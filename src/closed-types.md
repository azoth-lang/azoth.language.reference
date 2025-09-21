# Closed Types

Types can be marked `closed`. This means that the set of direct subtypes is closed and cannot be
extended in other packages. This allows the compiler to do exhaustiveness checking when matching on
these types. All sorts of types can be declared closed and this allows for many powerful variations.
The `closed` modifier is not inherited. That is, it is perfectly acceptable for a non-closed type to
subtype a closed type. However, it is often more useful to make entire hierarchies closed so that
exhaustive matching covers multiple layers of the hierarchy.
