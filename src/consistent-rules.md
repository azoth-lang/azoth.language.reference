# Consistent Rules

To make Azoth easier to understand and read, it is designed to follow consistent rules whenever
possible. Knowing these rules can help you quickly learn Azoth and read code using language features
and APIs you haven't learned yet. The rules are briefly stated in the list below. Some are discussed
in more detail in the following sections.

Rules:

* Generics are enclosed in square brackets (`[` and `]`)
* Parenthesis (`(` and `)`) are for grouping and invoking
* Curly braces (`{` and `}`) are for code and member blocks
* The pound sign ('`#`') indicates one of:
  * An attribute
  * An initializer (note that this use alters the meaning of `#[ ]`, `#( )`, and `#{ }`)
* The `=>` symbol is followed by an expression and indicates something evaluates to the result of
  the expression
* Punctuation using colon ('`:`') is about types (except for loop labels)
  * The colon ('`:`') indicates that something is of the type or kind to the right of the colon
  * The subtype operator (`<:`) indicates that one type is a subtype of another type
  * The one exception to this rule is that loop labels use colon `label: foreach ...`
* A prefixed-dot (e.g. `.x`) accesses a member on the entity that is determined by the context
* Backslash ('`\`') always escapes the thing it precedes
* A suffix exclamation point indicates something could cause an abort (e.g. `x as! int`)
* The `?` is used for optional types and operations on optional types
* The number of items in something is the `count` property
* Collection items are accessed with `.get(...)` and `.set(...)`
* The at sign ('`@`') stands for "address of" and is used for pointers

## Contextual Members

The prefixed-dot can access members on a number of different things where the context determines
what is being accessed. Typically this does not lead to ambiguity. However, if something is
ambiguous it will often be a compiler error and can be disambiguated by prefixing with the entity
the member is being accessed on. Depending on context it can mean:

* An argument name (e.g. `func(.arg = 42)`)
* A member of `self` (e.g. `.field = 42`)
* An object/value member of a closed type (e.g. `schedule_for(.Monday)`)

If a contextual member matches an argument name, that takes precedence. Otherwise, if multiple
meanings are possible, an error is reported. Accessing a member on `self` can be disambiguated using
`self` (e.g. `self.field`). Accessing a member of a closed type can be disambiguated by prefixing
with the type (e.g. `schedule_for(Day_Of_Week.Monday)`).
