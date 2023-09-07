# Ref Structs

Ref structs are declared by prefixing a standard struct with `ref`. They can be applied to either
`copy struct` or `move struct` types. These are very similar to C# `ref struct`. They allow stack
references to be used as fields within the struct. However, they are then limited to only being
allocated on the stack themselves. These types may not be used as the type of a field in an object.
