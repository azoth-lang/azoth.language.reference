# `azoth.collections` Namespace

## `Array[T]`

**TODO:** how to implement multidimensional arrays?

## `Array[Count: size, T]`

## `List[T]`

A dynamically sized list of elements stored in contiguous memory (i.e. not a linked list).
Internally, elements are stored as `Var[T]` which allows `List[T]` to be sliced into `Array[T]`
views. As long as such a view exists, the list cannot be resized and it `temp const`.
