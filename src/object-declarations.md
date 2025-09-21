# Object Declarations

An object declaration declares a reference type that has only a single instance. As with other
non-instance state in Azoth, they must be inherently constant. An object declaration is very much
like a class declaration. However, a single constant instance will be created. Since the object name
refers to both the declared type and that instance, both associated and instance members are
accessed from it. While object declarations are useful for creating singleton objects, they are also
very important in the creation of closed type hierarchies. Object types are conceptually empty
types. They cannot contain fields. The type of the value fully determines what can be done with it.
This becomes important because objects can also inherit from closed value types.

**TODO:** does this cause problems with name conflicts between instance and associated members?

An object declaration is the same as a class declaration except:

* The `object` keyword replaces the `class` keyword
* They are implicitly `const`
* They cannot be declared `drop`, `sealed`, or `closed`
* A single no-arg initializer is allowed and no named initializers are allowed. (This means that a
  record object must have no parameters.)
* They cannot declare fields (though they may inherit them). They may have properties.
* They can inherit from a closed value type (see [Closed Values](closed-values.md)).
* An object declaration can invoke a base constructor by placing the constructor parameters after
  the base class name. If this is done, that is the object's no-arg initializer. (This is consistent
  with the object expression syntax.)

```azoth
public object Void_Type: Type
{
    public get type_parameters(self) -> const List[Type]
        => #[];
}
```

**TODO:** Could object declarations actually allow mutability if they are thread-safe?
