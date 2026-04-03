# Coding Questions: Basics (Q1–Q10)

> **Difficulty:** 40% Basic | Topics: Variables, data types, loops, conditionals, functions

---

## Q1: Swap Two Variables Without a Temp Variable

**Question:** You receive two pipeline config values `source` and `target`. Swap them without using a third variable.

```python
source = "s3://bucket-a"
target = "s3://bucket-b"

# Swap
source, target = target, source

print("source:", source)
print("target:", target)
```

**Explanation:**
Python's tuple unpacking lets you swap in one line. Both right-hand values are evaluated first, then assigned. No temp variable needed.

---

## Q2: Check If a Number Is Even or Odd

**Question:** In a batch pipeline, jobs are numbered. Print "Even batch" or "Odd batch" for a given job number.

```python
def check_batch(job_number):
    if job_number % 2 == 0:
        return "Even batch"
    else:
        return "Odd batch"

print(check_batch(4))   # Even batch
print(check_batch(7))   # Odd batch
```

**Explanation:**
The `%` operator gives the remainder. If remainder is 0, the number is even. Simple conditional to branch logic.

---

## Q3: Sum of a List of Numbers

**Question:** You have a list of file sizes in MB. Calculate the total size.

```python
file_sizes = [120, 340, 55, 210, 430]

total = sum(file_sizes)
print("Total size (MB):", total)
```

**Explanation:**
Python's built-in `sum()` adds all elements. In data engineering you'll use this often for quick aggregations.

---

## Q4: Find the Maximum Value in a List

**Question:** A list of row counts per partition is given. Find the largest partition.

```python
row_counts = [10000, 45000, 23000, 67000, 12000]

largest = max(row_counts)
print("Largest partition:", largest)
```

**Explanation:**
`max()` scans the list and returns the highest value. `min()` works the same way for the smallest.

---

## Q5: Count Numbers Greater Than a Threshold

**Question:** Given latency readings from a data pipeline (in ms), count how many exceeded the 200ms threshold.

```python
latencies = [150, 300, 220, 95, 410, 180, 260]
threshold = 200

slow_count = 0
for lat in latencies:
    if lat > threshold:
        slow_count += 1

print("Slow responses:", slow_count)
```

**Explanation:**
A basic `for` loop with a counter. Each time the condition is met, increment. This is a building block for data quality checks.

---

## Q6: FizzBuzz Variant — Label Pipeline Stages

**Question:** For job IDs 1–20, print:
- "Extract" if divisible by 3
- "Load" if divisible by 5
- "Transform" if divisible by both
- The number otherwise

```python
for i in range(1, 21):
    if i % 3 == 0 and i % 5 == 0:
        print("Transform")
    elif i % 3 == 0:
        print("Extract")
    elif i % 5 == 0:
        print("Load")
    else:
        print(i)
```

**Explanation:**
The combined condition (`% 3 == 0 and % 5 == 0`) must come first. Order of `elif` matters here — a common interview gotcha.

---

## Q7: Reverse a List

**Question:** Records in your pipeline arrived in reverse chronological order. Reverse the list to process oldest-first.

```python
records = ["rec_005", "rec_004", "rec_003", "rec_002", "rec_001"]

records.reverse()   # in-place
print(records)

# OR using slicing (creates a new list)
reversed_records = records[::-1]
print(reversed_records)
```

**Explanation:**
`.reverse()` mutates the original list. `[::-1]` creates a new reversed copy. Know both — interviewers sometimes ask which is in-place.

---

## Q8: Write a Function With a Default Parameter

**Question:** Write a function `read_data(source, format="csv")` that prints what it would read.

```python
def read_data(source, format="csv"):
    print(f"Reading {source} as {format}")

read_data("s3://bucket/file.csv")
read_data("s3://bucket/file.parquet", format="parquet")
```

**Explanation:**
Default parameters make functions flexible. The default is used when the caller doesn't provide that argument — common in pipeline configs.

---

## Q9: Nested Loop — Generate All Combinations

**Question:** You have two lists: `environments = ["dev", "prod"]` and `regions = ["us-east-1", "eu-west-1"]`. Generate all env-region pairs.

```python
environments = ["dev", "prod"]
regions = ["us-east-1", "eu-west-1"]

for env in environments:
    for region in regions:
        print(f"{env}-{region}")
```

**Output:**
```
dev-us-east-1
dev-eu-west-1
prod-us-east-1
prod-eu-west-1
```

**Explanation:**
Nested loops iterate the outer list once per inner element. A 2-item × 2-item list gives 4 combinations.

---

## Q10: Use a Function to Validate Row Count

**Question:** Write a function that takes an expected and actual row count and returns True if they match, otherwise prints a warning and returns False.

```python
def validate_row_count(expected, actual):
    if expected == actual:
        return True
    else:
        print(f"WARNING: Expected {expected} rows but got {actual} rows")
        return False

result = validate_row_count(1000, 1000)
print(result)   # True

result = validate_row_count(1000, 987)
print(result)   # WARNING + False
```

**Explanation:**
Validation functions are critical in ETL pipelines. Returning a boolean lets the caller decide to continue or halt the pipeline.

---

> Next: [02_strings_lists.md](02_strings_lists.md)
