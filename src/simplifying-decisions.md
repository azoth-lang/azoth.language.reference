# Simplifying Decisions

In designing Azoth there are many options and many possible advanced features. For the sake of getting
an initial version that can be used to experiment with and learn from, it is necessary to rule out
some of those options for now. This section documents those decisions. They may be changed in the future
if it becomes clear that the functionality offered would be valuable.

## No Default Capability Declarations

It might seem reasonable for classes to declare a default reference capability. That is similar to
the supported `const` classes and default reference capabilities are provided in Pony. However, that
would cause confusion since the default is `read`. It would also require the use of the `read`
keyword on some types to force the read-only capability when the default capability was different.
For simplicity, default capability declarations will not be supported.
