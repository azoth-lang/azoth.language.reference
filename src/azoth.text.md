# `azoth.text` Namespace

The text types are those types in addition to `string` which support text/string manipulation.

## Encoding

Following the principle of [UTF-8 Everywhere](http://utf8everywhere.org/), strings are encoded in
UTF-8 by default in memory and all other places. The encodings classes must be used to read and
write in other encodings.

## Line Endings

To optimally support cross platform development, newlines are represented in code as simply `\n`.
Libraries should convert them when needed for output (for example to the console). Text reading
libraries should support the new lines types. The default is `Mixed` which can accept a mix of any
of the standard newline forms and converts them to `\n`. When writing text, libraries should support
the new line types. The default is line feed `\n`, but other types can be selected, including
native.

## `azoth.text.string`

The standard string type. This encodes strings in UTF-8 and supports string literal values. It is
designed to ensure that developers do not do things that do not work for unicode strings. As such,
it exposes iterators of grapheme clusters, `scalar_value`s and `byte`s. It does not allow slicing by
index, but instead encourages properly slicing a string at textually appropriate positions.

The `string` type is a constant copy struct. Internally, it contains a byte length and a reference
to the UTF8 bytes. This allows a string slice to reference a subsection of a larger string.

### `azoth.text.String_Builder`

The `String_Builder` type provides an efficient way to build up and work with mutable strings that
can then be converted to `string`. `String_Builder` should be internally represented as either a
growing `Raw_Bounded_List[byte]` or maybe as a `Raw_Bounded_List[Raw_Bounded_List[byte]]` that
stores multiple string chunks. C# uses a linked list stored in reverse order. That is the end of the
string is in the first node and as you traverse the nodes you move toward the beginning. However,
this does not provided constant time access to all the data of the `String_Builder`.

## `azoth.text.unicode.code_point`

A 32-bit type used for a [Unicode Code Point](https://unicode.org/glossary/#code_point). Code points
support user literals of a single character such as "`'c'`" or "`'♠'`". As defined in the Unicode
standard, a code point is any value in the Unicode codespace. That is, the range of integers from 0
to 0x10FFFF.

## `azoth.text.unicode.scalar_value`

A 32-bit type used for a [Unicode Scalar Value](https://unicode.org/glossary/#unicode_scalar_value).
Scalar values support user literals of a single character such as "`'c'`" or "`'♠'`". As defined in
the Unicode standard, a scalar value is any Unicode code point except high-surrogate and
low-surrogate code points. In other words, the ranges of integers 0 to 0xD7FF and 0xE000 to 0x10FFFF
inclusive.
