# Structs

Structs are similar to classes. They contain field and function members. Unlike classes, structs are
hybrid types. They are allocated on inline on the stack or in a field etc. However, they are still
passed by reference. The split is determined by the reference capability. To ensure that references
do not outlive a value on the stack, all structs are move types and must be declared with the `move`
keyword.

See the [Hybrid Types section of Categories of Types](categories-of-types.md#hybrid-types) for more
 information on the limitations and unique aspects of hybrid types.
