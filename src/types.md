# Types

Types in Azoth are made a little more complex by one of the unique features of Azoth, namely
reference capabilities. But that complexity gives a great amount of power and safety. There are
three major categories of types in Azoth. They are *extrema types*, *value types* and *reference
types*. The latter two of which have capabilities applied to them. Within those categories, further
subcategories exist. For example, Azoth supports *optional types*, *function types*, *type
expressions* (i.e. union and intersection types), and other special types.

The extrema types act in some ways as what is known in type theory as a top and bottom type. Value
types are types whose values are passed by value (e.g. they are typically copied). Reference types
are types whose values are automatically held by reference. Both value and reference types may be
*generic types*, which have *generic parameters*. Generic parameters can be types (both value and
reference types) or constant values. Generic parameters that are types are called *type parameters*
and can be used as types. *Optional types* modify types to add a `none` value. Function types allow
function references to be used as values. Type expressions allow the construction of new types by
intersecting and unioning existing types within certain limitations. Beyond those, there are
*stack reference types* and *internal reference types*, and *pointer types*.

```grammar
type
    : extrema_type
    | capability_type
    | viewpoint_type
    | type_parameter
    | optional_type
    | function_type
    | type_expression
    | variable_reference
    | internal_reference
    | pointer_type
    ;
```

## Reference Types

Reference types have distinct behavior when combined with optional types and have distinct behavior.
The following types are value types:

* Types defined by class or trait declarations (whether with or without a capability).
* Type parameters constrained to reference types
* Function types
* Variable references
* Internal references
