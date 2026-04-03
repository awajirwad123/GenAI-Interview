# Coding Questions: Advanced Basics (Q43–Q50)

> **Difficulty:** 20% Amazon-level  
> Topics: Generators, decorators basics, OOP for DE, error handling in pipelines, context managers, itertools

---

## Q43: Generator That Produces Fibonacci Numbers

**Question:** Write a generator that yields Fibonacci numbers up to a given limit. Use it to simulate event IDs.

```python
def fibonacci(limit):
    a, b = 0, 1
    while a <= limit:
        yield a
        a, b = b, a + b

for num in fibonacci(50):
    print(num, end=" ")
# 0 1 1 2 3 5 8 13 21 34
```

**Explanation:**
A generator function uses `yield` instead of `return`. Each call to the generator resumes from where it left off. Values are produced lazily — no list is built in memory. Perfect for large sequences.

---

## Q44: Generator to Read Large Files in Chunks

**Question:** A 10GB log file cannot be loaded into memory. Write a generator that yields `chunk_size` lines at a time.

```python
def read_in_chunks(filepath, chunk_size=100):
    """Yield lists of `chunk_size` lines from a file."""
    chunk = []
    with open(filepath, "r") as f:
        for line in f:
            chunk.append(line.strip())
            if len(chunk) == chunk_size:
                yield chunk
                chunk = []
    if chunk:
        yield chunk   # yield the last partial chunk

# Usage
for batch in read_in_chunks("big_log.txt", chunk_size=500):
    print(f"Processing batch of {len(batch)} lines")
    # process batch here
```

**Explanation:**
Accumulate lines into a chunk. When full, yield and reset. After the loop, yield any remaining lines. This is micro-batching — a core pattern in streaming and large-scale ETL.

---

## Q45: Write a Decorator to Time Any Function

**Question:** Write a decorator `@timer` that prints how long any function takes to run.

```python
import time

def timer(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} ran in {end - start:.4f} seconds")
        return result
    return wrapper

@timer
def process_records(records):
    total = sum(r["amount"] for r in records)
    return total

data = [{"amount": i} for i in range(1, 100001)]
total = process_records(data)
print("Total:", total)
```

**Explanation:**
A decorator wraps a function: `timer` returns `wrapper`, which runs the original function and captures its return value. `*args, **kwargs` passes through all arguments. This is the standard performance monitoring pattern.

---

## Q46: Use Try/Except/Finally in a Pipeline Step

**Question:** Write a function that reads a JSON file for a partition. Handle `FileNotFoundError` and invalid JSON gracefully.

```python
import json

def load_partition(filepath):
    try:
        with open(filepath, "r") as f:
            data = json.load(f)
        print(f"Loaded {len(data)} records from {filepath}")
        return data
    except FileNotFoundError:
        print(f"ERROR: File not found — {filepath}")
        return []
    except json.JSONDecodeError as e:
        print(f"ERROR: Invalid JSON in {filepath} — {e}")
        return []
    finally:
        print(f"Attempted to load: {filepath}")

result = load_partition("partition_001.json")
```

**Explanation:**
`except` catches specific errors. Catching specific exceptions (not bare `except:`) is best practice. `finally` always runs — use it for cleanup or logging. Returning `[]` allows the pipeline to continue without crashing.

---

## Q47: Raise Custom Exceptions for Pipeline Validation

**Question:** Create a custom exception `PipelineValidationError` and raise it when row count drops below a threshold.

```python
class PipelineValidationError(Exception):
    """Raised when pipeline data does not meet quality standards."""
    pass

def validate_pipeline_output(rows_loaded, min_expected):
    if rows_loaded < min_expected:
        raise PipelineValidationError(
            f"Only {rows_loaded} rows loaded — expected at least {min_expected}"
        )
    print(f"Validation passed: {rows_loaded} rows loaded")

try:
    validate_pipeline_output(800, 1000)
except PipelineValidationError as e:
    print(f"PIPELINE FAILED: {e}")
```

**Explanation:**
Custom exceptions let you create meaningful error types. Inheriting from `Exception` is the standard approach. This makes it easy to catch pipeline errors separately from Python errors.

---

## Q48: Build a Simple Pipeline Class With OOP

**Question:** Model a data pipeline as a class with `extract`, `transform`, and `load` methods.

```python
class DataPipeline:
    def __init__(self, name):
        self.name = name
        self.records = []

    def extract(self, source_data):
        self.records = source_data
        print(f"[{self.name}] Extracted {len(self.records)} records")

    def transform(self):
        self.records = [
            {**rec, "name": rec["name"].strip().upper()}
            for rec in self.records
            if rec.get("active", True)
        ]
        print(f"[{self.name}] Transformed: {len(self.records)} records remain")

    def load(self, sink):
        sink.extend(self.records)
        print(f"[{self.name}] Loaded {len(self.records)} records to sink")

# Run the pipeline
raw = [
    {"name": "  alice ", "active": True},
    {"name": "bob", "active": False},
    {"name": "carol  ", "active": True},
]

pipeline = DataPipeline("CustomerPipeline")
pipeline.extract(raw)
pipeline.transform()
destination = []
pipeline.load(destination)
print(destination)
```

**Explanation:**
Classes let you group related pipeline logic and maintain state (`self.records`) between steps. `{**rec, "name": ...}` creates a shallow copy of the dict with one key overridden — a clean immutable-style transform.

---

## Q49: Use `enumerate` and `zip` in Pipeline Processing

**Question:**
1. Use `enumerate` to print record numbers while processing
2. Use `zip` to pair headers with values when receiving raw CSV rows

```python
records = ["Alice,Engineering", "Bob,Marketing", "Carol,Engineering"]

# enumerate — add position numbers
print("=== enumerate ===")
for idx, record in enumerate(records, start=1):
    print(f"Record {idx}: {record}")

# zip — pair headers with values
print("\n=== zip ===")
headers = ["name", "department"]
for record in records:
    values = record.split(",")
    row_dict = dict(zip(headers, values))
    print(row_dict)
# {'name': 'Alice', 'department': 'Engineering'}
# {'name': 'Bob', 'department': 'Marketing'}
```

**Explanation:**
`enumerate(iterable, start=1)` yields `(index, item)` pairs. `zip(a, b)` pairs elements from two iterables. `dict(zip(headers, values))` recreates a dict from raw field values — common when parsing non-standard delimited files.

---

## Q50: Write a Context Manager for Pipeline Logging

**Question:** Create a context manager class `PipelineLogger` that logs start and end of any pipeline block, with elapsed time.

```python
import time

class PipelineLogger:
    def __init__(self, stage_name):
        self.stage_name = stage_name
        self.start_time = None

    def __enter__(self):
        self.start_time = time.time()
        print(f"[START] {self.stage_name}")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        elapsed = time.time() - self.start_time
        if exc_type:
            print(f"[FAILED] {self.stage_name} after {elapsed:.2f}s — {exc_val}")
        else:
            print(f"[DONE] {self.stage_name} completed in {elapsed:.2f}s")
        return False   # don't suppress exceptions

# Usage
with PipelineLogger("Transform Stage"):
    import time
    time.sleep(0.5)   # simulate work
    records = [i * 2 for i in range(1000)]

# [START] Transform Stage
# [DONE] Transform Stage completed in 0.50s
```

**Explanation:**
A context manager uses `__enter__` (setup) and `__exit__` (teardown). `with` guarantees `__exit__` runs even if an exception occurs. `exc_type` is `None` when no exception happened. `return False` means re-raise any exception. This is the pattern behind `with open(...)`.

---

> You've completed all 50 coding questions!
> 
> Now head to the **FAQ section** for conceptual interview preparation: [../python_faq/01_core_python.md](../python_faq/01_core_python.md)
