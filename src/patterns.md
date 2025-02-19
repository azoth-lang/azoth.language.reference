# Patterns

Patterns are used to match values. They can also be used to destructure values into variables to
access specific portions of a value.

Patterns can bind names to values. These bindings may be mutable (i.e. `var`) or immutable (i.e.
`let`). As with variable declarations, `let` bindings are preferred. How this is done depends on
whether the pattern is in a binding context or not. The pattern grammar is therefor parameterized
on whether it is in a binding context or not.

```grammar
pattern<binding>
  : type_pattern
  | none_pattern
  | optional_pattern
  | not_pattern
  | union_pattern
  | intersection_pattern
  | wildcard_pattern
  ;
```

## Refutability

Patterns come in two forms: *refutable* and *irrefutable*. An irrefutable pattern never fails to
match. They are used to destructure data or as base cases. Refutable patterns may not match a given
value. Some kinds of patterns are inherently irrefutable, for example the simple binding pattern.
While other kinds of patterns are inherently refutable, for example the optional pattern. However,
many patterns could be either refutable or irrefutable depending on the subpatterns used with it.
For example, the refutability of tuple patterns depends on the patterns used for the individual
tuple elements. Whether a pattern is allowed to be refutable or not depends on the context it is
used in. Patterns in variable declaration statements myst be irrefutable. Patterns in other contexts
can be refutable.

## Uses of Patterns

Patterns are used in a variety of places in Azoth. Some require irrefutable patterns and each may or
may not establish a binding context. The list below summarizes where patterns may be used and what
kind of pattern occurs in each location.

* `match` expression arms: refutable patterns in a non-binding context
* `is` expressions: refutable patterns in a non-binding context
* `foreach` expressions: refutable patterns in a binding context
* `let` and `var` statements: irrefutable patterns in a binding context
* function parameters: irrefutable patterns in a binding context

## Binding Pattern

Binding patterns establish bindings to names. The form they take depends on whether the pattern
occurs in a binding context. Inside of a binding context, an identifier creates a binding of the
kind determined by the binding context. That is, whether the binding context was established by a
`let` or `var` determines the binding mutability. (Note that in non-binding contexts, an identifier
would be a type pattern).

```grammar
pattern<binding=true>
  : identifier
  ;
```

Outside of a binding context, pattern bindings are introduced with the `let` or `var` keyword. These
create a binding context for the patterns they contain. Optionally, a type may be specified in which
case the pattern is refutable and matches only if the type matches. Because specifying the type is
inherently refutable, they cannot be specified inside of variable declaration statements.

```grammar
pattern<binding=false>
  : "let" pattern<binding=true> (":" type)?
  | "var" pattern<binding=true> (":" type)?
  ;
```

## Literal Patterns

```azoth
let x = 1;

match x
{
    1 => console.write_line("one"),
    2 => console.write_line("two"),
    3 => console.write_line("three"),
    _ => console.write_line("anything"),
}
```

`string` and `code_point` literals also work.

## Type Patterns

```azoth
let x = Foo();

match x
{
    Foo => console.write_line("Foo"),
    _ => console.write_line("not Foo"),
}
```

Any class type can be matched. If the type is not visible outside the package, then the match can be
exhaustive because no subclasses can be added without rebuilding the package. This can also be
achieved by marking the class "closed?"

**TODO:** clarify how this works with capabilities. Should it be a bare type. But what about cases
like `self |> ref var Foo` where the capability modifies what type you might be checking for?

## None Pattern

```grammar
none_pattern
  : "none"
  ;
```

## Optional Pattern

A modifier to identifier binding patterns that matches only if the value is not `none`.

```grammar
pattern<binding=true>
  : identifier "?"
  ;
```

```azoth
if value is let x?
{
    // x is value
}
```

If the `?` operator can be overloaded, then it should be possible to overload this to match for
other types besides optional.

## Not Patterns

The not pattern can be used to match any value that does not match a given pattern. When bindings
are established, this flips the context under which the binding is available. For example, if a
binding is inside a not pattern in the condition of an if, then the bound name will be available in
the else clause or after the if statement when there is no else clause.

```grammar
not_pattern
  : "not" pattern
  ;
```

## Multiple Patterns

These can be used to combine multiple patterns. Note that they match the syntax for type
expressions. This also distinguishes them from logical operators so that an `and` or `or` indicates
the end of a pattern expression.

```grammar
union_pattern
    : pattern "|" pattern
    ;
```

```grammar
intersection_pattern
    : pattern "&" pattern
    ;
```

```azoth
let x = 1;

match x
{
    1 | 2 => console.write_line("one or two"),
    3 => console.write_line("three"),
    _ => console.write_line("anything"),
}
```

Note that alternatives can contain the same binding name as long as the type matches in each alternative.

## Range Patterns

Ranges work exactly as they do in other places

```azoth
let x = 5;

match x {
    1..5 => console.write_line("one through five"), // TODO but .. isn't inclusive?
    _ => console.write_line("something else"),
}
```

How can this be overloaded to match other types?

## Tuple Patterns

```azoth
let #(x, y) = #(1, 2)
```

**TODO:** This should probably just be a destructure pattern `let x, y = #(1, 2)` because it applies
to any type that can be destructured, not just tuples.

## Wildcard Pattern

```azoth
foo(console: Console, _: int, y: int) -> void
{
    console.write_line("This code only uses the y parameter: \(y)");
}
```

Can be used to ignore sub-parts.

## Unexpected Values

There should be a syntax similar to `_` that serves as a catch all for match expressions, but
generates a warning if the compiler detects something will match it. This would be useful for
libraries that contain open types where you must have a `_` but don't want to accidentally match
added types. But is this different than `_ => NOT_IMPLEMENTED!()"?

Should you really be required to have a default case for matches on open types? If the library is
changed, you should deal with it.

## Unused Variables

Use `_x` to avoid the warning that variable `x` is unused. As with Rust, this has the effect of
binding the value to the variable which has lifetime effects, whereas just `_` does not bind.

## Conditional Patterns

```azoth
let num: int? = 4;

match num {
    x? when x < 5 => console.write_line("less than five: \(x)"),
    x => console.write_line(x),
    none => void,
}
```

## Object Property Patterns

```azoth
class Point
{
    public let x: int;
    public let y: int;

    public init(.x, .y) { }
}

main() -> void
{
    let p = Point(0, 7);

    let Point { x, y } = p;
    assert(0 == x);
    assert(7 == y);

    // TODO is this the correct syntax?
    let Point { .x=a, .y=b } = p;
    assert(0 == a);
    assert(7 == b);
}
```

## Arrays/Lists Patterns?

```azoth
let [x, y, ...] = list;
```

How do you expand this to work on other types? A special kind of match method.

```azoth
public class Foo
{
    match []() // creates array match method?
    {
    }
}
```

## Match Methods

Classes can declare how they are matched with match methods. These are like Scala unapply functions.

```azoth
class Foo
{
    match(.x, .y) => true; // match the x and y properties
    match() => #(int, int)? // Form allowing more sophisticated logic
    {
        return #(x, y);
    }
    match named(.x, .y); // matches can be named
    init match (.x, .y) // declare both an initializer and a match method at once
    {
    }
}
```

These are then matched as

```azoth
let v = Foo();

match v
{
    T(x,y) => ...,
    T.name(x, y) => ...,
    .name(x, y) => ..., // type assumed to be that of `v`
}
```

Should classes that are tuple like be supported as a shortcut for matching?

```azoth
public class T(a: int, b: int) // declares fields `a` and `b`, initializer and match method all at once
{
}
```

## Regular Expressions

Is there a way to use a regular expression to match a string?
