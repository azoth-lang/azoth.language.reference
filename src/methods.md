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
