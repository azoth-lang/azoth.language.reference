# Member Access

## Declared Visibility

In Azoth there are a number of levels of visibility an item can have. These are specified using the
access modifiers "`protected`", "`public`", and "`published`". If there is no access modifier on an
item, it is private. The access limitations imposed by these visibility levels are:

* "`published`": Access is not limited.
* "`published protected`": Access is limited to the containing declaration or subtypes of the
  containing type.
* "`public`": Access is limited to the the current package.
* "`protected`": Access is limited to the containing declaration or subtypes of the containing type
  within the current package.
* private: Access is limited to the current instance of the containing declaration. For namespace
  declarations, visibility is limited to the current file.

Note that private access is more restrictive than in other object oriented languages like C# or
Java. In those languages, private visibility allows other instances of the same class to access
those members. In Azoth, it does not.

```grammar
access_modifier
    : "published"
    | "published" "protected"
    | "public"
    | "protected"
    | // private
    ;
```
