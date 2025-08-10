# Values

A value declares a value type that is allocated inline (i.e. on the stack or directly in an object
field, etc.) and copied from place to place. Values still have capabilities because they can contain
references. To ensure proper behavior, value types cannot contain any `var` fields. This way it is
not possible to accidentally modify a copy of a value when intending to modify the original value.
