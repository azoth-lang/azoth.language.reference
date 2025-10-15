# Keyword Operator Overloading

In other languages certain types are treated specially by the compiler. These types are thus
"magic". In Azoth this is avoided by having all interaction with compiler behavior be controlled by
overloading keywords. These are like special operators that the compiler invokes to trigger the
desired special behavior. Thus any type can participate in the compiler special treatment by
declaring one of these operators. They are declared like operator overloads with keywords or the
special syntax they are activated by.

## Literal Operators

The literal operators `'_'` and `"_"` allow for the construction of values from user defined and
string literals respectively. As described in the literals section, the resulting type must be
`const`. This operator must be an implicit pure function. For simplicity, no spaces are allowed
around the underscore between the delimiters.

```azoth
public const value example
{
    public let value: string;

    public init(self, .value) { }

    public implicit operator '_'(value: string) -> const example
      => example(value);

    public implicit operator "_"(count: size, bytes: @byte) -> const example
      => example(count, bytes);
}
```

## Initialization Operators

i.e. `#(_)`, `#[_]`, `#{_}`, `#{_#>_}`

## Default Initializer

Overload operator `init` to declare a default initializer value that will be used to initialize
fields of the given type if they are not explicitly initialized. This should produce and "empty"
value. For example lists default initialize to the empty string and collections default initialize
to empty collections.

## User Defined Conversions

Are declared by overloading `as` and `as?`. Overloading `as` is used with non-failable conversions
and they mut be declared `implicit` or `explicit` to indicate whether the compiler can invoke them
implicitly. To declare failable conversions overload `as?`. When overloading `as?` the return type
must be optional and `none` is returned to indicate the conversion failed. All failable conversions
are explicit and so the `explicit` keyword is not used. The `as!` operator is derived from the `as?`
implementation.

## Foreach Operators

Overload `foreach` to indicate how to get an iterator and overload `next` to declare how elements
should be accessed.

## Truthiness

Overload `true` to indicate how `if` and `while` should treat a value. The overload must return
`bool` or `bool?`.

**TODO:** how does this interact with future plans to support overloading of short circuiting
  operators like `and` and `or`?

## Params Operator

The params operator is called to construct a `params` value from the individual parameters being
passed. Generally, these are chained so that the `params` operator takes some other collection or
iterator. That must be somehow grounded. One way to ground it is in a raw collection. But there
ought to be a way to ground it in an iterator.
