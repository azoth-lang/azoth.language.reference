# Execution Order

Expressions are evaluated left to right. Arguments to function calls are evaluated left to right. If
an expression contains only pure functions/operations, then the compiler may execute them in any
order for optimization purposes. The compiler may assume that functions marked pure are pure and can
be evaluated in any order.

**TODO:** how are functions marked pure (e.g. effect types?), or is it inferred from the reference
capabilities in the signature?
