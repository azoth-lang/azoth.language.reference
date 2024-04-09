# Value Types

The value types can be further divided into a number of subcategories. Most value types are *struct
types*. The *simple types* are predefined value types identified with keywords. The *tuple type* is
a predefined type for tuples of values.

```grammar
value_type
    : simple_type
    | struct_type
    | tuple_type
    ;
```
