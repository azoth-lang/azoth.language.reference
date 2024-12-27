# Structured Concurrency

**TODO:** Explain why structured concurrency and link to "Go Considered Harmful"
**TODO:** back-pressure and throttling in async operations.

## Principles

Principle: Async Everywhere - use async in lots of places

Operations that should be async, Disk, Network, Screen (writing to Console can be slower than
writing to file), inter thread and process communication, and sleeping.

* Principle: No Blocking Operations - it should be impossible to block a thread
* Principle: Don't divide the world into red and green methods

## Async Values

An async value is a value whose type follows the async scope pattern and is either declared with an
async block or is in an async parameter. Async values are used to restrict the starting of async
operations to enforce the rules of structured concurrency.

## Async Blocks

An asynchronous block is introduced with the `async` keyword. The compiler enforces that all
asynchronous operations started within an async block are complete before exiting the block. To
provide flexibility, it allows an asynchronous scope object to be provided to manage the async
block. The asynchronous scope object can be given a name via a `let` declaration.

```grammar
async_block
    : "async" block
    | "async" embedded_expression block
    | "async" "let" identifier (":" type)? "=" embedded_expression block
    ;
```

The async scope cannot be declared with `var` because assigning a new value would invalidate the
structured concurrency rules. The async scope object must conform to the pattern for async scopes.
For convenience, that pattern is defined mirrored in the `Async_Scope[Promise]` trait.

**TODO:** proper trait for async scope and how to handle optional cancellation context?

### Async Scopes

**TODO:** define what methods must exist on an async scope.

### Default Async Scope

**TODO:** define how the compiler selects the async scope when none is provided.

### Async Context

Inside of an async block two contextual values are available (see [Contextual
Arguments](optional-arguments.md#contextual-arguments)). The first is the async scope object. The
second is the cancellation token if the async scope defines one.

### Starting Async Operations

Async operations can be started with either the `go` or `do` keywords. The `go` keyword schedules
the operation to run on a separate thread as determined by the async scope. The `do` keyword begins
execution of the operation on the current thread. If an `await` is reached that is awaiting an
incomplete operation, then execution will return to the nearest `do` on the call stack. The
continuation of the operation will continue on a separate thread as determined by the async scope.
These operators are defined on the async scope object. However, they can only be used on an `async`
value. These operators can be used as binary operators, but within an `async` block the async scope
is implicit and they can be used as unary prefix operators. The compiler implicitly inserts the
async scope parameter.

```grammar
async_start_expression
    : "go" expression
    | embedded_expression "go" expression
    | "do" expression
    | embedded_expression "do" expression
    ;
```

Both operators return a promise-like object the exact type of which is determined by the async
scope. If the result of the expression being started is of type `void`, then the return value of the
start expression does not have to be used. However, if the expression being started returns a value,
then the return value of the start expression must be used or explicitly discarded.

Not all async scopes support both operators. Some support only one as indicated by the fact that
they define only that operator. In that case, trying to use the other will be a compiler error.

**TODO:** is the `do` keyword necessary or is it an optimization that isn't needed?

#### Reference Capabilities

Async operations conceptually work like anonymous functions. As such they capture a closure of the
variables used. Because async operations could potentially operate in parallel the compiler enforces
that this closure can contain only constant, isolated, and identity values. However, these can be
lent with the returned promise-like object being lent so that the compiler can enforce proper
reference access.

### Async Block Completion

The compiler ensures that every async operation started in an async scope completes before the
associated async block exits. If explicit `await`s are used or a compile time fixed number of async
operations are started in a block, this is equivalent to the compiler inserting `await` operations
at the end of the block for any promise that may not have been awaited. If an indeterminate number
of async operations are started, then a collection of async operations is created and managed. This
collection may not contain statically awaited promises. Any promise placed in the collection will be
removed immediately upon successful completion.

The async scope determines how exiting the block is handled. If the block is exited via an
exception, the async scope is notified. The default async scope uses this to request cancellation of
any incomplete async operations. The async scope also handles how to report multiple async
operations failing.

## Async Parameters

While async blocks provide the fundamentals for async operations, sometimes it is necessary to start
an async operation which will outlive the current function. To do this, an async scope must be
passed from a scope higher up the call stack and used to start async operations. To do this, use
async parameters.

An async parameter is declared by prefixing the parameter type with the async keyword.

```azoth
public fn example(_: async Async_Scope = context) -> Promise[int]
    => go compute_nth_prime(1_000_000);
```

The parameter type must follow the async scope pattern. Async parameters should typically be
declared as [contextual arguments](optional-arguments.md#contextual-arguments). Only an async value
may be passed as the argument to an async parameter.

## Awaiting Promises

Async operations return promise-like objects which must implement the awaitable pattern. For
convenience, that pattern is defined mirrored in the `Awaitable` and `Awaitable[T]` traits. A
promise is awaited using an await expression.

```grammar
await_expression
    : "await" embedded_expression
    ;
```

Await expressions can appear anywhere, not just inside of async blocks. When an incomplete promise
is awaited, it does not block the current thread. The behavior depends on the whether the nearest
async start expression on the call stack is a `go` or `do`. If it is a `go` then execution of the
current async operation is suspended until the promise is complete. If it is a `do` then execution
continues from the `do` expression. If there is no `go` or `do` above the await expression on the
call stack, then the main operation is suspended. Effectively, it is as if `main()` is called from
inside an async block using a `do` expression.

### Awaiting Callback

Sometimes when adapting external APIs it is necessary to adapt an async callback into an await. This
can be done using the `Async_Callback` type.

```azoth
async
{
    // Create an `Async_Callback` to bridge from a callback to an awaitable
    // Note: this takes an `Async_Scope` so that it is confined to an async block and enforces
    // structured concurrency. The body of this function will run on the thread used by the external
    // API to call the callback.
    let callback = Async_Callback(fn (x: int32, y: int32) => x + y);

    // Call the external API that expects a callback function
    start_operation(callback.function);

    // Await the callback to access the result of the callback function
    return await callback;
}
```

**TODO:** Should the function run on the callback thread or the thread pool?

### Awaiting Blocking APIs

**TODO:** is this necessary?

If an external API provides only a blocking function call, this can be adapted to async by using a
`Blocking_IO_Scope`. This scope defines only the `go` operator, not the `do` operator. The `go`
operator allocates a thread to the blocking operation which will be blocked until the operation
returns.

```azoth
async Blocking_IO_Scope()
{
    return await go blocking_operation();
}
```

## Design Notes

The design of async blocks and parameters has been chosen to try to ensure that it is not possible
for libraries using them to violate the structured concurrency rules. This is why, for example, it
isn't possible to use the `go` and `do` operators on an async scope unless it is an async value.
Otherwise, it would be possible to instantiate an async scope not associated to any block and use it
to start async operations.

Of course, it is probably impossible to fully preclude violations of structured concurrency given
that external functions can be directly called. But at least those violations should not be dressed
up in the syntax Azoth provides for structured concurrency.
