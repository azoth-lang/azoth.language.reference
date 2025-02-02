# Variable References

Standard reference types have values that are inherently a reference to objects on the heap. In
addition to these references, there are variable reference types. A variable reference type is
required to only be held on the stack, and allows one to reference values on the stack. Additional
limitations are applied to variable references to ensure that one can never reference a value on the
stack that has gone out of scope. It should be noted that while variable references allow values on
the stack to be referenced, they can also refer to values on the heap inside of objects.

It should also be noted that while variable references must be held on the stack, they can still be
included in value types via `ref structs`s (see [Ref Structs](ref-structs.md)).

Note: variable references are basically equivalent to the `ref` and `in` keywords in C#.

```grammar
stack_reference
    : struct_reference
    | variable_reference
    ;
```

## Struct References

A readonly reference to a value type either on the stack or the heap. This allows passing value
types without copying. Note that the value type could be a value on the stack, but it could also be
a value inside an object on the heap or an element of an array or list. If a variable reference
refers to a value on the heap, then that reference keeps the larger object it is referring into
alive and it cannot be garbage collected. Variable references cannot be combined with reference
types because the double reference indirection would make no sense and serve no purpose. However, if
via generics, `ref` is combined with a reference type `T` then the type `ref T` is instead taken as
the type `T`. Variable reference types are very similar to the `in` parameter modifier in C#, but
can be used as local variable types too.

```grammar
struct_reference
    : "ref" value_type
    ;
```

Note: though a variable reference is read-only, that does not mean the actual value is read-only and
that there aren't other mutable references to it.

## Mutable Variable References

A reference to a mutable variable on the stack or the heap. That variable may contain either a value
type or a reference type. It is important to understand that the mutability of the variable here
refers to the mutability of the variable binding. Thus all variable references must refer to a
variable or field defined with the `var` keyword. That variable may have an immutable reference
type. For example, `ref var const User` is a reference to a `var` referencing an immutable `User`
object. Thus the variable could be assigned a new reference value to a different immutable `User`
object, but the `User` object itself could not be mutated via this reference, or indeed any possible
reference.

```grammar
struct_reference
    : "ref" "var" type
    ;
```

## Variable Reference Safety

To ensure that stack references can never refer to an invalid memory location, they are only
permitted to be used as the types of parameters, local variables, return types, and fields of `ref
struct`s (ref structs are themselves confined to only be on the stack). They cannot be used as the
type of any object field. Additionally, the compiler will track which variables a variable reference
might refer to and prevent a reference from being returned from a function which might refer to
variables within the stack frame of that function.

## Working with Variable References

A value can be referenced by prefixing the variable or field name with `ref`. For example, given a
variable `var y: int = 42;` then `let x = ref y;` declares a variable `x` of type `ref int` and
assigns it a reference to `y`. To reference a variable prefix the name with `ref var`. For example,
`let z = ref var y;` declares a variable of type `ref var int` and assigns it a reference to `y`.

Variable references can be assigned new references and new values. A `var a: ref var int` can be assigned
a new reference by assigning into it an expression of type `ref var int`, but the variable it
references can be assigned by assigning into it an expression of type `int`. Likewise, to pass a
variable reference as a parameter, simply pass any expression of the appropriate reference type
which might be a variable with a variable reference type. A variable reference will be implicitly
dereferenced in contexts where the referenced type is expected.

## Equality

Variable references cannot be compared for equality. If an equality operator is used on two `ref`
types, they will be implicitly dereferenced and the operator evaluated on the result.
