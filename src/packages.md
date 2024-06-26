# Packages

The *package* is the fundamental unit of deployment in Azoth. A package may be an executable or
library. A package is compiled from one or more *source files* and a *package configuration*. Source
files are formally described as *compilation units*. A source file is a sequence of unicode scalar
values encoded in UTF-8. It is a non-fatal compilation error for a source file to begin with a
Unicode byte-order mark (BOM). Source files typically have a one-to-one correspondence with files in
a file system, but this is not required. The location of the source files relative to a root
directory for the package source files is the basis of namespaces within the package. The package
configuration specifies configuration values needed to properly compile the package such as the root
namespace, package name, referenced packages, and aliases for the referenced packages.

Roughly speaking, an Azoth package is compiled in the following steps:

1. Lexical analysis translates Unicode code points into a stream of tokens.
2. Syntactic analysis translates tokens into a structured representation of Azoth code.
3. Semantic analysis validates that structured representation for semantic rules such as type
   checking.
4. Intermediate language (IL) code is generated from the structured representation. This is the
   basis of package sharing.
5. Code generation translates the IL into executable code.
