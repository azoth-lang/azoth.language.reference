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
    | viewpoint_type
    | type_parameter
    | optional_type
    | function_type
    | type_expression
    | pointer_type
    ;
```
