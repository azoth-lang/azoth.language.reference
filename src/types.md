# Types

Types in Azoth are made a little more complex by one of the unique features of Azoth, namely
reference capabilities. But that complexity gives a great amount of power and safety. There are
three major categories of types in Azoth. They are *empty types*, *value types* and *reference
types*. The latter two of which have capabilities applied to them. In addition to those, Azoth
completes the set of possible types with *optional types*, *function types*, *type expressions*
(i.e. union and intersection types), and several other special types.

The empty types are those types for which there is no value of that type. Value types are types
whose values are passed by value (e.g. they are typically copied). Reference types are types whose
values are automatically held by reference. Both value and reference types may be *generic types*,
which have *generic parameters*. Generic parameters can be types (both value and reference types) or
constant values. Generic parameters that are types are called *type parameters* and can be used as
types. *Optional types* modify types to add a `none` value. Function types allow function references
to be used as values. Type expressions allow the construction of new types by intersecting and
unioning existing types within certain limitations. Beyond those, there are *variable reference
types* and *internal reference types*, and *pointer types*.

```grammar
type
    : empty_type
    | capability_type
    | value_type
    | reference_type
    | type_parameter
    | optional_type
    | type_expression
    | pointer_type
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
