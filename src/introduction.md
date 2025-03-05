# Introduction

Azoth is a new language that is under development. It will be a general-purpose multi-paradigm
language with structured concurrency and thread-safety. Documentation and resources can be found at
[azoth-lang.org](http://azoth-lang.org). Azoth should be a compelling alternative to other
high-level languages like C#, Scala, and Java. It's focus on compile time safety aids in creating
correct code. Other features of interest include:

* Write Once, Compile Anywhere
* Guaranteed Optimizations
* Asynchronous Everywhere
* Type Inference
* Generics with Partial Specialization
* Class Defined Interfaces
* Optional Exception Specifications
* Minimal Runtime

## Version

This reference currently documents what is intended to be v0.5 of Azoth. This should be a version
that is fully functional enough to validate the language design and write 95% of the standard
library.

Features planned for future versions can be found in the planned features sections.

## Purpose

This book serves is an informal reference to the Azoth language. It tries to be comprehensive by
covering all features of the language. It can serve as an introduction to the language for
programmers who are already familiar with another language like C#, Java, or Rust. Features unique
to Azoth are explained in more detail while features shared with other languages are only briefly
covered with a focus on stating how they work and any differences with other languages.

## Unique Features

This section introduces a few of Azoth's more unique features and links out to the relevant section
on each feature.

### Reference Capabilities

**TODO:** describe reference capabilities (See [Reference Capabilities](reference-capabailities.md))

### Inferred Throws Clauses

Azoth has checked exceptions similar to, but much better than, Java or C++. However, unlike either
of those languages, by default the exceptions thrown by a function are automatically inferred. Only
on externally exposed APIs of packages are exception specifications required. Additionally, Azoth's
powerful type system provides the tools needed to work with exception specifications so they don't
get in the way.

```azoth
// Throws clause is required because this functions is a published API
published fn function1()
    throws exception
{
    Function2();
}

// Throws clause is inferred because it is omitted and this is an internal function
internal fn function2()
{
    throw exception();
}
```

For more information see [Structured Error Handling](structured-errors.md).

### Implicit Traits

In many languages there is a distinction between an interface or trait that has no implementation
and a class which can have an implementation. While Azoth does have both classes and traits, classes
also define an implicit trait. Any class can implement the implicitly defined trait of another
class. In a class declaration, the base class appears after the first colon and classes whose trait
is being implemented appear after the subtype operator `<:`.

```azoth
public class My_Class: Base_Class <: Some_Class, Another_Class
{
}
```

This simplifies the type hierarchy and eliminates the practice often needed in C# and Java of
defining a matching interface for most classes. The feature also necessitates several of Azoth's
other unique features. Private members are accessible only from the same instance, not by other
instances of the same class. Public fields can be overridden by properties in subclasses, so there
is no need to declare properties for every field. For more information see [Traits](traits.md).

### Immutable by Default

All variable declarations and types are immutable by default. Mutability has to be opted into.

### Guaranteed Optimizations

Because of immutable by default and other features of the language, it becomes possible to safely
execute some code at compile time. For example, if you initialize a constant to the square root of
two, this computation will be performed during compilation and the constant will be directly
initialized with the resulting value.

Exactly how this feature will work is still being determined.

### Asynchronous Everywhere

Azoth enforces the [structured
concurrency](https://vorpus.org/blog/notes-on-structured-concurrency-or-go-statement-considered-harmful/)
programming paradigm and uses something similar to `async`/`await` support. However, it uses green
threads similar to the Go language. So, unlike other languages with `async`/`await` there is no
contagious spread of async functions (see [What Color is Your
Function?](https://journal.stuffwithstuff.com/2015/02/01/what-color-is-your-function/)). Instead, it
is safe to start and await an async operation in synchronous code and it will block the current
thread of execution without holding a thread.

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

The fragile base class problem occurs when a class overrides a base class's methods. This is avoided
by specifically marking methods that the base class will call subclass implementations with the
`open` keyword.

### Named Initializers

Initializers can be given names to indicate their purpose and meaning.

## Uncommon Features

This section lists features Azoth shares with other languages that are less common, but still
contribute to the distinctive flavor of the Azoth language.

* Type Inference on Local variable declarations
* Diverging Functions (return type `never`)
* A Specific Infinite Loop Keyword
* All `for` loops are iterator based
* Iterator performance often optimizes down to a C style for loop
* Operator Overloading
* Object Literals: allows creation of a single instance of an anonymous type
* Classes can be extended with additional methods in separate libraries
* Partial Classes: supports code generation
* Both reference and value types
* C# style generators with `yield` keyword
* `unsafe` code blocks and low-level language features like raw pointers
* C interop
