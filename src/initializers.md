# Initializers

Azoth supports three different forms of initializers. Similar to user defined literals, these initializers do not determine the type being constructed. The type constructed is inferred. These initializers are named after the type they are generally used to construct, but they actually could be used to construct any type. All of them are prefixed with a pound sign that is followed by a comma separated list of values, the difference it simply in what those values are bracketed with. The three types of initializers are:

| Initializer | Syntax    |
| ----------- | --------- |
| Tuple       | `#(x, y)` |
| List        | `#[x, y]` |
| Set         | `#{x, y}` |

**TODO:** dictionary initializers are also needed but the syntax hasn't been determined

## Construction

Initializers invoke special constructors whose names mirror the syntax. Their arguments can be one of a number of forms.

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
}
```

## Typed Construction Syntax

If needed or desired for clarity, the type of an initializer can be specified as `new Example #{x, y}` (or one of the other two initializer types). If the constructor is declared implicit, it can be called without specifying the type. If it is not implicit, the type must be stated.
