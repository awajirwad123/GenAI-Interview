# Coding Questions: Data Engineering Patterns (Q31–Q42)

> **Difficulty:** 40% Intermediate | 20% Amazon-level  
> Topics: JSON parsing, CSV file I/O, log parsing, ETL transforms, deduplication, aggregation, streaming simulation

---

## Q31: Parse a JSON String and Extract Fields

**Question:** An API returned a JSON string for a customer record. Extract `name` and `orders_count`.

```python
import json

raw_json = '{"customer_id": 42, "name": "Alice", "orders_count": 17, "region": "US"}'

data = json.loads(raw_json)

print("Name:", data["name"])
print("Orders:", data["orders_count"])
```

**Explanation:**
`json.loads()` converts a JSON string to a Python dict. `json.load()` (no 's') reads from a file object. Knowing the difference is a common interview question.

---

## Q32: Write Records to a JSON File

**Question:** Save a list of pipeline run metadata dicts to a JSON file.

```python
import json

runs = [
    {"job": "ingest", "status": "success", "rows": 50000},
    {"job": "transform", "status": "success", "rows": 48500},
    {"job": "load", "status": "failed", "rows": 0},
]

with open("pipeline_runs.json", "w") as f:
    json.dump(runs, f, indent=2)

print("Written to pipeline_runs.json")
```

**Explanation:**
`json.dump()` serializes Python objects to a file. `indent=2` makes it human-readable. The `with` block ensures the file is closed automatically even if an error occurs.

---

## Q33: Read a CSV File and Print Rows as Dicts

**Question:** Read `employees.csv` and print each row as a dictionary.

**Sample CSV (`employees.csv`):**
```
name,department,salary
Alice,Engineering,95000
Bob,Marketing,72000
Carol,Engineering,105000
```

```python
import csv

with open("employees.csv", "r") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(dict(row))

# {'name': 'Alice', 'department': 'Engineering', 'salary': '95000'}
# {'name': 'Bob', 'department': 'Marketing', 'salary': '72000'}
# ...
```

**Explanation:**
`csv.DictReader` uses the first row as column headers and returns each row as an `OrderedDict`. Wrapping in `dict()` gives a plain dict. Note: all values are strings — cast to `int`/`float` as needed.

---

## Q34: Filter CSV Records by a Condition

**Question:** From the same `employees.csv`, print only Engineering employees with salary > 90000.

```python
import csv

with open("employees.csv", "r") as f:
    reader = csv.DictReader(f)
    for row in reader:
        if row["department"] == "Engineering" and int(row["salary"]) > 90000:
            print(row["name"], row["salary"])

# Alice 95000
# Carol 105000
```

**Explanation:**
Filter logic goes inside the loop. Always cast numeric CSV fields (they come as strings). This is a basic ETL filter step.

---

## Q35: Parse Log Lines and Extract Error Messages

**Question:** Given a list of log lines, extract only ERROR lines and return their messages.

```python
logs = [
    "2024-01-15 INFO Pipeline started",
    "2024-01-15 ERROR Failed to connect to database: timeout",
    "2024-01-15 INFO Records loaded: 1200",
    "2024-01-15 ERROR S3 write failed: permission denied",
    "2024-01-15 WARNING Slow query detected",
]

errors = []
for line in logs:
    parts = line.split(" ", 3)   # split into at most 4 parts
    if parts[1] == "ERROR":
        errors.append(parts[3])   # the message part

for err in errors:
    print(err)
# Failed to connect to database: timeout
# S3 write failed: permission denied
```

**Explanation:**
`split(" ", 3)` splits on space but stops after 3 splits, keeping the rest of the message intact. Index `[1]` is the log level, `[3]` is the full message.

---

## Q36: Aggregate Total Sales by Region

**Question:** Given a list of transaction dicts, compute total sales amount per region.

```python
transactions = [
    {"region": "US", "amount": 500},
    {"region": "EU", "amount": 200},
    {"region": "US", "amount": 300},
    {"region": "APAC", "amount": 700},
    {"region": "EU", "amount": 150},
    {"region": "US", "amount": 450},
]

totals = {}
for txn in transactions:
    region = txn["region"]
    totals[region] = totals.get(region, 0) + txn["amount"]

for region, total in sorted(totals.items()):
    print(f"{region}: ${total}")

# APAC: $700
# EU: $350
# US: $1250
```

**Explanation:**
The `dict.get(key, 0)` pattern accumulates values without `KeyError`. `sorted()` on `.items()` gives alphabetical output. This is the Python equivalent of SQL `SUM(...) GROUP BY region`.

---

## Q37: Deduplicate Records by Composite Key

**Question:** A transaction list has duplicates based on `(user_id, order_id)`. Keep only the first occurrence.

```python
transactions = [
    {"user_id": "u1", "order_id": "o100", "amount": 50},
    {"user_id": "u2", "order_id": "o101", "amount": 75},
    {"user_id": "u1", "order_id": "o100", "amount": 50},  # duplicate
    {"user_id": "u3", "order_id": "o102", "amount": 120},
    {"user_id": "u2", "order_id": "o101", "amount": 75},  # duplicate
]

seen = set()
deduped = []

for txn in transactions:
    key = (txn["user_id"], txn["order_id"])
    if key not in seen:
        deduped.append(txn)
        seen.add(key)

print(f"Original: {len(transactions)}, Deduplicated: {len(deduped)}")
for t in deduped:
    print(t)
```

**Explanation:**
A composite key `(user_id, order_id)` uniquely identifies each record. The `seen` set provides O(1) duplicate checks. This pattern is the foundation of exactly-once processing in pipelines.

---

## Q38: Clean and Transform a Dataset

**Question:** A raw dataset has inconsistent formats. Clean it: trim whitespace, lowercase `region`, cast `revenue` to float, and skip rows missing `revenue`.

```python
raw_data = [
    {"name": "  Alice ", "region": "US", "revenue": "1200.50"},
    {"name": "Bob", "region": " eu ", "revenue": ""},
    {"name": "Carol", "region": "APAC", "revenue": "3450.00"},
    {"name": "  Dave", "region": "US", "revenue": "980"},
]

cleaned = []
for row in raw_data:
    if not row["revenue"].strip():
        print(f"Skipping {row['name'].strip()} — missing revenue")
        continue
    cleaned.append({
        "name": row["name"].strip(),
        "region": row["region"].strip().lower(),
        "revenue": float(row["revenue"]),
    })

print("\nCleaned records:")
for r in cleaned:
    print(r)
```

**Explanation:**
Data cleaning is a core DE task. Always validate before type casting (empty string `float("")` raises `ValueError`). The `continue` statement skips invalid rows without crashing the loop.

---

## Q39: Simulate a Streaming Pipeline With a Generator

**Question:** Simulate reading records from a large file one at a time (without loading everything into memory) using a generator.

```python
def stream_records(filepath):
    """Yield one record at a time from a CSV-like file."""
    with open(filepath, "r") as f:
        for line in f:
            line = line.strip()
            if line:
                yield line

# Usage
for record in stream_records("data.csv"):
    # process one record at a time
    print("Processing:", record)
    # In a real pipeline: transform, validate, write to sink
```

**Explanation:**
`yield` turns a function into a generator. The file is read one line at a time — never fully loaded into memory. This is how real streaming pipelines (Kafka consumers, large S3 reads) process data at scale.

---

## Q40: Write a Simple ETL Function (Extract → Transform → Load)

**Question:** Write a function that reads from a source list, transforms each record, and "loads" into a target list. Simulate a mini-ETL.

```python
def extract():
    """Simulate extracting raw records."""
    return [
        {"id": 1, "name": "alice", "score": "85"},
        {"id": 2, "name": "  Bob", "score": "92"},
        {"id": 3, "name": "carol", "score": ""},
    ]

def transform(records):
    """Clean and validate records."""
    transformed = []
    for rec in records:
        if not rec["score"].strip():
            print(f"Skipping record {rec['id']} — missing score")
            continue
        transformed.append({
            "id": rec["id"],
            "name": rec["name"].strip().title(),
            "score": int(rec["score"]),
        })
    return transformed

def load(records, sink):
    """Load records to target (simulated as a list)."""
    sink.extend(records)
    print(f"Loaded {len(records)} records")

# Run the ETL
raw = extract()
clean = transform(raw)
destination = []
load(clean, destination)

print(destination)
```

**Explanation:**
Breaking ETL into separate functions mirrors how real pipelines are architected. Each stage is independently testable. `title()` properly capitalizes names.

---

## Q41: Find Top-N Records by a Metric

**Question:** From a list of S3 objects with their sizes, return the top 3 largest files.

```python
files = [
    {"name": "archive_2023.gz", "size_mb": 1200},
    {"name": "events_jan.parquet", "size_mb": 450},
    {"name": "raw_logs.txt", "size_mb": 3400},
    {"name": "customer_export.csv", "size_mb": 780},
    {"name": "dim_products.json", "size_mb": 95},
]

top_3 = sorted(files, key=lambda f: f["size_mb"], reverse=True)[:3]

for f in top_3:
    print(f["name"], f["size_mb"], "MB")

# raw_logs.txt 3400 MB
# archive_2023.gz 1200 MB
# customer_export.csv 780 MB
```

**Explanation:**
`sorted(..., reverse=True)` sorts descending. `[:3]` slices the first three results. The `lambda` key extracts the size field. This is a "top-N" pattern — common in monitoring dashboards.

---

## Q42: Compute Moving Average of Batch Sizes

**Question:** You have a list of daily record counts ingested. Compute the 3-day moving average to detect trends.

```python
daily_counts = [1000, 1200, 950, 1300, 1100, 1400, 1250, 1350]
window = 3

moving_avg = []
for i in range(len(daily_counts) - window + 1):
    window_data = daily_counts[i:i + window]
    avg = sum(window_data) / window
    moving_avg.append(round(avg, 2))

print("Moving averages:", moving_avg)
# [1050.0, 1150.0, 1116.67, 1266.67, 1250.0, 1333.33]
```

**Explanation:**
Sliding window pattern: move the start index forward each step, take a slice of `window` size, compute average. This is foundational for time-series analysis in data pipelines.

---

> Next: [05_advanced_basics.md](05_advanced_basics.md)
