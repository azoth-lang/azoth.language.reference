# Variable Declarations

Variable declarations create space for the storage of values within a function etc. They come in two
forms. The `let` statement allows for the binding of a name to a value. The `var` statement creates
a variable that can be assigned new values as the program runs. Use of `let` is preferred. The
compiler will issue non-fatal errors for `var` statements that could be replaces with `let`.

```grammar
variable_declaration
    : let_statement
    | var_statement
    ;
```

## Let Statements

Let statements bind a name to a value. Each name bound via a let statement has a type. This type may
be explicitly declared or it may be inferred from the value the name is bound to.

By default let statements are inferred to have a read only reference capability even if they are
bound to a mutable value. To bind a name to a mutable value the let statement must explicitly be
declared as mutable. This can be done either by declaring the full type of the variable binding or
with the shorthand let syntax that allows for the declaration of a reference capability only. The
reason bindings default to read only is so that mutability will always be explicit in Azoth
programs. The designers of Azoth believe that immutability should be the default.

```grammar
let_statement
    : "let" identifier (":" type)? "=" expression ";"
    | "let" identifier (":" reference_capability) "=" expression ";"
    ;
```

## Var Statements

Var statements bind a name to a storage location that can be assigned new values during the
execution of the program. As with let statements each variable has a type and these types may be
explicitly declared or inferred from the initial variable value. However, variables can also be
declared without providing an initial value. These variables are said to be unassigned. All
variables must have been definitely assigned some value before being read from.

```grammar
var_statement
    : "var" identifier (":" type)? "=" expression ";"
    | "var" identifier (":" reference_capability) "=" expression ";"
    | "var" identifier ":" type ";"
    ;
```

## Mutability

As mentioned, by default any reference type variable binding is read only. Thus, the value
referenced cannot be mutated through that reference. For `var` declarations, the variable can be
assigned a new reference. Thus the mutability of the reference is distinct from the mutability of
the object being referenced. All for combinations are possible (e.g. `let x: T`, `let x: mut T`,
`var x: T`, `var x: mut T`).

The mutability of value/struct type variable bindings works differently. Since a struct value is
stored directly in the variable, the mutability of the value is determined by the mutability of the
binding. Thus `let x: T` for a struct type implies that `x` cannot be assigned a new value and that
the current value is read only. While `var x: T` indicates that `x` can both be assigned new value
and that the value may be mutated.

Some value types are inherently immutable. So while the variable binding may be mutable, the value
is still not mutable. It is only possible to assign the variable a new value. For example, `int` is
a `const struct` and cannot be mutated, so `var x: int` declares a variable of an immutable value
but the variable can be assigned a new integer value.

## Definite Assignment

As mentioned before variables in Azoth must be definitely assigned. This means that the compiler
will check that a value has been assigned to a variable along every code path before that variable's
value is read.

```azoth
let x: int;
let y = x + 5; // compile error: can't use x before it is assigned a value
```

## Rebinding

A `let` binding can be redeclared in order to bind a name to a different value.

```azoth
let x = 6; // x: int
let y = x + 6; // x is an int here
let x = "Hello"; // x: string
let s = x + " World"; // x is a string here
```

Note this can only be done with `let` bindings, not with `var`. This is to avoid confusion between
assigning a new value to a variable and redeclaring it.

## Scopes and Shadowing

Bindings declared in a scope are only usable inside that scope after they are declared.

```azoth
let x = 65;
{
    let y = 3;
    let z = x + y; // both `x` and `y` are in scope
}
let p = x; // `x` is in scope
let q = x + y; // compile error: `y` can't be accessed because it is out of scope
```

When a variable hides another variable in an outer scope with the same name, that is called
shadowing. Most languages allow shadowing everywhere. Azoth does not allow shadowing within
functions.

```azoth
let x = 65;
{
    let x = 2; // Not allowed, this `x` shadows the other `x`
}
let y = x; // `y` will be 65
```

Shadowing is allowed between a variable and a class member or declaration outside the class.
However, Azoth follows the tradition of C# in not allowing shadowing within the parameters or
variables of a function. This avoids programmer confusion and prevents bugs while almost never
causing inconvenience.

## Patterns?

A section about destructuring patterns?
