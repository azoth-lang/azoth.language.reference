# Methods

## Open Methods

By default, when one method calls another method on the same instance, the version of the method
declared in that class will be called rather than any overridden versions of that method. This helps
prevent the fragile base class problem where overriding a method unexpectedly changes the behavior
of a non-overridden method because it is implemented in terms of that method. This can also manifest
when a base class method is changed to be implemented in terms of another method.

If it is intended that the overridden version of a method should be called then that method must be
marked with the `open` keyword. Any method marked `open` will have its overridden version called
when called by another method on the same instance. Note that the `open` property does not inherit.
So if a method is declared `open` in the base class `A` and overridden in the class `B` then
instance methods in `B` calling that method will invoke the `B` version of the method even if it is
overridden in a subclass `C` of `B` unless the method in `B` is also declared `open`.

All `abstract` methods are implicitly `open` and it is a non-fatal error to declare them as `open`.

## Covariant Return Types

## Overriding

When overriding a method, it is possible to do one or more of:

* Rename a method
* Contravariantly change parameter types (i.e. make them less specific)
* Change visibility

To do any of these, uses the `overrides` keyword after the method signature and specify the aspect
being changed. To rename a method, provide the base method name being overridden. To change
parameter types, list the base method parameter types in parenthesis. To change the visibility,
provide the base visibility. Only the changed parts can be listed.

```azoth
public class B
{
  protected fn test(c: Cat) { ... }
}

public class C: B
{
  public fn example(a: Animal)
    overrides protected test(Cat)
  { ... }
}
```

Note that return types are covariant and do not need to be listed.

There are two possible syntaxes for overriding a method without changing any of these

```azoth
public class B
{
  protected fn test(c: Cat) { ... }
}

public class C1: B
{
  // First syntax
  protected fn test(c: Cat) overrides
  { ... }
}

public class C2: B
{
  // Second syntax
  protected override fn test(c: Cat)
  { ... }
}
```

## Hiding

Hiding a method in a base class means that the base class method is not overridden. If it is called,
its implementation will still run. But when using that method on the subclass, a different
implementation will be called. Hiding, like overriding, allows the visibility, name, and parameter
types to be changed.

Hiding is the default. If neither `overrides` or `hides` is used on a method then it hides the base
method with the same name and parameter types. It is a non-fatal error to hide a method without
declaring so with the `hides` keyword. This supports situations where methods are added to base
classes in published packages.
