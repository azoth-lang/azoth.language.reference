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

Types can be marked `closed`. This means that the set of direct subtypes is closed and cannot be
extended in other packages. This allows the compiler to do exhaustiveness checking when matching on
these types. All sorts of types can be declared closed and this allows for many powerful variations.
The `closed` modifier is not inherited. That is, it is perfectly acceptable for a non-closed type to
subtype a closed type. However, it is often more useful to make entire hierarchies closed so that
exhaustive matching covers multiple layers of the hierarchy. Since values and structs cannot
normally be subtyped, marking them closed allows them to be subtyped however the result isn't
totally independent types, but rather a discriminated union of types.

## Empty Types

An empty type with no members can be abbreviated by replacing the body with a semicolon (e.g.
`public class Example;`). This is most useful with record types.

## Nested Types

Types can be declared inside of other types. When this is done they are independent types that have
access to protected members and the outer types generic parameters. Nested types are associated
members.
