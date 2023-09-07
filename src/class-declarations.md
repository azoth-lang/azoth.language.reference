# Class Declarations

## Class Capabilities

A class is declared with one of three reference capabilities `mut`, `const`, or read-only which is
indicated by the absence of a reference capability. These capabilities dictate which kind of
reference capabilities may be applied to the fields of the class. Note that while the class
capability determines which capabilities may be applied to fields of the class, it does not limit
which capabilities may be used with references to that type. This is because subtypes may be
declared with a different class capability and thus contain fields that would not be allowed bye the
by the base class capability.

### Const Classes

A const class can contain only constant references and read-only value types. However, during the
constructor of the class, the fields can still be mutated to allow for initialization.

### Read-Only Classes

A read only class can contain only read-only and constant references and read-only value types.
However, during the constructor of the class, the fields can still be mutated to allow for
initialization.

**TODO:** consider requiring a `readonly` keyword for read-only classes to avoid them being the
default. Generally, classes should be `const` if possible.

### Mutable Classes

A mutable class may contain references with any capability and mutable value types. This includes
even isolated references. For users of the class, a hidden isolated reference is indistinguishable
from a `mut` reference.

## Move Classes

It might be thought that `iso` should be allowed as a class capability, but it is not. As described
in the section about mutable classes, they are permitted to contain `iso` references. Instead,
classes can be declared a movable classes. A movable class is allowed to contain fields whose types
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
