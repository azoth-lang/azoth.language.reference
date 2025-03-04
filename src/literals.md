# Literals

Literals are representations of values in source code. A portion of literals are simply keywords.

```grammar
literal
    : boolean_literal
    | integer_literal
    | decimal_literal
    | user_literal
    | string_literal
    | none_literal
    ;
```

## Boolean Literals

There are two boolean literal values: "`true`" and "`false`". The type of a boolean literal is
"`bool`".

```grammar
boolean_literal
    : "true"
    | "false"
    ;
```

## Integer Literals

Integer literals represent integer values for types like `int` and `uint`. There are no negative
integer literals. Instead, negative values are represented by constant expressions using the
negation operator on an integer literal. Integer literals can be written in decimal, hexadecimal, or
binary.

Digits within an integer literal can be grouped using digits separators. This is similar to how
comma can be used to group digits into groups of three. In Azoth, the separator may be an underscore
or narrow non-breaking space. Integer literals allow the digit separators to be used to group digits
in any way the developer wants. However, an integer literal may not start or end with a digit
separator. Nor may they contain consecutive digit separators. For hexadecimal and binary integer
literals, a digit separator may separate the prefix from the value.

Decimal integer literals may not start with the digit zero except for the value "0".

```grammar
integer_literal
    : decimal_integer_literal
    | hexadecimal_integer_literal
    | binary_integer_literal
    ;

digit_separator
    : "_"
    | \u(202F) // Narrow No-break Space
    ;

decimal_integer_literal
    : decimal_digit_or_separator+
    ;

decimal_digit_or_separator
    : [0-9]
    | digit_separator
    ;

hexadecimal_integer_literal
    : "0x" hexadecimal_digit_or_separator+
    ;

hexadecimal_digit_or_separator
    : [0-9a-fA-F]
    | digit_separator
    ;

binary_integer_literal
    : "0b" binary_digit_or_separator+
    ;

binary_digit_or_separator
    : [0-1]
    | digit_separator
    ;
```

Integer literals do not in of themselves determine the type for the value. Instead, they represent
arbitrary precision integer constants which provide a value for integer types inferred by the
context.

## Decimal Literals

Decimal literals represent floating point values for types like `float`, `float32` and `decimal`.
Like integer literals, digit separators can be used in decimal literals to group digits. The integer
portion, decimal portion and exponent may not start or end with a digit separator. This implies a
separator cannot appear to either side of the decimal point, nor to either side of the letter "e"
which introduces the exponent. A decimal literal also may not contain consecutive digit separators.
The integer and exponent portions of a decimal literal may not begin with a zero digit except for
when that portion is the value "0".

```grammar
decimal_literal
    : decimal_digit_or_separator+ "." decimal_digit_or_separator+ exponent_part?
    | decimal_digit_or_separator+ exponent_part?
    ;

exponent_part
    : ("e"|"E") sign? decimal_digit_or_separator+
    ;

sign
    : "+"
    | "-"
    ;
```

Note: A decimal literal requires a digit before the decimal sign even if this is only the value zero.
Also, if present, a decimal point must be followed by at least one digit.

## Escape Sequences

Both user literals and string literals may contain escape sequences. This allows them to contain
characters that would otherwise be difficult to represent. Each escape sequence represents a single
Unicode code point. Simple escape sequences provide for some common characters one may wish to
escape. Escape sequences begin with a backslash which may be followed by a delimiter. The escaped
value is determined by the character(s) following the backslash or delimiter. See the relevant
literal section for a discussion of delimiters. Characters following a backslash or delimiter
besides those documented below or the left parenthesis are an error. Interpolated string segments
are another kind of escape sequence which are introduced by a left parenthesis following the
backslash or delimiter.

Unicode escape sequences allow the escaping of any Unicode code point. They do this by giving the
hexadecimal value of the code point as one to six hexadecimal digits. It is an error for a Unicode
escape sequence to refer to a surrogate pair or a code point outside of those allowed by Unicode.

```grammar
escape_sequence
    : simple_escape_sequence
    | unicode_escape_sequence
    ;

simple_escape_sequence
    : "\" delimiter ["]     // Double Quote U+0022
    | "\" delimiter "'"     // Single Quote U+0027
    | "\" delimiter "\"     // Backslash U+005C
    | "\" delimiter "n"     // Newline U+000A
    | "\" delimiter "r"     // Carriage Return U+000D
    | "\" delimiter "0"     // Null U+0000
    | "\" delimiter "t"     // Horizontal Tab U+0009
    ;

unicode_escape_sequence
    : "\" delimiter "u(" [0-9a-f-A-F]{1,6} ")"
    ;

delimiter
    : "#"*
    ;
```

## User Literals

User literals provide literal values for user defined types. They are enclosed in single quotes and
may contain escape sequences. They may not contain interpolated segments. Examples of user literals
include "`'c'`", "`'♠'`", "`'2018-09-28'`", and "`'c29a3471-ea8d-40e3-bb2b-ef563687f'`". The type of
a user literal is determined from context and content using type inference and pattern matching. The
user defined type is then constructed from the string value of the user defined literal.

To enable user literals containing backslash, single quote, or other characters requiring escape
sequences to be more easily written, they may be delimited. For example, this allows user literals
to be used for regular expressions which use backslash for escaping characters. A delimiter consists
of one or more pound signs immediately before and after the user literal. A delimited user literal
is terminated by a single quote followed by the same number of pound signs as it was prefixed by. A
single quote followed by fewer pound signs is taken to be part of the user literal value. A single
quote followed by more pound signs terminates the user literal and is an error. Within a delimited
user literal, escape sequences are only matched when they contain the same delimiter. That is when
the backslash is followed by the same number of pound signs. A backslash followed by fewer or more
pound signs is taken to be part of the user literal value.

```grammar
user_literal
    : delimiter "'" user_literal_character* "'" delimiter
    ;

user_literal_character
    : escape_sequence
      // any character except newline characters
    | [^] - newline
    ;
```

### User Literal Construction

User literal values are constructed by calling the user literal operator. The resulting object must
be `const`. This operator must be an implicit operator. See the section on this operator for more
information on what parameter types it can accept.

```azoth
public struct Example
{
    public let value: string;

    public init(.value) { }

    public implicit operator '_'(value: string) -> const Example
    {
        return Example(value);
    }
}
```

### Type Determination

Which type a user literal is for is determined by type inference. This means one can declare the
type of a variable to force what literal will be constructed. However, this process can be further
restricted by filtering.

### Restricting User Literal Values

The values a user literal can have may be restricted to match a pattern. Types whose pattern does
not match the string value of the user literal will not be considered during type inference. To
restrict values, add preconditions to the operator overload. It is recommended that these
preconditions not restrict the value to only the legal values. Instead, a more liberal pattern
should be used and an exception thrown for illegal values. This will allow type inference to select
the type a developer probably intended when writing an invalid value and correctly report an error.

```azoth
public copy struct Date
{
    public implicit operator '_'(value: string) -> Date
        requires value.matches(#'\d+-\d+-\d+'#)
    {
        // construct date
    }
}
```

## String Literals

String literals are Unicode strings encoded in UTF-8. Just like user literals, string literals
support both escape sequences and delimiters. Also like user literals, the data type of string
literals is inferred by context and restricted by pattern matching. However, typically the `string`
type is the only type that makes use of string literals.

String literals are enclosed in double quotes and may be delimited. To delimit a string literal, it
is prefixed and suffixed by one or more pound signs. A delimited string literal is terminated by a
double quote followed by a number of pound signs equal to the prefix delimiter. A double quote
followed by fewer pound signs is taken as part of the string value. A double quote followed by more
pound signs terminates the string, but is an error. In delimited strings, escape sequences are only
matched when they contain the same delimiter. An escape sequence with more or fewer pound signs is
interpreted as part of the value.

```grammar
string_literal
    : delimiter ["] string_character* ["] delimiter
    ;

string_character
    : escape_sequence
      // any character except newline characters
    | [^] - newline
    ;
```

### String Literal Construction

String literals are constructed with an operator overload. The resulting object must be `const`.
This operator must be an implicit pure function. As with user literals, the type of a string literal
is inferred by context and may be restricted by a match pattern. See the section on this operator
for more information on what parameter types it can accept.

```azoth
public struct Example
{
    public let bytes: const Raw_Bounded_List[byte];

    public init(.value, .bytes) { }

    public implicit operator "_"(bytes: const Raw_Bounded_List[byte]) -> const Example
    {
        return Example(count, bytes);
    }
}
```

## Interpolated Strings

In addition to the standard escape sequences, strings may contain escaped expressions. These
expressions will be evaluated at runtime, converted to strings, and have their string values
inserted into the string. String literals containing escaped expressions are called interpolated
strings. The full semantics of this is described in [Interpolated String
Expressions](interpolated-strings.md). The current section describes their lexical analysis.

An escaped expression is introduced by "`\(`" and terminated with "`)`". As with other escape
sequences, in a delimited string, the delimiter must be placed between the backslash and open
parenthesis.

An interpolated string is first analyzed as a single token by the rules described here. Before
syntactic analysis, it is broken down into tokens for the parts of the string between the
expressions and the expressions. Then the escaped expressions are lexically analyzed again into
streams of tokens that are inserted between tokens for the string segments. This analysis may
produce further interpolated strings which must be reanalyzed.

In order to lexically determine the end of the escape expression, the expression must contain
balanced bracketing characters.

```grammar
// A rule for string_character in addition to previous ones
string_character
    : escaped_expression
    ;

escaped_expression
    : "\" delimiter "(" balanced_text ")"
    ;

balanced_text
    : balanced_text_part+
    ;

balanced_text_part            // longest match rule applies
    : balanced_text_character
    | comment                 // ignore contents of comments
    | identifier_string       // ignore contents of identifier strings
    | user_literal            // ignore contents of user literals
    | string_literal          // nested string literals
    | "(" balanced_text ")"
    | "[" balanced_text "]"
    | "{" balanced_text "}"
    ;

balanced_text_character
    : [^'"(){}\[\]]
    ;
```

## Multiline String Literals

Multiline string literals start with three double quotes followed by only whitespace on the line.
They continue until a line that begins with whitespace followed by three double quotes. The lines
with the double quotes are not included. So the final line does not end with a line break. They can
be indented. The indent of the close quotes is ignored, indentation beyond that is included in the
string. Double quotes can be used in a multiline string. A line can be continued by ending it with a
backslash. *Regardless of the line endings of the source file, lines in the string are terminated
with '\n'.* Multiline string literals should also support string interpolation. (see [Swift
Multiline String
Literals](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/StringsAndCharacters.html)
for more information).

```azoth
let s = """
    Indent to the left of this is ignored
        This is indented one tab
    This line continues \
    onto the next line.
    """;
```

## None Literal

The none literal is the literal value for optional types representing the lack of a standard value.
The none literal has the type `never?`.

```grammar
none_literal
    : "none"
    ;
```
