# Classes

A class declares a reference type that is allocated on the heap. That is the objects lifetime is
unbounded and the memory for it is reclaimed by the garbage collector when the object is no longer
reachable.

**TODO:** should there be a way to declare "static" classes like in C# which can have no instances?
It was decided that using `object` declarations for that doesn't make sense since they are about
creating an object. It is odd that in Scala object declarations are for declaring singletons but
companion objects are more like static classes. In Azoth, static classes could have a use as
implementations of virtual associated members. Thus they could be a way to pass generic arguments
that can have associated members but no instances. (In C# static classes cannot be used as generic
arguments.) Two possible syntaxes are `module` and `type`. Though the latter would require that
associated types be declared as `type alias`. But perhaps a modifier to the class declaration would
be best. However, this may not be needed. In C# they exist primarily as a way to have free standing
functions. But in Azoth functions can exist outside of classes.
