# Closed Structs and Struct Traits

Structs can be marked with the `closed` modifier. This transforms them into discriminated unions and
allows them to effectively be subtyped. The "subtypes" become members of the discriminated union.

A `closed` struct can only have a fixed set of subtype values or structs. These are declared with
`value` and `struct` declarations, though see the [Cases](cases.md) section for a shorthand syntax.
 Typically, these are declared inside the closed struct, though technically they can also be
declared outside the struct. Private initializers can be used to prevent that if desired. The
individual subtype values or structs can override members of the struct and add fields.The standard
mechanisms of `sealed`, `overrides`, and `hides` can be used to control this process. A closed
struct is effectively always `abstract`. That is, there can be no instances of the struct itself
created. Only values and instances of the subtypes.

```azoth
public closed struct release_stage
{
    let is_prerelease: bool;
    public init(.is_prerelease = false) { }

    public value Alpha: release_stage(true);
    public value Beta: release_stage(true);
    public value ReleaseCandidate: release_stage;
    public value Full: release_stage(true);
}

// Value declared outside of struct
public value PreAlpha: release_stage(true);

public closed struct token
{
    public value plus_token: token;
    public struct Identifier(value: string): token;
    public struct StringLiteral(value: string): token;
}
```

## Closed Struct Value Optimization

While a normal `value` declaration effectively declares a constant for the struct type and therefore
has the size of the struct type the value is declared for, `closed struct`s work differently. Since
the compiler knows that their is a fixed set of values, it does not have to pass the field values
around when using instances of the closed struct. Instead, it passes only an id from which it can
determine which value it is. Field values are stored separately in a constant table. This makes them
equivalent to enums in other languages like C# and Java where they are just an integer identifying
which value is used. Note, this does not apply to subtype structs. They do have space for any fields
of the closed struct.
