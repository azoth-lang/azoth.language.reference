# `void` Type

```grammar
void_type
    : "void"
    ;
```

The `void` type functions similarly to `void` in languages like C, Java, and C#. However, those
languages don't treat it as a proper type. In Azoth, `void` is a type that can be used as the
argument to a generic parameter. Semantically, the void type behaves somewhat like a special unit
type. However, rather than being a unit type, it represents the absence of a type. Functions that
return `void` don't return a value. Parameters of type `void` are dropped from the parameter list.
It is illegal to directly use `void` in a number of ways a type normally can be simply because the
result would be somewhat nonsensical. A variable or parameter can't be directly declared to have a
void type. An expression of type void can't be assigned into anything. However, when void is used as
the argument of a type parameter, it can cause variable and parameter declarations to be of type
`void`. This is not an error. Parameters of type `void` are removed from the parameter list.
Variables of type void are not allocated space. This can also create situations where an expression
of a generic type is actually of type `void` and assigned into a variable of the same generic type.
This is not an error. Effectively, the assignment is not performed.

Functions declared without a return arrow (`->`) and return type are defaulted to returning `void`.
Explicitly declaring a function as returning `void` is a non-fatal error.

The `void` type is not a supertype or subtype of any other type except for `never` which is a
subtype of all types including `void`. However, there is an implicit conversion from any type to
`void`. This implicit conversion allows one to explicitly override a `void` returning method with
one that returns a value.
