# Drop Types

Some types are `drop` types. A drop type imposes limitations but also enables powerful
functionality. Drop types are how Azoth manages resources besides memory. A drop type ensures that
the compiler is able to determine a point where every instance of the type goes out of scope. At
that point, the the drop type's drop method is called allowing for the release of non-memory
resources. Drop types can be fields of other drop types or be variable and parameter types. A drop
type is not allowed to be a field of a non-drop type.

**TODO:** explain moving and recovery of isolation.

If a class or trait is declared as a `drop` type then all subclasses or implementing types must also
be `drop` types. Struct and value types can also be drop types.

## When Drop is Called

Drop types support the additional special capability `own` which indicates what binding owns the
value. For local variable and parameter bindings, when the owning binding goes out of scope, that
value is dropped. To support this, the compiler inserts an implicit move at the point the variable
goes out of scope. This means that the compiler must be able to recover isolation of the value at
this point. That ensures that no references to a dropped value with persist after it is dropped
(other than `id` references which are safe).

If one does not want a value to be dropped when it goes out of scope, then one must move ownership
with the `move` keyword. For example, this can be used to return ownership of a drop type to the
caller of a function.

When a drop type is a field of another drop type, there is no point at which the field goes out of
scope. Instead, there will be a chain of drop types from some drop type that is on the stack to this
drop type field. When that stack-based drop type goes out of scope, isolation will be recovered on
it and its drop method will be called. That will recursively call the drop methods of all drop type
fields. This does mean that within the object subgraph there may be a reference cycle so that one
drop method can access another dropped object after it has been dropped. This means that care must
be taken when accessing reference typed fields from a drop method.

While it is possible that a cycle exists within the subgraph reachable from an object being dropped,
that subgraph is isolated and so it is not possible to resurrect dropped objects as it is from a
finalizer. This is further enforced by the fact that the self parameter of the drop method must be
`lent`.

## Non-constant

Drop type instances cannot be frozen or otherwise become `const`. This is because after being frozen
it would no longer make sense to drop them. This also means that the `drop` and `const` type
modifiers are exclusive of each other.

## Memory

Drop types do not control memory deallocation. This is because an `id` reference may still exist to
an instance even after the value is dropped.

## Conditional Drop Types

Types which may or may not have to manage ownership of a drop type are conditional drop types. For
example, a collection may contain elements that are drop types and be responsible for dropping them.
Conditional drop types are declared with the `drop?` modifier. They are drop types only if one of
their independent generic parameters is a drop type. Conditional drop types cannot be declared
`const` and instances can become constant only when that instance is not a drop type.
