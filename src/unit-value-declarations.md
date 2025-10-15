# Unit Value Declarations

Similar to object declarations, unit value declarations provide a way to declare a value type with
only a single value. As with other non-instance state in Azoth, they must be inherently constant. An
unit value declaration is very much like a value declaration. However, it will have a single value.
Since the unit value name refers to both the declared type and that value, both associated and
instance members are accessed from it. While unit value declarations are useful for creating
singleton values, they are also very important in the creation of closed type hierarchies. Unit
value types cannot contain fields. The type of the value fully determines what can be done with it.

**TODO:** does this cause problems with name conflicts between instance and associated members?

An unit value declaration is the same as a value declaration except:

* The `unit value` keywords replaces the `value` keyword
* They are implicitly `const`
* They cannot be declared `drop`, `sealed`, or `closed`
* A single no-arg initializer is allowed and no named initializers are allowed. (This means that a
  record unit value must have no parameters.)
* They cannot declare fields (though they may inherit them). They may have properties.
* An unit value declaration can invoke a base initializer by placing the initializer parameters
  after the base type name. If this is done, that is the unit value's no-arg initializer. (This is
  consistent with the object declarations syntax.)

```azoth
public closed const value Http_Method
{
  public abstract get name(self) -> string;
}

public unit value Http_Get inherits Http_Method
{
    public get name(self) -> string overrides => "GET";
}
```

## Unit Value Size

Unit values have no fields and so they have size zero unless they are inheriting from a value.
