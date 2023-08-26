# Structured Error Handling

Structured error handling provides a mechanism for error handling similar to exceptions in other
languages. It is not referred to as exceptions because it is both more flexible than exceptions in
other languages and lacks the stack traces that other languages provide.

## Typed Effect

Errors are treated as a typed effect. What errors a function or method can throw are declared on the
method with the `throws` effect. Functions that do no throw errors can be indicated with the `throws
never` effect. See the [Effects](effects.md) section for more information.

## Throws Declaration

As a typed effect, what errors a function throws are declared with a `throws` clause. This takes a
single type that may be thrown. However, union types may be used to allow a function to throw
multiple error types from a single domain of types. If a function cannot throw errors this is
indicated by `throws never` since the `never` type is the bottom or uninhabited type with no values.

Errors thrown do not need to be a subclass of any particular type or implement any trait. Any type
can be thrown, even value types.

```azoth
public fn example() -> void
    throws IOException|ParseException
{
    // ...
}
```

## Throws Inference

If a throws clause is omitted from a function or method then the types it throws will be inferred by
the compiler. This allows for statically enforcing what errors can be thrown while not requiring
cascading updates to throws clauses across a program when a new error is added.

There is one exception to throws inference. Published APIs for packages must have throws
declarations. Their throws clauses cannot be inferred. This ensures that published APIs have stable
interfaces since what errors a function/method can throw is part of the interface of that method.

## Throw Expressions

To throw an error use a throw expression.

```azoth
throw new FileNotFoundException("Example.txt");
```

The type of a throw expression is `never` so it is a subtype of any type. This allows throw
expressions to be used in interesting ways.

```azoth
let x = ReadLine() ?? throw new UserInputException();
```

## Try Expressions

Any expression containing one or more calls to functions or methods that throw exceptions must be
wrapped in a try expression. This clearly indicates in the code all places where an error can be
thrown from and makes all control flow explicit. It might be thought that this would be burdensome.
However, since Azoth aborts for unrecoverable errors the number of places an error can be thrown
from is dramatically reduced.

```azoth
let x = try foo();
```

A simple try expression will cause any errors thrown by the expression to be rethrown. Thus the
containing function must either catch it elsewhere, declare that it is thrown, or be inferred to
throw the error.

Note: having a try expression when it isn't needed or missing one when it is, is a non-fatal
compiler error. That is, it is an error, but a debug build can still produce an executable.

## Catch Expressions

A try expression can have a catch expression as well. This allows exceptions to be handled and
turned into values.

```azoth
let x = try foo() catch ex: SomeException => ex.Something;
```

A try expression with catch expressions will evaluate to the value of the tried expression if no
error is thrown. If an error is thrown, then if it matches the catch clause the expression will
evaluate to the value of the catch expression.

There can be multiple catch expressions and they can use expression blocks with braces. To rethrow
an exception, it is fine to use throw on the caught error (there is no `throw;` statement like
in other languages). Alternatively, you can wrap the error in a new type by passing the caught
error to its constructor.

A catch expression can omit the variable binding if the error value is not needed. However, the type
cannot be omitted. This is to discourage blanket catches of all errors.

```azoth
let x = try foo()
    catch FileException => none
    catch ex: NetworkException
    {
        if ex.IsFatal
            { throw ex; }

        => "error value";
    }
    catch ex: ParseException => throw new DataException(ex);
```

### Catch and Abort

If an exception should not be possible or this is a program where sophisticated errors handling
doesn't make sense, then a catch and abort expression to be used to easily catch an error and cause
an abort based on that error. This is done using the `catch!` syntax.

```azoth
let x = try foo() catch! SomeException;
```

**TODO:** given throws clause inference, it may not make sense to include this. The cases where one
would want to convert an error to an abort should be basically cases when the error is unreachable
and so `unreachable()` should probably be used instead.

### Catch to None

In some situations, the error cases are limited enough that it makes sense to just transform the
error into a `none` result. There is not a syntax specifically for that because making it too easy
would encourage its use more than is appropriate. However, it is fairly easy to catch an error and
transform it into a `none` value.

```azoth
let x = try foo() catch FormatError => none;
```

### Rethrow

There is a shorthand for catching a particular error and rethrowing it. This is useful to rethrow
one kind of exception while having a more general catch clause after it

```azoth
let x = try foo()
    rethrow SomeException
    catch ex: Exception
        => // handle exception
```

**TODO:** is this worth including?

### When Clause

Sometimes, whether an error should be caught is determined by more than just its type. A `when`
clause allows an additional condition to be applied to a catch or rethrow expression.

```azoth
let x = try foo()
    rethrow ex: SomeException when ex.IsFatal
    catch ex: Exception when ex.Kind == Something
        => // handle exception
```

## Try/Catch Statements

Try and catch can also be used as a statement. Note that this does not remove the need mark
individual expressions with `try`. Instead, it allows for handling exceptions within an entire block
of code. For a try/catch statement the `try` must be followed by a block and each `catch` must be
followed by a block. Note that `rethrow` expressions are unchanged.

```azoth
try
{
    let x = try foo();
    return x + 1;
}
catch e: IOException
{
    // handle exception
}
```

Note: Swift uses the `do` keyword for blocks instead of the `try` keyword. However, the `try` is
consistent with the use of it as a marker of code that may have exceptions passing through it. It
also leaves the `do` keyword available for other uses. Given that try expressions will clearly mark
where exceptions can be thrown. We don't necessarily need to precede try blocks with a keyword. We
could simply allow a catch on any block containing a try expression. However, this could make
control flow confusing because when looking at the try expression it might not be clear whether
there could be a catch block below.

## No Stack Trace

Structured error handling does not capture stack traces for errors. Stack traces are most useful for
unexpected errors that are actually unrecoverable. Structured error handling is for recoverable
errors. As such, it is expected that every error throw will be caught and properly handled. Because
of that along with the high overhead of capturing stack traces Azoth does not capture stack traces
for thrown errors. However, debuggers and other tools may capture stack traces for them for
debugging purposes.

## Performance

Project Midori found that by a combination of effects including removing error handling code from
the hot path, avoiding allocation, removing dynamic checks, and simplifying functions that can't
throw, exceptions actually performed better than returning errors with a `Result` type. Azoth has
been designed to allow a range of performance for structured error handling. To reduce overhead of
throwing exceptions, the exception object themselves can be constants or cached. Since there is no
stacktrace stored in the exception this is possible. In cases where even more performance is needed
throwing value types can further improve performance. Throwing a value type removes the overhead of
allocating memory while preserving the ability to include values specific to the error. It can also
allow efficient error processing without type checks by using enumerated value types.

## Defer

Azoth does not have finally blocks as C# and Java do. Instead, it takes inspiration had has `defer`
statements. A defer statement specifies an expression to be run any time the current scope is
exited. This allows for cleanup operations to be preformed. It has the advantage over finally blocks
that it has access to any values created so far in the scope. Often with finally blocks it is
necessary to place some code outside of the try block to enable it to be accessed from the finally
block. They can also necessitate initializing to null to allow the finally block to handle a case
when the full variable isn't assigned before the finally block runs.

**TODO:** defer statements may be unnecessary given the RAII pattern support provided by move types
with destructors. If all resources are managed that way then defer may be so rarely needed that it
would be better accomplished by a move type wrapping a lambda expression.
