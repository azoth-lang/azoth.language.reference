# Structs

Structs are similar to classes. They contain field and function members. Unlike classes, structs are
hybrid types. They are allocated inline on the stack or in a field etc. However, they are still
passed by reference. The split is determined by the reference capability. To ensure that references
do not outlive a value on the stack, the compiler ensures that isolation is recovered on the owning
instance before it goes out of scope. Structs used as fields in classes are not subject to this since
any reference to them will keep the containing object live and they will "go out of scope" when the
object is dead and thus there is no reference that can reach the struct.

Because isolation must be recovered, structs cannot be declared `const` and struct instances cannot
be frozen. However, a class containing an owned struct field can be frozen. This is because all
references to the struct field will keep that struct alive and isolation does not need to be
recovered when it goes out of scope. Thus any type `const Struct` must actually be a reference to a
field. Additionally, `id` references to a struct have sharing tracked and are counted as breaking
isolation. This ensures that an `id` reference can't outlive a struct on the stack.

See the [Hybrid Types section of Categories of Types](categories-of-types.md#hybrid-types) for more
information on the limitations and unique aspects of hybrid types.
