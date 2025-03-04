# Initializer Expressions

Azoth supports four different forms of initializer expressions. Similar to user defined literals,
these initializer expressions do not determine the type being initialized. The type initialized is
inferred. These initializer expressions are named after the type they are generally used to
initialize, but they actually could be used to initialize any type. All of them are prefixed with a
pound sign that is followed by a comma separated list of values, the difference it simply what those
values are bracketed with. The four types of initializer expressions are:

| Initializer | Syntax              |
| ----------- | ------------------- |
| Tuple       | `#(x, y)`           |
| List        | `#[x, y]`           |
| Set         | `#{x, y}`           |
| Dictionary  | `#{x #> 1, y #> 2}` |

**TODO:** syntax for dictionary still not certain.

## Initialization

Initializer expressions invoke special initializers whose names mirror the syntax. Their arguments
can be one of a number of forms.

```azoth
public class Example
{
    // Tuple Initializer with Regular parameters
    public init #() (x: int, y: int)
    {
        // construct
    }

    // List Initializer with params initializer
    public init #[] (params values: Array[int])
    {
        // construct
    }

    // Set Initializer with params initializer
    public init #{} (params values: Array[int])

    // Dictionary Initializer with params initializer
    public init #{#>} (params values: key_value_pair[int, int])
}
```

**TODO:** should these be operators instead of initializers to avoid yet another special syntax?
That does conflict with the typed initialization syntax given below.

## Typed Initialization Syntax

If needed or desired for clarity, these initializers can be called as regular initializers (e.g.
`Example(x, y)`).
