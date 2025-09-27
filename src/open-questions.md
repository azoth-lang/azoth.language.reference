# Open Questions

This section documents some open questions in the design of Azoth.

## User Defined Conversion Syntax

The syntax for declaring user defined conversions as not be specified.

## Java-style Inner Classes

In Java, an inner class as an implicit reference to the outer class. In C# that isn't needed,
instead one simply makes an explicit reference. In Azoth, the restriction that private members can
only be accessed from the same instance might mean that it makes sense to have inner classes like
that. However, it isn't clear what distinguishes that from a simple nested class.

## Advanced Operator Overloading

How to overload short-circuiting operators and other advanced operators.

## Initializer Questions

* If some return type parameters are specified, what does that do to the type parameters when
  initializing?
* Can the initializer method be generic so it can control type parameters?
* How are type parameters passed when calling a named initializer given that they may be
  changed/reduced it seems they belong with the name not the type?
* How can one combine specifying type parameters with returning a base type?
* Can you declare initializers in traits? (This allows them to act more like base classes e.g. a
  `Dictionary[Key, Value]` trait can have an initializer that returns the default dictionary
  implementation.)
* Since static methods can be overridden, can a named initializer override a static method in a
  trait? Likewise can an initializer override an initializer in a trait? This would would avoid the
  need to create static factory methods just to allow generics to construct types implementing a
  trait.

Idea:

* Perhaps it is like the initializer is a static method on a non-generic version of the type that
  has all the same type parameters by default?

## Thread-Safe Types

How do types like lockless collections declare that they are mutable but still safe to send between
threads?

## Declaring Initializers for Initializer Expressions

How should a type declare that it supports a particular initializer expression. Is this a special
initializer syntax or is it some kind of operator overload.
