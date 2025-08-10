# Struct Declarations

```grammar
struct
    : access_modifier "move" "struct" identifier //...
    ;
```

## Record Structs

Record structs provide a shorthand for declaring record types. That is types that are the
composition of a named, ordered sequence of other types. They work just like record classes. A
record struct is declared with a parameter list after the struct name (e.g. `move struct
Example(foo: int, bar: string)`). These parameters then become fields of the type. Declaring a
record struct implicitly declares three things in the struct. Additional members can be declared in
additional to the implicitly declared ones.

First, each of the parameters is declared as a field of the struct. For this reason, the parameters
may not be `lent`. A parameter may be declared `var` in which case the corresponding field will be
declared `var`. Each of the implicitly declared fields is `public` unless the struct is
`published` in which case they are declared `published`. These fields can implicitly override
abstract getters and setters. They cannot override or hide non-abstract ones.

Second, an initializer is declared with corresponding field parameters (e.g. `init(mut self, .foo,
.bar)` for the `Example` struct). The implicitly declared initializer is `public` unless the struct
is `published` in which case it is declared `published`. This initializer is otherwise not special
and additional initializers may be declared. Those initializers are not required to call the record
initializer. They must simply follow the initialization safety rules of all initializers. It is not
possible to add logic to this initializer or call a base class initializer. However, if an
initializer with the same parameter types is declared, it replaces the initializer that would
otherwise be generated. That initializer may contain logic and call a base or other initializer.

Third, a pattern match deconstructor is declared corresponding the the parameters/fields (e.g.
`operator is(self) -> Tuple[int, string]` for the `Example` struct). This allows the record struct
to be deconstructed as part of an assignment or pattern match. If another deconstuctor is explicitly
declared, the implicit one is still generated. It is not possible to replace this deconstructor. If
control of the deconstructor is needed, then a regular struct should be declared and members that
would be implicitly declared by a record struct can be manually written.
