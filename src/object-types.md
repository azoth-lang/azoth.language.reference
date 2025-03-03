# Object Types

Reference types are either object reference types or function reference types. Additionally, the
special any reference type may refer to an object or function of any type.

```grammar
object_type
    : identifier
    | "Type"
    | "Metatype"
    | "Any"
    ;
```

## Ordinary Object Types

An ordinary object type is a reference to an object. This object could be of a concrete class type
or a trait type.

## `Type` and `MetaType`

The type "`Type`" is the type of all objects that represent types during reflection and in generics.
The default type of a generic parameter is "`Type`" (thus "`foo[T]`" and "`foo[T: Type]` are
equivalent). Any generic parameter with this type is referred to as a "type parameter". Note that
"`Type`" is a reference type. This was necessary to allow metatypes to be subtypes of it.

Objects representing the type of classes and structs have a type that is a subtype of "`Type`". When
calling associated functions as "`Example.function()`", the expression "`Example`" has a type that
is a subtype of "`Type`". Metatypes describe the associated functions and initializers on the type
object. The meta type of a type can be accessed using the "`type`" property, for example
"`Example.type`".

These types can be very confusing. For some, an example clarifies the relationship.

```azoth
let example: Example = Example();
let metaType: Example.type = Example;
let type: Type = metaType; // Example.type <: Type
```

## `Any` Type

All object and function types are a subtype of the "`Any`" type. Note that variable
references are not subtypes of `Any`.

**TODO:** document that `Any` provides the reference hash function.
