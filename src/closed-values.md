# Closed Values

Values can be marked with the `closed` modifier. This transforms them into discriminated unions and
allows them to effectively be subtyped. The "subtypes" become members of the discriminated union. A
closed value can be subtyped by a unit value or another value declaration. Closed values are
abstract. One cannot create an instance of them, only of the subtypes.

A `closed` value can only have a fixed set of subtype values. These are declared with `value` and
`unit value` declarations, though see the [case declarations](case-declarations.md) section for a
 shorthand syntax. Typically, these are declared inside the closed value, though technically they
can also be declared outside the value. Private initializers can be used to prevent that if desired.
The individual subtype values can override members of the value and add fields. The standard
mechanisms of `sealed`, `overrides`, and `hides` can be used to control this process.

```azoth
public const closed value release_stage
{
    let is_prerelease: bool;
    public init(.is_prerelease = false) { }

    // These follow constant name conventions because they are acting as constants
    public unit value Alpha inherits release_stage(true);
    public unit value Beta inherits release_stage(true);
    public unit value ReleaseCandidate inherits release_stage;
    public unit value Full inherits release_stage(true);
}

// Value declared outside of closed value
public unit value PreAlpha inherits release_stage(true);

public const closed value token
{
    // These follow type name conventions because they are acting more as types
    public unit value plus_token: token;
    public const value identifier(value: string): token;
    public const value string_literal(value: string): token;
}
```

## Closed Unit Value Optimization

Unit values typically have size zero. However, when they inherit from a closed value they have the
size of that closed value. However, they affect the size of the closed value in a more complex way
that subtype values.

Even if a closed value has fields, if all subtypes are unit values, then those fields do not need to
be stored in the value. Instead, only a discriminator is stored which allows for determining the
unit value. The field values for each unit value are stored as constants.

If a closed value has subtypes that are themselves values, then the size of the value will be
determined by the size necessary to store any fields of the value along with the fields of the
largest subtype value. However, unit values do not have to have their inherit fields stored. They
can be identified just by the discriminator. This can reduce the overall size of the value type. For
example, if the closed value declared a single non-zero field and all but one of the subtypes were
unit values. This could be encoded so that the unit values were identified by a zero field value
plus a discriminator.
