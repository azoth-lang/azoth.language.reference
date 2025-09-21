# Pattern Match Expressions

Pattern match expressions are used to determine if a value matches a pattern and can introduce
naming bindings as well. They allow refutable patterns in a non-binding context. The result of the
expression is a `bool` indicating whether the pattern matches the expression.

```grammar
pattern_match_expression
  : expression "is" pattern<binding=false>
  ;
```
