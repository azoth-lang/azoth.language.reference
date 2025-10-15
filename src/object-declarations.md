# Object Declarations

An object declaration declares a reference type that has only a single instance. As with other
non-instance state in Azoth, they must be inherently constant. An object declaration is very much
like a class declaration. However, a single constant instance will be created. Since the object name
refers to both the declared type and that instance, both associated and instance members are
accessed from it. While object declarations are useful for creating singleton objects, they are also
very important in the creation of closed type hierarchies. Object types cannot contain fields. The
type of the value fully determines what can be done with it.

**TODO:** does this cause problems with name conflicts between instance and associated members?

An object declaration is the same as a class declaration except:

* The `object` keyword replaces the `class` keyword
* They are implicitly `const`
* They cannot be declared `drop`, `sealed`, or `closed`
* A single no-arg initializer is allowed and no named initializers are allowed. (This means that a
  record object must have no parameters.)
* They cannot declare fields (though they may inherit them). They may have properties.
* An object declaration can invoke a base initializer by placing the initializer parameters after
  the base class name. If this is done, that is the object's no-arg initializer. (This is consistent
  with the object expression syntax.)

```azoth
public object Void_Type inherits Type
{
    public get type_parameters(self) -> const List[Type]
        => #[];
}
```

**TODO:** Could object declarations actually allow mutability if they are thread-safe?

## Notes

It was considered that perhaps one should be allowed to inherit from object declarations unless they
were explicitly declared sealed. However, that violates the idea that they are a singleton object.
It would be rarely used requiring most object declarations to be declared sealed. Also, it is still
possible to use the singleton pattern to create a class with a single instance that can be inherited
from.
