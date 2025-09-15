# Drop Types

Some types are `drop` types. A drop type imposes limitations but also enables powerful
functionality. Drop types are how Azoth manages resources besides memory. A drop type ensures that
the compiler is able to determine a point where every instance of the type goes out of scope. At
that point, the compiler recovers isolation for the instance and the drop type's drop method is
called allowing for the release of non-memory resources. Drop types can be fields of other drop
types or be variable and parameter types. A drop type is not allowed to be a field of a non-drop
type.

**TODO:** explain moving and recovery of isolation.

If a class or trait is declared as a `drop` type then all subclasses or implementing traits must
also be `drop` types.

Struct and value types can also be drop types.

## Non-constant

Drop type instances cannot be frozen or otherwise become `const`. This is because after being frozen
it would not be possible to recover isolation in order to drop them. This also means that the `drop`
and `const` type modifiers are exclusive of each other.

## Memory

Drop types do not control memory deallocation. This is because an `id` reference may still exist to
instance even after isolation is recovered and the value is dropped.

## Conditional Drop Types

Types which may or may not have to manage ownership of a drop type are conditional drop types. For
example, a collection may contain elements that are drop types and be responsible for dropping them.
Conditional drop types are declared with the `drop?` modifier. They are drop types only if one of
their independent generic parameters is a drop type.
