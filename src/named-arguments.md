# Named Arguments

Named arguments allow arguments to be named when calling a function to improve clarity. Some
languages allow any argument to be named. In such languages, the argument name is typically the
parameter name. However, this makes parameter names part of the published API and changing one a
breaking change. Also, sometimes the parameter name and the argument name ought to be different. The
Swift language addresses this by having "argument labels" and "parameter names". However, all
parameters default to having an argument name. Indeed, by default it is whatever the parameter name
is. To avoid this, one must explicitly specify "`_`" as the argument label to indicate that there is
no argument label. Developers will naturally ignore or overlook this most of the time and easily
forget that changing the parameter name changes the argument label. To avoid these issues in Azoth,
parameters must be explicitly given argument names for them to be called with a named argument.

There are two ways to provide an argument name for a parameter. The name may precede the parameter
name and is distinguished with a '`.`' prefix. As a shorthand, the parameter name may be prefixed
with '`.`' to indicate that the parameter name should be used as the argument name. This shorthand
cannot be used when the parameter is being pattern matched (e.g. it is discarded with `_` or
destructed into multiple values).

```azoth
public fn example(.name1 n1: bool, .name2: bool) { ... }
```

**TODO:** maybe unpublished APIs should work differently and the parameter name would be used as the
argument name. This is similar to how `throws` is inferred internally but must be explicit for
published APIs.

## Passing Arguments by Name

If a function declares argument names, those arguments can be passed by name. This is done by
assigning the dot-prefixed name the argument value as one of the arguments.

```azoth
example(.name1 = 42, .name2 = true);
```

If a call matches a named argument, that takes precedence over an access to a member of `self`.

## Name Argument Order

Named arguments in a call can occur in any order and any position relative to the other unnamed
arguments.

## Named Arguments for Optional Parameters

Optional parameters are frequently given argument names because this allows callers to pass an
argument value for a single parameter that may not be the positionally first optional parameter.

## Initializer Parameters

In initializers, the shorthand for initializing fields looks similar to a named argument. It is
distinguished by the lack of `:` and a type. A field initializer parameter does not have an argument
name. To give it one, provide an argument name before the field name (e.g. `.arg_name .field`).

## When to use Named Arguments

It is recommended that named arguments be provided for:

* All optional parameters
* All `bool` and `bool?` parameters
* All parameters whose type has a literal when the meaning of the literal may not be obvious in the
  call context
* To allow a sort of named initialization syntax
* When it improves the readability of calling code
