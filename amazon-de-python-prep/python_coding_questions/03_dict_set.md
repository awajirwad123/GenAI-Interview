# Coding Questions: Dictionaries & Sets (Q21–Q30)

> **Difficulty:** 40% Intermediate | Topics: dict CRUD, set operations, grouping, deduplication, counting

---

## Q21: Count Event Types From a Log List

**Question:** Given a list of event strings, count how many times each event type appears.

```python
events = ["click", "view", "click", "purchase", "view", "click", "view", "view"]

counts = {}
for event in events:
    counts[event] = counts.get(event, 0) + 1

print(counts)
# {'click': 3, 'view': 4, 'purchase': 1}
```

**Explanation:**
`dict.get(key, default)` returns the current count (or 0 if missing), then we add 1 and store back. This is the manual version of `Counter` — know both.

---

## Q22: Use Counter for Word Frequency

**Question:** Same as above — count event types — but use Python's `collections.Counter`.

```python
from collections import Counter

events = ["click", "view", "click", "purchase", "view", "click", "view", "view"]

counts = Counter(events)
print(counts)
# Counter({'view': 4, 'click': 3, 'purchase': 1})

print(counts.most_common(2))
# [('view', 4), ('click', 3)]
```

**Explanation:**
`Counter` is a dict subclass built exactly for counting. `.most_common(n)` returns the top-n items by count — very useful for reporting.

---

## Q23: Group Records by a Field (GroupBy Pattern)

**Question:** Group a list of sales records by `region`.

```python
records = [
    {"region": "US", "amount": 200},
    {"region": "EU", "amount": 150},
    {"region": "US", "amount": 300},
    {"region": "EU", "amount": 100},
    {"region": "APAC", "amount": 450},
]

grouped = {}
for rec in records:
    region = rec["region"]
    if region not in grouped:
        grouped[region] = []
    grouped[region].append(rec["amount"])

print(grouped)
# {'US': [200, 300], 'EU': [150, 100], 'APAC': [450]}

# Calculate total per region
for region, amounts in grouped.items():
    print(f"{region}: {sum(amounts)}")
```

**Explanation:**
Build a dict where each key is a group and the value is a list of records for that group. This is a manual GroupBy operation — the same logic Pandas `.groupby()` uses internally.

---

## Q24: Merge Two Dictionaries

**Question:** You have default config and environment-specific overrides. Merge them so overrides win.

```python
defaults = {"timeout": 30, "retries": 3, "format": "csv"}
overrides = {"timeout": 60, "format": "parquet"}

# Method 1: unpacking (Python 3.5+)
merged = {**defaults, **overrides}
print(merged)
# {'timeout': 60, 'retries': 3, 'format': 'parquet'}

# Method 2: .update() (mutates defaults)
defaults.update(overrides)
print(defaults)
```

**Explanation:**
`{**dict1, **dict2}` creates a new dict. Keys in `dict2` overwrite `dict1`. When managing pipeline configs, this pattern lets defaults be overridden cleanly.

---

## Q25: Invert a Dictionary

**Question:** You have a mapping `column_alias -> column_name`. Invert it to look up by column name.

```python
alias_map = {"uid": "user_id", "ts": "timestamp", "evt": "event_type"}

inverted = {v: k for k, v in alias_map.items()}
print(inverted)
# {'user_id': 'uid', 'timestamp': 'ts', 'event_type': 'evt'}
```

**Explanation:**
Dict comprehension with `.items()` gives `(key, value)` pairs. Swapping `k` and `v` inverts the mapping. Assumes all values are unique (otherwise some keys will be lost).

---

## Q26: Find Common Elements Between Two Datasets (Set Intersection)

**Question:** Two teams ran different pipeline jobs. Find job IDs that both ran.

```python
team_a_jobs = {101, 102, 103, 105, 107}
team_b_jobs = {102, 104, 105, 106, 107}

common = team_a_jobs & team_b_jobs   # intersection
print("Both ran:", common)
# {102, 105, 107}

only_a = team_a_jobs - team_b_jobs   # difference
print("Only team A ran:", only_a)
# {101, 103}
```

**Explanation:**
Set operators: `&` = intersection (in both), `|` = union (in either), `-` = difference (in first but not second). Sets are ideal when you care about membership, not order.

---

## Q27: Deduplicate Records Using a Set of Tuples

**Question:** A list of `(user_id, event)` tuples has duplicates. Deduplicate while keeping unique combinations.

```python
raw_events = [
    ("u1", "click"),
    ("u2", "view"),
    ("u1", "click"),   # duplicate
    ("u3", "purchase"),
    ("u2", "view"),    # duplicate
]

unique_events = list(set(raw_events))
print(unique_events)
# [('u1', 'click'), ('u2', 'view'), ('u3', 'purchase')]  (order may vary)
```

**Explanation:**
Tuples are hashable, so they can be stored in a set. `set()` automatically removes duplicates. Note: you lose original order, unlike the seen-set pattern from Q16.

---

## Q28: Filter a Dictionary by Value

**Question:** A dict of `table -> row_count` is given. Return only tables with more than 10,000 rows.

```python
row_counts = {
    "orders": 52000,
    "temp_staging": 800,
    "customers": 15000,
    "archive": 100,
    "events": 98000,
}

large_tables = {table: count for table, count in row_counts.items() if count > 10000}
print(large_tables)
# {'orders': 52000, 'customers': 15000, 'events': 98000}
```

**Explanation:**
Dict comprehension with a condition filters key-value pairs just like list comprehension filters elements. This is cleaner than building a new dict in a `for` loop.

---

## Q29: Use defaultdict to Avoid KeyError

**Question:** Group filenames by their file extension without manually checking if the key exists.

```python
from collections import defaultdict

files = ["data.csv", "report.pdf", "sales.csv", "notes.txt", "events.json", "prices.csv"]

by_extension = defaultdict(list)
for filename in files:
    ext = filename.rsplit(".", 1)[-1]
    by_extension[ext].append(filename)

print(dict(by_extension))
# {'csv': ['data.csv', 'sales.csv', 'prices.csv'],
#  'pdf': ['report.pdf'],
#  'txt': ['notes.txt'],
#  'json': ['events.json']}
```

**Explanation:**
`defaultdict(list)` auto-creates an empty list for any new key. You never get a `KeyError`. `rsplit(".", 1)` splits from the right, so `"archive.tar.gz"` gives `"gz"` correctly.

---

## Q30: Check If Two Datasets Have the Same Schema (Keys)

**Question:** Two JSON records were loaded. Verify they have exactly the same set of keys (same schema).

```python
record_a = {"user_id": 1, "event": "click", "timestamp": "2024-01-01"}
record_b = {"user_id": 2, "region": "US", "timestamp": "2024-01-02"}

keys_a = set(record_a.keys())
keys_b = set(record_b.keys())

if keys_a == keys_b:
    print("Schemas match")
else:
    missing_in_b = keys_a - keys_b
    extra_in_b = keys_b - keys_a
    print(f"Fields missing in B: {missing_in_b}")
    print(f"Extra fields in B: {extra_in_b}")

# Fields missing in B: {'event'}
# Extra fields in B: {'region'}
```

**Explanation:**
`dict.keys()` returns a view that behaves like a set. Comparing with `==` checks exact equality. Set difference shows exactly what diverged — critical for schema evolution in data engineering.

---

> Next: [04_data_engineering_patterns.md](04_data_engineering_patterns.md)
