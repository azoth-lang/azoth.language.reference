# Value Declarations

Values are similar to classes. They contain field and function members. Unlike classes, values are
value types and by default are allocated inline. A variable with a value type directly contains the
data of the value. A variable of a class type contains a reference to the object which contains the
data. Most commonly, values are declared `const`. However, there are use cases for non-constant
values.

```grammar
struct
    : access_modifier ("move"|"const")? "value" identifier //...
    ;
```

## Capabilities when Copying

Capabilities on value types are handles the same when copying a value as if a reference was being
copied. For example, if a value is `iso` and is copied, then the alias and the original will now be
`mut`. This behavior is consistent with references and safe for the references inside of the value.

## Pseudo References

While most value types are declared `const`, there is one use of non-const value types that is
important enough to call out. That is as a pseudo reference. Some value types hold a single
reference and so act as if they were a reference type whose value was the referenced object. This
can be useful for producing values with special behaviors. For example, this is part of how array
slicing works. An array is a pseudo reference that internally contains a length and a reference to
the start of the array slice.

### Drop Value Types

Since value types can act like references, it is critical that they be able to act like references
to drop types. This is what declaring a value `drop` does. Obviously, their will be copies of the
value but moving it and recovering isolation will ensure that there is only one reference to the
object referenced by the value and allow it to be dropped.

## Record Values

Record values provide a shorthand for declaring record types. That is types that are the
composition of a named, ordered sequence of other types. They work just like record classes. A
record value is declared with a parameter list after the struct name (e.g. `move struct
Example(foo: int, bar: string)`). These parameters then become fields of the type. Declaring a
record value implicitly declares three things in the value. Additional members can be declared in
additional to the implicitly declared ones.

First, each of the parameters is declared as a field of the struct. For this reason, the parameters
may not be `lent`. A value type cannot have `var` fields so the parameters cannot be declared `var`.
Each of the implicitly declared fields is `public` unless the struct is `published` in which case
they are declared `published`. These fields can implicitly override abstract getters and setters.
They cannot override or hide non-abstract ones.

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
