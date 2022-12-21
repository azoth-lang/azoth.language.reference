# Feature Ideas being Considered

## `var` is `mut` for Value Types

Currently, `var` and `mut` have distinct meaning for value types. It is assumed this will be needed to implement certain pseudo references. Also, on struct constructors it makes sense to have a `ref mut self` parameter. However, perhaps using `var` should always imply `mut` for mutable structs. Logically it seems if you can assign a new value you should be able to mutate the value.

## Returns That Can't be Ignored

Part of the Midori project's safety came from not being able to ignore the return value of a function except by explicit statement. This had the effect of ensuring that return codes and error results couldn't be ignored. If my error model includes `Result<void>` then it is possible people could ignore that. Perhaps there should be some special handling to say that certain types shouldn't be ignored when returned.

For example, there was a bug in the compiler where `ExpectToken(PublicKeyword);` should have been `children.Add(ExpectToken(PublicKeyword));`. If return values couldn't be ignored, this would have been a compiler error.

Any function that is a pure function should not have its return value ignored because you can't be calling it for side effects.

The syntax of `ignore foo()` is confusing because it seems you are ignoring the function call rather than the return value. Possible syntaxes include:

| Syntax             | Notes                                                                                   |
| ------------------ | --------------------------------------------------------------------------------------- |
| `_ = foo()`        | Consistent with the use of underscore as unused placeholder. Not obvious or searchable. |
| `__ = foo()`       | Double underscore should be unique and more noticeable.                                 |
| `_ignored = foo()` | Uses the underscore along the ideas of unused variables and is very clear.              |
| `ignored = foo()`  | Reads more like a proper keyword.                                                       |
| `discard foo()`    | Syntax used by Nim, again seems like you are discarding the call, not the value         |

## Exact Types

An exact type would be a type that references to a superclass but can't hold a subclass type. Since most types will have vtables, their pointers would be fat pointers. Exact would provide a way to ensure that reference and pointer types were thin pointers. This could be done with a special syntax like `!Foo` for an exact references and`@!Foo` for exact pointers. A user on Reddit said the P6 language allows constrained type declarations. This could provide another way of supporting exact types. So `type Exact_Foo = T where typeof(T)=Foo` and the `typeof` operator would be taken to mean the concrete type.

More recently, the syntax `^Foo` has been considered. The intent being that you "dereference" the type to get a non-reference type.

## Make good use of other symbols

I should seriously consider what other symbols are unused and how they could be used to good effect. Of course, it might be good to leave some symbols for future use, but it is so hard to know if those uses will make sense for the available symbols. Possible symbols are:

* `&`
* `|`
* `$`

And of course combinations of symbols and unary prefix and suffix versions of existing operators.

## Omit `-> void` for Procedures

Instead of requiring that procedures have `-> void` in their declaration, just leave it off. This is what Rust does. It also reduces noise and boilerplate. The one disadvantage is that `void` is still a valid type and it kind of obscures the role of the `void` type.

One way to do this would be to add a `function` or `func` keyword in front of function declarations similar to Rust's `fn` keyword. This would then make function declarations unambiguous even without the `-> void`. Note that operators and properties would not be preceded by `function` as they already have `operator`, `get`, or `set` in front of them. This would make all declarations consistent in that they would be a keyword followed by a name. Alternatively, the keyword could be `method`? But then you would still need `function` for non-methods.

Note that anonymous functions might indicate that `-> void` should be allowed. The expression `(x) -> returns_void()` seems reasonable. Yet, that means the arrow accepts a return type of void. This is consistent with essentially saying that a return statement with no value is `return void`.

## Operator for Await

Given that async will be more pervasive in my language. Perhaps it makes sense to give it an operator. One idea is to use `!`. It conveys the "do it" sense. Indeed, Haskell uses it as the force evaluation operator. Other options include `>>` and `|>`. Those are reversible which might be useful. Both give the sense of directing output or ordering. If await has an operator, should `async` have one too?

## Use `|` as Remainder Operator

Now that it isn't taken up by "or", the pipe could be used as the remainder operator, fitting the mathematical usage. However, that still seems a pretty rare operator. Perhaps it should be used for something else more common

## Use `_` or `*` for Wild Card Types

Java style wild card types could be done using underscore.  For example, `List[_]` would be a list of anything. `List[_ <: Foo]` a list of things that inherit from `Foo`. Of course, then it isn't clear how to get the opposite type relation. `List[_ :> Foo]` seems strange. `List[_/Foo]` as in the wild card is above the `Foo`. Maybe the `in` and `out` keywords are the correct thing here. So `List[out Foo]` and `List[in Foo]` works pretty well. It is just missing the sense of wild card. That would be read as a list that I can take out `Foo`s from and a list that you can put `Foo`s in. Adding the underscore back could be `List[Foo out _]` and `List[Foo in _]` (note this order so that it is "get Foo out of _" and "put Foo in _" but that has reversed the sense).

## Use `[ ]` for Compile Time Computing

Use square brackets instead of angle brackets for generics, but then allow other values etc. These are conceptually evaluated at compile like the same way as templates. Allow meta functions with only compile time arguments which are then guaranteed to be executed at compile time. Const expressions could be done as some form of this as `const[foo]`.

## All Statements are Expressions of Type `never`

The `never` type is a true, first class type. There are already `if`, `match` and loop expressions. This could be expanded to that everything that is a statement can be used as an expression returning some variant of `never`. Note that type `never?` signifies that the value will always be `none` since there are no values of type `never`

| Expression                                 | Type      |
| ------------------------------------------ | --------- |
| `loop` without break                       | `never`   |
| `while` loop without break                 | `never?`  |
| `for` loop without break                   | `never?`  |
| loops with break that doesn't have a value | `void?` ? |
| `let`                                      | ?         |
| `var`                                      | ?         |

## Closure Syntax

It isn't clear what a good closure syntax would be. One wants the type and the declaration to reflect the call syntax.  That would imply parens must be used around the closures arguments. However any syntax using parens will be confusing unless there is some prefix introducing the closure.  Yet, if there is a prefix there, then it would be very reasonable to expect the same prefix to be involved with function declarations.

One possible syntax is `\(x) -> x`. This has the advantage of evoking the similarity of lambda with the backslash. It also kind of fits with the idea that backslash means escape. In this case we are escaping the standard meaning of left paren. This would require the parens never be dropped which is annoying.  However it would mean the arrow could be used similar to Rust to give the return type. It would be possible, though confusing to allow `\x x` when `x` was not a keyword. This resurrects the idea of using the fat arrow.  If it allows an expression where a block is expected then `\x => x` would be a substitute for `\x { return x; }`. Then is `\x { => x; }` valid though? If that is valid, then why can't you use the same syntax with a function to return? Also, why would `\x { return x; }` return from the lambda rather than the outer function?

## Copy with Change Syntax

If immutability is used with true object orientation, there will be many more instances where a copy with only a few changes will be needed. There should be a short syntax for this. Similar to how Rust has the syntax for taking all the other fields of a struct from an existing one.

## Interpolated String Localization

Interpolated strings don't fit well with localization. The language would ideally steer people into the pit of success which would be an easy transition to localized strings. That would imply that interpolated strings are only for programmer output and not user display. That could be done by making interpolation always call the debug format. On the other hand almost all the programs I've written haven't needed localization and interpolation is such a good feature that it would be bad to not support it for user display strings.

## Simple Dependent Types

Full dependent types are complicated and confusing. But basic dependent types, say that an indexer object can only be used with the collection instance it indexes could be useful and easy. Something like this may already come along with lifetimes. On the other hand, this feature could make working with lifetimes easier.

## Nested functions

The equivalent of Rust's nested functions might be a constant whose value is a lambda expression.

## Change `?` from Suffix to Prefix

Some languages use the `?` as a prefix for optional types. While it looks a little strange, it resolves all ambiguity with all the other type prefixes. Also, `?T` can be read as "optional T".

## Sum Types

```azoth
type Foo = Bar | Baz;
```

Effectively making `Foo` a base class of `Bar` and `Baz` except they must be matched to split them.
