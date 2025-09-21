# Capability Expressions

The [capability types](capability-types.md) require some special expressions to work with them.
However, these also find use with drop types.

```grammar
capability_expression
    : move_expression
    | freeze_expression
    | id_expression
    ;
```

## Move Expression

A move expression allows for moving the value out of a variable or field. This is useful both for
drop types and to recover isolation on reference types. Without a move expression, ownership cannot
be passed from a variable to a called function.

```grammar
move_expression
    : "move" identifier
    | "move" embedded_expression "." identifier
    ;
```

For variables, if it is a reference type then the flow typing of that variable is changed to `id`.
If it is a hybrid type, then the variable becomes inaccessible on all code paths after the move. The
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

## Implicit Move and Freeze

When calling a method, the self parameter can cause an implicit move or freeze. If the self
parameter is `iso self` then an implicit move can occur when calling the method. If the self
parameter is `const self` then an implicit freeze can occur when calling the method.
