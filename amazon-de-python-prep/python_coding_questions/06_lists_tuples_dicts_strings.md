# 50 Coding Questions: Lists, Tuples, Dictionaries & String Manipulation

> **Level:** Easy to Moderate | **Target:** Amazon DE2
> **Coverage:** All core scenarios — data cleaning, deduplication, transformation, aggregation, parsing
> **Format per question:** Problem → Code → Explanation

---

# LISTS (Q1–Q15)

---

## Q1: Remove All Negative Numbers From a List

**Scenario:** A pipeline collected temperature sensor readings. Remove all invalid negative readings.

```python
readings = [23, -5, 18, -1, 42, 0, -99, 35]

valid = [r for r in readings if r >= 0]
print(valid)
# [23, 18, 42, 0, 35]
```

**Explanation:** List comprehension with a condition filters elements in one pass. Equivalent to a `for` + `append` loop but cleaner and faster.

---

## Q2: Find the Second Largest Number in a List

**Scenario:** Find the second highest revenue day from daily revenue figures.

```python
revenues = [1500, 3200, 2800, 3200, 1900, 2100]

unique_sorted = sorted(set(revenues), reverse=True)
second_largest = unique_sorted[1]
print("Second largest:", second_largest)
# 2800
```

**Explanation:** `set()` removes duplicates first (so two equal maximums don't fool us). `sorted(..., reverse=True)` gives descending order. Index `[1]` is the second largest.

---

## Q3: Rotate a List by N Positions

**Scenario:** A circular pipeline buffer holds the last 5 batch IDs. Rotate left by 2 positions to reorder processing.

```python
batches = [101, 102, 103, 104, 105]
n = 2

rotated = batches[n:] + batches[:n]
print(rotated)
# [103, 104, 105, 101, 102]
```

**Explanation:** Slicing from index `n` gives the tail, slicing up to `n` gives the head. Concatenating them rotates left. For right rotation: `batches[-n:] + batches[:-n]`.

---

## Q4: Split a List Into Chunks of Size N

**Scenario:** You have 1000 records and a downstream API accepts max 100 records per call. Batch them.

```python
records = list(range(1, 21))   # simulating 20 records
chunk_size = 5

chunks = [records[i:i + chunk_size] for i in range(0, len(records), chunk_size)]
for chunk in chunks:
    print(chunk)
# [1, 2, 3, 4, 5]
# [6, 7, 8, 9, 10]
# ...
```

**Explanation:** `range(0, len(records), chunk_size)` steps through starting indices. Each slice `[i:i+chunk_size]` is one batch. This is the standard micro-batching pattern.

---

## Q5: Count How Many Times Each Number Appears

**Scenario:** Count how many records belong to each partition ID.

```python
partition_ids = [1, 2, 1, 3, 2, 1, 4, 3, 2, 1]

counts = {}
for pid in partition_ids:
    counts[pid] = counts.get(pid, 0) + 1

print(counts)
# {1: 4, 2: 3, 3: 2, 4: 1}
```

**Explanation:** `dict.get(key, 0)` safely fetches current count (or 0 if absent). Adding 1 accumulates. This is manual frequency counting — the foundation of aggregation logic.

---

## Q6: Move All Zeros to the End of a List

**Scenario:** A data quality scan flagged rows with score 0. Push them to the end for separate handling.

```python
scores = [4, 0, 3, 0, 7, 0, 2, 5]

non_zeros = [s for s in scores if s != 0]
zeros = [s for s in scores if s == 0]
result = non_zeros + zeros

print(result)
# [4, 3, 7, 2, 5, 0, 0, 0]
```

**Explanation:** Two list comprehensions separate the two groups. Concatenation combines them. Order within each group is preserved.

---

## Q7: Find All Indexes of a Value in a List

**Scenario:** Find all positions in a pipeline run log where "FAILED" status appeared.

```python
statuses = ["OK", "FAILED", "OK", "OK", "FAILED", "FAILED", "OK"]

failed_indexes = [i for i, s in enumerate(statuses) if s == "FAILED"]
print(failed_indexes)
# [1, 4, 5]
```

**Explanation:** `enumerate()` gives `(index, value)` pairs. The comprehension collects indexes where the condition matches. `list.index()` only finds the first match — this finds all.

---

## Q8: Interleave Two Lists

**Scenario:** Merge two ordered event streams (from two producers) in alternating order.

```python
stream_a = ["a1", "a2", "a3"]
stream_b = ["b1", "b2", "b3"]

merged = [item for pair in zip(stream_a, stream_b) for item in pair]
print(merged)
# ['a1', 'b1', 'a2', 'b2', 'a3', 'b3']
```

**Explanation:** `zip()` pairs elements. The nested comprehension unpacks each pair in alternating order. `zip` stops at the shorter list — use `zip_longest` from `itertools` if lengths differ.

---

## Q9: Remove Consecutive Duplicates From a List

**Scenario:** A sensor emitted repeated readings. Keep only the first of each consecutive duplicate.

```python
readings = [5, 5, 7, 7, 7, 3, 5, 5, 9]

deduped = [readings[0]]
for r in readings[1:]:
    if r != deduped[-1]:
        deduped.append(r)

print(deduped)
# [5, 7, 3, 5, 9]
```

**Explanation:** Start with the first element. For each subsequent element, only add it if it differs from the last item already in the result. This preserves non-consecutive duplicates (like the final `5`).

---

## Q10: Compute the Running Total (Cumulative Sum) of a List

**Scenario:** Compute the cumulative bytes transferred in a file upload pipeline.

```python
bytes_per_second = [100, 200, 150, 300, 250]

running_totals = []
total = 0
for b in bytes_per_second:
    total += b
    running_totals.append(total)

print(running_totals)
# [100, 300, 450, 750, 1000]
```

**Explanation:** A running accumulator adds each value and records the cumulative state. This is the manual equivalent of `itertools.accumulate()` — useful in time-series pipeline metrics.

---

## Q11: Partition a List Into Two Groups Based on a Condition

**Scenario:** Split records into "high-value" (amount ≥ 500) and "low-value" groups in one pass.

```python
amounts = [120, 750, 300, 1200, 450, 600, 200]

high = []
low = []
for a in amounts:
    (high if a >= 500 else low).append(a)

print("High:", high)
# High: [750, 1200, 600]
print("Low:", low)
# Low: [120, 300, 450, 200]
```

**Explanation:** The ternary expression `(high if ... else low)` picks the correct list before calling `.append()`. Single pass, preserves order, avoids iterating the list twice.

---

## Q12: Find the Intersection of Two Lists (Preserving Order)

**Scenario:** Find records that exist in both today's batch and the reference master list. Keep today's order.

```python
todays_batch = ["r3", "r1", "r5", "r2", "r8"]
master = {"r1", "r2", "r3", "r4", "r6"}   # use set for O(1) lookup

matched = [r for r in todays_batch if r in master]
print(matched)
# ['r3', 'r1', 'r2']
```

**Explanation:** The reference list is converted to a `set` for O(1) membership checks. The comprehension preserves the order of `todays_batch`. Using a list for `master` would be O(n²).

---

## Q13: Transpose a Matrix (List of Lists)

**Scenario:** A dataset arrives row-by-row. Transpose it to access it column-by-column.

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

transposed = [list(row) for row in zip(*matrix)]
print(transposed)
# [[1, 4, 7], [2, 5, 8], [3, 6, 9]]
```

**Explanation:** `zip(*matrix)` unpacks the rows and zips their columns together. `list()` converts each zip tuple to a list. This is how you "rotate" a table — rows become columns.

---

## Q14: Sort a List of Tuples by the Second Element

**Scenario:** Sort `(job_name, duration_seconds)` pairs by duration to find slowest jobs.

```python
jobs = [("ingest", 300), ("transform", 95), ("load", 210), ("validate", 60)]

sorted_jobs = sorted(jobs, key=lambda x: x[1], reverse=True)
for name, dur in sorted_jobs:
    print(f"{name}: {dur}s")
# ingest: 300s
# load: 210s
# transform: 95s
# validate: 60s
```

**Explanation:** `key=lambda x: x[1]` tells `sorted()` to compare by the second element of each tuple. `reverse=True` gives descending (slowest first).

---

## Q15: Detect All Duplicates in a List

**Scenario:** Find which record IDs appear more than once in a batch (potential double-processing).

```python
record_ids = ["r1", "r2", "r3", "r1", "r4", "r2", "r2"]

from collections import Counter

counts = Counter(record_ids)
duplicates = [rid for rid, cnt in counts.items() if cnt > 1]

print("Duplicate IDs:", duplicates)
# ['r1', 'r2']
```

**Explanation:** `Counter` efficiently counts occurrences. The comprehension filters for keys with count > 1. This is an O(n) duplicate detection — faster than nested loop O(n²) approaches.

---

# TUPLES (Q16–Q25)

---

## Q16: Unpack a Tuple Into Named Variables

**Scenario:** A pipeline returns a result tuple `(status, rows_processed, elapsed_time)`. Unpack clearly.

```python
result = ("success", 48500, 12.3)

status, rows, elapsed = result

print(f"Status: {status}")
print(f"Rows:   {rows:,}")
print(f"Time:   {elapsed}s")
# Status: success
# Rows:   48,500
# Time:   12.3s
```

**Explanation:** Tuple unpacking assigns each element to a named variable. Much clearer than `result[0]`, `result[1]` etc. The number of variables must match the tuple length exactly.

---

## Q17: Use a Tuple as a Dictionary Key

**Scenario:** Cache results keyed by `(region, date)` — a composite key that tuples enable.

```python
cache = {}

def get_aggregated(region, date, data):
    key = (region, date)
    if key not in cache:
        # compute and store
        cache[key] = sum(r["amount"] for r in data if r["region"] == region)
    return cache[key]

data = [
    {"region": "US", "amount": 100},
    {"region": "EU", "amount": 200},
    {"region": "US", "amount": 300},
]

print(get_aggregated("US", "2024-01-01", data))  # 400
print(get_aggregated("US", "2024-01-01", data))  # 400 (from cache)
```

**Explanation:** Tuples are hashable (unlike lists), so they can be dict keys. Composite keys are a standard pattern for multi-dimensional lookups and caching.

---

## Q18: Convert a List of Lists Into a List of Tuples

**Scenario:** Data comes in as lists from a CSV parser. Convert to immutable tuples for use as dict keys.

```python
rows = [["u1", "NY", 500], ["u2", "LA", 200], ["u3", "NY", 800]]

tuples = [tuple(row) for row in rows]
print(tuples)
# [('u1', 'NY', 500), ('u2', 'LA', 200), ('u3', 'NY', 800)]

# Now usable as dict keys
lookup = {t: True for t in tuples}
```

**Explanation:** `tuple(list)` converts a list to a tuple. Tuples are immutable so Python allows them as dict keys and set members. Lists cannot be used as keys — you'll get `TypeError: unhashable type: 'list'`.

---

## Q19: Use `*` to Capture Remaining Elements (Extended Unpacking)

**Scenario:** A log record has a fixed header `(timestamp, level)` followed by a variable-length message. Unpack safely.

```python
log = ("2024-01-15", "ERROR", "disk", "full", "on", "/dev/sdb")

timestamp, level, *message_parts = log

print("Time:", timestamp)
print("Level:", level)
print("Message:", " ".join(message_parts))
# Time: 2024-01-15
# Level: ERROR
# Message: disk full on /dev/sdb
```

**Explanation:** `*variable` in unpacking captures all remaining elements into a list. This is called extended unpacking (Python 3+). It's flexible when you know the prefix structure but not the tail length.

---

## Q20: Return Multiple Values From a Function Using a Tuple

**Scenario:** A validation function should return both the status and a message.

```python
def validate_file(filepath, min_size_kb):
    import os
    if not os.path.exists(filepath):
        return False, f"File not found: {filepath}"
    size_kb = os.path.getsize(filepath) / 1024
    if size_kb < min_size_kb:
        return False, f"File too small: {size_kb:.1f}KB < {min_size_kb}KB"
    return True, f"Valid ({size_kb:.1f}KB)"

ok, msg = validate_file("data.csv", 10)
print(ok, msg)
```

**Explanation:** Python functions can return multiple values via implicit tuple packing. The caller unpacks them into named variables. This avoids returning a dict for simple multi-value returns.

---

## Q21: Count Occurrences of an Element in a Tuple

**Scenario:** A pipeline config is stored as a tuple of stage names. Count how many times "retry" appears.

```python
stages = ("extract", "validate", "retry", "transform", "retry", "load", "retry")

retry_count = stages.count("retry")
print("Retries:", retry_count)
# Retries: 3
```

**Explanation:** `tuple.count(value)` returns how many times `value` appears. It scans the entire tuple — O(n). Tuples also have `.index(value)` to find the first occurrence.

---

## Q22: Find the Index of an Element in a Tuple

**Scenario:** Given a tuple of pipeline stage names, find the position of "load".

```python
pipeline = ("extract", "clean", "validate", "transform", "load", "report")

load_pos = pipeline.index("load")
print("Load starts at step:", load_pos)
# Load starts at step: 4
```

**Explanation:** `tuple.index(value)` returns the zero-based index of the first match. Raises `ValueError` if not found — wrap in `try/except` for safe lookups.

---

## Q23: Zip Two Lists Into a List of Tuples (Key-Value Pairs)

**Scenario:** You have separate lists of column names and row values. Combine them.

```python
headers = ["user_id", "region", "revenue"]
values = ["u42", "US", 1500]

record = list(zip(headers, values))
print(record)
# [('user_id', 'u42'), ('region', 'US'), ('revenue', 1500)]

# Convert directly to dict
record_dict = dict(zip(headers, values))
print(record_dict)
# {'user_id': 'u42', 'region': 'US', 'revenue': 1500}
```

**Explanation:** `zip()` produces tuples of paired elements. `dict(zip(keys, vals))` is the standard pattern for reconstructing a record from separate header and value arrays.

---

## Q24: Named Tuple as a Lightweight Row Record

**Scenario:** Represent a pipeline run record with named fields without defining a full class.

```python
from collections import namedtuple

PipelineRun = namedtuple("PipelineRun", ["job_name", "status", "rows", "duration_sec"])

run = PipelineRun(job_name="ingest", status="success", rows=50000, duration_sec=12.4)

print(run.job_name)    # ingest
print(run.status)      # success
print(run[2])          # 50000 — also accessible by index
print(run._asdict())   # convert to OrderedDict
```

**Explanation:** `namedtuple` creates a tuple subclass with named fields. Immutable like a tuple, readable like a class. Great for representing fixed-schema rows in pipelines without the overhead of a full class.

---

## Q25: Sort a List of Named Tuples by a Field

**Scenario:** Sort pipeline run records by `duration_sec` descending.

```python
from collections import namedtuple

PipelineRun = namedtuple("PipelineRun", ["job", "duration_sec"])

runs = [
    PipelineRun("ingest", 300),
    PipelineRun("transform", 95),
    PipelineRun("load", 210),
]

sorted_runs = sorted(runs, key=lambda r: r.duration_sec, reverse=True)
for r in sorted_runs:
    print(f"{r.job}: {r.duration_sec}s")
# ingest: 300s
# load: 210s
# transform: 95s
```

**Explanation:** `namedtuple` fields are accessible as attributes (`.duration_sec`) or by index. `lambda r: r.duration_sec` extracts the sort key. `operator.attrgetter("duration_sec")` is a cleaner alternative.

---

# DICTIONARIES (Q26–Q40)

---

## Q26: Safely Access a Nested Dictionary Key

**Scenario:** A JSON config has deeply nested keys that may or may not exist. Access safely.

```python
config = {
    "pipeline": {
        "source": {
            "type": "s3",
            "bucket": "my-data-lake"
        }
    }
}

# Safe access with .get()
bucket = config.get("pipeline", {}).get("source", {}).get("bucket", "default-bucket")
print("Bucket:", bucket)
# Bucket: my-data-lake

# Missing key falls back to default
region = config.get("pipeline", {}).get("source", {}).get("region", "us-east-1")
print("Region:", region)
# Region: us-east-1
```

**Explanation:** Chaining `.get(key, {})` at each level returns an empty dict instead of raising `KeyError` when a level is missing. The final `.get()` provides the default for the leaf value.

---

## Q27: Count Null/None Values Per Key in a List of Dicts

**Scenario:** Data quality check — count how many records have `None` for each field.

```python
records = [
    {"name": "Alice", "age": 30, "region": "US"},
    {"name": "Bob",   "age": None, "region": "EU"},
    {"name": None,    "age": 25, "region": None},
    {"name": "Carol", "age": None, "region": "APAC"},
]

null_counts = {}
for rec in records:
    for key, val in rec.items():
        if val is None:
            null_counts[key] = null_counts.get(key, 0) + 1

print(null_counts)
# {'age': 2, 'region': 1, 'name': 1}
```

**Explanation:** Double loop: outer over records, inner over fields. `is None` is the correct null check. This gives a column-level null summary — a fundamental data quality metric.

---

## Q28: Build a Lookup Dictionary From a List

**Scenario:** Given a list of user dicts, build a `user_id → user` lookup for O(1) access.

```python
users = [
    {"user_id": "u1", "name": "Alice", "region": "US"},
    {"user_id": "u2", "name": "Bob", "region": "EU"},
    {"user_id": "u3", "name": "Carol", "region": "APAC"},
]

lookup = {u["user_id"]: u for u in users}

# O(1) access
print(lookup["u2"]["name"])   # Bob
print(lookup.get("u99"))      # None (no KeyError)
```

**Explanation:** Dict comprehension with `user_id` as key. Converting a list to a lookup dict transforms O(n) searches into O(1) lookups — critical for join-like operations in Python pipelines.

---

## Q29: Flatten a Nested Dictionary to Dot-Notation Keys

**Scenario:** An API response has nested JSON. Flatten it to single-level for loading into a relational table.

```python
def flatten_dict(d, parent_key="", sep="."):
    flat = {}
    for k, v in d.items():
        new_key = f"{parent_key}{sep}{k}" if parent_key else k
        if isinstance(v, dict):
            flat.update(flatten_dict(v, new_key, sep))
        else:
            flat[new_key] = v
    return flat

nested = {
    "user": {"id": 1, "name": "Alice"},
    "event": {"type": "click", "timestamp": "2024-01-01"}
}

print(flatten_dict(nested))
# {'user.id': 1, 'user.name': 'Alice', 'event.type': 'click', 'event.timestamp': '2024-01-01'}
```

**Explanation:** Recursion handles arbitrary nesting depth. `isinstance(v, dict)` detects nested dicts. `parent_key.key` builds dot-notation path names. This is a core pattern for normalizing semi-structured data.

---

## Q30: Compute Aggregates Per Group From a List of Dicts

**Scenario:** Given sales transactions, compute `count`, `total`, and `average` amount per region.

```python
transactions = [
    {"region": "US", "amount": 500},
    {"region": "EU", "amount": 200},
    {"region": "US", "amount": 300},
    {"region": "APAC", "amount": 700},
    {"region": "EU", "amount": 100},
]

agg = {}
for txn in transactions:
    r = txn["region"]
    if r not in agg:
        agg[r] = {"count": 0, "total": 0}
    agg[r]["count"] += 1
    agg[r]["total"] += txn["amount"]

for region, stats in sorted(agg.items()):
    avg = stats["total"] / stats["count"]
    print(f"{region}: count={stats['count']}, total={stats['total']}, avg={avg:.0f}")
# APAC: count=1, total=700, avg=700
# EU:   count=2, total=300, avg=150
# US:   count=2, total=800, avg=400
```

**Explanation:** A nested dict accumulates multiple aggregates per group. Compute `avg` only at output time (divide by count). This is the Python equivalent of SQL `COUNT(*), SUM(), AVG() GROUP BY region`.

---

## Q31: Remove Keys From a Dictionary

**Scenario:** A schema has PII fields (`ssn`, `email`) that must be stripped before the record is written to the data lake.

```python
record = {
    "user_id": "u1",
    "name": "Alice",
    "email": "alice@example.com",
    "ssn": "123-45-6789",
    "region": "US"
}

pii_keys = {"email", "ssn"}

clean_record = {k: v for k, v in record.items() if k not in pii_keys}
print(clean_record)
# {'user_id': 'u1', 'name': 'Alice', 'region': 'US'}
```

**Explanation:** Dict comprehension with exclusion condition. Using a `set` for `pii_keys` gives O(1) membership checks. This pattern is essential for data masking and compliance in DE pipelines.

---

## Q32: Find Keys Present in Dict A but Not in Dict B (Schema Diff)

**Scenario:** Compare today's schema against yesterday's. Find new and removed columns.

```python
yesterday = {"id", "name", "region", "amount"}
today = {"id", "name", "region", "score", "category"}

new_columns = today - yesterday
removed_columns = yesterday - today

print("New columns:    ", new_columns)
# {'score', 'category'}
print("Removed columns:", removed_columns)
# {'amount'}
print("Unchanged:      ", today & yesterday)
# {'id', 'name', 'region'}
```

**Explanation:** `dict.keys()` returns a set-like view. Set operations (`-`, `&`, `|`) compare schemas directly. Schema drift detection is a real production DE concern.

---

## Q33: Sort a Dictionary by Its Values

**Scenario:** Display a `column → null_count` dict sorted by highest null count first.

```python
null_counts = {"name": 3, "age": 15, "region": 1, "email": 8}

sorted_by_nulls = dict(sorted(null_counts.items(), key=lambda item: item[1], reverse=True))
print(sorted_by_nulls)
# {'age': 15, 'email': 8, 'name': 3, 'region': 1}
```

**Explanation:** `dict.items()` returns `(key, value)` pairs. `sorted()` with `key=lambda item: item[1]` sorts by value. Wrapping back in `dict()` reconstructs the ordered dict.

---

## Q34: Transpose a List of Dicts Into a Dict of Lists (Column Store Format)

**Scenario:** Convert row-oriented records to columnar format for analytical processing.

```python
rows = [
    {"name": "Alice", "score": 85, "region": "US"},
    {"name": "Bob",   "score": 92, "region": "EU"},
    {"name": "Carol", "score": 78, "region": "US"},
]

columns = {}
for row in rows:
    for key, value in row.items():
        columns.setdefault(key, []).append(value)

print(columns)
# {'name': ['Alice', 'Bob', 'Carol'], 'score': [85, 92, 78], 'region': ['US', 'EU', 'US']}
```

**Explanation:** `dict.setdefault(key, [])` returns the existing list or creates a new one. This "column store" format is how columnar formats (Parquet, Arrow) organize data — each column is a contiguous array.

---

## Q35: Use `ChainMap` to Layer Configurations

**Scenario:** A pipeline has default config, environment overrides, and job-level overrides. Layer them with precedence.

```python
from collections import ChainMap

defaults = {"timeout": 30, "retries": 3, "format": "csv", "batch_size": 1000}
env_overrides = {"timeout": 60, "format": "parquet"}
job_overrides = {"batch_size": 500}

# Job > Env > Defaults  (first dict wins)
config = ChainMap(job_overrides, env_overrides, defaults)

print(config["timeout"])      # 60   (env wins over default)
print(config["batch_size"])   # 500  (job wins over default)
print(config["retries"])      # 3    (only in defaults)
print(config["format"])       # parquet (env wins)
```

**Explanation:** `ChainMap` stacks dicts in priority order — first dict wins on key conflicts. No data is copied. Changes to the first map are visible immediately. Perfect for layered config systems.

---

## Q36: Convert a Flat Dict to a Nested Dict Using Dot-Notation Keys

**Scenario:** Reconstruct a nested JSON structure from a flat column-store format.

```python
def unflatten_dict(flat, sep="."):
    result = {}
    for key, value in flat.items():
        parts = key.split(sep)
        d = result
        for part in parts[:-1]:
            d = d.setdefault(part, {})
        d[parts[-1]] = value
    return result

flat = {"user.id": 1, "user.name": "Alice", "event.type": "click"}

print(unflatten_dict(flat))
# {'user': {'id': 1, 'name': 'Alice'}, 'event': {'type': 'click'}}
```

**Explanation:** Split each dot-notation key into parts. Navigate (or create) nested dicts using `setdefault`. Assign the value at the leaf. This is the inverse of Q29's flatten operation.

---

## Q37: Merge and Deduplicate a List of Dicts by a Key

**Scenario:** Two data sources returned overlapping customer records. Merge, keeping the latest version (second source wins).

```python
source_a = [
    {"id": 1, "name": "Alice", "score": 80},
    {"id": 2, "name": "Bob",   "score": 70},
]
source_b = [
    {"id": 2, "name": "Bob",   "score": 95},  # updated score
    {"id": 3, "name": "Carol", "score": 88},
]

merged = {}
for rec in source_a + source_b:
    merged[rec["id"]] = rec   # later records overwrite earlier ones

result = list(merged.values())
for r in sorted(result, key=lambda x: x["id"]):
    print(r)
# {'id': 1, 'name': 'Alice', 'score': 80}
# {'id': 2, 'name': 'Bob', 'score': 95}  ← updated
# {'id': 3, 'name': 'Carol', 'score': 88}
```

**Explanation:** Iterating `source_a + source_b` means source_b records overwrite source_a records for the same ID. The dict keyed on `id` is the dedup structure — this is upsert logic in pure Python.

---

## Q38: Build an Inverted Index From Documents

**Scenario:** Build a word → list of document IDs index for a small search feature.

```python
documents = {
    "doc1": "pipeline etl extract load",
    "doc2": "extract transform data",
    "doc3": "data pipeline orchestration",
}

index = {}
for doc_id, text in documents.items():
    for word in text.split():
        index.setdefault(word, []).append(doc_id)

print(index["pipeline"])   # ['doc1', 'doc3']
print(index["extract"])    # ['doc1', 'doc2']
print(index["data"])       # ['doc2', 'doc3']
```

**Explanation:** Loop over documents and words, adding `doc_id` to each word's list. `setdefault()` initializes the list if needed. Inverted indexes are the foundation of search engines and text-based lookups.

---

## Q39: Identify Rows Where Any Field Has Changed (Change Data Capture)

**Scenario:** Compare a new record against the previous version. Report which fields changed.

```python
def get_changes(old, new):
    changes = {}
    all_keys = set(old) | set(new)
    for key in all_keys:
        old_val = old.get(key)
        new_val = new.get(key)
        if old_val != new_val:
            changes[key] = {"old": old_val, "new": new_val}
    return changes

old_record = {"id": 1, "name": "Alice", "score": 80, "region": "US"}
new_record = {"id": 1, "name": "Alice", "score": 95, "status": "active"}

diff = get_changes(old_record, new_record)
for field, change in diff.items():
    print(f"{field}: {change['old']} → {change['new']}")
# score: 80 → 95
# region: US → None
# status: None → active
```

**Explanation:** Union of all keys ensures new and removed fields are both detected. `dict.get(key)` returns `None` for missing keys — treating absence as `None` makes the comparison uniform. This is CDC (Change Data Capture) logic in pure Python.

---

## Q40: Pivot a List of Records Into a Dict of Column Arrays

**Scenario:** Group `(date, metric, value)` records into a pivot table: dates as rows, metrics as columns.

```python
raw = [
    {"date": "2024-01", "metric": "revenue", "value": 5000},
    {"date": "2024-01", "metric": "users",   "value": 200},
    {"date": "2024-02", "metric": "revenue", "value": 6200},
    {"date": "2024-02", "metric": "users",   "value": 240},
]

pivot = {}
for row in raw:
    d = row["date"]
    if d not in pivot:
        pivot[d] = {}
    pivot[d][row["metric"]] = row["value"]

for date, metrics in sorted(pivot.items()):
    print(date, metrics)
# 2024-01 {'revenue': 5000, 'users': 200}
# 2024-02 {'revenue': 6200, 'users': 240}
```

**Explanation:** A nested dict with `date → metric → value` structure pivots the EAV (Entity-Attribute-Value) format into a wide table. This mirrors SQL PIVOT operations.

---

# STRING MANIPULATION (Q41–Q50)

---

## Q41: Extract the Filename and Extension From a File Path

**Scenario:** Parse S3 object keys to get just the filename and determine the format.

```python
path = "s3://my-data-lake/raw/2024/01/events_20240115.parquet"

filename = path.split("/")[-1]          # 'events_20240115.parquet'
name, ext = filename.rsplit(".", 1)     # split from right, once

print("Filename:", filename)
print("Name:", name)
print("Extension:", ext)
# Filename: events_20240115.parquet
# Name: events_20240115
# Extension: parquet
```

**Explanation:** `.split("/")[-1]` gets the last path segment. `.rsplit(".", 1)` splits on the last dot only — correct for `archive.tar.gz` (ext = `gz`). `.split(".")[-1]` also works but `.rsplit` is more explicit.

---

## Q42: Validate That a String Is a Valid Integer

**Scenario:** CSV values are always strings. Validate before casting to avoid `ValueError`.

```python
def is_valid_integer(s):
    s = s.strip()
    if not s:
        return False
    return s.lstrip("-").isdigit()

test_values = ["42", "-7", "3.14", "abc", "", "  100  "]

for val in test_values:
    print(f"{repr(val):12} → {is_valid_integer(val)}")
# '42'         → True
# '-7'         → True
# '3.14'       → False
# 'abc'        → False
# ''           → False
# '  100  '    → True
```

**Explanation:** `.lstrip("-")` removes a leading minus (for negative numbers) before calling `.isdigit()`. `.strip()` handles whitespace. Always validate before `int()` to avoid crashes on dirty data.

---

## Q43: Replace Multiple Patterns in a String

**Scenario:** Normalize a log timestamp by replacing both `/` and `T` separators to produce a consistent format.

```python
raw_timestamp = "2024/01/15T10:30:00"

normalized = raw_timestamp.replace("/", "-").replace("T", " ")
print(normalized)
# 2024-01-15 10:30:00
```

**Explanation:** Multiple `.replace()` calls chain neatly. Each creates a new string — Python strings are immutable. For many replacements, `str.translate()` or `re.sub()` with a mapping is more efficient.

---

## Q44: Extract All Numbers From a String Using Regex

**Scenario:** A log line contains embedded metrics. Extract all numeric values.

```python
import re

log_line = "Processed 1500 records in 12.5 seconds, 3 failures detected"

numbers = re.findall(r'\d+\.?\d*', log_line)
print(numbers)
# ['1500', '12.5', '3']

# Cast to appropriate types
values = [float(n) if '.' in n else int(n) for n in numbers]
print(values)
# [1500, 12.5, 3]
```

**Explanation:** `\d+\.?\d*` matches integers and decimals. `re.findall()` returns all non-overlapping matches as strings. The comprehension casts based on whether a decimal point is present.

---

## Q45: Pad a String to Fixed Width (Column Alignment)

**Scenario:** Format a pipeline summary report with aligned columns.

```python
jobs = [("ingest", 48500, "success"), ("transform", 1200, "failed"), ("load", 48500, "success")]

print(f"{'Job':<15} {'Rows':>10} {'Status':<10}")
print("-" * 35)
for job, rows, status in jobs:
    print(f"{job:<15} {rows:>10,} {status:<10}")

# Job              Rows   Status
# -----------------------------------
# ingest          48,500 success
# transform        1,200 failed
# load            48,500 success
```

**Explanation:** `:<15` pads right with spaces to width 15 (left-align). `:>10` pads left (right-align). `:,` adds thousands separators. f-string format specs control precise column alignment.

---

## Q46: Check if a String Contains Only Allowed Characters

**Scenario:** Validate that a table name contains only `a-z`, digits, and underscores (no special chars).

```python
import re

def is_valid_table_name(name):
    return bool(re.fullmatch(r'[a-z][a-z0-9_]*', name))

names = ["sales_2024", "Sales", "2024_data", "my-table", "orders_v2", ""]

for name in names:
    print(f"{name!r:20} → {is_valid_table_name(name)}")
# 'sales_2024'         → True
# 'Sales'              → False  (uppercase)
# '2024_data'          → False  (starts with digit)
# 'my-table'           → False  (dash not allowed)
# 'orders_v2'          → True
```

**Explanation:** `re.fullmatch()` requires the pattern to match the whole string (not just a part). `[a-z][a-z0-9_]*` means: start with a lowercase letter, followed by any number of lowercase letters, digits, or underscores.

---

## Q47: Truncate Long Strings With Ellipsis

**Scenario:** Display column values in a report. Truncate any value longer than 30 characters to keep rows tidy.

```python
def truncate(text, max_len=30, suffix="..."):
    text = str(text)
    if len(text) <= max_len:
        return text
    return text[:max_len - len(suffix)] + suffix

values = [
    "Short value",
    "This is a really long description that exceeds the limit",
    "Exactly thirty characters here",
]

for v in values:
    print(repr(truncate(v, 30)))
# 'Short value'
# 'This is a really long descript...'
# 'Exactly thirty characters here'
```

**Explanation:** Subtract the suffix length from `max_len` before slicing, so the total output never exceeds `max_len`. `str(text)` handles non-string inputs safely.

---

## Q48: Parse a Key=Value Config String

**Scenario:** A pipeline receives inline config as a string: `"host=localhost;port=5432;db=analytics"`. Parse it into a dict.

```python
config_str = "host=localhost;port=5432;db=analytics;timeout=30"

config = {}
for pair in config_str.split(";"):
    if "=" in pair:
        key, value = pair.split("=", 1)   # maxsplit=1 to protect values with '='
        config[key.strip()] = value.strip()

print(config)
# {'host': 'localhost', 'port': '5432', 'db': 'analytics', 'timeout': '30'}
```

**Explanation:** Split on `;` to get pairs, then split on `=` with `maxsplit=1` to protect values containing `=` (e.g., base64 encoded strings). The check `if "=" in pair` skips empty/malformed entries.

---

## Q49: Count Words, Sentences, and Unique Words in a Text Column

**Scenario:** Basic NLP-style profiling of a `description` column in your dataset.

```python
text = "The pipeline failed. The pipeline retried. Retry succeeded after retry."

words = text.lower().split()
word_count = len(words)
sentence_count = text.count(".")
unique_words = len(set(words))

print(f"Words:        {word_count}")
print(f"Sentences:    {sentence_count}")
print(f"Unique words: {unique_words}")
# Words:        11
# Sentences:    3
# Unique words: 7
```

**Explanation:** `.lower().split()` normalizes and tokenizes. `text.count(".")` is a quick sentence estimate (imprecise for real NLP but fine for profiling). `set()` on words gives unique count.

---

## Q50: Build a CSV Line From a Dictionary

**Scenario:** Given a record dict, generate a properly escaped CSV line to write to a file.

```python
import csv
import io

def dict_to_csv_line(record, headers):
    output = io.StringIO()
    writer = csv.DictWriter(output, fieldnames=headers)
    writer.writerow(record)
    return output.getvalue().strip()

headers = ["user_id", "name", "region", "amount"]
record = {"user_id": "u42", "name": "Alice, Jr.", "region": "US", "amount": 1500}

line = dict_to_csv_line(record, headers)
print(line)
# u42,"Alice, Jr.",US,1500
```

**Explanation:** Using `csv.DictWriter` with `io.StringIO` correctly handles values containing commas, quotes, and newlines — something manual `",".join()` cannot do. Always use the `csv` module for CSV writing.

---

> **Total: 50 Questions**
>
> | Topic        | Questions | Count |
> |--------------|-----------|-------|
> | Lists        | Q1–Q15    | 15    |
> | Tuples       | Q16–Q25   | 10    |
> | Dictionaries | Q26–Q40   | 15    |
> | Strings      | Q41–Q50   | 10    |
>
> Return to: [01_basics.md](01_basics.md) | [04_data_engineering_patterns.md](04_data_engineering_patterns.md)
