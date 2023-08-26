# Design By Contract

Functions, methods, and types can be annotated with contracts that indicate and enforce
preconditions, post-conditions and invariants.

Preconditions and post conditions are declared on functions and methods with the `requires` and
`ensures` keywords respectively. Multiple preconditions and post conditions can be specified by
repeated use of `requires` and `ensures`. All preconditions must be declared before any post
conditions.

## Preconditions

Precondition expressions have read only access to all argument values including `self`.

## Post Conditions

Post conditions have readonly access to the original value of all arguments including `self` and the
current state of `self` and the return value. The previous value of `self` can be accessed with the
special variable `old` and the previous values of parameters can be accessed with the special
expression `old()` (e.g. `old(x)` gives the original value of `x`). The return value of the function
can be accessed by using `return` as a variable.

**TODO:** is it really reasonable to preserve the prior state of arguments including self if they
have been mutated. That may be an unreasonable cost. It may also require copying that isn't
otherwise possible in the language. Perhaps this could be available only for copy types?

## Invariants

Invariants are declared on the class or struct level with the `invariant` keyword. These must hold
before and after every public method call on the type.

**TODO:** Midori has `protected` and `private` invariants that held for protected and private
methods respectively.

## Enforcement

* By default, all contracts are checked at runtime.
* The compiler is free to prove contracts false, and issue non-fatal compile-time errors.
* The compiler is free to prove contracts true, and remove these runtime checks.

Which one happens varies by call site so that at one call site the compiler might prove a
precondition satisfied while at another one there must be a runtime check.

If a contract is not satisfied at runtime this is an abort error. See [Error
Handling](error-handling.md) and [Aborting](aborting.md).
