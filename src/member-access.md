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
* private: Access is limited to the current instance of the containing declaration. For declarations
  in a namespace, visibility is limited to the current file.

Note that private access is more restrictive than in other object oriented languages like C# or
Java. In those languages, private visibility allows other instances of the same class to access
those members. In Azoth, it does not.

```grammar
access_modifier
    : "published"
    | "published" "protected"
    | "public"
    | "protected"
    | ε // private
    ;
```

**TODO:** This system is actually really good, but needs more explanation. Good points: 1. only
`published` things are ever visible outside the package. 2. `protected` on namespace members
provides a way to limit visibility within a single package. 3. Not too many access modifiers. 4.
File visibility is good for code generation and local aliases.

## File Visibility

Declarations visible within a file will not have name conflicts with other files. For example a
file-scoped `class Foo` in one file will not conflict with a file-scoped `class Foo` in another
file. However, if either were public then the file-scoped one would report an error for a duplicate
class name if they were declared in the same namespace. This feature allows one to use the type
alias features (e.g. `type alias X = Foo;`) to give types local names in a way that is equivalent to
C# using alias directive (e.g. `using X = Foo;`).

## Namespace Dependencies

**TODO:** this whole section is speculation on what might be needed.

Is it possible to mimic the restrictions created by projects in C# hiding members from each other?
Note that before the stupid transitive reference thing, you could chain A references B references C.
If you turn those each into sub-namespaces, the `protected` keyword would let you hide members of
`B` from `C` in many circumstances (though you still need to expose some upward?). But members of
`B` would then not be visible to `A`. (One way would be to nest `A.B.C` but that doesn't seem like a
good idea.)

It is hoped that packages can be units of deployment, and don't need to be used just to control
accessibility.

Perhaps what is needed is an access modifier that exposes something upward, but not horizontally and
then a way to alias that to re-expose it to specific things?
