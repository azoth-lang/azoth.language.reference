# Types

Types in Azoth are made a little more complex by one of the unique features of Azoth, namely
capabilities. But that complexity gives a great amount of power and safety. There are three major
categories of types in Azoth. They are *reference types*, *value types*, and *hybrid types*. All of
which have capabilities applied to them. Within those categories, further subcategories exist. For
example, Azoth supports *optional types*, *function types*, *type expressions* (i.e. union and
intersection types), and other special types.

The never type is what is known in type theory as the bottom type. Value types are types whose
values are passed by value (i.e. they are copied). Reference types are types whose values are
automatically held by reference. Hybrid types blend aspects of value types and reference types. All
three may be *generic types*, which have *generic parameters*. Generic parameters can be types or
constant values. Generic parameters that are types are called *type parameters* and can be used as
types. *Optional types* modify types to add a `none` value. Function types allow function references
to be used as values. Type expressions allow the construction of new types by intersecting and
unioning existing types within certain limitations. In addition to those, there are *pointer types*.

```grammar
type
    : never_type
    | void_type
    | capability_type
    | viewpoint_type
    | type_parameter
    | optional_type
    | function_type
    | type_expression
    | pointer_type
    ;
```

## Reference Types

Reference types have distinct behavior when combined with optional types. The following types are
reference types:

* Types defined by class or trait declarations (whether with or without a capability).
* Type parameters constrained to reference types
* Function types
