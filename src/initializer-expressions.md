# Initializer Expressions

Azoth supports four different forms of initializers. Similar to user defined literals, these
initializers do not determine the type being constructed. The type constructed is inferred. These
initializers are named after the type they are generally used to construct, but they actually could
be used to construct any type. All of them are prefixed with a pound sign that is followed by a
comma separated list of values, the difference it simply in what those values are bracketed with.
The four types of initializers are:

| Initializer | Syntax              |
| ----------- | ------------------- |
| Tuple       | `#(x, y)`           |
| List        | `#[x, y]`           |
| Set         | `#{x, y}`           |
| Dictionary  | `#{x #> 1, y #> 2}` |

**TODO:** syntax for dictionary still not certain.

## Construction

Initializers invoke special constructors whose names mirror the syntax. Their arguments can be one
of a number of forms.

```azoth
public class Example
{
    // Tuple Constructor with Regular parameters
    public implicit new #() (x: int, y: int)
    {
        // construct
    }

    // List Constructor with params constructor
    public implicit new #[] (params values: Array[int])
    {
        // construct
    }

    // Set Constructor with params constructor
    public implicit new #{} (params values: Array[int])

    // Dictionary Constructor with params constructor
    public implicit new #{#>} (params values: key_value_pair[int])
}
```

**TODO:** should these be operators instead of constructors to avoid yet another special syntax?
That does conflict with the type construction syntax given below. Also, what if you want a struct
from one of these? Do they work with init too?

## Typed Construction Syntax

If needed or desired for clarity, the type of an initializer can be specified as `new Example #{x,
y}` (or one of the other two initializer types). If the constructor is declared implicit, it can be
called without specifying the type. If it is not implicit, the type must be stated.
