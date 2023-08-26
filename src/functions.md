# Functions

Every Azoth program has at least one function, the main function:

```azoth
public fn main() -> void
{
}
```

This is the simplest possible function. It is public, meaning it is visible everywhere including outside of this package (see [Access Modifiers](access-modifiers.md). That is followed by the name of the function. The empty parentheses indicate the function takes no arguments. The `->` indicates this is a function declaration and is followed by the return type. This function doesn't return a value which is indicated with the special type `void`.

So how to take arguments? Here is a main function that takes the console as an argument, so it can print hello world.

```azoth
public fn main(console: mut Console) -> void
{
    console.write_line("Hello World");
}
```

Function parameters are declared like `let` bindings. In fact, they are completely equivalent to them except the type must always be fully annotated. No type inference is done on function parameters. Multiple parameters are separated by commas. In the rare circumstance where a parameter needs to be a mutable binding, preceded it with the var keyword.

```azoth
public fn write_it(console: mut Console, var x: int) -> void
{
    console.write_line("x = {x}");
    x -= 1; // we can assign a new value to `x` because we declared it with var
    console.write_line("x - 1 = {x}");
}
```

## Returning from a Function

To return a value simply declare the correct return type and use a return statement.

```azoth
public fn add_two(x: int) -> int
{
    return x + 2;
}
```

To return from a function whose return type is `void`, use a return statement without a value, as `return;`.

## Ignoring Function Return Values

By default values returned from functions cannot be ignored. They must be explicitly discarded with
`_ = ...`. However, functions can be marked with an attribute that indicates the return value can be
ignored.

## Diverging Functions

Azoth has special syntax for [diverging functions](https://en.wikipedia.org/wiki/Divergence_(computer_science)). Those are functions that never return by normal means. That could be because they always cause an abort (terminate the program), throw an exception, or loop forever.

```azoth
public fn diverges() -> never
{
    abort("Stop the program now");
}
```

The function `abort` is a special function that crashes the program immediately, outputting the message passed to it. The `diverges` function never returns, because it crashes the program. This is indicated with the special return type `never`.

A diverging function can be used where an expression of any type is expected. Thus the type `never` is a subtype of all other types. In formal type theory, it is the [bottom type](https://en.wikipedia.org/wiki/Bottom_type). This can be useful in situations where two expressions are expected to have the same type, but you want one to be an error. For example, using an [`if` expression](choice-expressions.md#if-expression), we might write:

```azoth
let y: string = if condition => "We're good" else => abort();
```

The `never` type is a first class type and can be used anywhere a type can be. However, there can never be a value of that type.

## Function References

We can also create variable bindings that reference a function.

```azoth
let f: (int) -> int = ...;
```

`f` is a variable that references a function taking an `int` and returning an `int`. It can be assigned a function and called as a function. For example:

```azoth
// Without type inference
let f: (int) -> int = add_two;

// With type inference
let f = add_two;  // f: (int) -> int

// Calling f
let five = f(2);
```

## Params List

The `params` keyword allows for the creation of variadic functions. That is function that can take a variable number of arguments. It also allows for the invocation of variadic functions using a list of arguments. Additionally, it can operate with tuples, not just lists.

### Declaring a Variadic Function

A variadic function is declared by prefixing a parameter with the `params` keyword. That parameter should have a type that can be constructed with a list of values (see the next section). That is, the type of this argument is not limited to `Array[T]`. It may have any reference capability. If the `params` parameter is the last parameter, this is always valid. If the parameter is not the last parameter, none of the following parameters can have default values.

```azoth
public fn example(params items: List[int]) -> void
{
    // ...
}
```

**TODO:** Syntax for `params` requiring at least one parameter?

### Calling Variadic Functions

A variadic function can be called as if it were a normal function except that a variable number of arguments can be passed in place of the `params` parameter. This could be zero arguments.

```azoth
example(1, 2, 3, 4, 5);
```

### Construction of the List

When gathering the parameters into the collection type of the parameter, the compiler will generate code based on the available constructors and methods of the type. It will first look for a constructor taking `size, Raw_Array[T]` with any reference capability for the raw array. If one of those exists, it will construct a raw array of the values and pass the number of values and the raw array to that constructor. Even if that constructor is `unsafe` (which it should be), no safety error will be generated if it is outside an unsafe context. If no such constructor is present, it will look for a constructor named `capacity` taking a single argument of `size`. If it exists, it will construct the collection using that constructor passing the number of values, then call `add()` repeatedly, passing each value. If no such constructor is present, it will look for a default constructor, use it and add the values. If none of these exists, an error will be generated.

### Calling Variadic Functions with a List

Sometimes you wish to pass a pre-existing list of values in place of the variable number of arguments. This can be done by prefixing the argument in place of the variadic arguments with the params keyword. Normal type compatibility and conversion rules apply between the passed collection argument and the expected collection type.

```azoth
let numbers = #[1, 2, 3, 4];
example(params numbers)
```

## Params Tuple

The `params` keyword can also be used with the tuple type. While this typically leads to functions with a fixed arity, variadic generics can be used to create variadic functions accepting a variable number of parameters of varying type. The `params` keyword works with the tuple type analogously to how it works with list. It can be used both for declaring a function and for invoking that function with an existing tuple. Additionally, it can be used with any function to take a tuple and expand it out so each element is passed as an argument. Thus the place where the params argument is passed is effectively replaced by an access to each element of the tuple for a number of arguments equal to the length of the tuple.
