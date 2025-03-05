# Optional Parameters

Optional parameters allow arguments to be omitted and a default or contextual value will be used.

## Default Values

A default value is specified after the parameter type declaration using an equal sign. The
expression must be a constant expression that can be evaluated at compile time.

```azoth
public fn example(answer: int = 42) { ... }

// calling
example();
```

## Contextual Parameters

A contextual parameter allows an argument to be omitted if the compiler can find a contextual value
to pass instead. Contextual parameters are declared using an equal sign and the `context` keyword.

```azoth
public fn example(answer: int = context) { ... }

// calling
with 42
{
    example();
}
```

Contextual parameters will be passed the nearest lexically scoped type compatible contextual value.
Contextual values come from three sources:

1. Any contextual parameter is itself a contextual value (even if the discard pattern `_` is used
   for the argument).
2. The value of a `with` block is a contextual value within the block.
3. The async scope and cancellation token of an `async` block are contextual values within the
   block.

If no contextual value can be found for a contextual argument, then it is an error. The programmer
must explicitly specify the argument.
