# Aliases

The keyword "`alias`" introduces an alias for a type name. It does not create a separate type.

```azoth
public alias PromiseResult[T] = Promise[Result[T]]
```

Aliases can have any access modifier. It may be useful/desirable to have aliases for other things
such as namespaces etc. For that reason, or just for increased clarity it might be preferable to use
the keyword pair `type alias` for type aliases.

Type aliases inside of type declarations are "static" and associated with the type not the instance.
They cannot be virtual.
