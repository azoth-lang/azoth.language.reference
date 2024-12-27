# `azoth.collections.raw` Namespace

The `azoth.collections.raw` namespace contains special types that are declared by the
compiler and form the basis of memory allocations for all kinds of collections. These allow for the
allocation of many values in contiguous chunks both on the heap and the stack.

**TODO:** it would be neat to allow array structs and bounded list structs to be dynamically sized
similar to how C# can `stackalloc` an array whose size is determined by a variable or expression or
C has flexible array members. However, there are serious challenges with how these would be declared
and initialized because the compiler would need to determine the size of the containing object
during allocation. Thus the size could not be a simple initializer parameter. Instead there would
need to be some special syntax for allocating. For that to not be limited to unsafe types, it would
need to be flexible enough to apply to any type containing one or more of dynamically sized struct
collections.

## Naming

The types described here are named prefixed with `Raw` rather than `Unsafe` because the standard
library may wish to declare similar types with the `Unsafe` prefix that are more safe than this but
still expose specific unsafe functionality. For example, `Unsafe_Array` might expose an array that
may be uninitialized but prevents accessing outside of the array bounds.

## `Raw_Inline_Array[Count: size, T]`

This allows arrays of values to be allocated directly on the stack or in another object.

Initialization of this is possible because the self parameter is `ref self` and the size is known at
compile time. Thus the compiler can allocate the space directly on the stack or in the containing
object and then call the initializer on it by passing a stack reference to the allocated memory.

```azoth
published move struct Raw_Inline_Array[Count: size, T]
{
    published init(ref mut self, ensureZeroed: bool) {...}
    // TODO should generic parameters be publicly exposed members and this getter wouldn't be needed?
    published get count() => Count;
    // Overloading `self` on `ref` vs `iref` allows for determining the proper return type
    published unsafe fn at(ref mut self, index: size) -> ref var T {...};
    published unsafe fn at(iref mut self, index: size) -> iref var T {...};
    published unsafe fn at(ref self, index: size) -> ref T {...};
    published unsafe fn at(iref self, index: size) -> iref T {...};
}
```

## `Raw_Inline_Bounded_List[Capacity: size, T]`

This allows a list whose size can grow within a bounded limit to be allocated directly on the stack
or in another object. Initialization of this is possible because the initializer self parameter is
`ref self` and the size is known at compile time. Thus the compiler can allocate the space directly
on the stack or in the containing object and then call the initializer on it by passing a stack
reference to the allocated memory.

```azoth
published move struct Raw_Inline_Bounded_List[Capacity: size, T]
{
    published init(ref mut self) {...}
    // TODO should generic parameters be publicly exposed members and this getter wouldn't be needed?
    published get capacity() => Capacity;
    published get count() => size;
    published fn add(ref mut self, value: T);
    published unsafe fn shorten(ref temp iso self, count: size); // temp iso to prevent a reference to the removed items(s)
    // Overloading `self` on `ref` vs `iref` allows for determining the proper return type
    published unsafe fn at(ref mut self, index: size) -> ref var T {...};
    published unsafe fn at(iref mut self, index: size) -> iref var T {...};
    published unsafe fn at(ref self, index: size) -> ref T {...};
    published unsafe fn at(iref self, index: size) -> iref T {...};
}
```

## `Raw_Hybrid_Array[F, T]` and `Raw_Hybrid_Array[F, Count: size, T]`

These are reference types that allocate a hybrid of a fixed portion and an array of a given size and
type. The size may be determined either at runtime or compile time. For unmanaged types, it is safe
to allocate an array without initializing it. However, for managed types, it is likely necessary to
zero the memory so that the garbage collector does not try to follow uninitialized references.
However, even when zeroed, that doesn't mean that the value is a valid value for the type. It may
not respect reference nullability and class invariants. Individual values in the array should not be
used without first initializing them.

While it is guaranteed that unmanaged types can be allocated uninitialized, no guarantee is made
about whether other types will be zeroed. However, zeroing can improve safety by eliminating the
chance that previously allocated and released values can be read. As such, a initializer parameter
is provided that allows for requesting that the memory be definitely zeroed.

This type is rarely used, but is the foundation for other collection types. As with other objects,
the Azoth compiler is able to optimize `Raw_Hybrid_Array` so that the memory allocation occurs on
the stack rather than the heap when it can determine references to it do not escape and the array is
small enough for this to be beneficial. The effective declarations of `Raw_Hybrid_Array[F, T]` and
`Raw_Hybrid_Array[F, Count: size, T]` are:

```azoth
published class Raw_Hybrid_Array[F, T]
{
    published init(mute self, .fixed, .count, ensureZeroed: bool) {...}
    published var fixed: F;
    published let count: size;
    published unsafe fn at(mut self, index: size) -> iref var T {...};
    published unsafe fn at(self, index: size) -> iref T {...};
}

published class Raw_Hybrid_Array[F, Count: size, T]
{
    published init(mut self, .fixed, ensureZeroed: bool) {...}
    published var fixed: F;
    // TODO should generic parameters be publicly exposed members and this getter wouldn't be needed?
    published get count() => Count;
    published unsafe fn at(mut self, index: size) -> iref var T {...};
    published unsafe fn at(self, index: size) -> iref T {...};
}
```

Note that the `at` methods return `iref` to allow slices to be implemented by referencing directly
into the middle of the array. This also avoids the need to copy large structs from an array.

## `Raw_Hybrid_Bounded_List[F, T]` and `Raw_Hybrid_Bounded_List[F, Capacity: size, T]`

These reference types provide a hybrid of a fixed portion and a list whose size can grow within a
bounded limit. This allows for the memory to be allocated without initialization and initialized
element by element as the values are assigned. It also allows the size to be reduced thereby
allowing elements to be effectively "freed" and no longer referenced by the list. This type forms
the basis of an efficient implementation of `List[T]`. No bounds checking is performed when indexing
into the list. Thus it would be possible to access uninitialized elements. This type also does not
limit `count <= capacity`. If the size is increased beyond the capacity, then the result is
undefined. It may cause the garbage collector to crash or keep invalid memory "live".

```azoth
published class Raw_Hybrid_Bounded_List[F, T]
{
    published init(.fixed, .capacity) {...}
    published var fixed: F;
    published let capacity: size;
    published get count() => size;
    published unsafe fn shorten(count: size); // unsafe because there may be iref to them
    published unsafe fn at(mut self, index: size) -> iref var T {...};
    published unsafe fn at(self, index: size) -> iref T {...};
    published fn add(mut self, value: T);
}

published class Raw_Hybrid_Bounded_List[F, Capacity: size, T]
{
    published init(.fixed) {...}
    published var fixed: F;
    // TODO should generic parameters be publicly exposed members and this getter wouldn't be needed?
    published get capacity() => Capacity;
    published get count() => size;
    published unsafe fn shorten(count: size); // unsafe because there may be iref to them
    published unsafe fn at(mut self, index: size) -> iref var T {...};
    published unsafe fn at(self, index: size) -> iref T {...};
    published fn add(mut self, value: T);
}
```
