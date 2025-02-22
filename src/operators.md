# Operators

```grammar
operator_expression
  : equality_expression
  | comparison_expression
  | math_expression
  | range_expression
  | assignment_expression
  | logical_operator_expression
  | reference_expression
  | pointer_expression
  ;
```

## Operator Precedence

| Operators                                                         | Category or name            |
| ----------------------------------------------------------------- | --------------------------- |
| `f()`, `x.y`                                                      | Primary                     |
| `+x`, `-x`, `await`, `move`, `freeze`                             | Unary                       |
| `x ^ y`                                                           | Exponentiation              |
| `x * y`, `x / y`                                                  | Multiplicative              |
| `x + y`, `x - y`                                                  | Additive                    |
| `as`, `as!`, `as?`                                                | Conversion                  |
| `..`, `..<`, `<..`, `<..<`                                        | Range                       |
| `x < y`, `x <= y`, `x > y`, `x >= y`, `x @== y`, `x @=/= y`, `is` | Relational and type-testing |
| `x == y`, `x =/= y`                                               | Equality                    |
| `not x`                                                           | Logical Not                 |
| `x and y`                                                         | Logical And                 |
| `x or y`, `x xor y`                                               | Logical Or                  |
| `x implies y`, `x iff y`                                          | Implicative                 |
| `x ?? y`                                                          | Coalesce                    |
| `=` `+=` `-=` `*=` `/=` `??=`                                     | Assignment                  |

**TODO:** Swift combines `as` and `is` in a level below range but above relational.
**TODO:** Swift put `??` above relational. That seems wrong since comparisons are lifted to optional.

## Equality Operators

## Comparison Operators

* Reference equality is at this level because it cannot be applied to `bool` or `bool?` so when
  mixed with equality, one wouldn't expect it to have the same precedence. For example,
  `should_be_same() == a @== b` only works if `@==` has higher precedence than `==`.

## Math Operators

## Range Operators

* In C# the range operator has higher precedence than the multiplicative operators but that leads to
  issues where one can't do math to determine the ends of the range (e.g. `0..list.Count-1` doesn't
  work as expected). That is somewhat mitigated by the `..<` operator in Azoth which reduces the
  need for math on the endpoint. However, it still seems to make sense that math and conversion
  should have higher precedence than range. It is unlikely that one would create a range and then
  immediately try to do a conversion on it.

## Assignment Operators

## Logical Operators

* `not` has a lower precedence than the traditional unary precedence. This is the same as Python
  uses and the rationale behind it is that this should result in less parenthesizing. For example,
  `not x < y` would usually not make sense as `(not x) < y` as comparing a boolean value with
  relational operators don't make sense. Grouping the logical operators like this at the bottom
  means, that they are all right below the relational operators, already at the level where we can
  expect boolean values. It also fits with the fact that `not` is a keyword that is separated from
  the argument with a space unlike `!` which is usually written unseparated (i.e. `!x`).

## Reference and Pointer Operators
