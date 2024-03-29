# Reference Types

Reference types are either object reference types or function reference types. Additionally, the
special any reference type may refer to an object or function of any type.

```grammar
reference_type
    : object_type
    | function_type
    | any_type
    ;
```

## Object Reference types

An object reference type (object type for short) is a reference to an object. This object could be
of a concrete class type or a trait type or one a several built in object types. The two special
object types are `Type` and `Metatype`.

```grammar
object_type
    : identifier
    | "Type"
    | "Metatype"
    ;
```

### `Type` and `MetaType`

The type "`Type`" is the type of all objects that represent types during reflection and in generics.
The default type of a generic parameter is "`Type`" (thus "`foo[T]`" and "`foo[T: Type]` are
equivalent). Any generic parameter with this type is referred to as a "type parameter". Note that
"`Type`" is a reference type. This was necessary to allow metatypes to be subtypes of it.

Objects representing the type of classes and structs have a type that is a subtype of "`Type`". When
calling associated functions as "`Example.function()`", the expression "`Example`" has a type that
is a subtype of "`Type`". Metatypes describe the associated functions and constructors on the type
object. The meta type of a type can be accessed using the "`Type`" property, for example
"`Example.Type`".

These types can be very confusing. For some, an example clarifies the relationship.

```azoth
let example: Example = new Example();
let metaType: Example.Type = Example;
let type: Type = metaType; // Example.Type <: Type
```

## Function Types

Function types are the parameter and return types of a function. Note that function types are
contravariant in their parameter types and covariant in their return types.

```grammar
function_type
    : "(" type_list ")" "->" type
    ;

type_list
    : type{",", 0, *}
```

**TODO:** function types may need something to indicate how they handle their closure/state.
Currently thinking that `const () -> void` means the closure is constant, `mut () -> void` means it
is mutable (and thus can't be passed between threads), `() -> void` means the closure is read only
(and thus has no side effect) and `move () -> void)` means that it contains `move` types and is
consumed. The latter also makes the function type itself a move type. Thus move functions can only
be called once. One question is whether there needs to be a distinction between `mut` functions and
read only functions.

### Function Type Subtyping

Function types are contravariant in their parameter types and covariant in their return types. That
is if F1 <: F2 then for each parameter Px, F2.Px <: F1.Px. And F1.Return <: F2.Return.

The `lent` attribute of the parameters and returns also comes into play. If you translate `lent`
into which parameters can be referenced by the return value, then for a function type to be a
subtype of another it must be as or more restrictive in what parameters can be referenced by the
return value. This can get very confusing when flipping whether the return type is `lent`.

## `Any` Type

```grammar
any_type
    : "Any"
    ;
```

All reference types (object, and function) are a subtype of the "`Any`" type. Note that variable
references are not included in this.
