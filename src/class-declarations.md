# Class Declarations

## Const Classes

Marking a class `const` indicates that all instances of this type will be constant. It constrains
all initializers to return a `const` reference. Consequently it constrains all field bindings to be
immutable (i.e. `let`) and have `const` types. The compiler will then treat all references
to that type as `const` except for a few special cases like the `self` reference of the initializer.

**TODO:** should it be an error to use `const T` with a `const` type? That would avoid debates about
whether one should use `const`. However, it also means changing a type to `const` requires updating
any existing `const T` uses.

Constantness is inherited. All subtypes of a constant type are required to be `const` themselves.

## Drop Classes

Classes can be declared as drop classes. A drop class is allowed to contain fields whose types are
drop types. A drop class can have a destructor. The compiler ensures that for all droppable classes,
isolation is recovered before the last reference leaves scope and that the destructor is called.
Thus, while multiple references to a droppable reference type may exist at one time, there will
always conceptually be one stack frame that "owns" the instance. To pass ownership one must `move`
an `iso` reference to the instance to another method or field.

The drop declaration of a class is inherited by all subtypes (i.e. all subtypes are also droppable).
Subclasses must be declared `drop` as well.

## Empty Classes

An empty class with no members can be abbreviated by replacing the class body with a semicolon (e.g.
`public class Example;`). This is most useful with record classes.

## Record Classes

Record classes provide a shorthand for declaring record types. That is types that are the
composition of a named, ordered sequence of other types. A record class is declared with a parameter
list after the class name (e.g. `class Example(foo: int, bar: string)`). These parameters then
become fields of the type. Declaring a record class implicitly declares three things in the class.
Additional members can be declared in additional to the implicitly declared ones.

First, each of the parameters is declared as a field of the class. For this reason, the parameters
may not be `lent`. A parameter may be declared `var` in which case the corresponding field will be
declared `var`. Of course a `const` class cannot have `var` fields. Each of the implicitly declared
fields is `public` unless the class is `published` in which case they are declared `published`.
These fields can implicitly override abstract getters and setters. They cannot override or hide
non-abstract ones.

Second, an initializer is declared with corresponding field parameters (e.g. `init(.foo, .bar)` for
the `Example` class). The implicitly declared initializer is `public` unless the class is
`published` in which case it is declared `published`. This initializer is otherwise not special and
additional initializers may be declared. Those initializers are not required to call the record
initializer. They must simply follow the initialization safety rules of all initializers. It is not
possible to add logic to this initializer or call a base class initializer. However, if an
initializer with the same parameter types is declared, it replaces the initializer that would
otherwise be generated. That initializer may contain logic and call a base or other initializer.

Third, a pattern match deconstructor is declared corresponding the the parameters/fields (e.g.
`operator is(self) -> Tuple[int, string]` for the `Example` class). This allows the record class to
be deconstructed as part of an assignment or pattern match. If another deconstuctor is explicitly
declared, the implicit one is still generated. It is not possible to replace this deconstructor. If
control of the deconstructor is needed, then a regular class should be declared and members that
would be implicitly declared by a record class can be manually written.

## Sealed Classes

I class can be declared with the `sealed` modifier. This indicates that no subtypes of the type can
be declared. Note this is distinct from `closed` which allows subtypes declared in the current
package. A class cannot be declared both `sealed` and `closed`.
