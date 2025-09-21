# Cases

While `closed` types are vary powerful, declaring all the subtypes can be quite verbose. The `case`
keyword provides a convenient shorthand for declaring them. It can also be used to declare a member
inside of a non-closed class or trait declaration. The `case` declaration combines a number of
assumptions into one declaration.

## Access Modifiers

First, a case declaration does not default to a private declaration. Instead, it defaults to either
`public`, or if the containing type is `published`, it defaults to `published`.

## Cases are Sealed

Cases that produce a class declaration produce a sealed class. Values and structs are effectively
sealed by default. However, cases do not allow the `closed` modifier so that they cannot be
unsealed.

Note: This greatly simplifies case declarations and prevents surprises. We do not want people to be
able to unexpectedly inherit from cases. If inheritance is intended, the types should really be
declared outside of the closed type.

## Cases of a Class or Trait

A class or trait can be subtyped with a class, trait, or object declaration. However for simplicity
 and because of the expected use case, a case declaration can only result in an object or class
declaration. In a case declaration, the containing class name is implicit. Any record parameters are
listed after the case name. Parameters to the base initializer are separated from the name with a
colon and listed in parenthesis. Traits and members can still be optionally declared or omitted the
same as an object declaration. When the case generates a class, it inherit from the containing class
(e.g. `:` rather than `<:`). To override this, list the containing type as a supertype in the traits
list.

```azoth
public class Containing
{
    // An object case
    case Case1: (42);

    // A class case
    case Case2(foo: int): (62);

    // A class case that implements the trait
    case Case2 <: Containing;
}
```

If possible, a case in a class or a trait will produce an object declaration. If that is not
possible then a class declaration will be generated. A class is generated when one of the following
is the case

1. The case has record parameters.
2. The case declares initializers with arguments.
3. A class specific modifier is applied to the case (e.g. `const`, `drop`, etc.).

Note: Requiring an explicit declaration when a case implements the trait of a class simplifies the
case declaration and makes it clear to the programmer what will be generated.

## Cases of a Value

## Cases of a Struct

Non-closed structs do not allow any subtypes. Consequently, there can be no subtype value or struct
declarations. Closed structs do, however, allow subtype structs. Thus cases can only be declared in
a closed struct.

When used inside of a closed struct, the case declaration is a shorthand for declaring a `struct`
that is a subtype of that struct. The struct name is implicit. A case can be made a record struct by
listing parameters immediately after the case name. Parameters to the base initializer are separated
from the name with a colon and listed in parenthesis. Traits and members can still be optionally
declared or omitted the same as a struct declaration.

```azoth
public struct Containing
{
    // Instead of this
    public struct Value1: Containing <: Trait
    {
        public init(.foo) { base.init(42); }
    }

    // Shorten it to this
    case Value1: (42) <: Trait
    {
        // members here
    }
}
```

## Case Modifiers

Accessibility modifiers can be applied to cases in which case they replace the default modifier.
Other modifiers compatible with the fact that cases are `sealed` can be applied. These include
`const` and `drop`. The modifiers `abstract`, `sealed`, and `closed` are not allowed.
