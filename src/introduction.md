# Introduction

Azoth is a new language that is under development. It will be a general-purpose multi-paradigm
language with structured concurrency, and capabilities. Documentation and resources can be found at
[azoth-lang.org](http://azoth-lang.org). Azoth should be a compelling alternative to other
high-level languages like C#, Scala, and Java. It's focus on compile time safety aids in creating
correct code. Other features of interest include:

* Write Once, Compile Anywhere
* Guaranteed Optimizations
* Asynchronous Everywhere
* Type Inference
* Generics with Partial Specialization
* Implicit Traits
* Inferred Exception Specifications
* Minimal Runtime

## Version

This reference currently documents what is intended to be v0.5 of Azoth. This should be a version
that is fully functional enough to validate the language design and write 95% of the standard
library.

Features planned for future versions can be found in the planned features sections.

## Purpose

This book serves as an informal reference to the Azoth language. It tries to be comprehensive by
covering all features of the language. It can serve as an introduction to the language for
programmers who are already familiar with another language like C#, Java, Scala, or Rust. Features
unique to Azoth are explained in more detail while features shared with other languages are only
briefly covered with a focus on stating how they work and any differences with other languages.

## Unique Features

This section introduces a few of Azoth's more unique features and links out to the relevant section
on each feature.

### Structured Concurrency

**TODO:** document what structured concurrency is and how it is supported.

### Capabilities

Capabilities are modifiers on types that express either a permission or constraint on the value. For
example, in the type `mut Example`, the capability is `mut` which is short for "mutable". This
capability indicates that this reference or value allows both reading and writing. In most
languages, all references are implicitly mutable. In Azoth, there are additional capabilities. For
example, the default capability is "read", written without a capability keyword (e.g. just
`Example`). This means that through this reference it is not possible to modify the value or
anything it references. There may however be another reference that can modify them. All
capabilities are transitive or "deep". That is, they don't affect only the reference or value they
are on, but also everything reachable through that value or reference.

Capabilities are valuable because they allow developers to express intent and reason about their
code, but also because they ensure thread safety in Azoth. When calling a method, you can know which
parameters it might modify because it will indicate which ones it requires `mut` on. When passing
values between threads, the compiler uses capabilities to ensure that there can never be shared
mutable state between threads that is not thread-safe.

In addition to permission capabilities there are capabilities that express a constraint. The two
most important ones are `const` and `iso`. A `const Example` indicates that the value and everything
reachable from it are constant. They cannot be modified through this reference nor through any other
in the program. An `iso Example` indicates that this is the only reference to an isolated sub-graph
of objects. That is there are no other references into the graph of objects reachable from this
reference. Note that there can still be references out from this subgraph to `const` or `id` values.
Having an isolated reference allows for more operations. For example, given an `iso` reference to a
data structure, one can `freeze` it and make the whole data structure `const`. That would not be
possible if there were other references to any object in the data structure that might still be able
to mutate it. Additionally, both `const` and `iso` values are safe to pass between threads.

In academic literature, these are generally referred to as "reference capabilities". In Azoth, they
are called just "capabilities" because they can be applied to things like value types that aren't
references. Capabilities are one of the most unique features of Azoth since there are very few
production languages with anything similar.[^1] The Pony language has genuine reference capabilities
but it uses a different mechanism for managing them and different names for them. Both of those
aspects make them less ergonomic and more difficult to program with. Arguably, Rust has a limited
form of reference capabilities with its mutable references and read-only references (e.g. `&mut T`
and `&T`). However, Rust's system is much more restrictive and less flexible because it must support
memory management without a garbage collector and capabilities aren't fully transitive. Azoth's
capabilities most closely resemble those of Project Midori as described on [Joe Duffy's
Blog](https://joeduffyblog.com/) and in the Microsoft's paper Uniqueness and Reference Immutability
for Safe Parallelism.

There are additional capabilities such as `id` and `own` as well as the parameter modifier `lent`.
For a full explanation, see the [Capabilities](capabilities.md) section.

[^1]: In additional to Pony and Rust, there is also [Language 42](https://l42.is/)

### Write Once, Compile Anywhere

**TODO:** document what this means

### Inferred Exception Specifications

Azoth has checked exceptions similar to, but much better than, Java or C++. However, unlike either
of those languages, by default the exceptions thrown by a function are automatically inferred. Only
on externally exposed APIs of packages are exception specifications required. Additionally, Azoth's
powerful type system provides the tools needed to work with exception specifications so they don't
get in the way.

```azoth
// Throws clause is required because this functions is a published API
published fn function1()
    throws Exception
{
    Function2();
}

// Throws clause is inferred because it is omitted and this is a public function
public fn function2()
{
    throw Exception();
}
```

For more information see [Structured Error Handling](structured-errors.md).

### Implicit Traits

In many languages there is a distinction between an interface or trait that has no implementation
and a class which can have an implementation. While Azoth does have both classes and traits, classes
also define an implicit trait. Any class can implement the implicitly defined trait of another
class. In a class declaration, the base class appears after the first colon and classes whose trait
is being implemented appear after the subtype operator `<:` along with any traits being implemented.

```azoth
public class My_Class: Base_Class <: Some_Class, Some_Trait
{
}
```

This simplifies the type hierarchy and eliminates the practice often needed in C# and Java of
defining a matching interface for many classes. The feature also necessitates several of Azoth's
other unique features. Private members are accessible only from the same instance, not by other
instances of the same class. Public fields can be overridden by properties in subclasses, so there
is no need to declare properties for every field.

### Immutable by Default

All variable declarations and types are immutable by default. Mutability has to be opted into. This
is accomplished by having `let` be the default way of declaring a variable and requiring `var` to
opt into mutability of the binding. For parameters, `let` is assumed and they must be marked `var`
to allow the binding to be mutated. This is also accomplished by the default capability being read.

### Guaranteed Optimizations

Because of immutable by default and other features of the language, it becomes possible to safely
execute some code at compile time. For example, if you initialize a constant to the square root of
two, this computation will be performed during compilation and the constant will be directly
initialized with the resulting value.

Exactly how this feature will work is still being determined.

### Asynchronous Everywhere

Azoth enforces the [structured concurrency](https://vorpus.org/blog/notes-on-structured-concurrency-or-go-statement-considered-harmful/)
programming paradigm and uses something similar to `async`/`await` support. However, it uses green
threads similar to the Go language. So, unlike other languages with `async`/`await` there is no
contagious spread of async functions (see [What Color is Your Function?](https://journal.stuffwithstuff.com/2015/02/01/what-color-is-your-function/)).
Instead, it is safe to start and await an async operation in synchronous code and it will block the
current thread-of-execution without holding a thread.

In C#, asynchronous code can easily lead to race conditions as two threads share data across an
asynchronous boundary. In Azoth, capabilities protect the developer from any such possible race
conditions so that asynchronous code is fully safe.

### Generics with Partial Specialization

Generics as supported by C# and Java are much easier to reason about than C++ style templates.
However, they can be more restrictive. The type system of Azoth allows for much greater flexibility
in generic types. For example, it is possible to use the type `void` as a type parameter to a
generic class or function. Additionally, it is possible to provide specialized implementations of a
generic class or function for specific types for better performance.

**TODO:** this may be dropped from the language

### Open Methods

The fragile base class problem occurs when a class overrides a base class's methods. This is avoided
by specifically marking methods that the base class will call subclass implementations with the
`open` keyword.

### Named Initializers

Initializers can be given names to indicate their purpose and meaning.

### Minimal Runtime

**TODO:** Document how it has a runtime but it is minimal

## Uncommon Features

This section lists features Azoth shares with other languages that are less common, but still
contribute to the distinctive flavor of the Azoth language.

* Type Inference on Local variable declarations
* Diverging Functions (return type `never`)
* A Specific Infinite Loop Keyword
* All `for` loops are iterator based
* Iterator performance often optimizes down to a C style `for` loop
* Operator Overloading
* Object Expressions: allows creation of a single instance of an anonymous type
* Classes can be extended with additional methods in separate libraries
* Partial Classes: supports code generation
* Both reference and value types
* C# style generators with `yield` keyword
* `unsafe` code blocks and low-level language features like raw pointers
* C interop
