# Drop Methods

Classes, values, and structs declared as `drop` types may have a drop method. Because they are drop
types the compiler enforces that there is either one owning value. Thus the compiler can determine
when that value goes out of scope and call the drop method on it. Note that this does not mean that
the memory of an object or those referenced is immediately freed. That is still the responsibility
of the garbage collector. Rather, drop methods provide a means of managing other resources via the
resources acquisition is initialization (RAII) pattern.

Any class, value, or struct with a field whose type is a drop type gets a default drop method and
thus must be a drop type itself.

```grammar
destructor
    : "public" "drop" "(" "lent" "mut" "self" ")" expression_block
    ;
```

## Finalizers

Most garbage collected languages do not have destructors or drop methods. They instead have
finalizers. A finalizer is run at an indeterminate time by the garbage collector after it has
determined the object to be garbage. As such, finalizers have a host of problems including:

* object resurrection
* not called in a timely manner
* garbage collection is based on memory pressure not resource scarcity of other resources
* executed in unspecified order
* exceptions in finalizers cannot be handled
* may not be run during program termination

Because of these issues, Azoth does not have finalizers. This is possible because it provides drop
methods as a means to manage non-memory resources in a reliable way.

## Default Drop Methods and Dropping

Any type with fields that are drop types will get a default drop method if no drop method is
declared. This drop method will call the drop methods of all fields first and then the base class
drop method.

If a type has fields that are drop types and a drop method is declared, then immediately before the
call to the base class drop method, the drop methods of all fields will be automatically called.

## Exceptions in Drop Methods

All drop methods implicitly are `throws never`. This is because drop methods are called while
exceptions are unwinding the stack. Thus if an exception were to be thrown from a drop method it
might hide the original exception that is leading to the drop method being called.

**TODO:** Allow the drop method to receive info about an in progress error and allow drop method
definitions to override the default of `throws never`

## Base Class Drop Method

A base class drop method can be called similar to other base class methods with the `base.drop()`
syntax.

## No Resurrection

Since the only reference to the value is about to go out of scope, it is not possible to persist new
references to `self` past the end of the destructor. The reference capability system will prevent
this.
