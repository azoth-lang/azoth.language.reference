# Type Declarations

Type declarations declare type constructors that can be used to create instances of types. There are
four kinds of type declaration: traits, classes, values, and structs. Traits declare types that
cannot be directly instantiated but instead can be implemented by other types. Classes create
reference types. Each class also creates a trait that is implicitly declared and can be implemented
by other types. Values declare value types that are copied by value. Structs declare hybrid types. A
hybrid type can be moved by value but that value can be referenced. Whether a hybrid type is a value
or reference is determined by the capability (see ...).

This section introduces certain aspects that are common to all or most type declarations.

## Const Types

Types declarations can be declared with the `const` modifier. This means that all instances of that
type must be `const`. This changes the default capability to `const` for the type. A `const` type
can only be subtyped by another `const` type. Constant types cannot have `var` fields and all fields
must themselves be `const`. Any non-const or `var` field wouldn't really be variable since all
instances are `const`.

**TODO:** should it be an error to use `const T` with a `const` type? That would avoid debates about
whether one should use `const`. However, it also means changing a type to `const` requires updating
any existing `const T` uses.

## Drop Types

Type declarations can be declared with the `drop` modifier. See [Drop Types](drop-types.md). This is
how Azoth manages non-memory resources. All subtypes of a `drop` type must be `drop` types. The
compiler ensures that for all droppable types, isolation is recovered before the last reference
leaves scope and that the drop method is called. Thus, while multiple references to a droppable
reference type may exist at one time, there will always be one stack frame that owns the instance.
To pass ownership one must `move` an `iso` or `own` reference to the instance to another method or
field.

Because `drop` type instances can never be `const`, `drop` type declarations cannot be declared
`const`.

## Closed Types

Types can be declared `closed`. This limits how they can be subtyped and allows values and structs
to be subtyped when they otherwise can't. Closed types enable exhaustive matching on type. They are
a complex enough topic that they are described in their own section, see [Closed
Types](closed-types.md).

## Empty Types

An empty type with no members can be abbreviated by replacing the body with a semicolon (e.g.
`public class Example;`). This is most useful with record types.
