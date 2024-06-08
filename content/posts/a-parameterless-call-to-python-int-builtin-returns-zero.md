+++
title = "A parameterless call to Python's `int` builtin returns zero"
date = 2024-02-07T19:21:54Z
tags = ["Python"]
+++

# A parameterless call to Python's `int` builtin returns 0

The following test passes:
```python
assert int() == 0
```

Which fits in with the notion from set theory that the numerical representation of
nothing is 0. Unfortunately though calling `int` on python's empty set does not 
return 0.
The same goes for the `float` builtin.

This is useful for example to aggregate a collection of objects using a `defaultdict`.
Here is a simple example of this:

```python
import collections
import dataclasses
import random


@dataclasses.dataclass
class Item:
    id: str

my_items = [Item(id=id) for id in random.choices(["foo", "bar", "baz"], k=10)]
counts = collections.defaultdict(int)

for item in my_items:
    # Indexing `counts` for a non-present `id` will create a new
    # key with value 0, since `int` is used as the default factory. 
    counts[item.id] += 1
```
