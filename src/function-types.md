# Function Types

Function types are a composite type composed of the parameter and return types of a function. In
addition to this, they account for which parameters are lent and for the reference capability of any
possible closure. Note that function types are contravariant in their parameter types and covariant
in their return types.

Because the closure is effectively passed as a parameter to the function, this makes the closure
capability contravariant and effectively flips the subtyping relationship for the closure
capability. This is reflected in the syntax by listing the capability as if it were the first
parameter. It is not the capability of what you can do to the function object. It is the capability
of what the function can do to its closure. This also leads to a different set of capabilities
making sense.

```grammar
function_type
    :  "(" function_capability ("," parameter_type_list)? ")" "->" type
    |  "(" parameter_type_list? ")" "->" type // read function omits the capability and comma
    ;

function_capability
    : "iso"
    | "own"
    | "mut"
    | "const"
    ;

parameter_type_list
    : parameter_type{",", 1, *}
    ;

parameter_type
    : "lent"? type
    ;
```

**TODO:** Are temp capabilities and `id` also needed?

**TODO:** maybe `iso` should mean the closure is `iso` and so other references aren't allowed and it
is safe to pass the function reference between threads?

**TODO:** Maybe `own` should instead be a prefix `drop` making it a drop type.

**TODO:** how is ownership tracked for `drop` functions?

**TODO:** Maybe `id` should no allow invocation, but exists as the capability after a function
reference is moved? This would probably mean `id` goes in front.

**TODO:** should their be `unsafe` function types for referencing `unsafe` functions?

**Idea:** In the bytecode, don't have full blown closures and function types. Instead, have only
function types that take no closure or context. Implement function types as a struct of the closure
reference and the byte code function type with the closure as the first parameter. There may be an
issue with the closure type though since the public type should not include the closure type.
Options: use an associated type for the closure type (requires subtyping, e.g. `Function[Closure,
...] <: Function[...]`), have an existential type for the closure type (not sure existential types
will be supported), use some sort of special erased type that only still has a capability and is
unsafe (`Any` may not work because upcasting to it could change the vtable pointer?). This could
also provide a way to have true function pointers for C interop with `@() -> void` etc.

## Function Type Capabilities

The capabilities of a function type are really the capability of the closure passed to the function.
There are five function type capabilities.

The `iso` capability indicates that the closure will be consumed by the function. Thus the closure
is or contains isolated values. An `iso` function can only be invoked once. Calling it effectively
moves the function reference into the called function leaving behind `void`. The closure may still
contain mutable references and calling the function may mutate objects reachable from the closure.

The `own` capability indicates that the closure has drop types in it and it makes the function a
drop type. The closure may still contain mutable references and calling the function may mutate
objects reachable from the closure.

The `mut` capability indicates that the closure contains `mut` types and calling the function may
mutate objects reachable from the closure.

The read capability indicates that the closure is read and calling the function will not mutate any
objects reachable from the closure.

The `const` capability indicates that the closure contains only `const` and `id` values. Calling it
will not not mutate any objects reachable from the closure. Furthermore, it is safe to pass `const`
function references between threads.

## Function Type Subtyping

Function types are contravariant in their parameter types and covariant in their return types. That
is if `F1 <: F2` then for each parameter `Px`, `F2.Px <: F1.Px`. And `F1.Return <: F2.Return`.

The `lent` attribute of the parameters also comes into play. For `F1 <: F2`, any lent parameters of
`F2` must also be lent in `F1`. The inverse is not the case. A subtype function can have more lent
parameters since this is an additional restriction on the behavior of the function.

The capability of a function is essentially contravariant thus for function types `const <: read <:
mut <: iso` and `mut <: own`. Note that `own` is not a subtype of `iso` because `own` makes a
function a drop type while `iso` doesn't. Also, this is not fully contravariant because `const <:
read`. This is because the function reference also has a hidden reference to the closure. That
closure must be `const` to be passed between threads.
