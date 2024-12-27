# The Azoth Programming Language Reference

Note: This is the new version of the reference. Sections are being moved from the old version to the
new version. If something is missing, it may be documented in the [old version](../old/book.md).

1. [Introduction](introduction.md)
   * [Simplifying Decisions](simplifying-decisions.md)
2. [Lexical Structure](lexical-structure.md)
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
   * [Extrema Types](extrema-types.md)
   * [Capability Types](capability-types.md)
   * [Value Types](value-types.md)
   * [Simple Types](simple-types.md)
   * [Struct Types](struct-types.md)
   * [Tuple Types](tuple-types.md)
   * [Reference Types](reference-types.md)
   * Type Parameters
   * [Optional Types](optional-types.md)
   * [Function Types](function-types.md)
   * [Type Expressions](type-expressions.md)
   * [Pointer Types](pointer-types.md)
   * Generic Types
   * [Stack References](stack-references.md)
   * [Internal References](internal-references.md)
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
   * [Initializer Expressions](initializer-expressions.md)
   * [Generators](generators.md)
   * [Lambda Functions](lambda-functions.md)
   * [Object Expressions](object-expressions.md)
7. [Statements](statements.md)
   * [Variable Declarations](variable-declarations.md)
   * Local Constant Declarations
8. [Namespaces and Import Directives](namespaces.md)
9. [Functions](functions.md)
    * [Optional Arguments](optional-arguments.md)
10. [Traits](traits.md)
11. Classes
    * [Class Declarations](class-declarations.md)
    * Members
    * [Fields](fields.md)
    * [Initializers](initializers.md) TODO: change to initializers
    * [Destructors](destructors.md)
    * [Methods](methods.md)
    * [Associated Functions](associated-functions.md)
    * Properties
    * [Operator Overloading](operator-overloading.md)
    * [Partial Classes](partial-classes.md)
    * [Object Declarations](object-declarations.md)
12. [Structs](structs.md)
    * [Struct Declarations](struct-declarations.md)
    * [Struct Initializers](struct-initializers.md)
    * [Ref Structs](ref-structs.md)
    * [Value Declarations](value-declarations.md)
13. [Closed Types](closed-types.md)
    * [Closed Traits and Classes](closed-traits-and-classes.md)
    * [Closed Structs](closed-structs.md)
    * [Cases](cases.md)
    * [Closed Type Idioms](closed-type-idioms.md)
14. [Extensions](extensions.md)
15. Other Declarations
    * Constant Declarations
    * [Aliases](aliases.md)
16. Type Variables
    * Generics
    * [Self Type](self-type.md)
    * [Associated Types](associated-types.md)
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
    * [`azoth.collections.raw` Namespace](azoth.collections.raw.md)
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
