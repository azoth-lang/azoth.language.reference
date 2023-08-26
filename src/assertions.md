# Assertions

```azoth
Debug.Assert(condition);
Release.Assert(condition);
```

Both of these will abort if the condition is not true. Debug assertions are only checked when
compiling for debug while release assertions are checked even in release builds.
