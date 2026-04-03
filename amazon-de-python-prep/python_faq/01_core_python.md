# Python FAQ: Core Python Concepts (Q1–Q25)

---

**Q1: What are Python's key built-in data types?**

A: `int`, `float`, `str`, `bool`, `list`, `tuple`, `dict`, `set`, `NoneType`. Lists are mutable ordered sequences. Tuples are immutable. Dicts store key-value pairs. Sets store unique unordered elements. Knowing when to use each is critical for efficient code.

---

**Q2: What is the difference between `=` and `==` in Python?**

A: `=` is the assignment operator — it stores a value in a variable. `==` is the comparison operator — it checks if two values are equal and returns `True` or `False`. Confusing the two is a common beginner bug.

---

**Q3: What does `None` represent in Python?**

A: `None` is Python's null value — it represents the absence of a value. It is the only instance of `NoneType`. Functions without a `return` statement return `None` by default. Always check `if value is None` (not `== None`) as best practice.

---

**Q4: What is the difference between a list and a tuple?**

A: Lists are mutable (can be changed after creation) and defined with `[]`. Tuples are immutable (cannot be changed) and defined with `()`. Tuples are faster, use less memory, and are used as dictionary keys or fixed records. Lists are used for collections that will be modified.

---

**Q5: What is list comprehension and why use it?**

A: List comprehension is a compact way to build lists: `[expr for item in iterable if condition]`. It is more readable and faster than a `for` loop with `.append()`. Example: `[x*2 for x in range(10) if x % 2 == 0]` gives all even doubles 0–18.

---

**Q6: What is a dictionary comprehension?**

A: Similar to list comprehension but builds a dict: `{key_expr: val_expr for item in iterable}`. Example: `{name: len(name) for name in ["Alice", "Bob"]}` → `{"Alice": 5, "Bob": 3}`. Used often for data transformations.

---

**Q7: What is the `*args` and `**kwargs` syntax?**

A: `*args` allows a function to accept any number of positional arguments as a tuple. `**kwargs` accepts any number of keyword arguments as a dict. Example: `def log(*args, **kwargs)`. Used in decorators and flexible APIs.

---

**Q8: What is the difference between `is` and `==`?**

A: `==` checks value equality (do both objects have the same value?). `is` checks identity (do both variables point to the exact same object in memory?). `None`, `True`, and `False` should be compared with `is`, not `==`.

---

**Q9: What is a Python lambda function?**

A: A lambda is a small anonymous function defined in one line: `lambda arguments: expression`. Example: `double = lambda x: x * 2`. Used as quick inline functions in `sorted()`, `map()`, `filter()`. For complex logic, use a named `def` instead.

---

**Q10: What does `map()` do?**

A: `map(function, iterable)` applies a function to every element and returns a lazy iterator. Example: `list(map(str, [1, 2, 3]))` → `["1", "2", "3"]`. Similar to a list comprehension but returns a map object, not a list directly.

---

**Q11: What does `filter()` do?**

A: `filter(function, iterable)` keeps only elements where the function returns `True`. Example: `list(filter(lambda x: x > 0, [-1, 2, -3, 4]))` → `[2, 4]`. Like `map()`, it returns a lazy iterator.

---

**Q12: What is the difference between `range()` and `list(range())`?**

A: `range(n)` returns a lazy range object that generates numbers on demand — no memory is used upfront. `list(range(n))` materializes all numbers into a list. For loops with `range()` don't need `list()` — using it wastes memory.

---

**Q13: What is `enumerate()` and when should you use it?**

A: `enumerate(iterable, start=0)` yields `(index, value)` pairs. Use it when you need both the index and the value in a loop instead of `for i in range(len(lst))`. Example: `for i, name in enumerate(names): print(i, name)`.

---

**Q14: What is `zip()` and when is it useful?**

A: `zip(a, b)` pairs corresponding elements from two iterables into tuples. Stops at the shorter iterable. Example: `list(zip([1,2,3], ["a","b","c"]))` → `[(1,"a"), (2,"b"), (3,"c")]`. Used to combine headers with row values in CSV parsing.

---

**Q15: What is string formatting? What is an f-string?**

A: F-strings (format strings) are the modern way to embed expressions inside string literals: `f"Hello {name}"`. Introduced in Python 3.6. Faster and more readable than `%` formatting or `.format()`. You can embed any valid expression: `f"Total: {sum(values):.2f}"`.

---

**Q16: What is the difference between `append()`, `extend()`, and `insert()` for lists?**

A: `append(x)` adds `x` as a single element at the end. `extend(iterable)` adds all elements of the iterable at the end. `insert(i, x)` adds `x` at index `i`. `[1,2].append([3,4])` → `[1,2,[3,4]]`; `[1,2].extend([3,4])` → `[1,2,3,4]`.

---

**Q17: How does Python handle mutable default arguments? What is the gotcha?**

A: Mutable defaults (like `def f(lst=[])`) are created once and shared across all calls. This causes bugs where state persists between calls. The fix: use `None` as default and create the object inside the function:
```python
def f(lst=None):
    if lst is None:
        lst = []
```

---

**Q18: What is the difference between `pop()` and `remove()` for lists?**

A: `pop(index)` removes and returns the item at `index` (default: last element). `remove(value)` removes the first occurrence of `value` by value, not index. `pop()` is O(1) at the end, O(n) at arbitrary positions. `remove()` is always O(n).

---

**Q19: What is string immutability? How does it affect performance?**

A: Strings in Python are immutable — once created, they cannot be changed. Operations like `s += "x"` create a new string object. Concatenating in a loop (`+=`) is O(n²). Use `"".join(list_of_strings)` for efficient string building.

---

**Q20: What is `pass` in Python?**

A: `pass` is a no-op statement that does nothing. It's used as a placeholder where Python syntax requires a statement but you don't want any logic — e.g., in empty `except` blocks, stub class/function bodies, or planned future code. It prevents `SyntaxError`.

---

**Q21: What is the difference between `break`, `continue`, and `pass` in loops?**

A: `break` exits the loop entirely. `continue` skips the rest of the current iteration and moves to the next. `pass` does nothing — the loop continues normally. All three serve different control-flow purposes.

---

**Q22: What are Python's truthy and falsy values?**

A: Falsy values: `False`, `0`, `0.0`, `""`, `[]`, `{}`, `()`, `set()`, `None`. Everything else is truthy. This means `if my_list:` is equivalent to `if len(my_list) > 0:` and is the preferred Python style.

---

**Q23: What is `global` and `nonlocal` in Python?**

A: `global x` inside a function lets you modify a module-level variable. `nonlocal x` lets an inner (nested) function modify a variable in the enclosing function's scope. Overusing `global` is bad practice — prefer passing values as arguments.

---

**Q24: What is the `__name__ == "__main__"` guard?**

A: When Python runs a file directly, `__name__` is set to `"__main__"`. When it's imported as a module, `__name__` is the module name. The guard `if __name__ == "__main__":` ensures entrypoint code only runs when the file is executed directly, not when imported.

---

**Q25: What is the difference between `sorted()` and `.sort()`?**

A: `.sort()` sorts a list in-place (modifies the original, returns `None`). `sorted()` returns a new sorted list without modifying the original. `sorted()` works on any iterable; `.sort()` only works on lists. Use `sorted()` when you need to preserve the original.

---

> Next: [02_memory_gil.md](02_memory_gil.md)
