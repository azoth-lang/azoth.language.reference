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

## Generator Start Control

By default the body of a generator function does not start running until the first item is
requested. Sometimes this is not desirable and validation and initialization should be run when the
function is called. This can be done with the optional `init yield` statement. When this statement
is used, any code before it is run when the function is initially called and code after it is part
of the generator. This means other yield expressions cannot be used before it.

## Generator Drop Types

Generators can create drop types. When the generator is dropped, the current `yield` expression acts
as if it were a `yield return;`. That is, any `yield` expression can be a return point. A generator
must produce a drop type if there are any local variables that are drop types that must be dropped
when the function or method exits.

## Generator Method Builders

**TODO:** should these instead be based on keyword operator overloads?

The compiler transforms generator functions similar to how the C# compiler accomplishes this by
creating a class with a state machine in it. However, this process is controlled similar to how the
async transform is controlled in C#. An attribute with a type can be placed either on the generator
function itself or on the type being generated and it specifies the type the compiler uses during
the transform. This allows generator functions to be used with types besides just `Iterator[T]`.

The method builder adapts the state machine to the type returned by the generator method. It does
this via a factory method generic over the state machine type and by providing associated functions
to adapt the yield expressions to the type.

```azoth
/// The generator method builder is parameterized on the type of the compiler generated state
/// machine, the type parameter of the return type (e.g. `T` in `Iterator[T]`) and the throws type
/// of the generator function (i.e. the type in the `throws` clause).
/// For non-generic return types, there is a builder overload without the `T` type parameter.
published trait Generator_Method_Builder[State_Machine, T, Throws]
    where State_Machine: struct, Generator_State_Machine[Yield_Input, Yield_Input_Throws, Yield_Output, Yield_Output_Throws]
{
    /// The type that the yield expression returns from the caller
    published type Yield_Input; // e.g. `void` for iterators

    /// The type that the yield expression throws from the caller
    published type Yield_Input_Throws; // e.g. `never` for iterators

    /// The type that the yield expression outputs to the caller
    published type Yield_Output; // e.g. `T?` for iterators

    /// The type that the yield expression throws to the caller
    published type Yield_Output_Throws; // e.g. `never` for iterators

    /// The type generators using this method builder generate
    published type Generator_Type; // e.g. `Iterator[T]` for iterators

    /// Create the value returned from the generator function
    published fn \init(state_machine: State_Machine) -> Generator_Type;

    /// Called to prepare the output of a `yield` expression
    published fn \yield(state_machine: mut State_Machine, value: T) -> Yield_Output;

    /// Called to produce the output of a `yield return` expression
    published fn \return(state_machine: mut State_Machine) -> Yield_Output;
}

published drop trait Generator_State_Machine[Yield_Input, Yield_Input_Throws, Yield_Output, Yield_Output_Throws]
{
    /// `Yield_Input` of `void` indicates that no values are input and `never` indicates that this
    /// method cannot be called.
    published fn \yield(mut self, input: Yield_Input) -> Yield_Output
        throws Yield_Output_Throws;

    /// The yield expression should throw the given error. `Yield_Input_Throws` of `never` indicates
    /// this method cannot be called.
    published fn \throw(mut self, error: Yield_Input_Throws) -> Yield_Output
        throws Yield_Output_Throws;

    published drop(lent self)
        throws Yield_Output_Throws;
}
```

## Void Generators

For certain types there is no value to be yielded. In this case, the `Yield_Output` is `void` and
the `yield` expression takes no value (e.g. `yield;`). This may be useful if one was trying to
implement coroutines on top of generator functions.

## Generators as Coroutines

Similar to [PEP 342 – Coroutines via Enhanced Generators](https://peps.python.org/pep-0342/), the
generator method builders allow generator functions to be used as basically coroutines. This is why
the generator state machine has both input and output yield types and error types. This is also why
the `\throw()` method exist on the state machine.
