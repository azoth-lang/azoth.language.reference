# Traits

A trait defines an interface for a type. Traits can't have fields, but they can define behavior.
That behavior is then inherited by all classes that are subtypes of the trait. A class can implement
multiple traits by listing them in the base type list. Traits can be subtypes of other types. Trait
members that do not provide implementations are not marked "`abstract`".

```azoth
public trait Example <: SomeClass
{
}
```

```grammar
trait_declaration
    : access_modifier "closed"? ("const"|"drop")? "trait" identifier generic_parameters? base_types trait_body
    ;

trait_body
    : "{" trait_member* "}"
    ;

trait_member
    : named_function
    | get_function
    | set_function
    | operator_overload
    ;
```

## Const Traits

A trait may be marked `const`. This indicates any subtype of it must be `const`. This allows the
compiler to assume all references to such a trait are `const`. See constant classes and structs for
more information.

## Drop Traits

As with classes, structs and values, a trait may be declared as a `drop` type. This requires that
all subtypes of the trait be a `drop` class, struct, value, or trait.
