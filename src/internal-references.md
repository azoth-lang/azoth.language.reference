# Internal References

In addition to regular reference types that allow values to reference objects on the heap and stack
reference which reference values on the stack or heap from the stack, there are also internal
references. These allow for an object to reference a value inside of another object on the heap. In
many ways they operate like stack references, but because they reference into the heap no tracking
is necessary to ensure they don't refer to invalid memory and they can be stored inside the fields
of classes and structs. These are distinguished from standard reference types because of their
performance impact. Because an internal reference can point into the middle of an object, it can
cause the garbage collector to need to do more work to determine which object is being referenced.
However, internal references are incredibly valuable in enabling the implementation of slices
referring into an existing array or list. For this reason they are included in Azoth.

`iref T` and `iref var T`

```grammar
internal_reference
    : internal_struct_reference
    | internal_variable_reference
    ;
```

## Internal Struct References

Internal struct references provide a read-only reference to a field inside of an object or an
element in an array. Just like simple struct references, they may only refer to fields whose type is
a struct. Referring to a field whose type was a reference type wouldn't make sense. It would have a
needless double reference. However, if via generics, `iref` is combined with a reference type `T`
then the type `iref T` is instead taken to be the type `T`.

```grammar
struct_reference
    : "iref" value_type
    ;
```

## Internal Variable References

An internal variable reference provides a mutable reference to a field inside an object or an
element in an array. Just like simple variable references, they may refer to both value types and
reference types.

```grammar
struct_reference
    : "iref" "var" type
    ;
```

## Working with Internal References

Similar to stack references, internal references allow both for mutation of the referenced variable
and for assigning a new reference.

## Conversion to Stack References

Internal references are implicitly convertible to stack references. When this is done, the compiler
will infer that the resulting stack reference is safe to return from the function as it refers to a
value on the heap that will be kept alive as long as the reference exists.

**TODO:** should this be subtype instead?
