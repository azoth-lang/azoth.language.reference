# Guaranteed Optimizations

## Empty `object` Declarations

Object declarations with no fields can be optimized to not have an object reference. Thus no
allocation is required. Instead, it can be a vtable reference only. When something is known to be of
that type, even the vtable can be optimized away. However, when accessed via a trait, the object
reference can simply be null.
