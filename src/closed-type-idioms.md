# Closed Type Idioms

Closed types are quite flexible and serve the functions of several other constructs in other
languages. As such, it is helpful to be aware of several idioms in Azoth for achieving that
functionality.

## Enums

Many languages have enumerated types that allow one of only a fixed set of values (e.g. C# & Java).
Though it should be noted some languages use the term enum for what is more properly termed a
discriminated union which will be covered in the next section (e.g. Rust & Swift).

The enum idiom provides a short way to declare enumerations in Azoth. Enumerations can be either
value types or reference types. While most enumerations are value types, it can be useful to have
reference type enumerations to subtype other reference types.

A value type enum is simply declared using a `const closed value` with `case`s. Such enums can have
initializers and methods. Due to the closed object value optimization, they can have any number of
fields without increasing the size of enum instances. Initializers can be private to further prevent
accidental additions.

```azoth
public const closed value day_of_week
{
    case Monday;
    case Tuesday;
    case Wednesday;
    case Thursday;
    case Friday;
    case Saturday: (false);
    case Sunday: (false);

    public let is_weekday: bool;
    init(.is_weekday = true) { }
}
```

A reference type enum is simply declared using a `closed const class` with `case`s. This allows them
to be part of a larger reference type hierarchy. Initializers can be private to further prevent
accidental additions. Consider this example adapted from one often used for Java. (Though this code
could be improved using units of measure.)

```azoth
/// universal gravitational constant (m^3/kg/s)
public const G: float64 = 6.67300E-11;

public closed const class Planet <: Celestial_Body
{
    case Mercury: (3.303e+23, 2.4397e6);
    case Venus:   (4.869e+24, 6.0518e6);
    case Earth:   (5.976e+24, 6.37814e6);
    case Mars:    (6.421e+23, 3.3972e6);
    case Jupiter: (1.9e+27,   7.1492e7);
    case Saturn:  (5.688e+26, 6.0268e7);
    case Uranus:  (8.686e+25, 2.5559e7);
    case Neptune: (1.024e+26, 2.4746e7);

    public let mass: float64;   // in kilograms
    public let radius: float64; // in meters

    init(.mass, .radius) { }

    public get surface_gravity() -> float64
        => G * .mass / (.radius * .radius);

    public fn surface_weight(other_mass: float64) -> float64
        => other_mass * .surface_gravity;
}

public fn main(args: const Array[string], console: mut Console) -> byte {
    if args.count =/= 1
    {
        console.error.write_line("Usage: Planet <earth_weight>");
        return -1;
    }
    let earth_weight = float64.parse(args.at(0));
    let mass = earth_weight/EARTH.surface_gravity;
    foreach p in Planet.values() // TODO how to get all values
    {
        // TODO how does the planet get a good to_display_string()?
        console.write_line("Your weight on \(p) is \(p.surfaceWeight(mass))");
    }
    return 0;
}
```

**TODO:** this example demonstrates the need for an easy way to get all values and convert to
string.

## Discriminated Unions of Values

While simple enums are very useful, discriminated unions which union together multiple value types
into a single type are even more powerful. This is why Rust and Swift allow their enums to be
discriminated unions.

In Azoth, an idiomatic discriminated union is simply declared with `closed value` and `case`s
taking record parameters.

```azoth
public closed value Barcode
{
    case Missing;
    case UPC(Start: byte, Left: uint16, Right: uint16, End: byte);
    case QR_Code(value: string);
}
```

The cases can be objects or full values. Cases do not have to use record syntax.
