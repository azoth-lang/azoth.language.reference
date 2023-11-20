# With Expression

A when expression provides two benefits. It introduces a contextual value. See [contextual
arguments](optional-arguments.md#contextual-arguments) for more information on contextual values.
And it helps to manage resources.

With expressions come in two forms. An anonymous with expression and a named with expression that
declares a name for the context object.

```grammar
with_expression
    : "with" embedded_expression block
    | "with" "let" identifier (":" type)? "=" embedded_expression block
    ;
```

The with expression cannot be declared with `var` because that would make no sense with the
contextual value and would defeat the scoped based resource management.

## With Statements?

A way to declare a context variable until the end of the current scope?

```grammar
with_statement
    : "with" embedded_expression ";"
    | "with" "let" identifier (":" type)? "=" embedded_expression ";"
    ;
```

## With Pattern

If the type defines an `exit_normally(self)` method it will be called when the with block exits
normally. If it defines an `exit_errored[T](self, error: T)` method it will be called when the with
block is exited via an exception. If the type has a destructor it will be called after the exit
method whenever the block exits.

**TODO:** is this the best way to do this? Perhaps they should be overloads of a `with` operator?

## Use Cases

### Locks

Given the safe concurrency, locks may be completely unnecessary, but if needed, would be easy to
manage.

```azoth
with mutex.lock()
{
    // lock held here
}
```

### Reading Files and Streams

```azoth
with let file = File.Open(path)
{
    let contents = file.read_all();
    // ...
}
```

### Transactions

Transaction scopes take advantage of the ability to differentiate between error and successful
return.

```azoth
with let transaction = db.begin_transaction()
{
    transaction.update(...);

    if something => throw new Error(); // will cause rollback

    // reaching the end of the block successfully will commit the transaction
}
```

### Precision Context

For a precision context, the primary use is to define the contextual value, not to dispose of any
resources.

```azoth
with decimal.precision(32)
{
    var x = 1 as decimal / 3;
}
```

### Defining Using Generators

Perhaps there is an easy way to declare types meant to work in a `with` expression using generators.

```azoth
published class Database
{
    #Generator_Method_Builder[With_Builder]
    published fn begin_transaction(mut self) -> Transaction
    {
        var t = self.start_transaction();
        try
        {
            defer if t.is_active => t.commit();
            new yield; // Needed to force a call to begin?
            yield return;
        }
        catch
        {
            t.rollback();
            throw;
        }
    }
}
```

## References

* [PEP 343 – The “with” Statement](https://peps.python.org/pep-0343/)
