# Associated Types

**TODO:** expand these notes.

Associated types are direct type declarations within a trait, class, or struct and are therefore
"virtual". Abstract types can have abstract associated types. In an abstract class this is `public
abstract type T;`. In a trait, the `abstract` keyword is not needed.

Associated types can have type constraints placed on them with `where`. For consistency, one cannot
just declare supertypes on them (i.e. `public type T <: Base` must be written `public type T where T
<: Base`).

By default, associated types are invariant type variables. Therefore they cannot be changed in
subtypes after they have been assigned. However, they can be marked `in` or `out` to make them
covarient or contravariant and then they can be modified in subtypes consistent with those
limitations. The `in` or `out` must be repeated in the subtype declaration.

Associated types are associated members of a type.

Associated types do not have capabilities attached to them. They are a bare type.
