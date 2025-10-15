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

**TODO:** syntax for dictionary is still not certain.

## Initialization Operators

Initializer expressions invoke special initialization operators whose names mirror the syntax. Their
arguments can be one of a number of forms.

```azoth
public class Example
{
    // Tuple Initializer with Regular parameters
    public operator #(_) (x: int, y: int)
    {
        // construct
    }

    // Tuple Initializer with type list parameters
    public operator[T...] #(_) (values: T...)
    {
        // construct
    }

    // List Initializer with params initializer
    public operator #[_] (params values: Array[int])
    {
        // construct
    }

    // Set Initializer with params initializer
    public operator #{_} (params values: Array[int])

    // Dictionary Initializer with params initializer
    public operator #{_#>_} (params values: Iterator[key_value_pair[int, int]])
}
```

## Params Initializers

Types that provide initialization operators should typically provide `params` initializers as well
so that they can be explicitly initialized when type inference fails (e.g. `Example(x, y)`).
