# Class Declarations

## Const Classes

Marking a class `const` indicates that all instances of this type will be constant. It constrains
all constructors to return a `const` reference. Consequently it constrains all field bindings to be
immutable (i.e. `let`) and have `const` reference types. The compiler will then treat all references
to that type as `const` except for a few special cases like the `self` reference of the constructor.

**TODO:** should it be an error to use `const T` with a `const` type? That would avoid debates about
whether one should use `const`. However, it also means changing a type to `const` requires updating
any existing `const T` uses.

Constantness is inherited. All subtypes of a constant type are required to be `const` themselves.

## Move Classes

Classes can be declared a movable classes. A movable class is allowed to contain fields whose types
are movable value types and movable reference types. A movable class can have a destructor. The
compiler ensures that for all moveable classes, isolation is recovered before the last reference
leaves scope and that the destructor is called. Thus, while multiple references to a moveable
reference type may exist at one time, there will always conceptually be one stack frame that "owns"
the instance. To pass ownership one must `move` an `iso` reference to the instance to another method
or field.

The move declaration of a class is inherited by all subtypes (i.e. all subtypes are also movable).

Note: it doesn't make sense to have an `iso class` because that would not obviously imply that they
could contain movable value types and have constructors. Furthermore, presumably one would still
want to allow non-`iso` references to these classes so it wouldn't be as if they were also `iso`.
Finally, `move` is inherited by subtypes where as other class capabilities are not.
