# Types

There are three main categories of types in Azoth: *empty types*, *value types* and *reference
types*. In addition to those, there are a number of other categories of types that are special in
some way.

The *empty types* are those types for which there is no value of that type. Both value and reference
types may be *generic types*, which have *generic parameters*. Generic parameters can be types (both
value and reference types) or constant values. Generic parameters that are types are called *type
parameters* and can be used as types. *Optional types* modify value types or reference types to add
a `none` value. Additionally, *type expressions* allow the construction of new types by intersecting
and unioning existing types within certain limitations.

```grammar
type
    : empty_type
    | value_type
    | reference_type
    | type_parameter
    | optional_type
    | type_expression
    ;
```

The value types can be further divided into a number of subcategories. Most value types are *struct
types*. The *simple types* are predefined value types identified with keywords. The *tuple type* is
a predefined type for tuples of values. *Pointer types* allow unsafe code to directly use pointers
to addresses in memory.

```grammar
value_type
    : simple_type
    | struct_type
    | tuple_type
    | pointer_type
    ;
```
