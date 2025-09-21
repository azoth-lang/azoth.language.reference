# Type Members

Type declarations declare members that include methods,

## Instance vs Associated Members

Each type member is either an instance or associated member. Instances members declare aspects of
instances of that type (e.g. method, properties, fields, etc.). Associated members declare aspects
of the type itself. They are accessed from the type name and have a singular existence regardless of
how many instances of the type exist.

Instance Members:

* Methods
* Fields
* Properties
* Drop Methods
* Operator Methods

Associated Members:

* Initializers
* Associated Functions
* Associated Types
* Associated Constants
* Nested Types

## Abstract Members

Type members can be abstract meaning that they must be defined in a subtype. Any type with abstract
instance members must itself be abstract meaning that instances of it cannot be created. Traits are
inherently abstract. Classes can be declared abstract with the `abstract` modifier. Values and
structs are abstract only if they are closed.

Associated members can also be abstract. If a type has abstract associated members then this
restricts the use of that type. It cannot be used as a generic argument to a generic parameter in
any situation where it might be possible for the abstract member to be used. (Exact rules still to
be determined, see https://github.com/dotnet/csharplang/discussions/8922)

Abstract associated members must be marked with the `abstract` modifier.

Abstract instance members are commonly distinguished by the lack of a method body (e.g. `fn
example(self);`). The `abstract` modifier cannot be used with such members. The one exception to
instance members not using `abstract` is that in an abstract class, struct, or value, the `abstract`
keyword can be used to make a property declaration abstract (e.g.`public abstract var x: int`).
