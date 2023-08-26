# Generators

The `yield` keyword acts basically as `yield return` does in C#.

Python uses `return` for the equivalent of `yield break`, however that causes an inconsistency where
removing yield from a method changes the meaning of the return. Azoth should use either `yield
break;` or `yield return;` or something else.

Python has `yield from` for recursively yielding another iterator (Python already used `from` for
other things). A paper proposed `yield foreach` for the same thing in C#. Azoth doesn't have a
`foreach` or `from` keyword and isn't likely to get either. Options for this are `yield for`, `yield
from`, and `yield all`. It is important to make sure this can be implemented in an efficient way
that doesn't easily lead to exponential blowup.