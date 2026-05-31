# Function Types

Function types are object types. Conceptually, it is an object with an invoke method. The function
type is determined by the parameters and return type of that method including the capability on the
`self` parameter. The object (i.e. the `self` parameter) is the closure that was captured with the
function. Function types are contravariant in their `self` capability and parameter types. They are
covariant in their return types. Typically, the capability used with the value matches the self
capability. This ensures that the function can be invoked. However, those two capabilities are
technically independent. If the object capability is not a subtype of the self parameter capability
then the function cannot be invoked. The unique nature of having two important capabilities and
being contravariant in the self parameter capability is supported by having different default
capabilities for function types.

Additionally, function types may or may not be drop types depending on whether the closure contains
any drop types. If the function `self` parameter is `iso` then invoking the function will drop the
closure. Otherwise, the closure will be dropped when the function goes out of scope.

**TODO:** should their be `unsafe` function types for referencing `unsafe` functions?

**TODO:** maybe function types should default to `mut` to allow subtyping to work better?

**TODO:** this design isn't good because the equivalent of `FnOnce` should allow any other function
to be passed where it is expected. But an `iso` function consumes the self argument in a way that
prevents it.

Contents:

* [Default Capabilities](#default-capabilities)
* [Function Type Syntax](#function-type-syntax)
* [Function Type Subtyping](#function-type-subtyping)
* [Most Common Function Types](#most-common-function-types)
* [Function Type Behavior](#function-type-behavior)
* [Implementation](#implementation)

## Default Capabilities

The default self parameter capability is `id`. This is because it is simultaneously the subtype of
all self parameter capabilities and safe to send between threads. Thus it is the most flexible. Note
that because `let` `const` fields can be accessed through an `id` reference, it is actually possible
for such a closure to capture constant values.

The default capability of the function type is the capability of the self parameter type.

## Function Type Syntax

Function types are preceded by a capability like all object types. They are then a list of parameter
types in parentheses followed by an arrow and the return type. Additionally, the capability of the
`self` parameter is listed as if it were the first parameter unless it is `id` which is the default.
If a capability for the function is omitted, the default is the capability of the self parameter.

It is not necessary to explicitly denote which functions are drop types. For any such function,
there will be a single `own` instance that has ownership. This indicates that the function is a drop type.

```grammar
function_type
    : capability "(" function_capability ("," parameter_type_list)? ")" "->" type
    | capability "(" parameter_type_list? ")" "->" type // id self function omits the capability and comma
    ;

function_capability
    : "iso"
    | "mut"
    | "read"
    ;

parameter_type_list
    : parameter_type{",", 1, *}
    ;

parameter_type
    : "lent"? type
    ;
```

Note that for the self parameter only the capabilities `iso`, `mut`, `read`, and the default of `id`
are allowed. The `const` capability is not included because there is no need for it when `id` is
more flexible while still being sendable.

**TODO:** are temp capabilities on the self parameter also needed?

## Function Type Subtyping

Function types are contravariant in their parameter types and covariant in their return types. That
is if `F1 <: F2` then for each parameter `Px`, `F2.Px <: F1.Px` and `F1.Return <: F2.Return`. It is
also contravariant in the self parameter type `iso <: mut <: read <: id`.

The `lent` attribute of the parameters also comes into play. For `F1 <: F2`, any lent parameters of
`F2` must also be lent in `F1`. The inverse is not the case. A subtype function can have more lent
parameters since this is an additional restriction on the behavior of the function.

Function types are, of course, covariant in their capability as all object types are. Since this is
typically identical with the self parameter capability, it is effectively invariant if you want to
maintain the ability to invoke the function.

## Most Common Function Types

While the combination of the capability of the function type and the self parameter provides lots of
possibilities, most of the time that is not used. Generally, a function type will be one of:

* `(T...) -> TReturn`
* `(read, T...) -> TReturn`
* `(mut, T...) -> TReturn`
* `(iso, T...) -> TReturn`
* `own (T...) -> TReturn`
* `own (read, T...) -> TReturn`
* `own (mut, T...) -> TReturn`
* `own (iso, T...) -> TReturn`

## Function Type Behavior

A function with an `iso` self parameter can be called only once. The closure is consumed by calling
it.

## Implementation

**Idea:** In the bytecode, don't have full blown closures and function types. Instead, have only
function types that take no closure or context. Implement function types as a struct of the closure
reference and the byte code function type with the closure as the first parameter. There may be an
issue with the closure type though since the public type should not include the closure type.
Options: use an associated type for the closure type (requires subtyping, e.g. `Function[Closure,
...] <: Function[...]`), have an existential type for the closure type (not sure existential types
will be supported), use some sort of special erased type that only still has a capability and is
unsafe (`Any` may not work because upcasting to it could change the vtable pointer?). This could
also provide a way to have true function pointers for C interop with `@() -> void` etc.
