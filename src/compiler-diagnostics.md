# Compiler Diagnostics

The Azoth compiler emits diagnostics for warnings, and both fatal and non-fatal errors.

## Fatal Errors

Fatal compiler errors prevent the package from being compiled or debugged.

Errors should be categorized as fatal only when it is impossible for the compiler to generate
machine code. For example, an error that a semicolon is missing should be non-fatal because the
compiler can correctly compile the code by inserting the missing semicolon. However, an that a
return expression has the wrong type is fatal because the compiler cannot determine how it would
convert the given type to the return type.

## Non-fatal Errors

Non-fatal compiler errors still allow the package to be compiled for a debug build and debugged.
However, non-fatal compiler errors prevent compiling the package for release.

Errors should be categorized as non-fatal whenever possible. In addition, many diagnostics that
would be warnings for other compilers should be non-fatal errors. Anything that would be a warning
in another compiler that should not be ignored and are definitely incorrect should be non-fatal
errors. For example, diagnostics about violations of naming conventions should be non-fatal errors.

## Warnings

Warnings indicate code that is very likely to be incorrect in some way but aren't definitely
incorrect. However, it is very important that warnings should be generated only for things that are
likely to be incorrect so that developers don't feel the need to ignore warnings.
