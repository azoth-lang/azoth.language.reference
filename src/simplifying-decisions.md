# Simplifying Decisions

In designing Azoth there are many options and many possible advanced features. For the sake of getting
an initial version that can be used to experiment with and learn from, it is necessary to rule out
some of those options for now. This section documents those decisions. They may be changed in the future
if it becomes clear that the functionality offered would be valuable.

## No Borrowing

In addition to basic reference capabilities, one could imagine supporting something like borrowing
in rust. For example, imagine a method that needs `iso` access to a reference for thread safety
reasons, but guarantees that is won't hold onto that reference the function returns. This would
allow the caller to recover `iso` after calling the function even if it returned some other value
that in a non-borrowing function might have to be assumed to reference the original `iso` reference.

This could be very useful in some cases. However, it is much more complex than basic reference
capabilities and there are a number of different ways it could be added to the language. Questions
include:

* Does it apply to parameters only or also variables?
* Is it just a flag indicating borrowed or not? What about more complex relationships between
  parameters and returns?
* How does it combine with other reference capabilities? Can every capability be borrowed
* How does it combine with references to structs and variables? For example, isn't a reference to
  something on the stack just a borrowed reference?

## No Separate Capabilities on Structs

One could imagine that `var` only provides the ability to assign a new value to the variable and
that mutability is fully independent of that. So `var mut T` would be needed for a mutable struct.
This would also mean that `let T` and `let mut T` where `T` was a struct would allow for variables
whose assignment couldn't be changed but whose mutability could be chosen. For simplicity, this will
not be supported. `var T` will be mutable and `let T` will not. However, note that is T is
inherently not mutable then of course `var T` will only allow assignment not mutation.

## Heap References Can Always be Internal

Once could imagine that a reference to a struct or variable on the heap could be either a reference
to the inside of an object or to the beginning of the object. The difference matters for the garbage
collector. However, internal references are much more common and are also inclusive of both
possibilities. To keep the language simply, only internal references will be supported for now.
