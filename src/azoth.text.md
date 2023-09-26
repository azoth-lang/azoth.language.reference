# `azoth.text` Namespace

The text types are those types in addition to `String` which support text/string manipulation.

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

## `azoth.text.String`

The standard string type. This encodes strings in UTF-8 and supports string literal values. It is
designed to ensure that developers do not do things that do not work for unicode strings. As such,
it exposes iterators of grapheme clusters, `scalar_value` and `bytes`. It does not allow slicing by
index, but instead encourages properly slicing a string at textually appropriate positions.

To enable strings to represent both constant strings and mutable strings in a very efficient way,
the `String` type has a more complex implementation that usual. It is a psuedo reference struct that
wraps and enum struct with three cases. The first case is an empty string with no reference to data.
The second case is a non-zero length and a constant internal reference to the string data. This is
used for constant strings and slices. The third case is a mutable reference to the sealed private
`String_Builder` class. The compiler should be able to optimize this into a struct that only
consumes the space for a `size` and an `iref` byte (i.e. twice the native bit size). This is
possible because the non-zero constraint on the length means that a zero length can be used to
distinguish the constant string case from the others (per the kind of optimization that Rust is able
to perform). Then the non-zero constraint of the `String_Builder` reference can be used to
distinguish the mutable case from the zero case. Typically, the reference to `String_Builder` would
require both a vtable reference and data reference. But because it is a private sealed class, the
compiler should be able to optimize it to a data reference only. While getting the compiler to
optimize to that specifically will take some work and probably should have attributes to ensure the
compiler always does, it should be possible and provides an ideal experience for users while also
providing top tier performance.

### `azoth.text.String_Builder`

This is a private type used internally by `String` to provide the mutable string capability. It is
listed here only for clarity because the `String` documentation describes it. `String_Builder`
should be internally represented as either a growing `Raw_Bounded_List[byte]` or maybe as a
`Raw_Bounded_List[Raw_Bounded_List[byte]]` that stores multiple string chunks. C# uses a linked list
stored in reverse order. That is the end of the string is in the first node and as you traverse the
nodes you move toward the beginning. However, this does not provided constant time access to all the
data of the `String_Builder`.

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
