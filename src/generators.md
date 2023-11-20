# Generators

The `yield` keyword acts basically as `yield return` does in C#.

Python uses `return` for the equivalent of `yield break`, however that causes an inconsistency where
removing yield from a method changes the meaning of the return. Azoth uses `yield return;` for this.

## Recursive Generators

Having generator functions recursively yield all the items from another generator can lead to very
poor performance. It may even be able to change the big-O performance since it is effectively
performing multiple operations for every element in proportion to the depth of the recursion.

Python has `yield from` for recursively yielding another iterator (Python already used `from` for
other things). A paper proposed `yield foreach` for the same thing in C#. Azoth will likewise use
`yield foreach`.

In order to ensure that this can be implemented efficiently, one isn't allowed to use it on an
arbitrary iterator. Instead, it can only be used to make recursive calls on the main function or on
nested functions that themselves are generators of the same type. This allows the compiler to
transform the recursion into iteration and avoid the possible performance impacts of a truly
recursive generator.

**TODO:** see https://github.com/dotnet/csharplang/discussions/378 for possible ways to implement
efficiently. Could we just use a sort of continuation passing style?

## Void Generators

For certain types like `Iterator[void]` there is no value to be yielded. In this case, the `yield`
expression takes no value (e.g. `yield;`). This would be useful it one was trying to implement
coroutines on top of generator functions.

## Generator Start Control

By default the body of a generator function does not start running until the first item is request.
Sometimes this is not desireable and validation and initialization should be run when the function
is called. This can be done with the optional `new yield` statement. When this statement is used.
Any code before it is run when the function is initially called and code after it is part of the
generator. This means other yield expressions cannot be used before it.

**TODO:** should it switch to `init yield` if

## Generator Method Builders

The compiler transforms generator functions similar to how the C# compiler accomplishes this by
creating a class with a state machine in it. However, this process is controlled similar to how the
async transform is controlled in C#. An attribute with a type can be placed either on the generator
function itself or on the type being generated and it specifies the type the compiler uses during
the transform. This allows generator functions to be used with types besides just `Iterator[T]`.
