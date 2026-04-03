# Python FAQ: Memory Management & GIL (Q26–Q40)

---

**Q26: How does Python manage memory?**

A: Python uses a private heap to store objects. The memory manager handles allocation and deallocation automatically. Python also uses a memory pool (via `pymalloc`) for small objects (≤512 bytes) to reduce the overhead of frequent `malloc`/`free` OS calls.

---

**Q27: What is garbage collection in Python?**

A: Python uses two mechanisms: **reference counting** (primary) and a **cyclic garbage collector** (secondary). Every object has a reference count. When it drops to zero, memory is freed immediately. The cyclic GC handles reference cycles (e.g., A → B → A) that reference counting alone can't free.

---

**Q28: What is a reference count and how does it work?**

A: Every Python object has an internal counter tracking how many references point to it. When you do `x = obj`, the count increases. When `x` goes out of scope or is reassigned, the count decreases. When it reaches 0, the object is immediately deallocated. Use `sys.getrefcount(obj)` to inspect.

---

**Q29: What causes a memory leak in Python?**

A: Common causes: circular references not cleaned by GC, global variables holding large objects, unbounded caches (like dicts or lists that grow forever), and unclosed file handles or database connections. In data pipelines, the most common cause is accumulating records in a list without processing and clearing it.

---

**Q30: What is the GIL (Global Interpreter Lock)?**

A: The GIL is a mutex in CPython that allows only one thread to execute Python bytecode at a time. It simplifies memory management but limits true parallelism in CPU-bound multithreaded programs. For I/O-bound tasks (like network calls), threads still work well because the GIL is released during I/O.

---

**Q31: Why does the GIL exist?**

A: CPython's memory management (especially reference counting) is not thread-safe. The GIL prevents two threads from simultaneously modifying reference counts and corrupting memory. It was a design tradeoff that made Python simpler to implement and extend with C libraries.

---

**Q32: How does the GIL affect data engineering workloads?**

A: Most DE work is I/O-bound (reading files, querying databases, calling APIs). The GIL releases during I/O operations, so multithreading still provides speedups for these workloads. For CPU-bound transformations, use `multiprocessing` instead, as each process has its own GIL.

---

**Q33: What is the difference between shallow copy and deep copy?**

A: A **shallow copy** (`copy.copy()` or `list[:]`) creates a new container but the nested objects are still shared. A **deep copy** (`copy.deepcopy()`) recursively copies all nested objects. For a list of dicts, a shallow copy means modifying a nested dict will affect both the original and the copy.

---

**Q34: What `is` the difference between `copy.copy()` and `copy.deepcopy()`?**

A: `copy.copy()` is O(n) — copies only the top-level container. `copy.deepcopy()` is O(n×depth) — recursively copies everything. For performance-sensitive pipelines with large nested structures, prefer redesigning to avoid deep copies entirely rather than paying the cost.

---

**Q35: What is object interning in Python?**

A: Python automatically reuses (interns) small integers (-5 to 256) and some strings. `a = 100; b = 100; a is b` → `True` because both point to the same object. This is an optimization — don't rely on `is` for value comparison, use `==`.

---

**Q36: What is `__slots__` and when should you use it?**

A: `__slots__` restricts a class to only the attributes listed in the tuple, preventing the creation of a `__dict__` per instance. This reduces memory significantly for classes with many instances. Example:
```python
class Record:
    __slots__ = ("id", "value")
```
Use when creating millions of small objects (e.g., streaming records).

---

**Q37: What is the difference between stack and heap in Python?**

A: The **call stack** holds function frames (local variables, arguments). When a function returns, its frame is popped. The **heap** holds all Python objects. Variables on the stack don't hold data — they hold references (pointers) to objects on the heap. CPython manages both automatically.

---

**Q38: How can you reduce memory usage when processing large datasets in Python?**

A: Use generators instead of lists (lazy evaluation), process data in chunks, use `del` to remove unneeded large objects, use `gc.collect()` to force cyclic GC, and prefer typed arrays (`array` module or NumPy) over lists of numbers. In Pandas, use chunked reading (`chunksize` parameter) for large CSVs.

---

**Q39: What is `sys.getsizeof()` and what does it tell you?**

A: `sys.getsizeof(obj)` returns the memory size of an object in bytes. `sys.getsizeof([1,2,3])` returns only the list container's size, not the elements inside. For a true recursive size measurement, you'd need a deep-size utility. Useful for quick memory sanity checks.

---

**Q40: What happens when you assign a list to another variable in Python?**

A: Both variables point to the same list object in memory — no copy is made. Modifying through one variable changes what the other sees. Example:
```python
a = [1, 2, 3]
b = a
b.append(4)
print(a)  # [1, 2, 3, 4]
```
To avoid this, explicitly copy: `b = a[:]` or `b = a.copy()`.

---

> Next: [03_concurrency.md](03_concurrency.md)
