# Introduction

Azoth is a new language that is under development. It will be a general-purpose multi-paradigm
language with structured concurrency and thread-safety. Documentation and resources can be found at
[azoth-lang.org](http://azoth-lang.org). Azoth should be a compelling alternative to other high
level languages like C# and Java. It's focus on compile time safety aids in creating correct code.
Other features of interest include:

* write once, compile anywhere
* guaranteed optimizations
* asynchronous everywhere
* type inference
* generics with partial specialization
* class defined interfaces
* optional exception specifications
* minimal runtime

## Purpose

This book serves is an informal reference to the Azoth language. It tries to be comprehensive by
covering all features of the language. It can serve as an introduction to the language for
programmers who are already familiar with another language like C#, Java or Rust. Features unique to
Azoth are explained in more detail while features shared with other languages are only briefly
covered with a focus on stating how they work and any differences with other languages.

Those less familiar with programming would do better to read the [Introduction to Programming in
Azoth](https://github.com/azoth/azoth.language.introduction/blob/master/book.md) book. If you are
already very comfortable with computer programming and just want to see what sets Azoth apart from
other languages, continue on to the next section for a brief summary of some of the unique features
of Azoth.

## Unique Features

This section introduces a few of Azoth's more unique features and links out to the relevant section
on each feature.

### Reference Capabilities

**TODO:** describe reference capabilities

### Inferred Throws Clauses

Azoth has checked exceptions similar to Java or C++. However, unlike either of those languages, by
default the exceptions thrown by a function are automatically inferred. Only on externally exposed
APIs of packages are exception specifications required.

```azoth
// Throws clause is required because this functions is publicly exposed
published Function1() -> void
    throws exception
{
    Function2();
}

// Throws clause is inferred because it is omitted and this is an internal function
internal Function2() -> void
{
    throw new exception();
}
```

For more information see [Exceptions](exceptions.md).

### Interfaces of Classes

In many languages there is a distinction between an interface or trait that has no implementation
and a class which can have implementation. Azoth doesn't have separate interfaces. Instead, any
class can implement the implicitly defined interface of another class. In a class declaration, the
base class appears after the first colon and classes whose interface is being implemented appear
after the subtype operator `<:`.

```azoth
public class MyClass: BaseClass <: ClassAsInterface1, ClassAsInterface2
{
}
```

This simplifies the type hierarchy and eliminates the practice often needed in C# and Java of
defining a matching interface for most classes. The feature also necessitates several of Azoth's
other unique features. Members declared private are accessible only from the same instance, not by
other instances of the same class. Public fields can be overridden by properties in subclasses, so
there is no need to declare properties for every field. For more information see
[Traits](traits.md).

### Immutable by Default

All variable declarations and types are immutable by default. Mutability has to be opted into.

### Guaranteed Optimizations

Because of immutable by default and other features of the language, it becomes possible to safely
execute some code at compile time. For example, if you initialize a constant to the square root of
two, this computation will be performed during compilation and the constant will be directly
initialized with the resulting value.

Exactly how this feature will work is still being determined.

### Asynchronous Everywhere

Azoth will have [structured
concurrency](https://vorpus.org/blog/notes-on-structured-concurrency-or-go-statement-considered-harmful/)
something similar to async/await support. However, other languages async becomes contagious and must
be passed up the call chain throughout your code. Because of pervasive support for async execution,
that is not the case in Azoth. Instead, it is safe to await a task in synchronous code and it will
block the current thread of execution without holding a thread.

In C#, asynchronous code can easily lead to race conditions as two threads share data across an
asynchronous boundary. In Azoth, reference capabilities protect the developer from any such possible
race conditions so that asynchronous code is fully safe.

### Generics with Partial Specialization

Generics as supported by C# and Java are much easier to reason about than C++ style templates.
However, they can be more restrictive. The type system of Azoth allows for much greater flexibility
in generic types. For example, it is possible to use the type `void` as a type parameter to a
generic class or function. Additionally, it is possible to provide specialized implementations of a
generic class or function for specific types for better performance.

### Open Methods

The fragile base class problem occurs when a class overrides a base classes methods. This is avoided
by specifically marking methods that the base class will call subclass implementations with the
`open` keyword.

### Named Constructors

Constructors can be given names to indicate their purpose and meaning.

## Uncommon Features

This section lists features Azoth shares with other languages that are less common, but still
contribute to the distinctive flavor of the Azoth language.

* Type Inference on Local variable declarations
* Diverging Functions (return type `never`)
* A Specific Infinite Loop Keyword
* All for loops are iterator based
* Iterator performance often optimizes down to a C style for loop
* Operator Overloading
* Object Literals - allows creation of single instance of anonymous type
* Classes can be extended with additional methods in separate libraries
* Partial Classes - supports code generation
* Both reference and value types
* C# style generators with `yield` keyword
* `unsafe` code blocks and low level language features like raw pointers
* C interop
