# Python FAQ: Decorators, functools & itertools (Q101–Q125)

---

## DECORATORS (Q101–Q112)

---

**Q101: What is a decorator and what problem does it solve?**

A: A decorator is a function that wraps another function to add behavior (logging, timing, retry, auth) without modifying the original code. It follows the Open/Closed principle — open for extension, closed for modification. Syntax: `@my_decorator` above the function definition. Under the hood, `@decorator` is equivalent to `func = decorator(func)`.

---

**Q102: Why should you use `functools.wraps` inside a decorator?**

A: Without `functools.wraps`, the wrapped function loses its `__name__`, `__doc__`, and other metadata. `functools.wraps(func)` copies the original function's attributes onto the wrapper. Always use it:
```python
from functools import wraps

def my_decorator(func):
    @wraps(func)   # preserves func.__name__, __doc__, __module__
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```
Without it, `help(func)` and logging would show `wrapper` instead of the original function name.

---

**Q103: How do you write a decorator that accepts its own arguments?**

A: Add one more nesting level — a function that accepts decorator arguments returns the actual decorator:
```python
from functools import wraps

def retry(max_attempts=3, exceptions=(Exception,)):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    if attempt == max_attempts:
                        raise
                    print(f"Attempt {attempt} failed: {e}")
        return wrapper
    return decorator

@retry(max_attempts=4, exceptions=(ConnectionError,))
def fetch_data(url):
    raise ConnectionError("timeout")
```
Three levels deep: `retry(args)` → `decorator(func)` → `wrapper(*args, **kwargs)`.

---

**Q104: What is a class-based decorator and when would you use one?**

A: A class implements `__call__` to be callable like a function. Useful when the decorator needs to maintain state:
```python
from functools import wraps

class CallCounter:
    def __init__(self, func):
        wraps(func)(self)   # copy metadata
        self.func = func
        self.calls = 0

    def __call__(self, *args, **kwargs):
        self.calls += 1
        print(f"[{self.func.__name__}] called {self.calls} time(s)")
        return self.func(*args, **kwargs)

@CallCounter
def process(data):
    return len(data)

process([1, 2, 3])   # called 1 time(s)
process([1, 2])      # called 2 time(s)
print(process.calls) # 2
```
Class decorators are useful for call counters, circuit breakers, or any decorator that accumulates state across calls.

---

**Q105: What is `@property` and when do you use it?**

A: `@property` turns a method into a read-only attribute. The caller accesses it like a field, not a function call. Use it when a value should be computed from other attributes:
```python
class PipelineRun:
    def __init__(self, rows_loaded, rows_expected):
        self.rows_loaded = rows_loaded
        self.rows_expected = rows_expected

    @property
    def success_rate(self):
        if self.rows_expected == 0:
            return 0.0
        return round(self.rows_loaded / self.rows_expected * 100, 2)

run = PipelineRun(48500, 50000)
print(run.success_rate)   # 97.0  — no parentheses needed
```
Add `@property.setter` to allow assignment. Use `@property` to encapsulate computation behind simple attribute access.

---

**Q106: What is `@staticmethod` vs `@classmethod`?**

A: `@staticmethod` defines a method that doesn't need `self` or `cls` — it's just a function namespaced inside the class. `@classmethod` receives the **class** as the first argument (`cls`), not the instance. Use `@staticmethod` for utility functions logically related to the class. Use `@classmethod` for alternative constructors:
```python
class DataRecord:
    def __init__(self, data):
        self.data = data

    @staticmethod
    def validate_schema(record):
        return "id" in record and "timestamp" in record

    @classmethod
    def from_json(cls, json_string):
        import json
        return cls(json.loads(json_string))

rec = DataRecord.from_json('{"id": 1, "timestamp": "2024-01-01"}')
print(DataRecord.validate_schema({"id": 1}))  # False (no timestamp)
```

---

**Q107: How do you stack multiple decorators?**

A: Stack them by listing multiple `@decorator` lines. They apply bottom-up (innermost first):
```python
from functools import wraps
import time

def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        t = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__}: {time.time()-t:.4f}s")
        return result
    return wrapper

def logger(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@timer      # applied second (outer)
@logger     # applied first (inner)
def process(data):
    return sum(data)

process([1, 2, 3])
# Calling process        ← logger runs first
# process: 0.0001s       ← timer wraps logger
```
`@timer @logger def f` is equivalent to `f = timer(logger(f))`.

---

**Q108: What is memoization? How do you implement it as a decorator?**

A: Memoization caches function results so repeated calls with the same arguments return instantly. Use `functools.lru_cache` (built-in) or implement manually:
```python
from functools import lru_cache

@lru_cache(maxsize=128)
def expensive_lookup(key):
    # Simulate slow computation
    return key * 2

print(expensive_lookup(10))   # computed
print(expensive_lookup(10))   # returned from cache instantly
print(expensive_lookup.cache_info())
# CacheInfo(hits=1, misses=1, maxsize=128, currsize=1)
```
`lru_cache` evicts the Least Recently Used entry when `maxsize` is reached. Use `maxsize=None` for unlimited cache (`functools.cache` in Python 3.9+). Only works for **hashable** arguments.

---

**Q109: What is `functools.wraps` doing exactly at the bytecode level?**

A: `@wraps(func)` copies `__wrapped__`, `__name__`, `__qualname__`, `__doc__`, `__dict__`, and `__module__` from `func` to `wrapper`. It also sets `wrapper.__wrapped__ = func`, making the original function accessible via introspection tools and `inspect.unwrap()`. This is why `help()`, `pydoc`, and logging show the correct function metadata even after wrapping.

---

**Q110: What is `functools.partial` and when is it useful?**

A: `partial(func, *args, **kwargs)` creates a new callable with some arguments pre-filled:
```python
from functools import partial

def filter_by_region(records, region):
    return [r for r in records if r["region"] == region]

filter_us = partial(filter_by_region, region="US")
filter_eu = partial(filter_by_region, region="EU")

data = [{"region": "US", "v": 1}, {"region": "EU", "v": 2}, {"region": "US", "v": 3}]
print(filter_us(data))   # only US records
```
Use `partial` when you repeatedly call a function with the same arguments (pipeline stage config, map functions, API call wrappers).

---

**Q111: Write a decorator that validates function arguments.**

A: Type validation decorator that checks arguments match expected types before calling:
```python
from functools import wraps

def validate_types(**expected_types):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            import inspect
            sig = inspect.signature(func)
            bound = sig.bind(*args, **kwargs)
            bound.apply_defaults()
            for param_name, expected_type in expected_types.items():
                val = bound.arguments.get(param_name)
                if val is not None and not isinstance(val, expected_type):
                    raise TypeError(f"'{param_name}' must be {expected_type.__name__}, got {type(val).__name__}")
            return func(*args, **kwargs)
        return wrapper
    return decorator

@validate_types(records=list, batch_size=int)
def process_batch(records, batch_size=100):
    return records[:batch_size]

process_batch([1, 2, 3], batch_size=2)    # OK
# process_batch("not a list", batch_size=2)  # raises TypeError
```

---

**Q112: What is the difference between a decorator and a context manager? When to use each?**

A: A **decorator** wraps a function call — adds behavior before/after every time the function runs. A **context manager** wraps a block of code — manages resource acquisition and release for a specific code block. Use decorators for cross-cutting concerns on functions (logging, retry, timing). Use context managers for resource management (file handles, DB connections, locks). They can work together — `contextlib.contextmanager` creates context managers using `yield`.

---

## functools (Q113–Q118)

---

**Q113: What is `functools.reduce()` and how does it work?**

A: `reduce(function, iterable[, initializer])` applies a function cumulatively to items, reducing the iterable to a single value:
```python
from functools import reduce

# Product of all numbers
numbers = [2, 3, 4, 5]
product = reduce(lambda acc, x: acc * x, numbers)
print(product)   # 120

# Merge list of dicts (later dicts win on key collision)
configs = [{"a": 1, "b": 2}, {"b": 3, "c": 4}, {"d": 5}]
merged = reduce(lambda acc, d: {**acc, **d}, configs)
print(merged)   # {'a': 1, 'b': 3, 'c': 4, 'd': 5}
```
Use when you need to fold a sequence into a single value with custom logic. `sum()`, `max()`, `min()`, `all()`, `any()` are all specialized reduces.

---

**Q114: What is `functools.cached_property`?**

A: `@cached_property` computes a property once and caches the result as an instance attribute. Unlike `@lru_cache`, it's per-instance (each instance has its own cache), not shared:
```python
from functools import cached_property

class DataSet:
    def __init__(self, records):
        self.records = records

    @cached_property
    def statistics(self):
        print("Computing statistics (once)...")
        values = [r["value"] for r in self.records]
        return {"count": len(values), "mean": sum(values)/len(values), "max": max(values)}

ds = DataSet([{"value": 10}, {"value": 20}, {"value": 30}])
print(ds.statistics)   # Computing... + result
print(ds.statistics)   # Only result (cached on instance as attribute)
```
Use for expensive computed properties that won't change after the object is created.

---

**Q115: What does `functools.total_ordering` do?**

A: If you define `__eq__` and one of (`__lt__`, `__le__`, `__gt__`, `__ge__`) on a class, `@total_ordering` fills in the other comparison methods automatically. Useful for sortable data records:
```python
from functools import total_ordering

@total_ordering
class PipelineRun:
    def __init__(self, duration):
        self.duration = duration

    def __eq__(self, other):
        return self.duration == other.duration

    def __lt__(self, other):
        return self.duration < other.duration

runs = [PipelineRun(300), PipelineRun(95), PipelineRun(210)]
print(sorted(runs, key=lambda r: r.duration))   # sorted by duration
print(runs[0] > runs[1])   # True (30 > 95 → False actually, 300 > 95 → True)
```

---

**Q116: What is `operator.itemgetter` and when does it outperform lambda?**

A: `operator.itemgetter(key)` extracts a dict key or tuple index — faster than `lambda x: x[key]` because it's implemented in C:
```python
from operator import itemgetter, attrgetter

records = [{"name": "c", "score": 85}, {"name": "a", "score": 92}, {"name": "b", "score": 78}]

# Both are equivalent, but itemgetter is ~20% faster for large lists
sorted_by_score = sorted(records, key=itemgetter("score"), reverse=True)
sorted_by_name = sorted(records, key=itemgetter("name"))

# attrgetter for object attributes
from collections import namedtuple
Record = namedtuple("Record", ["name", "score"])
objs = [Record("c", 85), Record("a", 92)]
sorted(objs, key=attrgetter("score"))
```
Use `itemgetter`/`attrgetter` in hot paths with large datasets — the C implementation avoids Python function call overhead on every comparison.

---

## itertools (Q117–Q125)

---

**Q117: What is `itertools.chain()` and when is it useful?**

A: `chain(*iterables)` treats multiple iterables as a single lazy sequence — no intermediate list is created:
```python
from itertools import chain

partitions = [
    [1, 2, 3],
    [4, 5],
    [6, 7, 8, 9]
]

# Flat iteration without creating a full list in memory
for record in chain(*partitions):
    print(record, end=" ")
# 1 2 3 4 5 6 7 8 9

# Equivalent to but more memory-efficient than:
flat = [x for part in partitions for x in part]
```
Use `chain()` when flattening partitions or chaining generators — the entire flattened sequence is never materialized in memory.

---

**Q118: What is `itertools.groupby()` and what is its critical requirement?**

A: `groupby(iterable, key)` groups consecutive elements with the same key value. **Critical:** the input must be sorted by the key first, or groups will be split:
```python
from itertools import groupby

records = [
    {"region": "EU", "amount": 100},
    {"region": "US", "amount": 200},
    {"region": "EU", "amount": 150},   # different group than first EU!
    {"region": "US", "amount": 300},
]

# WRONG — without sorting, EU appears in two groups
# RIGHT — sort first
sorted_records = sorted(records, key=lambda r: r["region"])

for region, group in groupby(sorted_records, key=lambda r: r["region"]):
    amounts = [r["amount"] for r in group]
    print(f"{region}: {sum(amounts)}")
# EU: 250
# US: 500
```
`itertools.groupby` is a lazy streaming groupby — it processes one group at a time. Pandas `groupby()` does not require sorting.

---

**Q119: What are `itertools.islice()` and `itertools.takewhile()`?**

A: `islice(iterable, stop)` lazily takes the first N items from any iterator (like slicing but without materializing). `takewhile(predicate, iterable)` yields items while the predicate is True:
```python
from itertools import islice, takewhile

# islice — take first 5 records from a generator (no list needed)
def infinite_event_id_generator():
    n = 0
    while True:
        yield f"event_{n}"
        n += 1

first_5 = list(islice(infinite_event_id_generator(), 5))
print(first_5)
# ['event_0', 'event_1', 'event_2', 'event_3', 'event_4']

# takewhile — process records while status is OK, stop at first failure
statuses = ["ok", "ok", "ok", "failed", "ok", "ok"]
good_runs = list(takewhile(lambda s: s == "ok", statuses))
print(good_runs)
# ['ok', 'ok', 'ok']   — stops at the first 'failed'
```

---

**Q120: What is `itertools.combinations()` vs `itertools.permutations()`?**

A: `combinations(iterable, r)` yields all unique r-element subsets (order doesn't matter). `permutations(iterable, r)` yields all ordered arrangements:
```python
from itertools import combinations, permutations

columns = ["user_id", "region", "date"]

print("Combinations (pairs):")
for pair in combinations(columns, 2):
    print(pair)
# ('user_id', 'region'), ('user_id', 'date'), ('region', 'date')

print("\nPermutations (pairs):")
for pair in permutations(columns, 2):
    print(pair)
# ('user_id', 'region'), ('user_id', 'date'),
# ('region', 'user_id'), ('region', 'date'), ...
```
Use `combinations` to generate candidate join keys or feature pairs. Use `permutations` when order matters (query plan ordering). C(n,r) combinations vs P(n,r) = n!/(n-r)! permutations.

---

**Q121: What is `itertools.product()` and when is it useful in DE?**

A: `product(*iterables)` yields the Cartesian product — all combinations across multiple iterables:
```python
from itertools import product

environments = ["dev", "staging", "prod"]
regions = ["us-east-1", "eu-west-1"]
formats = ["csv", "parquet"]

for env, region, fmt in product(environments, regions, formats):
    print(f"{env}/{region}/{fmt}")
# dev/us-east-1/csv
# dev/us-east-1/parquet
# dev/eu-west-1/csv
# ... (12 combinations total)
```
Use to generate all test cases, all partition paths, or all parameter combinations for integration tests. Equivalent to nested `for` loops.

---

**Q122: What is `itertools.accumulate()` and how does it differ from `reduce()`?**

A: `accumulate(iterable, func)` yields all intermediate results (like a running aggregation), while `reduce()` returns only the final value:
```python
from itertools import accumulate
import operator

daily_sales = [100, 200, 150, 300, 250]

# Running total (all intermediate values)
running_total = list(accumulate(daily_sales))
print("Running total:", running_total)
# [100, 300, 450, 750, 1000]

# Running maximum
running_max = list(accumulate(daily_sales, func=max))
print("Running max:", running_max)
# [100, 200, 200, 300, 300]
```
Use `accumulate` when you need the intermediate states (running metrics dashboards). Use `reduce` when you only need the final result.

---

**Q123: What is `itertools.cycle()` and `itertools.repeat()`?**

A: `cycle(iterable)` loops through the iterable infinitely. `repeat(obj, n)` yields the same object n times (or infinitely):
```python
from itertools import cycle, repeat, islice

# cycle — round-robin partition assignment
partitions = cycle(["partition_0", "partition_1", "partition_2"])
records = ["r1", "r2", "r3", "r4", "r5", "r6", "r7"]

assignments = list(zip(records, islice(partitions, len(records))))
for rec, part in assignments:
    print(f"{rec} → {part}")
# r1 → partition_0 | r2 → partition_1 | r3 → partition_2
# r4 → partition_0 | r5 → partition_1 | r6 → partition_2 | r7 → partition_0

# repeat — fill default values
defaults = list(repeat("N/A", 5))   # ['N/A', 'N/A', 'N/A', 'N/A', 'N/A']
```
`cycle` is useful for round-robin load balancing. `repeat` is useful as a fill value with `zip`.

---

**Q124: What is `itertools.zip_longest()` and when do you need it over `zip()`?**

A: `zip()` stops at the shortest iterable. `zip_longest(*iterables, fillvalue=None)` continues to the longest, filling missing values:
```python
from itertools import zip_longest

schema_v1 = ["id", "name", "region"]
schema_v2 = ["id", "name", "region", "score", "category"]

print("Regular zip (stops at shortest):")
for a, b in zip(schema_v1, schema_v2):
    print(f"  {a} | {b}")
# id | id | name | name | region | region  (score, category dropped!)

print("\nzip_longest (handles unequal lengths):")
for a, b in zip_longest(schema_v1, schema_v2, fillvalue="<missing>"):
    print(f"  {a} | {b}")
# id | id | name | name | region | region | <missing> | score | <missing> | category
```
Always use `zip_longest` when comparing schemas of different lengths. `zip` silently drops extra elements — a dangerous default in schema comparison.

---

**Q125: Combine itertools tools to build a lazy pipeline**

**Scenario:** Lazily process a large log file — filter errors, extract fields, take the first 100, enumerate them.

```python
from itertools import islice, filterfalse

# Simulated log generator
def log_stream():
    logs = [
        "2024-01-01 INFO start",
        "2024-01-01 ERROR disk full",
        "2024-01-01 WARNING slow query",
        "2024-01-01 ERROR timeout",
        "2024-01-01 INFO end",
    ]
    for log in logs * 20:   # simulate 100 log lines
        yield log

def parse_error(line):
    parts = line.split(" ", 3)
    return {"ts": parts[0], "msg": parts[3]} if parts[1] == "ERROR" else None

# Lazy pipeline: filter → parse → first 100 → enumerate
errors = (parse_error(line) for line in log_stream() if "ERROR" in line)
first_100 = islice(errors, 100)

for i, error in enumerate(first_100, start=1):
    print(f"{i:3}. {error}")
    if i >= 5:
        print("... (truncated)")
        break
```

**Explanation:** Generator expressions, `filterfalse`, and `islice` chain lazily — no lists, no loading everything into memory. Each item flows through the pipeline one at a time. This is the Python model of a streaming data pipeline.

---

> Next: [07_testing_typehints_logging.md](07_testing_typehints_logging.md)
