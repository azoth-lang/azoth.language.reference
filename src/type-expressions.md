# Type Expressions

Type expressions allow types to be composed into further types. All type operators have the same
precedence. The order of type operations can be controlled using parenthesized expressions.

```grammar
type_expression
    : union_type
    | intersection_type
    | "(" type ")"
    ;
```

## Domains

For union type expression, all the types operated on must be from a single domain. There are two
ways type domains are formed.

1. All reference types and optional reference types form a domain.
2. All of the subtypes of each closed value or struct form a separate domain.

It is important to note that for the reference types, a single domain contains them all. Whereas for
the closed value and struct types, each closed value or struct creates a separate domains. Optional
value types cannot be unioned. Instead, the result of the union can be optional.

## Union Types

Union types are the union of other data types. A value of any of the types being unioned is a legal
value for the union type. Methods and properties common to all types in the union type can be
directly accessed on a value of the union type. Other methods and properties require that the value
be destructured to the specific type using `match` etc. Types being unioned are bare types and do
not each have their own capabilities. Thus `mut Foo | const Foo` is invalid, instead something like
`mut (Foo | Bar)` would be allowed.

```grammar
union_type
    : type "|" type
    ;
```

## Intersection Types

An intersection of reference types can be created using the type intersection operator. It is not
possible to intersect value or struct types. For a value to be of the intersection type, it must be
a subtype of all the types being intersected. Instances of an intersection type have all the methods
and properties of all the types being intersected.

```grammar
intersection_type
    : type "&" type
    ;
```

It is not possible to intersect value or struct types because there is no value that is a subtype of
multiple value or struct types.
