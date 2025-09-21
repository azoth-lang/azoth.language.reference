# Statements

Blocks contain statements. Statements include variable declarations, constant declarations, and
expression statements. There is also a special empty statement.

```grammar
statement
    : variable_declaration
    | constant_declarations
    | expression_statement
    | ";" // empty statement
    ;
```

## Expressions Statements

Expression statements are statements that are simply expressions. Block, choice, loop and `with`
expressions can be used without a closing semicolon because their extent is delimitated by curly
braces. Other expression statements require a semicolon terminator to delimit the end of the
statement.

```grammar
expression_statement
    : block
    | choice_expression
    | loop_expression
    | with_expression
    | expression ";"
    ;
```

## Empty Statement

The empty statement is a special statement that has no effect. It may be useful in situations where
a statement is expected, but no action is desired.

**TODO:** Perhaps there should be no empty statement. With curly brace blocks there may be no need
for it. Rather, it may just be an opportunity for mistakes. It could also confuse the case of
abstract implied by no method body.
