# Destructors

Classes and structs declared as `move` types may have a destructor. Because they are move types the
compiler enforces that there is either one owning reference for reference types or a single copy of
the value for value types. Thus the compiler can determine when that instance goes out of scope and
call the destructor on it. Note that this does not mean that the memory of an object or those
referenced is immediately freed. That is still the responsibility of the garbage collector. Rather,
destructors provide a means of managing other resources via the resources acquisition is
initialization (RAII) pattern.

Any class or struct with a field whose type has a destructor gets a default destructor and thus must
be a move type itself.

```grammar
destructor
    : "public" "delete" "(" "iso" "self" ")" expression_block // reference types
    ;
```

**TODO:** should the destructor self parameter be `lent` so that it can't be resurrected?

## Finalizers

Most garbage collected languages do not have destructors and instead have finalizers. A finalizer is
run at an indeterminate time by the garbage collector after it has determined the object to be
garbage. As such, finalizers have a host of problems including:

* object resurrection
* not called in a timely manner
* garbage collection is based on memory pressure not resource scarcity of other resources
* executed in unspecified order
* exceptions in finalizers cannot be handled
* may not be run during program termination

Because of these issues, Azoth does not have finalizers. This is possible because it provides
destructors as a means to manage non-memory resources in a reliable way.

## Default Destructors and Destruction

Any type with fields that have destructors will get a default destructor if no destructor is
declared. This destructor will call the destructors of all fields first and then the base class
destructor.

If a type has fields that have destructors and a destructor is declared, then immediately before the
call to the base class destructor, the destructors of all fields will be automatically called. This
means that after the base class destructor is called, all move reference type fields have the `id`
capability and all move value type fields cannot be accessed.

## Exceptions in Destructors

All destructors implicitly are `no throw`. This is because destructors are called while exceptions
are unwinding the stack. Thus if an exception were to be thrown from a destructor it might hide the
original exception that is leading to the destructor being called.

**TODO:** Allow the destructor to receive info about an in progress error and allow destructors
definitions to override the default of `no throw`

## Base Class Destructors

A base class destructor can be called similar to other base class methods with the `base.delete()`
syntax.

## No Resurrection

Since the only reference to the object is about to go out of scope, it is not possible to persist
new references to `self` past the end of the destructor. The reference capability system will
prevent this.
