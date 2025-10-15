# Type System

## Soundness

The soundness of the Azoth type system will be very important because it will not have a virtual
machine for runtime checks. Without that layer of runtime checks, there is a risk that an unsound
type system will lead to memory safety issues.

## Type Aliases

Type aliases should be a safe addon to the core type system. This is because they ought type check
equivalent to a program with all aliases replaced with the type they alias. The one thing to be
careful of is if a type alias allows for the declaration of additional constraints beyond those of
the type it is aliasing. But that should be safe so long as all use sites check that the constraints
are satisfied and that any constraints are expanded onto use site entities. For example, an
extension to a type alias would need to repeat all the constraints of the alias. If like C#, all
constraints must always be explicit at each site, then that may already be covered.

## Union Types

Union types should be possible to transform into core types by introducing a new trait which all the
unioned types implement. The union can then be represented by that trait. A few things must be
accounted for. These union traits must have the correct relationships to each other. For example if
one union contains a subset of the types of another union then its union trait must be a subtype of
the other union trait. Additionally, for unions of values or structs, methods may need to be made
generic over the type so that they still take the type by value.

## Intersection Types

Like union types it should be possible to transform intersection types by introducing new traits
which implement all the intersected types and which any type in the intersection implements.
