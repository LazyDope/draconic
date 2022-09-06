# draconic

The Draconic scripting language, designed to allow a safe in-memory runtime for user scripting with bindings to
native APIs hosted in the parent application.

Requires Python 3.8+.

## Notable Differences from Python

The Draconic language is based on, and implemented in, the CPython implementation of the Python language. However, there
are a number of notable differences between Draconic and Python:

### In-Place Operator Unfurling

In Python, in-place operators (e.g. `a += 1`) are handled differently from binary operators and assignments (
e.g. `a = a + 1`). In Draconic, all in-place operators are unfurled to their binary-operator equivalent. In most cases
this does not create a semantic difference, except in the following cases:

#### Dict/Set Union Operations

**Python**

```pycon
>>> a = {"a": 1, "b": 2}
>>> b = {"b": 3, "c": 4}
>>> a_reference = a

>>> a |= b
>>> a
{"a": 1, "b": 3, "c": 4}
>>> a_reference
{"a": 1, "b": 3, "c": 4}
>>> a is a_reference
True
```

**Draconic**

```pycon
>>> a = {"a": 1, "b": 2}
>>> b = {"b": 3, "c": 4}
>>> a_reference = a

>>> a |= b
>>> a
{"a": 1, "b": 3, "c": 4}
>>> a_reference
{"a": 1, "b": 2}
>>> a is a_reference
False
```

Use `dict.update()` or `set.update()` instead for an in-place operation.

#### Function Nonlocal References

**Python**

```pycon
>>> a = 1
>>> def incr_locally():
...     a += 1
...     return a
>>> incr_locally()
UnboundLocalError: local variable 'a' referenced before assignment
```

**Draconic**

```pycon
>>> a = 1
>>> def incr_locally():
...     a += 1
...     return a
>>> incr_locally()
2
```

### Unequal Patma Bindings

While Python checks that all bindings in a pattern matching case with multiple options are the same across all branches,
Draconic does not do any such check.

**Python**

```pycon
>>> a = 1
>>> match a:
...     case (1 as one) | (2 as two):
...         print(one)
SyntaxError: alternative patterns bind different names
```

**Draconic**

```pycon
>>> a = 1
>>> match a:
...     case (1 as one) | (2 as two):
...         print(one)
1
```

### Function Dunder Attributes

As access to `__dunder__` attributes is not allowed in Draconic, the `function.__name__` and `function.__doc__` 
attributes are exposed as `function.name` and `function.doc` instead.

**Python**

```pycon
>>> def foo():
...     """I am foo"""
...     pass
>>> print(foo.__name__)
foo
>>> print(foo.__doc__)
I am foo
```

**Draconic**

```pycon
>>> def foo():
...     """I am foo"""
...     pass
>>> print(foo.name)
foo
>>> print(foo.doc)
I am foo
```

### Try/Except

As Draconic currently has no user-defined class implementation, it is impossible to provide a type by name to an 
`except` clause. Instead, Draconic requires `except` clause types to be a string literal or tuple of string literals.
These are matched against the name of the exception, with no subclass checking.

**Python**

```pycon
>>> try:
...     1/0
... except Exception:
...     print("I catch all exceptions!")
... except ZeroDivisionError:
...     print("You divided by zero!")
I catch all exceptions!
```

**Draconic**

```pycon
>>> try:
...     1/0
... except "Exception":
...     print("I catch all exceptions!")
... except "ZeroDivisionError":
...     print("You divided by zero!")
You divided by zero!
```

### Starred Unpacking

Assigning to a starred variable outside of a tuple or list in Python throws a `SyntaxError`, but is valid in Draconic

**Python**

```pycon
>>> *a = [1, 2, 3]
SyntaxError: starred assignment target must be in a list or tuple
```

**Draconic**

```pycon
>>> *a = [1, 2, 3]
>>> a
[1, 2, 3]
```

This has syntactical equivalents in Python:

```pycon
>>> *a, = [1, 2, 3]
>>> [*b] = [1, 2, 3]
>>> (*c,) = [1, 2, 3]
>>> assert a == b == c == [1, 2, 3]
```
