# Function Types

Function types are the parameter and return types of a function. Note that function types are
contravariant in their parameter types and covariant in their return types.

```grammar
function_type
    : function_capability "(" parameter_type_list ")" "->" type
    ;

function_capability
    : "move"
    | "const"
    | "mut"
    | "id"
    | ε
    ;

parameter_type_list
    : parameter_type{",", 0, *}
    ;

parameter_type
    : "lent"? type
    ;
```

**TODO:** function types may need something to indicate how they handle their closure/state.
Currently thinking that `const () -> void` means the closure is constant, `mut () -> void` means it
is mutable (and thus can't be passed between threads), `() -> void` means the closure is read only
(and thus has no side effect) and `move () -> void)` means that it contains `move` types and is
consumed. The latter also makes the function type itself a move type. Thus move functions can only
be called once. One question is whether there needs to be a distinction between `mut` functions and
read only functions.

**Answer:** `mut () -> void` is needed because accessing something through a read only reference
must now allow mutation. If it were possible to hide a function type inside a class and invoke it
from a read only method, then it could cause mutation. This also implies that function type
capabilities have different subtype relationship. `const <: read` as normal, but `read <: mut`, `mut
<: move`. I think is a symptom of the fact that a function type is sort of a pair of a closure value
and a function to call with the first parameter being the closure. Since the closure is a parameter,
it is contravariant. Are the temp capabilities needed?

**TODO:** `id` is a weird edge case. It almost seems useless since `id <: const` however, it might
allow some functions to be called even from within `id` contexts.

**Idea:** In the bytecode, don't have full blown closures and function types. Instead, have only
function types that take no closure or context. Implement function types as a struct of the closure
reference and the byte code function type with the closure as the first parameter. There may be an
issue with the closure type though since the public type should not include the closure type.
Options: use an associated type for the closure type (requires subtyping, e.g. `Function[Closure,
...] <: Function[...]`), have an existential type for the closure type (not sure existential types
will be supported), use some sort of special erased type that only still has a capability and is
unsafe (`Any` may not work because upcasting to it could change the vtable pointer?). This could
also provide a way to have true function pointers for C interop with `@() -> void` etc.

## Function Type Subtyping

Function types are contravariant in their parameter types and covariant in their return types. That
is if F1 <: F2 then for each parameter Px, F2.Px <: F1.Px. And F1.Return <: F2.Return.

The `lent` attribute of the parameters and returns also comes into play. If you translate `lent`
into which parameters can be referenced by the return value, then for a function type to be a
subtype of another it must be as or more restrictive in what parameters can be referenced by the
return value. This can get very confusing when flipping whether the return type is `lent`.
