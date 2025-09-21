# Closed Traits and Classes

Both classes and traits can be marked with the `closed` modifier. A closed trait or class can only
be directly subtyped within the same package as the type is declared in. This means for a closed
class or trait `A`, only classes and traits in the same package may list it as their supertype or
implemented types. This applies equally to object and value declarations.

It should be noted that while a closed class or trait cannot be directly subtyped, it can be
indirectly subtyped. For example, given `published closed trait A` and `published trait B <: A` in a
given package, another package can declare `class C <: B`. Thus `C` is a subtype of `A` even though
it does not directly subtype it. This is allowed because an exhaustive match is still possible on
`A` by having a single `B` case that covers all subtypes of `B`.

It should also be noted that if a non-abstract class is declared `closed` then it is possible for
instances of that type to be created and matching on the type will require a case to match such
instances. For example, given `closed class B` with subclasses `C` and `D`, a match on `B` would not
be exhaustive when covering the `C` and `D` cases. In addition, a final `B` case would be needed to
handle instances of `B`.
