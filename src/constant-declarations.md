# Constant Declarations

Constants can be declared in namespaces using the `const` keyword. A constant must have a type whose
capability is `const` and all independent parameters are `const`. However, not all generic
parameters must be `const`. For example, one could have a `const Factory[iso String]` that
constructed isolated strings. They must have an initializer and that initializer must be evaluated
at compile time.
