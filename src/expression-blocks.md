# Expression Blocks

```grammar
expression_block
    : block
    | result_expression
    ;
```

## Blocks

A block is zero or more statements enclosed in curly braces. If the block contains a result
expression, then the type of the expression is the type of the result expression. If a block
contains no result expression, then the type of the expression is "`void`".

```grammar
block
    : "{" statement* "}"
    ;
```

## Result Expressions

A result expression is used to provide the value a block evaluates to. Certain control flow
expressions like "`if`" allow a result expression to be used directly as the body of the expression.
In that case the block has the type of the expression and yields the expression's value. The type of
the block is determined by the type of the result expression for the block. It is an error to have
statements or expressions after the control flow is terminated by the result expression.

```grammar
result_expression
    : "=>" expression
    ;
```

A result expression can't be used outside of a block. However, it can be used to create a block that
can be used as an expression with a value.

```azoth
var x = => "Hello"; // ERROR
x = {
        DoSomething();
        => "Goodbye";
    };
```
