# Out of Memory and Stack Overflow

Both out of memory errors and stack overflows are treated as unrecoverable errors and cause program
abort.

To avoid out of memory errors when allocating, you can use the `new?` construction to receive `none`
instead of triggering an out of memory error.

**TODO:** how to handle when an out of memory error occurs inside the constructor of the type being
constructed with `new?`. This should probably return `none`.
