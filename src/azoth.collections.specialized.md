# `azoth.collections.specialized` Namespace

The `azoth.collections.specialized` contains two special types that declared by the compiler and
form the basis of memory allocations for all kinds of collections. These allow for the allocation of
many values in contiguous chunks both on the heap and the stack.

**TODO:** it would be neat to allow array structs and bounded list structs to be dynamically sized
similar to how C# can `stackalloc` and array whose size is determined by a variable or expression or
C has flexible array members. However, there are serious challenges with how these would be declared
and initialized because the compiler would need to determine the size of the containing object
during allocation. Thus the size could not be a simple initializer parameter. Instead there would
need to be some special syntax for allocating. For that to not be limited to unsafe types, it would
need to be flexible enough to apply to any type containing one or more of dynamically sized struct
collections.

## `Unsafe_Array[T]` and `Unsafe_Array[Count: size, T]`

These are reference types that allocate an array of a given size and type. The size may be
determined either at runtime or compile time. Because the entire array would have to be initialized
upon construction if it contained references, these arrays may only be used with unmanaged types.
Because of this, the array is constructed in a completely uninitialized state and may contain random
values. Individual values in the array should not be used without first initializing them.

 This type is rarely used, but is the foundation for other collection types. As with other objects,
 the Azoth compiler is able to optimize `Unsafe_Array` so that the memory allocation occurs on the
 stack rather than the heap when it can determine the array is small enough for this to be
 beneficial. The effective declarations of `Unsafe_Array[T]` and `Unsafe_Array[Count: size, T]` are:

```azoth
public mut class Unsafe_Array[T]
    where T: unmanaged
{
    public new(.count) {...}
    public let count: size;
    public unsafe at(mut self, index: size) -> iref var T {...};
    public unsafe at(self, index: size) -> iref T {...};
}

public mut class Unsafe_Array[Count: size, T]
{
    public new() {...}
    // TODO should generic parameters be publicly exposed members and this getter wouldn't be needed?
    public get count() => Count;
    public unsafe at(mut self, index: size) -> iref var T {...};
    public unsafe at(self, index: size) -> iref T {...};
}
```

Note that the `at` methods return `iref` to allow slices to be implemented by referencing directly
into the middle of the array.

## `Unsafe_Array_Struct[Count: size, T]`

This allows arrays of values to be allocated directly on the stack or in another object.
Initialization of this is possible because the constructor self parameter is implicitly `ref var
self` and the size is known at compile time. Thus the compiler can allocate the space directly on
the stack or in the containing object and then call the initializer on it by passing a stack
reference to the allocated memory.

```azoth
public mut move struct Unsafe_Array_Struct[Count: size, T]
{
    public init() {...}
    // TODO should generic parameters be publicly exposed members and this getter wouldn't be needed?
    public get count() => Count;
    // TODO how does it determine these should be `ref` instead of `iref` when on the stack?
    public unsafe at(mut self, index: size) -> iref var T {...};
    public unsafe at(self, index: size) -> iref T {...};
}
```

## `Unsafe_Bounded_List[T]` and `Unsafe_Bounded_List[Capacity: size, T]`

This provide a list whose size can grow within a bounded limit. This allows for the memory to be
allocated without initialization and initialized element by element as the values are assigned. It
also allows the size to be reduced thereby allowing elements to be effectively "freed" and no longer
referenced by the list. This type forms the basis of an efficient implementation of `List[T]`. No
bounds checking is performed when indexing into the list. Thus it would be possible to access
uninitialized elements. This type also does not limit `count <= capacity`. If the size is increased
beyond the capacity, then the result is undefined. It may caused the garbage collector to crash or
keep invalid memory "live".

```azoth
public mut class Unsafe_Bounded_List[T]
{
    public new(.capacity) {...}
    public let capacity: size;
    public var count: size = 0;
    public unsafe at(mut self, index: size) -> iref var T {...};
    public unsafe at(self, index: size) -> iref T {...};
    public add(mut self, value: T);
}

public mut class Unsafe_Bounded_List[Capacity: size, T]
{
    public new() {...}
    // TODO should generic parameters be publicly exposed members and this getter wouldn't be needed?
    public get capacity() => Capacity;
    public var count: size = 0;
    public unsafe at(mut self, index: size) -> iref var T {...};
    public unsafe at(self, index: size) -> iref T {...};
    public add(mut self, value: T);
}
```

## `Unsafe_Bounded_List_Struct[Capacity: size, T]`

```azoth
public mut move struct Unsafe_Bounded_List_Struct[Capacity: size, T]
{
    public init() {...}
    // TODO should generic parameters be publicly exposed members and this getter wouldn't be needed?
    public get capacity() => Capacity;
    public var count: size = 0;
    // TODO how does it determine these should be `ref` instead of `iref` when on the stack?
    public unsafe at(mut self, index: size) -> iref var T {...};
    public unsafe at(self, index: size) -> iref T {...};
    public add(mut self, value: T);
}
```
