# Fields and Properties

Fields and properties are intimately connected. In other languages, they are quite distinct, but in
Azoth, all public or protected field declarations create an implicit property declaration. However,
properties can also be created and overridden with getters and setters.

## Private Fields

Private fields of classes, values, and structs are declared like variable bindings using the `let`
or `var` keyword. Thought values only permit `let` fields, not `var` fields.

## Field Properties

Any field declaration that is more visible than private implicitly creates a private field and a
more visible property as well. In traits, no field is created. Only the property is created by the
field declaration. This means that a visible field declaration can be overridden by getters and
setters or fields in the subtypes. A `let` field declaration creates only a getter while a `var`
field declaration creates both a getter and setter.

A field property may wish to have a private settable field that only exposed a getter. This can be
done with the special field getter syntax. This syntax is an access modifier followed by `get` in
front of the `var` field declaration (e.g. `public get var x: int`).

## Field and Property Access

When accessing a field name from withing a type declaration, the field is always implicitly
accessed. When accessing from outside the declaration, only the property getter and setter can be
accessed. To explicitly access a getter or setter from inside a type declaration the syntax
`.get.property()` and `.set.property(value)` can be used. This also provides a means for getters and
setters to be used as method groups both within the type and outside the type (e.g. `v.get.property`
and `v.set.property`).

## Getters

Getters are declared like methods except with the `get` keyword in place of `fn`. A getter must have
no parameters other than `self` and a non-void return type. Getters are then accessed as properties
(e.g. `value.getter`).

## Setters

Setters are declared like methods except with the `set` keyword in place of `fn`. A setter has
exactly one parameter in addition to the `self` parameter. This parameter is conventionally named
`value` but doesn't have to be. The return type of the setter is `void`. The return value of an
assignment to a setter is automatically supplied by the language rather than by the setter.

## Associated Properties

There are no associated fields. Having one would violate thread safety and require initialization at
an indeterminate time. Many cases what is desired is actually an associated constant. However, one
can declare associated properties by declaring getters and setters without `self` parameters.

## Unique Names

Typically all type members must have a unique name. For fields, getters, and setters they can all
share the same name. That is, a field can have the same name as a getter and setter even if they are
declared separately. Like any other method, setters can be overloaded on the parameter type.
