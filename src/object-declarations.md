# Object Declarations

An object declaration declares a reference type that has only a single instance. As with other
non-instance state in Azoth, they must be inherently constant. An object declaration is very much
like a class declaration. However, a single constant instance will be created. Since the object name
refers to both the declared type and that instance, both associated and instance members are
accessed from it. While object declarations are useful for creating singleton objects, they are also
very important in the creation of closed type hierarchies.

**TODO:** does this cause problems with name conflicts between instance and associated members?

An object declaration is the same as a class declaration except:

* The `object` keyword replaces the `class` keyword
* They are implicitly `const`
* They cannot be declared `move`, `sealed`, or `closed`
* A single no-arg initializer is allowed and no named initializers are allowed. (This means that a
  record object must have no parameters.)

```azoth
public object Void_Type: Type
{
    public get type_parameters(self) -> const List[Type]
        => #[];
}
```

**TODO:** Could object declarations actually allow mutability if they are thread-safe?
