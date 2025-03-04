# Appendix: Ideas

This appendix describes various ideas for features that could be added to the Azoth language in the
future. Sometimes, it clarifies why a particular feature is currently not in the language.

Sections:

* [Reference Capabilities](#reference-capabilities)
* [Reachability](#reachability)
* [Mutability and Move](#mutability-and-move)
* [Operators](#operators)
* [Preprocessor](#preprocessor)
* [Documentation Comments](#documentation-comments)
* [Declarations](#declarations)
* [Expressions](#expressions)
* [Types](#types)
* [Parameters](#parameters)
* [Generics](#generics)
* [Misc](#misc)

## Reference Capabilities

As part of the memory management system, there will be reference capabilities. However, it isn't
clear what the correct set of reference capabilities is. For now, implement extra ones and see which
ones work well. Then reduce and possibly rename them. In the table below, the `B` right stands for
"Bounded". It means there is a reference leaving the ownership boundary and so the lifetime of this
reference is bounded. Given how many of these there are, initially the `leaked` and `held iso`
capabilities won't be implemented because they are unlikely to be useful.

| Rights  | Name                         | Syntax                           |
| ------- | ---------------------------- | -------------------------------- |
| IRWI̅R̅W̅B | Owning Writable              | `owned mut T`                    |
| IRWI̅R̅W̅  | Isolated Writable            | `iso mut T`                      |
| IRWĨR̅W̅B | Potentially Owning Writable  | `held mut T`  (alt. `uni mut T`) |
| IRWĨR̅W̅  | Leaked or Isolated Writeable | `held iso mut T`                 |
| IRWR̅W̅B  | Borrowed                     | `mut T`                          |
| IRWR̅W̅   | Leaked                       | `leaked mut T`                   |
| IRI̅R̅W̅B  | Owning Read-only             | `owned T`                        |
| IRI̅R̅W̅   | Isolated Read-only           | `iso T`                          |
| IRI͠RW̅B  | Potentially Owning Read-only | `held T`     (alt. `uni T`)      |
| IRI͠RW̅   | Leaked or Isolated Read-only | `held iso T`                     |
| IRW̅B    | Shared                       | `T`                              |
| IRW̅     | Leaked Read-only (const)     | `leaked T`                       |
| IB      | Id                           | `id T`                           |
| I       | Id of Leaked                 | `leaked id T`                    |

These capabilities can be assigned/passed from one to another by a set of operations which influence
how the source type is treated while the new reference exists. The table below summarizes the kinds
of assignment.

| Action | Syntax   | Description                                                                                                  |
| ------ | -------- | ------------------------------------------------------------------------------------------------------------ |
| Move   | `move x` | Move the value out of the source variable. The source variable is no longer usable. Ownership is transferred |
| Borrow | `mut x`  | Borrow the value (writable). Source variable becomes `id T` as long as reference exists.                     |
| Share  | `x`      | Share the value (read-only). Source variable becomes shared `T` as long as reference exists.                 |
| Tag    | `tag x`  | Create an id for this object. Source variable unaffected except lifetime                                     |

If isolated is a needed but rare reference capability. It could be represented in a different way.
As an owned reference that explicitly doesn't have a bounds. The syntax could be `owned T ~> none`
and `owned mut T ~> none`. The same idea could be extended to leaked identity `id T ~> none`. This
seems a little strange, so alternatively, leaked could be `id T <~ forever`, `mut T <~ forever`, and
`T <~ forever`. The idea is that this indicates a reference can hold onto them for an unlimited
amount of time.

## Reachability

In addition to reference capabilities, the reachability system enforces memory safety. In Rust,
borrow checking is treated as part of type checking. I think it is less confusing to think of
reachability checking as a separate set of checks. Reachability ensures that nothing can be deleted
while it is referenced and that nothing can be mutated while there are other references to it.
Reachability is automatically inferred within the body of a function. At function boundaries,
reachability is handled by reachability annotations. These annotations come in two forms. First as
annotations on the return type and second as something like effect types.

Every object forms the root of an ownership tree formed by all the references with ownership. Every
object in this tree is said to be inside the object's ownership boundary. Every object not in the
tree is outside the ownership boundary. Any reference crossing created by a function that crosses
into or out of the ownership boundary of the parameters or return type must be annotated. Since they
cross the boundary, they are by definition non-owning. Thus these annotations represent something
more complex than mere reachability. If a method returns a reference to an object owned by self.
That object is reachable from self or from the reference, but the annotation must be `T ~> self`
because the borrowed reference goes into the ownership boundary of self.

Ownership annotations on return types come in two forms. First, `T <~ w, x` indicates that `w` and
`x` may reference object's inside the ownership boundary of the returned `T`. Second, `T ~> y, z`
indicates that the returned value may reference object's inside the ownership boundary of `y` and
`z`. When a type has both, either may be listed first `T <~ w, x ~> y, z`. The reason for this is
that when read left-to-right, the transitivity of reachability means that reachability does extend
between everything on the left and right of the operator. Ownership annotations may also occur
within generics (i.e. `List[T ~> x]`).

Ownership annotations not involving the return type are listed as effects of the function. For
example, a function taking two parameters `x` and `y` that creates a reference from `x` to `y` would
be annotated `may x ~> y`. They may be chained and should be read left to right. For example `may x
~> y ~> z <~ p ~> q ~> r` is equivalent to `may (((((x ~> y) ~> z) <~ p) ~> q) ~> r)` which is
equivalent to `may p ~> x ~> y ~> z ~> q ~> r`. Expressions where all the arrows go in the same
directions are preferred.

If a reference exists, but hasn't escaped the current function, then it does not yet fully restrict
the original reference. This supports what in Rust is called two-phase borrows. For example, in the
expression `list.at(list.count - 1) = x;`, the left expression is evaluated first and creates a
temporary borrow of list. Then `list.count` is evaluated. This needs to temporarily share `list`.
Without the escape rule, this would be illegal because list has already been borrowed. However, that
borrow has never escaped the current function yet. So it is safe to share the list as long as that
share ceases to exist before the borrow is used. Note that in Azoth all "field access" from outside
an object is treated as property access and counts as the reference escaping the function. This is
because fields can be overridden and may act like functions.

Would a bidirectional reachability operator make any sense `<~>`?

## Mutability and Move

Mutability expressions and move expressions have been an ongoing source of confusion as I think
about implementing the compiler. Specifically, the question is how to handle them around
initializers and functions returning some form of ownership. Should `f(g())` require a mutability
expression like `f(mut g())` if `f` is to mutate the value returned by `g`? Similarly, should
`f(move G())` require a move expression? The move seems unnecessary there. However, it could have
effects on lifetimes. For example, in `f(G(a))` will the reference to `a` be available after this
call or could it still be held because ownership was taken and put somewhere? Another example is
`f(g(mut a))`. Should it be possible for `f` to indirectly mutate `a` by taking mutability of the
return value from `g`? It seems sufficient that `a` could be mutated in this expression without
worrying too much exactly when. Also, requiring `mut` everywhere seems like it could get very old.
An expression like `f(mut g(mut h(mut j(mut a)), mut k(mut b)))` is difficult to read. Shouldn't
`f(g(h(j(mut a)), k(mut b)))` be sufficient? Another interesting example is `f(a.foo())` where `a`
could be mutated in `f` because `foo()` returns a mutable type. But there, the real issue is not
knowing `foo` is mutably borrowing it. It seems the only solution to that would be a separate access
operator which takes the target mutability. What comes to mind first is `f(a!foo())`, but that does
somewhat conflict with the `!` used for abort in `as!` and probably elsewhere. Other options
`a|foo()`, `a~foo()`.

One of the sources of confusion is the variable declaration shorthand which allows `let x = mut
g()`. Here the `mut` seems to be applied to the result of the function making it seem that could and
perhaps should be done elsewhere.

Given all this, for now, a simple scheme will be adopted. Mutability and move expressions can be
applied only to variable usages and field accesses. It remains an open question if that should
include property accesses since they are really function calls. However, they look like field
access. Method calls and initializer calls will have mutable and ownership types that do not require
`mut` or `move`. Variables with inferred types will continue to infer read-only even if assigned
from mutable expressions. To enable easy mutable declaration and also remove the confusion around
variable declaration, and simplify it, the special form `let x: mut = g()` will mean infer a mutable
type. Some thought has been given to dropping the colon or otherwise changing this. However, all the
other syntax for this are less clear and confusing in some regard.

## Operators

### Symmetric Operators

Allow binary operator overloads to be declared as "symmetric". This allows them to be called with
their arguments in either order. Thus when overloading an operator for two different types, one
doesn't need to overload it twice, once for each order of types, but can simply write one function
which will be used for both orders. This idea comes from JAI where not only operators, but any
function of two arguments can be declared symmetric. If the function is meant to represent a
mathematical operation, then that makes sense, but seems odd otherwise.

### Comparison Chaining

Make `a < b < c` legal. This would also allow `a < b > c` which seems confusing. Perhaps there
should be rules that the direction of comparisons can't change in a chain. So `a < b <= c` and `a >
b >= c` are legal but `a < b >= c` and `a > b <= c` aren't. An equal would be allowed in the middle:
`a < b == c < d`. What about not equal? That seems confusing. What does `a < b =/= c < d` mean?

### Advanced Operator Overloading

A number of additional options for operator overloading are possible. One could allow overloading of
additional symbols and sequences of symbols. Such mixfix operator overloading could build on the
underscore syntax already used for overloading unary operators and surrounding operators (i.e.
`_>>_<<_` would be a ternary operator). Such operators could have no precedence, or the programmer
could be allowed to specify their precedence.

Additionally, one could declare an operator to be commutative. This would be similar to the idea of
allowing symmetric operators. It would mean that overloads of the operator could have their
arguments passed in either order. Of course, with imprecision of number types, that could change
behavior. Perhaps it would also make sense to allow the programmer to specify the associativity of
new operators.

### Operator Partial Order

Instead of having a total order of precedence on the operators, have only a partial order. So if two
operators had no relative precedence, it would be an error to use them without disambiguating
parentheses. An example of where this could be helpful is the "`xor`" operator (see "`xor`" Operator
idea). Note that this may not be a true partial order. Rather precedence may be a DAG. For example,
if library "A" declares an operator to be higher precedence than equality and library "B" declares
one to be lower precedence than equality, that doesn't mean the two operators should have a relative
precedence.

### "`xor`" Operator

Use "`xor`" as the logical exclusive or operator. It could have no precedence relative to the
"`and`" and "`or`" operators. It is unlikely anyone would know the precedence of it. In fact, there
may be disagreement about the correct precedence. This has been omitted from the language for now to
avoid imposing a precedence relative to the "`and`" and "`or`" operators before operator partial
ordering is supported.

### "`implies`" Operator

Eiffel has one. Being an operator allows it to short-circuit.

### "`iff`" Operator

`(a iff b) == not (a xor b)` This operator would be distinct from `==` because `none == none` for
optional booleans but `none iff none` would be `none`.

### "`^`" Exponent Operator

When used as a binary operator "`^`" should be a right associative exponentiation operator. I had
thought this could be confusing with caret as the dereference operator. However, C makes use of `*`
as both the multiply and dereference operator. Exponents are much rarer than multiply, and pointers
in Azoth are much rarer than in C. So it shouldn't be an issue.

### Dot Product and Cross Product Operators

These seem like fairly standard useful operators. They may not be defined on the primitive types,
but they could probably exist and be given precedence that mathematicians would expect.

### Use `<<` and `>>` as Operators

These character sequences are now available in the language and seem like they are evocative of
operations like directing data etc. They could be useful.

### Checked Underflow

While overflow causes abandonment, underflow does not. That seems like it could be an issue in some
situations. Perhaps there should be a way to cause checked underflow.

### Checked Arithmetic Operators

Add `+?`, `-?`, `*?` and `/?` operators for checked arithmetic. If the operation overflowed, `none`
would be returned. This is equivalent to the `checked_`*x* set of functions available in Rust. Note
that since operators are lifted to optional types, chaining these operators would be fine i.e. `x +?
y +? z` would type check.

## Operator for Await

Given that async will be more pervasive in my language. Perhaps it makes sense to give `await` an
operator. One idea is to use `!`. It conveys the "do it" sense. Indeed, Haskell uses it as the force
evaluation operator. However, that conflicts with its use to mark things that could abort. Other
options include `>>` and `|>`. Those are reversible which might be useful. Both give the sense of
directing output or ordering. If the operator could be used postfix, that would also address any
concerns about `await` being a prefix which is then hard to operate on the result of.

## Use `|` as Remainder Operator

Now that it isn't taken up by "or", the pipe could be used as the remainder operator, fitting the
mathematical usage. However, that still seems a pretty rare operator. Perhaps it should be used for
something else more common. There may also be conflicts with `|` used for union types and "or"
patterns.

## Preprocessor

C# offers a preprocessor which doesn't suffer from the issues of the C/C++ preprocessor. A
preprocessor could be very useful in Azoth for conditional compilation of packages for different
target platforms and controlling compilation. However, Azoth packages are meant to be
cross-platform, and having different versions for different platforms could be bad. There is an idea
to support different platforms through native packages. That may obviate some of the need for a
preprocessor. Preprocessor directives would be introduced with "`##`" but otherwise function similar
to the C# preprocessor. While the list below includes begin and end region directives, it should be
carefully evaluated whether these should be added to Azoth.

* `##define`
* `##undefine`
* `##if`
* `##else`
* `##elseif`
* `##endif`
* `##error`
* `##warning`
* `##pragma`
* `##line`
* `##region`
* `##endregion`

The preprocessor may also be involved in language oriented programming. Originally, the thought was
that there would be a `##lang` directive that worked similar to Racket lang directives. That is, it
would cause the rest of the file to be parsed as that language. However, it is now thought that code
blocks and spans set in backticks like Markdown make more sense. The `##lang` or something similar
may instead be used to control the default language for fenced code blocks.

## Documentation Comments

### Extended Markdown

Currently, documentation only supports a subset of [CommonMark](https://commonmark.org/). Ideally,
it should support a much wider of range syntax, something like [Markua](http://markua.com/).

### Compile Code in Documentation Comments

Code in documentation comments should either be compiled, or have a way of causing it to be
compiled.

## Declarations

### `type` Declarations

Currently, there are type aliases (e.g. `alias type ...`) and there are associated types (e.g. `type
...` inside of a type declaration). Perhaps one should be allowed to declare types outside of a type
declaration. This would have the effect of introducing a new type that was distinct from the type it
was set to. There are many questions about how that should work though. The reason that aliases are
aliases is so that they don't cause issues where the new type doesn't interoperate with the types it
is constructed from.

Alternatively, type aliases could be declared without the `alias` keyword. Then within type
declarations a pure alias could be declared using something like `const type` or `sealed type`.

**TODO:** record classes and structs serve this purpose. Document in lang design and remove from
here.

## Expressions

### `else match` Expression

Allow a match to occur immediately after an else. Currently only if can occur there.

### `repeat {} while condition;` Loops

Rust doesn't have `do {} while condition;` loops. While they are rare, they do come up. Using `loop
{} while <exp>;` to avoid introducing a new keyword was considered. However, someone reading the
code wouldn't know to look for the while at the end or would have to check all loops to see if they
ended with a while. Instead, the Swift style syntax was chosen. This makes it clear from the first
keyword that this is a loop construct and not just some kind of action.

### Loop Else

Sometimes it is useful to execute some code if a loop is never run. This could be done with an else
clause of the while and for loop.

```azoth
while condition
{
    // do work
}
else
{
    // condition was false to start with
}

for let x in expression
{
    // do work
}
else
{
    // no items in collection
}
```

This can be useful for definite assignment. If the loop assigns a variable, it may be the case that
the loop never runs and the variable may be unassigned. However, you can assign the variable in the
else clause to a reasonable default so that the variable will definitely be assigned after the loop.

Note: this is different from the python style loop else construct which runs as long as the loop
completed successfully.

Note: Alternatively, a different keyword or group of keywords could be used for loop else. Options
include `otherwise`, `loop else`, `while else`, `for else`, or `if none`.

### Change Block Delimiters

The curly braces use up lines when declaring blocks. However, using only indention is problematic
and doesn't allow for good auto formatting. Consider alternate block delimiters. Possibly use "`--`"
to end a block. However, a block start is also needed to separate the condition of an if expression
from the block unless parens are going to be required around the condition again. There is also a
problem determining when a function signature ends and the body begins (consider requires clauses
etc).

```azoth
public fn function()
    if(condition)
        statement1;
        statement2;
    --
--
```

```azoth
public fn function()    {
    if condition    {
        statement1;
        statement2; }   }
```

### `set` Expressions/Statements

Instead of allowing assignment expressions anywhere. Use a set expression "`set x = 5`". This makes
a set as long as a redeclaration with "`let`". It allows the single equals sign "`=`" to be used as
both assignment and comparison operations. Finally, it prevents any ambiguity for destructuring
assignments "`set #(x, y) = function()`". If this was done, then `/=` could be used for the not
equal operator since it would be distinct from divide assign, though that could still be confusing
to users.

### Dictionary Initializers

There is already initializer syntax for lists "`#[...]`", tuples "`#[...]`", and sets "`#{...}`".
However, the syntax for dictionary initializers has not be finalized. Note that the named arguments
syntax has been defined and this is distinct from that syntax. The key difference is that a named
parameter assigns a value to a symbol whereas a dictionary associates a key and a value, both of
which are expressions. Because of that, the separator between key and value must be something clear
and unique that won't otherwise occur in an expression. Possible syntaxes with comments:

```azoth
// Key and Value Separator
#{"x"=:5, "y"=:6} // Looks like a type ascription with the value missing
#{"x":=5, "y":=6} // Looks like a declaration with the type omitted
#{"x"<-5, "y"<-6} // Direction feels wrong, a set maps from keys to values
#{"x"~5, "y"~6}
#{"x"~~5, "y"~~6}
#{"x"~>5, "y"~>6} // Gives another meaning to ~>
#{"x"=>5, "y"=>6} // Is this 100% consistent with the result syntax? Is it ambiguous? Too much like a function or pattern match
#{"x"\=5, "y"\=6}
#{"x"~=5, "y"~=6}
#{"x"#>5, "y"#>6}
#{"x"#=5, "y"#=6}
#{"x"+>5, "y"+>6}
#{"x"==>5, "y"==>6}
#{"x" -> 1, "y" -> 2} // Uses `->` for something other than return type

// Value and Key Separator (value before key)
#{5@"x", 6@"y"} // Conflicts with '@' for address of
#{5#="x", 6#="y"}

// Prefix to Key
#{#"x" 5, #"y" 6} // Confusing and # could accidentally combine with the kwy value
#{%"x" 5, %"y" 6}
#{&"x" 5, &"y" 6}
#{~"x" 5, ~"y" 6}
#{'"x" 5, '"y" 6} // Conflicts with user literals
#{''"x" 5, ''"y" 6} // Conflicts with user literals

// Other
#{:"x": 5, :"y": 6} // Uses colon for none type
#{%"x": 5, %"y": 6} // Uses colon for none type
#{%"x"=5, %"y"=6}
#{"x", 1; "y", 2 }
#{.at("x") = 1, .at("y") = 2 }
#{("x", 1), ("y", 2)} // While more verbose and mundane, it is pretty clear
// if the syntax of default methods is added so dictionary(key) works
#{("x") = 1, ("y") = 2 }
#{.("x") = 1, .("y") = 2 }
```

Idea: make `=>` a type constructor so that `T => S` is a key value pair of `T` and `S`. Then the
type matches the dictionary initializer. Or something similar with whatever is chosen.

## Types

### Logarithmic Numbers

Support in the standard library for numbers that are represented in the [Logarithmic number
system](https://en.wikipedia.org/wiki/Logarithmic_number_system).

### Relative Pointers

JAI will likely have relative pointers to pack pointers into smaller spaces that the 64-bit address
space requires. Instead a relative pointer is smaller, but is a pointer relative to its own
location. Thus to get the actual pointer, you must add the address of the relative pointer to the
value of the regular pointer. Relative pointers would require the ability to specify their size.
However, this requires that the programmer knows something about where things are allocated. For
example, that all the values will be allocated in a single block of memory. Additionally, relative
pointers introduce the possibility of overflow. Assigning a pointer into a relative pointer would
cause overflow if the address pointed to is outside of what the relative pointer can point to. Early
versions of JAI used "`*~s32 Node`" for a 32-bit relative pointer to a node. Notice a signed int is
used. It may be possible to implement relative pointers in the standard library using structs.

### Move and Copy Reference Types

Copy initializers can be defined for reference types. However, would it be useful to have reference
types that are implicit copy? Would it be useful to have reference types that are move by default.
It seems that this might be useful for implementing something like a type state. Consider a socket
object. This should be a reference type because it is large and may have multiple subtypes. However,
it could go through three states represented by types (i.e. `AvailableSocket`, `OpenSocket`,
`ClosedSocket`). There would be methods that took the one and returned the other. For example, the
`close()` method would consume the socket and then return a `ClosedSocket`. However, this may not
require move reference types. Indeed, one may not want move for most methods on socket. The close
method could be declared `fn close(iso self) -> iso ClosedSocket`. It would then take ownership of
self. That is a little strange because one can't use the `move` keyword with the self reference. The
dot operator would have to do this implicitly.

### Tuple Base Class

It probably makes sense to have all tuple types implement a common interface. C# has them implement
several interfaces about structural equality.

### Product Types for Tuples

Azoth has sum types (`|`) and intersection types (`&`). For consistency with those, it may make
sense to make tuple types be product types. These could be constructed with the `*` operator. The
issue with that is how would tuples of one item and empty tuples work?

### Closed Type Options

Provide a syntax for explicitly listing out all the options for a closed type. This ensures that
more options aren't added in unexpected places in the codebase.

### Constant Types

**TODO:** This has been adopted, document it better

Expose types in the language for constants of known values. For example, `bool[true]` would be the
type of a boolean known to be true. Likewise, `int[0]` the type of an int know to be zero at compile
time. One possible use for this is in a units of measure library where `m^3` could be handled by
overloading the `^` operator on `int[1]`, `int[2]`, `int[3]` etc. Alternatively, the types could be
`const[true]`, `const[1]`, etc. or `Const[true]`, `Const[1]`, etc.

### Units of Measure

It would be really good to be able to have good units of measure either directly in the language or
as a really clean library. This might be a useful place for an effect that says all code uses units
of measure. Units of measure may call for a space/juxtaposition operator between the value and the
unit. There may need to be a lot of flexibility in how units of measure can be done. Some situations
call for types that hold a value and a unit. Other situations call for a type for the quantity but
the units are always converted to some standard unit. Finally, sometimes the C# style ability to
attach units to any numeric type will make the most sense. Note with the last one, the units aren't
just part of the expression, but are part of each type declaration.

### Use `_` or `*` for Wild Card Types

Java style wild card types could be done using underscore.  For example, `List[_]` would be a list
of anything. `List[_ <: Foo]` a list of things that inherit from `Foo`. Of course, then it isn't
clear how to get the opposite type relation. `List[_ :> Foo]` seems strange. `List[_/Foo]` as in the
wild card is above the `Foo`. Maybe the `in` and `out` keywords are the correct thing here. So
`List[out Foo]` and `List[in Foo]` works pretty well. It is just missing the sense of wild card.
That would be read as a list that I can take out `Foo`s from and a list that you can put `Foo`s in.
Adding the underscore back could be `List[Foo out _]` and `List[Foo in _]` (note this order so that
it is "get Foo out of _" and "put Foo in _" but that has reversed the sense).

### Change `?` from Suffix to Prefix

Some languages use the `?` as a prefix for optional types. While it looks a little strange, it
resolves all ambiguity with all the other type prefixes. Also, `?T` can be read as "optional T".

## Parameters

### "`out`" Parameters

The `ref` types function as reference parameters. Does it make sense to add an `out` keyword like
C#? Or does the ability to return tuples make `out` not useful enough? They might be really useful
with external functions.

## Generics

### Default To Inferred

Generic parameters can be given a default value. However, there are cases where one expects that
users of a class will often want to pass one argument but have later arguments inferred. As an
example, fixed size arrays `Unsafe_Array[n: size, T]` will need the size specified, but will often
be able to infer the type parameter. Of course, this can already be fairly easily done with
`Unsafe_Array[5, _]` at the use site. However, it could be useful to allow the class to declare that
a parameter can be inferred if it is not used. One possible syntax for this would be
`Unsafe_Array[n: size, T = _]`. Another possible syntax would be to allow curried or nested
generics: `Unsafe_Array[n: size][T]`. Thus the first argument would be specified, but the second
inferred. That syntax might imply that using the type would require double brackets as
`Unsafe_Array[5][int]` which is odd.

### Reified Generics

Ability to explicitly create functions with specific types filled in. This would be like "bake" in
JAI to some extent. As an example of how this could be used, given a function that uses reflection
to serialize a type to JSON, one could reify it to get a JSON serializer that was optimized for the
given type because the compiler knew exactly all the steps/operations needed.

### Inline Generic Constraints

One could use the subtype operator to allow inline generic constraints. So instead of `class Foo[T]
where T <: Bar` one could write `class F[T <: Bar]`. The drawback of this is what Rust noticed that
generic parameters can get really cluttered and confusing.

### Existential Types

Existential types should be cleaner than the `forSome` keyword used in Scala. Simple existential
types could be handled with something like Java wildcards using the `_`. However, in other places
that means infer this type. The two meanings need to be compatible. For more complicated situations,
named wildcards `_T` could allow for existential types that are further constrained. For example, a
list concatenation could be `fn concat(x: List[_T], y: List[_T]) -> List[_T]`. That expresses the
relationship between the types. However, a change of implementation might require that a variable of
that type be declared. As such, if there were a short simple way of declaring existential types it
would be better to consistently use it. If it weren't hard to type, "∃" could be used as `fn
concat(x: List[∃T], y: List[∃T]) -> List[∃T]`. However, it seems weird to repeat the there exists,
one expects something more like `∃T fn concat(x: List[T], y: List[T]) -> List[T]`. As a straw man
syntax, if tilde were used it would be `fn concat(x: List[~T], y: List[~T]) -> List[~T]`. A fully
anonymous existential parameter could then be `Array[~, T]`. Notice here that we have used it for a
parameter that is not even a type. A type alias for arrays could be `type alias Array[T] = Array[~,
T]`.

As an alternative syntax, existential types could be treated as existential or implicit parameters.
The concat example could be `fn concat[~T](x: List[T], y: List[T]) -> List[T]`.

Another important consideration is whether an existential type can be used for a field `var list:
List[∃T]` and it can actually be assigned lists of different types.

Another syntax would be to use `*` in place of `~`. That would not be ambiguous because `*` is not
otherwise used as a unary operator.

## Misc

### Package Wide Namespace Alias

Just like you can give a package an alias in the project file when referencing the package, allow
you to specify aliases for namespaces inside that package. This could avoid the need to alias the
package because you could move all of the declarations to a namespace that didn't conflict. In fact,
this could almost replace the package alias ability.

### Allow `!` at the End of Function Names

Scheme uses `!` at the end of functions to indicate they are mutating. That may not make sense for
Azoth where mutation is probably more common and less frowned on. Rust uses `!` at the end of names
to indicate macros. It is nice to have a clear distinction for macros, but the syntax doesn't seem
to fit with a macro. Since `!` is not used to mean "not", it could be allowed at the end of function
names and used to indicate divergent functions. This would make it clear that execution will
terminate there. However, divergent functions are likely rare and it may not be worth using up the
`!` character on their names.

### Indexing Tuples with `at[]()`

Alternatively, tuples can be accessed using the "`at[n:size]()`" method similar to how arrays and
lists are index. However, for tuples, the index must be known at compile time so the access can be
type checked. So the index must be passed as a generic argument.

```azoth
let t = #(1, 2, 3);

let x = t.at[0]();
let y = t.at[1]();
let z = t.at[2]();
```

Note that the "`at`" method can't be a meta-function because it must return a reference to a runtime
value.

**TODO:** I think this runs into the problems I thought of with having the return type be an
arbitrary function of the type arguments. (Example bad case?)

### Default Value Initializers

Fields whose type is optional can be implicitly initialized with the value `none`. Perhaps there
should be a special initializer that, if present, the compiler calls to implicitly initialize a
field. This would allow developers to create their own types like optional types which can be
implicitly initialized.

### Use `read` and `write` for Properties

Instead of using `get` and `set` for properties, use `read` and `write`. This corrects what Kevlin
Henny talks about that "get" is side-effecting in English and isn't the opposite of "set".
Alternatively, "assign" could be used. If assignment were turned into a set statement, then set
might make sense again. However, it could be confusing since `read` is also the name of a reference
capability in some situations.

### Copy with Change Syntax

If immutability is used with true object orientation, there will be many more instances where a copy
with only a few changes will be needed. There should be a short syntax for this. Similar to how Rust
has the syntax for taking all the other fields of a struct from an existing one.

### Interpolated String Localization

Interpolated strings don't fit well with localization. The language would ideally steer people into
the pit of success which would be an easy transition to localized strings. That would imply that
interpolated strings are only for programmer output and not user display. That could be done by
making interpolation always call the debug format. On the other hand almost all the programs I've
written haven't needed localization and interpolation is such a good feature that it would be bad to
not support it for user display strings.

### Interval Literals

Allows ranges or intervals to have literals that are math style interval syntax (e.g. `'[3..6)'` or
`'[3,6)'`). The issue with this is that it puts the constant values inside of the literal. Using
interpolation for that seems strange (e.g. `'[\(3)..\(6))'`).
