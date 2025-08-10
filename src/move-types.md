# Move Types

Some types are `move` types. A move type imposes limitations but also enables powerful
functionality. Move types are how Azoth manages resources besides memory. They also provide
deterministic control of memory deallocation. A move type ensures that the compiler is able to
determine a point where every instance of the type goes out of scope. At that point the move types
destructor is called allowing for the release of non-memory resources and the deallocation of memory
allocated on the heap for the move type. Move types can be fields of other move types or be variable
and parameter types. A move type is not allowed to be a field of a non-move type.

**TODO:** explain moving and recovery of isolation.

If a class or trait is declared as a `move` type then all subclasses or implementing traits must
also be `move` types. All struct declarations are move types. Value types cannot be move types.
