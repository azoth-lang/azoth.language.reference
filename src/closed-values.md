# Closed Values

Values can be marked with the `closed` modifier. This transforms them into discriminated unions and
allows them to effectively be subtyped. The "subtypes" become members of the discriminated union. A
closed value can be subtyped by either another value declaration or an object declaration. This
expands object declarations to value types. While that may seem odd, objects are empty types and so
are essentially values defined only by their type. Closed values are abstract. One cannot create an
instance of them only of the subtypes.

A `closed` value can only have a fixed set of subtype values or objects. These are declared with
`value` and `object` declarations, though see the [case declarations](case-declarations.md) section
 for a shorthand syntax. Typically, these are declared inside the closed value, though technically
they can also be declared outside the value. Private initializers can be used to prevent that if
desired. The individual subtype values or objects can override members of the value and add fields.
The standard mechanisms of `sealed`, `overrides`, and `hides` can be used to control this process.

```azoth
public const closed value release_stage
{
    let is_prerelease: bool;
    public init(.is_prerelease = false) { }

    // These follow constant name conventions because they are acting as constants
    public object Alpha: release_stage(true);
    public object Beta: release_stage(true);
    public object ReleaseCandidate: release_stage;
    public object Full: release_stage(true);
}

// Object declared outside of struct
public object PreAlpha inherits release_stage(true);

public const closed value token
{
    public object plus_token: token; // follows type name conventions
    public const value identifier(value: string): token;
    public const value string_literal(value: string): token;
}
```

## Closed Value Object Optimization

While an `object` declaration inheriting from a value type logically creates a constant for the
value type and therefore has the size of the value type the object is declared for, they work
differently. Since the compiler knows that there is a fixed set of objects inherited from a value,
it does not have to pass the fields around for the objects. Instead, it passes only an id from which
it can determine which object it is. Field values are stored separately in a constant table. This
makes them equivalent to enums in other languages like C# and Java where they are just an integer
identifying which value is used. Note, this does not apply to subtype values. They do have space for
any fields of the closed value.
