# Statements

Blocks contain statements. Statements include variable declarations, constant declarations, and
expression statements.

```grammar
statement
    : variable_declaration
    | constant_declarations
    | expression_statement
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

## No Empty Statement

Many languages allow an empty statement consisting of only a semicolon. This fits with the fact that
control flow constructs operate on statements by allowing them to be empty. In Azoth, this is
accomplished with an empty block (i.e. `{ }`). Empty statements are disallowed to help avoid
confusion about abstract members. An abstract method `fn example(self);` could never be mistaken for
a method with an empty statement body because there is no empty statement.
