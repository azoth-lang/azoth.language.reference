# Closed Values and Structs

Structs can be marked with the `closed` modifier. This transforms them into discriminated
unions and allows them to effectively be subtyped. The "subtypes" become members of the
discriminated union. Closed structs are abstract. One cannot create an instance of them
only of the subtypes.

A `closed` struct can only have a fixed set of subtype structs. Unlike values, closed structs do not
 allow object declarations to inherit from them. This is because structs cannot be declared `const`.
Thus only structs can inherit from them, but see the [Cases](cases.md) section for a shorthand
syntax. Typically, these are declared inside the closed struct, though technically they can also be
declared outside the struct. Private initializers can be used to prevent that if desired. The
individual subtype structs can override members of the struct and add fields. The standard
mechanisms of `sealed`, `overrides`, and `hides` can be used to control this process.

```azoth
public closed struct Game_Entity
{
    // Common fields and initializer

    public struct Player: Game_Entity { /* ... */ }
    public struct Enemy: Game_Entity { /* ... */ }
    public struct Obstacle: Game_Entity { /* ... */ }
    public struct Animal: Game_Entity { /* ... */ }
}

// Struct declared outside of the closed struct
public struct Wall: { /* ... */ }
```
