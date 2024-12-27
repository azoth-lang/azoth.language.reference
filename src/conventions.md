# Conventions

## Naming Conventions

The following naming conventions are enforced by compiler warnings.

| Item                | Convention                                         |
| ------------------- | -------------------------------------------------- |
| Packages            | `dotted.snake_case`                                |
| Namespaces          | plural `snake_case`                                |
| Const Copy Structs  | `snake_case`                                       |
| All Other Types     | `Pascal_Case_With_Underscores`                     |
| Type Parameters     | concise `Pascal_Case_With_Underscores`<sup>1</sup> |
| Functions           | `snake_case()`                                     |
| Divergent Functions | `ALL_CAPS()`                                       |
| Local variables     | `snake_case`                                       |
| Fields              | `snake_case`                                       |
| Constants           | `PascalCase`<sup>2</sup>                           |
| Global variables    | `ALL_CAPS`                                         |

<sup>1</sup> Type parameters are often single uppercase letters (i.e. "`T`").<br>
<sup>2</sup> Acronyms are capitalized and run together (i.e. "`ScreenDPIToPrint`").

## Curly Brace Placement

Curly braces should almost always be on their own line, never K&R style with the open brace on the
same line. However, for control flow statements with a single statement, the braces and statement
can appear on a single line indented. If the single statement is short it may appear on the same
line as the control flow. For `break`, `next` and `return` this should be a result expression.

```azoth
foreach x in 1..10
{
    if f(x)
    {
        // Multiple lines of work
    }
    if g(x)
        { doSomething(x); }
    if bar => break;
}
```

## Parameter placements

Function parameters should all be on one line or each on a separate line. If on separate lines, the
left parenthesis is on the line with the function and the first parameter goes on the line after.
The close parenthesis goes on the line with the last parameter. The exception is the self parameter
which stays on the line with the function unless it doesn't fit. When parameters are on separate
lines the return type is on its own line starting with `->`.

```azoth

public fn example(self,
    arg1: int,
    arg2: int,
    arg3: int,
    arg4: int)
    -> int
{
    // ...
}
```

## Code Width

Stay within 80 characters most of the time, never go beyond 100. This is not because of what fits on
screen, but rather, what humans can easily read and which parts of text humans focus on.

## Other

* Prefer named initializers to a method that makes an instance (for example making a child node,
  copy or converted value).
* Use implicit self whenever possible and explicit self only when necessary.
* Use spaces instead of tabs (need compiler warning for this).
* Use the preferred line ending of the platform you are developing on or `\n` (i.e. Unix) line
  endings if developing on multiple platforms
