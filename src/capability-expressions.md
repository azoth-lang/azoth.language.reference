# Capability Expressions

The [reference capability types](reference-capabailities.md) require some special expressions to
work with them. However, these also find use with movable types.

```grammar
capability_expression
    : move_expression
    | freeze_expression
    | id_expression
    ;
```

## Move Expression

A move expression allows for moving the value out of a variable or field. This is useful both for
move types and to recover isolation on reference types. Without a move expression, ownership cannot
be passed from a variable or field to a called function.

```grammar
move_expression
    : "move" identifier
    | "move" embedded_expression "." identifier
    ;
```

For variables, if it is a reference type then the flow typing of that variable is changed to `id`.
If it is a value type, then the variable becomes inaccessible on all code paths after the move. The
compiler issues fatal errors for attempts to access a potentially moved value type.

For fields, the field must be an optional type and have a mutable binding. The value will be moved
from the field leaving the value `none` in the field.

## Freeze Expression

A freeze expression allows one to convert a reference to `const`. After freezing a variable, that
variable is changed to `const T`. The freeze expression also returns the constant reference.

```grammar
freeze_expression
    : "freeze" identifier
    ;
```

## Id Expression

**TODO:** remove since this is no longer the idiom for reference equality (`id` could be applied to
structs but they still shouldn't be compared for reference equality).

Typically, conversion to `id T` is implicit. However, it can be helpful to explicitly get an
identity reference to something. The `id` expression allows this. This is used as part of the idiom
for checking reference equality (i.e. `id x == id y`).

```grammar
id_expression
    | "id" embedded_expression
    ;
```

## Implicit Move and Freeze

When calling a method, the self parameter can cause an implicit move or freeze. If the self
parameter is `iso self` then an implicit move can occur when calling the method. If the self
parameter is `const self` then an implicit freeze can occur when calling the method.
