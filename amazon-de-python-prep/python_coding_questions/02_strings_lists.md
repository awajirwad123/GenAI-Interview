# Coding Questions: Strings & Lists (Q11–Q20)

> **Difficulty:** 40% Basic–Intermediate | Topics: String manipulation, list comprehensions, slicing, sorting, filtering

---

## Q11: Clean Whitespace From a List of Table Names

**Question:** A downstream system returned table names with extra spaces. Strip and lowercase all entries.

```python
raw_tables = ["  Orders  ", "CUSTOMERS", " product_catalog ", "SALES"]

cleaned = [table.strip().lower() for table in raw_tables]
print(cleaned)
# ['orders', 'customers', 'product_catalog', 'sales']
```

**Explanation:**
`.strip()` removes leading/trailing whitespace. `.lower()` normalizes case. List comprehension applies both in one pass — cleaner than a `for` loop with `.append()`.

---

## Q12: Count Occurrences of a Character in a String

**Question:** In a log line `"ERROR: s3://bucket/path/to/file.csv"`, count how many `/` characters appear (to determine path depth).

```python
log_line = "ERROR: s3://bucket/path/to/file.csv"

slash_count = log_line.count("/")
print("Path depth slashes:", slash_count)
# 5
```

**Explanation:**
`.count(char)` scans the string and returns how many times the substring appears. No loop needed.

---

## Q13: Split a CSV Line Into Fields

**Question:** Parse a single CSV record: `"Alice,32,Engineer,New York"` into individual fields.

```python
record = "Alice,32,Engineer,New York"

fields = record.split(",")
print(fields)
# ['Alice', '32', 'Engineer', 'New York']

name, age, role, city = fields
print(f"Name: {name}, Age: {age}")
```

**Explanation:**
`.split(delimiter)` splits a string into a list wherever the delimiter appears. Unpacking into variables makes each field accessible by name.

---

## Q14: Check If a String Starts With a Known Prefix

**Question:** Filter only S3 paths (starting with `"s3://"`) from a mixed list of file paths.

```python
paths = [
    "s3://data-lake/raw/orders.csv",
    "/local/data/temp.csv",
    "s3://data-lake/processed/sales.parquet",
    "hdfs://namenode/user/data.json"
]

s3_paths = [p for p in paths if p.startswith("s3://")]
print(s3_paths)
```

**Explanation:**
`.startswith()` returns `True` if the string begins with the given prefix. Useful for routing logic in pipelines that handle multiple source types.

---

## Q15: Reverse a String

**Question:** A column value came in reversed from a legacy system. Reverse the string `"gnimertS atad"`.

```python
text = "gnimertS atad"

reversed_text = text[::-1]
print(reversed_text)
# "ataD gnimaertS"  — note: original was "ataD gnimertS"
```

**Explanation:**
`[::-1]` is Python slicing with a step of -1, which walks the string backwards. Works on strings, lists, and tuples.

---

## Q16: Remove Duplicates From a List While Preserving Order

**Question:** A log processor produced event IDs with duplicates. Remove duplicates but keep the original order.

```python
event_ids = [101, 203, 101, 405, 203, 101, 506]

seen = set()
unique_ids = []
for eid in event_ids:
    if eid not in seen:
        unique_ids.append(eid)
        seen.add(eid)

print(unique_ids)
# [101, 203, 405, 506]
```

**Explanation:**
A `set` tracks what we've already seen (O(1) lookup). We only append to the result list if it's a new value. Order is preserved unlike `list(set(...))`.

---

## Q17: Sort a List of Dictionaries by a Key

**Question:** Sort a list of pipeline run records by `duration_sec` ascending.

```python
runs = [
    {"job": "ingest", "duration_sec": 320},
    {"job": "transform", "duration_sec": 95},
    {"job": "load", "duration_sec": 210},
]

sorted_runs = sorted(runs, key=lambda r: r["duration_sec"])

for run in sorted_runs:
    print(run["job"], run["duration_sec"])
# transform 95
# load 210
# ingest 320
```

**Explanation:**
`sorted()` takes a `key` function that tells it what to sort by. `lambda r: r["duration_sec"]` extracts the numeric field. Add `reverse=True` for descending order.

---

## Q18: Flatten a List of Lists

**Question:** Each partition returned a list of records. Combine all partitions into a single flat list.

```python
partitions = [
    ["rec_1", "rec_2"],
    ["rec_3"],
    ["rec_4", "rec_5", "rec_6"]
]

flat = [record for partition in partitions for record in partition]
print(flat)
# ['rec_1', 'rec_2', 'rec_3', 'rec_4', 'rec_5', 'rec_6']
```

**Explanation:**
A nested list comprehension: the outer loop iterates over partitions, the inner loop iterates within each. Reads as: "for each record in each partition".

---

## Q19: Extract Unique Words From a Log Message

**Question:** From the error string `"disk full disk error disk write error"`, find unique words.

```python
message = "disk full disk error disk write error"

words = message.split()
unique_words = list(set(words))
print(sorted(unique_words))
# ['disk', 'error', 'full', 'write']
```

**Explanation:**
`.split()` with no argument splits on any whitespace. Wrapping in `set()` keeps only unique values. `sorted()` gives consistent ordering.

---

## Q20: Join a List Into a Formatted String

**Question:** You have a list of column names. Produce a SQL-style SELECT string: `"SELECT col1, col2, col3 FROM table"`.

```python
columns = ["user_id", "event_type", "timestamp", "region"]
table = "events"

select_clause = ", ".join(columns)
query = f"SELECT {select_clause} FROM {table}"
print(query)
# SELECT user_id, event_type, timestamp, region FROM events
```

**Explanation:**
`", ".join(list)` stitches list items together with the given separator string. `f-strings` embed variables cleanly. This pattern is used in dynamic SQL generation.

---

> Next: [03_dict_set.md](03_dict_set.md)
