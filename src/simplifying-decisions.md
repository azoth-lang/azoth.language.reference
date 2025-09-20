# Simplifying Decisions

In designing Azoth there are many options and many possible advanced features. For the sake of getting
an initial version that can be used to experiment with and learn from, it is necessary to rule out
some of those options for now. This section documents those decisions. They may be changed in the future
if it becomes clear that the functionality offered would be valuable.

## No Exclusive Mutability

It might make sense to have a capability that provides exclusive mutability while not guaranteeing
isolation. In Pony this is called `trn`. In Azoth, it might be `xmut`. This capability would allow
freezing. A temporary version of this capability would enable some mutable iterators to not require
`temp iso`. They could instead take `temp xmut` and still allow other references to read from the
collection while iterating.
