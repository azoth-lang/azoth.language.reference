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

For any type expression, all the types operated on must be from a single domain. There are two ways
type domains are formed.

1. All reference and optional reference types form a domain.
2. All of the subtypes of each closed struct form a separate domain.
3. All of the optional variants of the subtypes of each closed struct form a separate domain.

It is important to note that for the reference types, a single domain contains them all. Whereas for
the close struct types, each closed struct creates two separate domains, one for the optional
variant and one for the non-optional variant.

**TODO:** Do not include optional types here. Make that have to be done to the union or
intersection. Also clarify that reference capabilities apply across the whole type (e.g. `mut Foo |
const Foo` isn't valid, instead `mut (Foo | Bar)`). This needs to properly handle function types.

## Union Types

Union types are the union of other data types. A value of any of the types being unioned is a legal
value for the union type. Methods and properties common to all types in the union type can be
directly accessed on a value of the union type. Other methods and properties require that the value
be destructured to the specific type using `match` etc.

```grammar
union_type
    : type "|" type
    ;
```

## Intersection Types

Intersection types are the intersection of other data types. For a value to be of the intersection
type, it must be a subtype of all the types being intersected. Instances of an intersection type
have all the methods and properties of all the types being intersected.

```grammar
intersection_type
    : type "&" type
    ;
```

**TODO:** it may not be possible to intersect value types.
