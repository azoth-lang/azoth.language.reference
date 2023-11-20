# The Azoth Programming Language Reference

Note: This is the new version of the reference. Sections are being moved from the old version to the new version. If something is missing, it may be documented in the [old version](../old/book.md).

1. [Introduction](introduction.md)
   * [Simplifying Decisions](simplifying-decisions.md)
2. Lexical Structure
   * [Packages](packages.md)
   * [Grammars](grammars.md)
   * [Lexical Analysis](lexical-analysis.md)
   * [Newlines](newlines.md)
   * [Whitespace](whitespace.md)
   * [Comments](comments.md)
   * [Tokens](tokens.md)
   * [Identifiers](identifiers.md)
   * [Keywords](keywords.md)
   * [Reserved Words](reserved-words.md)
   * [Literals](literals.md)
   * [Operators and Punctuators](operators-and-punctuators.md)
3. Basic Concepts
   * [Syntactic Analysis](syntactic-analysis.md)
   * Declarations
   * [Member Access](member-access.md)
   * Signatures and Overloading
   * Scopes
   * [Execution Order](execution-order.md)
   * [Compiler Diagnostics](compiler-diagnostics.md)
4. [Types](types.md)
   * [Empty Types](empty-types.md)
   * [Simple Types](simple-types.md)
   * [Struct Types](struct-types.md)
   * [Tuple Types](tuple-types.md)
   * [Pointer Types](pointer-types.md)
   * [Reference Types](reference-types.md)
   * [Reference Capabilities](reference-capabailities.md)
   * [Stack References](stack-references.md)
   * [Internal References](interal-references.md)
   * Type Parameters
   * [Optional Types](optional-types.md)
   * [Type Expressions](type-expressions.md)
   * Generic Types
5. [Conversions](conversions.md)
6. [Expressions](expressions.md)
   * [Expression Blocks](expression-blocks.md)
   * [Choice Expressions](choice-expressions.md)
   * [Loop Expressions](loop-expressions.md)
   * [With Expression](with-expression.md)
   * [Capability Expressions](capability-expressions.md)
   * Operators
     * Logical Operators
   * [Bitwise Operations](bitwise-operations.md)
   * [Boolean Expression](boolean-expression.md)
   * [Interpolated Strings](interpolated-strings.md)
   * [Pattern Match Expressions](pattern-match-expressions.md)
   * [Initializers](initializers.md)
   * [Generators](generators.md)
7. [Statements](statements.md)
   * [Variable Declarations](variable-declarations.md)
   * Local Constant Declarations
8. [Namespaces](namespaces.md)
9. Declarations
   * Constant Declarations
   * Type Aliases
10. [Functions](functions.md)
    * [Anonymous Functions](anonymous-functions.md)
    * [Optional Arguments](optional-arguments.md)
11. Classes
    * [Class Declarations](class-declarations.md)
    * Members
    * [Fields](fields.md)
    * [Constructors](class-constructors.md)
    * [Destructors](destructors.md)
    * [Methods](methods.md)
    * [Associated Functions](associated-functions.md)
    * Properties
    * [Operator Overloading](operator-overloading.md)
    * [Partial Classes](partial-classes.md)
    * [Object Literals](object-literals.md)
12. [Structs](structs.md)
    * [Struct Initializers](struct-initializers.md)
    * [Struct Constructors](struct-constructors.md)
    * [Ref Structs](ref-structs.md)
13. [Traits](traits.md)
    * [Move Traits](move-traits.md)
14. [Enumerations](enumerations.md)
    * Enumeration Structs
    * Enumeration Classes
15. [Extensions](extensions.md)
16. Generics
17. [Structured Concurrency](structured-concurrency.md)
18. [Design by Contract](contracts.md)
19. [Error Handling](error-handling.md)
    * [Aborting](aborting.md)
    * [Structured Error Handling](structured-errors.md)
    * [Out of Memory and Stack Overflow](memory-exhaustion.md)
    * [Assertions](assertions.md)
20. [Effects](effects.md)
21. Attributes
22. [Patterns](patterns.md)
23. [Unit Testing](unit-testing.md)
24. [Unsafe Code](unsafe.md)
    * [Pointers](pointers.md)
25. [External Declarations](external.md)
26. [Documentation Comments](documentation-comments.md)
27. Standard Library
    * [Localization](localization.md)
    * [Global Namespace](std-lib-global-namespace.md)
    * [`azoth` Namespace](azoth.md)
    * [`azoth.collections` Namespace](azoth.collections.md)
    * [`azoth.collections.specialized.raw` Namespace](azoth.collections.specialized.raw.md)
    * [`azoth.io` Namespace](azoth.io.md)
    * [`azoth.math` Namespace](azoth.math.md)
    * [`azoth.memory` Namespace](azoth.memory.md)
    * [`azoth.text` Namespace](azoth.text.md)
28. Reflection
29. [Conventions](conventions.md)
30. [Version Number Scheme](version-numbers.md)
31. [Planned Features](planned-features.md)
    * [Global and Package Qualifiers](planned-qualifier.md)
    * [Additional Types](planned-types.md)
    * [Aliases](planned-aliases.md)
    * [Additional Expressions](planned-expressions.md)
    * [Operator Features](planned-operators.md)
    * [Compile-time Function Execution](planned-ctfe.md)
    * [Language-Oriented Programming](planned-lop.md)

Appendices:

* [Ideas](ideas.md)
* [Glossary](glossary.md)
* [Implementer's Notes](implementers-notes.md)
* [Influences](influences.md)
* [Syntax Reference](syntax-reference.md)
