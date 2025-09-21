# Nested Functions

Functions can be declared inside of other functions and methods. When this is done, they are visible
only to the containing function or method. They do not have access to any variables in the scope of
the containing method. If that is desired, a lambda function must be used. It is a non-fatal error
for a nested function to occur before the return statement of the enclosing function or method.
Nested functions must always be declared private.
