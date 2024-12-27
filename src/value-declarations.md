# Value Declarations

Value declarations are the equivalent of object declarations for a value type (i.e. a struct).
However, it is only possible to inherit from a closed struct. So either a value declaration cannot
inherit from anything or it must inherit from a closed struct.

An object declaration is the same as a struct declaration except:

* The `value` keyword replaces the `struct` keyword
* They cannot be declared `move`, `mut`, `ref`, or `closed`
* A single no-arg initializer is allowed and no named initializers are allowed. (This means that a
  record value must have no parameters.)
