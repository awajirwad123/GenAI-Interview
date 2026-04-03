# Coding Questions: boto3 Hands-On & SQL-to-Python Patterns (Q1–Q20)

> **Level:** Intermediate to Amazon DE1 | **Target:** Amazon Data Engineer 1
> **Section A:** boto3 real patterns with error handling (Q1–Q10)
> **Section B:** SQL-to-Python translation — side-by-side comparisons (Q11–Q20)

---

# SECTION A: boto3 HANDS-ON PATTERNS (Q1–Q10)

---

## Q1: List All Objects in an S3 Prefix (Handling Pagination)

**Scenario:** List all Parquet files under `s3://my-data-lake/raw/2024/` — the result may span multiple pages.

```python
import boto3

def list_s3_objects(bucket, prefix, suffix=".parquet"):
    """
    List all S3 objects matching a prefix/suffix.
    Handles pagination automatically.
    """
    s3 = boto3.client("s3")
    paginator = s3.get_paginator("list_objects_v2")

    matching_keys = []
    for page in paginator.paginate(Bucket=bucket, Prefix=prefix):
        for obj in page.get("Contents", []):
            if obj["Key"].endswith(suffix):
                matching_keys.append({
                    "key": obj["Key"],
                    "size_mb": round(obj["Size"] / 1024 / 1024, 2),
                    "last_modified": obj["LastModified"].isoformat()
                })

    return matching_keys

# Usage (requires AWS credentials configured)
# files = list_s3_objects("my-data-lake", "raw/2024/")
# for f in files:
#     print(f)

# Simulated output:
example_output = [
    {"key": "raw/2024/01/events.parquet", "size_mb": 45.2, "last_modified": "2024-01-31T12:00:00"},
    {"key": "raw/2024/02/events.parquet", "size_mb": 52.8, "last_modified": "2024-02-29T12:00:00"},
]
print(example_output)
```

**Explanation:** S3 `list_objects_v2` returns max 1000 objects per call. Always use a **paginator** — never assume one call returns everything. `get_paginator()` + `paginate()` automatically follows `NextContinuationToken`. `page.get("Contents", [])` safely handles empty pages (no `KeyError`).

---

## Q2: Download a File From S3 With Error Handling

**Scenario:** Download a partition file from S3. Handle the most common errors: file not found, permission denied, and network issues.

```python
import boto3
from botocore.exceptions import ClientError, NoCredentialsError

def download_s3_file(bucket, key, local_path):
    s3 = boto3.client("s3")
    try:
        s3.download_file(bucket, key, local_path)
        print(f"Downloaded: s3://{bucket}/{key} → {local_path}")
        return True
    except ClientError as e:
        error_code = e.response["Error"]["Code"]
        if error_code == "404" or error_code == "NoSuchKey":
            print(f"ERROR: Object not found — s3://{bucket}/{key}")
        elif error_code == "403":
            print(f"ERROR: Access denied — s3://{bucket}/{key}")
        else:
            print(f"ERROR: AWS error {error_code} — {e}")
        return False
    except NoCredentialsError:
        print("ERROR: AWS credentials not configured")
        return False

# Usage
# success = download_s3_file("my-data-lake", "raw/2024/01/events.parquet", "/tmp/events.parquet")
```

**Explanation:** All AWS API errors raise `ClientError`. Always check `e.response["Error"]["Code"]` to handle specific cases differently — `404`/`NoSuchKey` (missing), `403` (IAM permission issue), others (throttling, service errors). `NoCredentialsError` handles missing AWS config separately.

---

## Q3: Upload a File to S3 With Metadata and Encryption

**Scenario:** Upload a processed output file to S3, attaching job metadata and enabling server-side encryption.

```python
import boto3
import os
from datetime import datetime

def upload_to_s3(local_path, bucket, key, job_name, run_id):
    s3 = boto3.client("s3")
    file_size = os.path.getsize(local_path)

    extra_args = {
        "ServerSideEncryption": "AES256",   # SSE-S3 encryption
        "Metadata": {
            "job-name": job_name,
            "run-id": run_id,
            "upload-timestamp": datetime.utcnow().isoformat(),
            "source-size-bytes": str(file_size),
        },
        "ContentType": "application/octet-stream",
    }

    s3.upload_file(local_path, bucket, key, ExtraArgs=extra_args)
    print(f"Uploaded: {local_path} → s3://{bucket}/{key}")
    print(f"  Metadata: job={job_name}, run={run_id}, size={file_size:,} bytes")

# Usage
# upload_to_s3("/tmp/output.parquet", "my-data-lake", "processed/2024/output.parquet",
#              job_name="daily_ingest", run_id="run_20240115_001")
```

**Explanation:** `ExtraArgs` passes optional parameters to S3. Metadata values must all be strings — cast numbers with `str()`. Encryption at rest (`AES256`) is a compliance requirement at most enterprises. Metadata enables traceability — you can inspect any S3 file and know which job created it.

---

## Q4: Multipart Upload for Large Files

**Scenario:** Upload a 5GB Parquet file to S3. Standard `upload_file` handles this automatically, but here's the explicit multipart pattern for custom control.

```python
import boto3
import math

def multipart_upload(local_path, bucket, key, part_size_mb=50):
    s3 = boto3.client("s3")
    part_size = part_size_mb * 1024 * 1024   # bytes

    mpu = s3.create_multipart_upload(Bucket=bucket, Key=key)
    upload_id = mpu["UploadId"]
    parts = []

    try:
        with open(local_path, "rb") as f:
            part_number = 1
            while True:
                data = f.read(part_size)
                if not data:
                    break
                resp = s3.upload_part(
                    Bucket=bucket, Key=key,
                    PartNumber=part_number,
                    UploadId=upload_id,
                    Body=data
                )
                parts.append({"PartNumber": part_number, "ETag": resp["ETag"]})
                print(f"Uploaded part {part_number} ({len(data)/1024/1024:.1f} MB)")
                part_number += 1

        s3.complete_multipart_upload(
            Bucket=bucket, Key=key,
            UploadId=upload_id,
            MultipartUpload={"Parts": parts}
        )
        print("Multipart upload complete!")

    except Exception as e:
        # Abort to clean up incomplete multipart upload (avoids storage charges)
        s3.abort_multipart_upload(Bucket=bucket, Key=key, UploadId=upload_id)
        raise RuntimeError(f"Multipart upload failed and was aborted: {e}")
```

**Explanation:** Multipart upload: create → upload parts → complete. Each part must be ≥5MB (except the last). Always `abort_multipart_upload` on failure — incomplete uploads incur S3 storage costs. In practice, use `boto3.s3.transfer.TransferConfig` with `multipart_threshold` for automatic multipart handling.

---

## Q5: Read JSON Lines From S3 Without Downloading

**Scenario:** Process each JSON record in an S3 file line-by-line, without writing to disk.

```python
import boto3
import json
import io

def stream_jsonl_from_s3(bucket, key):
    """Stream and parse a JSON Lines file from S3 without saving locally."""
    s3 = boto3.client("s3")
    response = s3.get_object(Bucket=bucket, Key=key)
    body = response["Body"]   # StreamingBody

    record_count = 0
    for line in io.TextIOWrapper(body, encoding="utf-8"):
        line = line.strip()
        if not line:
            continue
        try:
            record = json.loads(line)
            yield record
            record_count += 1
        except json.JSONDecodeError as e:
            print(f"WARNING: Skipping malformed JSON line: {e}")

    print(f"Streamed {record_count} records from s3://{bucket}/{key}")

# Usage
# for record in stream_jsonl_from_s3("my-bucket", "data/events.jsonl"):
#     process(record)
```

**Explanation:** `get_object()["Body"]` returns a `StreamingBody` — wrap it with `io.TextIOWrapper` for line-by-line text iteration. No disk I/O needed. Memory usage = one line at a time, regardless of file size. Generator (`yield`) makes this composable with other pipeline stages.

---

## Q6: Check If an S3 Object Exists Without Downloading It

**Scenario:** Check if a "success marker" file (`_SUCCESS`) exists in an S3 prefix before starting a downstream job.

```python
import boto3
from botocore.exceptions import ClientError

def s3_object_exists(bucket, key):
    """Check existence using HEAD request — no data transfer."""
    s3 = boto3.client("s3")
    try:
        s3.head_object(Bucket=bucket, Key=key)
        return True
    except ClientError as e:
        if e.response["Error"]["Code"] in ("404", "NoSuchKey"):
            return False
        raise   # re-raise unexpected errors (permissions, network)

def wait_for_upstream(bucket, success_key, job_name):
    if s3_object_exists(bucket, success_key):
        print(f"Upstream ready. Starting {job_name}...")
        return True
    else:
        print(f"Waiting: s3://{bucket}/{success_key} not found yet. Skipping {job_name}.")
        return False

# Usage
# ready = wait_for_upstream("my-data-lake", "raw/2024/01/15/_SUCCESS", "daily_transform")
```

**Explanation:** `head_object()` only fetches metadata (size, ETag, last-modified) — no data downloaded. Much cheaper than `get_object()` for existence checks. This is the standard pattern for pipeline dependency checking and S3-based job orchestration.

---

## Q7: Write Records to DynamoDB in Batch

**Scenario:** Save pipeline run metadata to DynamoDB — use batch writes for efficiency.

```python
import boto3
from decimal import Decimal

def batch_write_to_dynamodb(table_name, records):
    """
    Write records to DynamoDB using batch_writer.
    Handles retries and chunking automatically (max 25 items per batch).
    """
    dynamodb = boto3.resource("dynamodb")
    table = dynamodb.Table(table_name)

    with table.batch_writer() as batch:
        for rec in records:
            # DynamoDB requires Decimal for floats, not float
            item = {k: Decimal(str(v)) if isinstance(v, float) else v
                    for k, v in rec.items()}
            batch.put_item(Item=item)

    print(f"Wrote {len(records)} items to {table_name}")

pipeline_runs = [
    {"run_id": "run_001", "job": "ingest",    "status": "success", "duration": 12.5, "rows": 50000},
    {"run_id": "run_002", "job": "transform", "status": "failed",  "duration": 3.2,  "rows": 0},
    {"run_id": "run_003", "job": "load",      "status": "success", "duration": 8.1,  "rows": 49800},
]

# batch_write_to_dynamodb("pipeline-runs", pipeline_runs)
```

**Explanation:** `batch_writer()` context manager chunks requests (DynamoDB batch limit = 25 writes per call) and retries unprocessed items automatically. Convert `float` to `Decimal` — DynamoDB rejects floats by default. `boto3.resource()` high-level API is cleaner for table operations than `boto3.client()`.

---

## Q8: Query AWS Glue Data Catalog for Table Metadata

**Scenario:** Before running a job, fetch the target table's schema from the Glue Data Catalog.

```python
import boto3
from botocore.exceptions import ClientError

def get_glue_table_schema(database, table_name):
    glue = boto3.client("glue")
    try:
        response = glue.get_table(DatabaseName=database, Name=table_name)
        table = response["Table"]
        columns = table["StorageDescriptor"]["Columns"]
        partition_keys = table.get("PartitionKeys", [])

        print(f"Table: {database}.{table_name}")
        print(f"Location: {table['StorageDescriptor']['Location']}")
        print(f"Format: {table['StorageDescriptor']['InputFormat'].split('.')[-1]}")
        print(f"\nColumns ({len(columns)}):")
        for col in columns:
            print(f"  {col['Name']:<25} {col['Type']}")
        if partition_keys:
            print(f"\nPartition Keys: {[k['Name'] for k in partition_keys]}")

        return columns, partition_keys

    except ClientError as e:
        if e.response["Error"]["Code"] == "EntityNotFoundException":
            print(f"Table {database}.{table_name} not found in Glue Catalog")
            return [], []
        raise

# glue_schema, partitions = get_glue_table_schema("my_database", "orders")
```

**Explanation:** Glue Data Catalog is the central metadata store for AWS data lakes. `StorageDescriptor.Columns` contains column names and types. `PartitionKeys` lists the partition fields. Always check `EntityNotFoundException` — the table may not exist in the catalog yet.

---

## Q9: Trigger and Monitor a Glue Job Run

**Scenario:** Start a Glue ETL job with custom parameters and poll until completion.

```python
import boto3
import time
from botocore.exceptions import ClientError

def run_glue_job(job_name, args=None, poll_interval=30):
    glue = boto3.client("glue")

    # Start the job
    start_resp = glue.start_job_run(
        JobName=job_name,
        Arguments=args or {},
        MaxCapacity=2.0   # DPUs
    )
    run_id = start_resp["JobRunId"]
    print(f"Started Glue job '{job_name}' — Run ID: {run_id}")

    # Poll until terminal state
    terminal_states = {"SUCCEEDED", "FAILED", "STOPPED", "ERROR", "TIMEOUT"}
    while True:
        resp = glue.get_job_run(JobName=job_name, RunId=run_id)
        state = resp["JobRun"]["JobRunState"]
        print(f"  Status: {state}")
        if state in terminal_states:
            if state == "SUCCEEDED":
                rows = resp["JobRun"].get("ExecutionTime", 0)
                print(f"Job succeeded in {rows}s")
                return True
            else:
                error = resp["JobRun"].get("ErrorMessage", "No error details")
                print(f"Job {state}: {error}")
                return False
        time.sleep(poll_interval)

# run_glue_job("my-etl-job", args={"--date": "2024-01-15", "--env": "prod"})
```

**Explanation:** Start job → poll `get_job_run()` in a loop until terminal state. Job arguments use `--key` prefix (Glue convention). `MaxCapacity` sets compute resources (DPUs). In production, use EventBridge/Step Functions instead of polling — but polling is correct for scripts and testing.

---

## Q10: Read from S3 Using Pandas with Chunked Processing

**Scenario:** Process a large CSV from S3 in chunks using Pandas — without writing to disk or loading everything into memory.

```python
import boto3
import pandas as pd
import io

def process_s3_csv_in_chunks(bucket, key, chunk_size=10000, filter_col=None, filter_val=None):
    """Read a CSV from S3 and process it in chunks using Pandas."""
    s3 = boto3.client("s3")
    response = s3.get_object(Bucket=bucket, Key=key)
    body = response["Body"].read().decode("utf-8")

    total_rows = 0
    results = []

    for chunk in pd.read_csv(io.StringIO(body), chunksize=chunk_size):
        # Optional filter
        if filter_col and filter_val:
            chunk = chunk[chunk[filter_col] == filter_val]
        total_rows += len(chunk)

        # Example transform: compute per-chunk aggregation
        if not chunk.empty:
            agg = chunk.groupby("region")["amount"].sum().reset_index()
            results.append(agg)

    # Combine all chunk results
    if results:
        final = pd.concat(results, ignore_index=True)
        final = final.groupby("region")["amount"].sum().reset_index()
        final.columns = ["region", "total_amount"]
        print(final.to_string(index=False))
    print(f"\nTotal rows processed: {total_rows:,}")

# process_s3_csv_in_chunks("my-bucket", "data/sales.csv", chunk_size=5000,
#                          filter_col="status", filter_val="completed")
```

**Explanation:** `get_object()["Body"].read()` loads the raw bytes. `io.StringIO` wraps as a file-like object for Pandas. `chunksize` returns a `TextFileReader` iterator — each chunk is a DataFrame. Aggregate each chunk, then combine. Handles files larger than RAM.

---

# SECTION B: SQL-TO-PYTHON TRANSLATION (Q11–Q20)

---

## Q11: SELECT with WHERE — Filter Records

**SQL:**
```sql
SELECT name, salary
FROM employees
WHERE department = 'Engineering' AND salary > 90000;
```

**Python:**
```python
employees = [
    {"name": "Alice", "department": "Engineering", "salary": 95000},
    {"name": "Bob",   "department": "Marketing",   "salary": 72000},
    {"name": "Carol", "department": "Engineering", "salary": 105000},
    {"name": "Dave",  "department": "Engineering", "salary": 85000},
]

# Using list comprehension
result = [
    {"name": e["name"], "salary": e["salary"]}
    for e in employees
    if e["department"] == "Engineering" and e["salary"] > 90000
]
print(result)
# [{'name': 'Alice', 'salary': 95000}, {'name': 'Carol', 'salary': 105000}]

# Using Pandas (equivalent)
import pandas as pd
df = pd.DataFrame(employees)
result_df = df[
    (df["department"] == "Engineering") & (df["salary"] > 90000)
][["name", "salary"]]
print(result_df)
```

**Key difference:** SQL is declarative ("what"). Python is imperative ("how"). Both achieve the same O(n) scan.

---

## Q12: GROUP BY with Aggregates

**SQL:**
```sql
SELECT department, COUNT(*) AS headcount, AVG(salary) AS avg_salary
FROM employees
GROUP BY department;
```

**Python (manual):**
```python
from collections import defaultdict

groups = defaultdict(list)
for e in employees:
    groups[e["department"]].append(e["salary"])

for dept, salaries in sorted(groups.items()):
    print(f"{dept:<15} headcount={len(salaries)}, avg_salary={sum(salaries)/len(salaries):,.0f}")
```

**Python (Pandas):**
```python
result = df.groupby("department").agg(
    headcount=("salary", "count"),
    avg_salary=("salary", "mean")
).reset_index()
print(result)
```

**Key insight:** Always prefer Pandas `groupby()` for tabular data over manual dicts. Manual dicts are useful when data is already in Python and you need only one pass.

---

## Q13: ORDER BY + LIMIT (Top-N)

**SQL:**
```sql
SELECT name, salary
FROM employees
ORDER BY salary DESC
LIMIT 3;
```

**Python:**
```python
# Using sorted() + slice
top_3 = sorted(employees, key=lambda e: e["salary"], reverse=True)[:3]
for e in top_3:
    print(e["name"], e["salary"])

# Using heapq.nlargest (more efficient for large lists, small N)
import heapq
top_3_efficient = heapq.nlargest(3, employees, key=lambda e: e["salary"])
```

**Key insight:** `heapq.nlargest(k, iterable)` is O(n log k) — better than `sorted()[: k]` which is O(n log n) when `k << n`.

---

## Q14: INNER JOIN Between Two Datasets

**SQL:**
```sql
SELECT o.order_id, c.name, o.amount
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id;
```

**Python (hash join):**
```python
orders = [
    {"order_id": "o1", "customer_id": "c1", "amount": 500},
    {"order_id": "o2", "customer_id": "c2", "amount": 200},
    {"order_id": "o3", "customer_id": "c9", "amount": 900},  # no match
]
customers = [
    {"customer_id": "c1", "name": "Alice"},
    {"customer_id": "c2", "name": "Bob"},
]

# Build hash table on the smaller dataset
customer_lookup = {c["customer_id"]: c for c in customers}

# INNER JOIN — only include rows with a match
inner_join = [
    {"order_id": o["order_id"], "name": customer_lookup[o["customer_id"]]["name"], "amount": o["amount"]}
    for o in orders
    if o["customer_id"] in customer_lookup
]
print(inner_join)
# [{'order_id': 'o1', 'name': 'Alice', 'amount': 500},
#  {'order_id': 'o2', 'name': 'Bob',   'amount': 200}]
```

**Key insight:** Build the hash table on the SMALLER dataset (customers). INNER JOIN = only rows where both sides match (filter with `if key in lookup`).

---

## Q15: LEFT JOIN — Keep All Rows from Left Table

**SQL:**
```sql
SELECT o.order_id, c.name, o.amount
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id;
```

**Python:**
```python
left_join = [
    {
        "order_id": o["order_id"],
        "name": customer_lookup.get(o["customer_id"], {}).get("name"),  # None if no match
        "amount": o["amount"]
    }
    for o in orders
]
print(left_join)
# [{'order_id': 'o1', 'name': 'Alice', 'amount': 500},
#  {'order_id': 'o2', 'name': 'Bob',   'amount': 200},
#  {'order_id': 'o3', 'name': None,    'amount': 900}]   ← kept with None
```

**Key insight:** LEFT JOIN = keep ALL rows from the left table. Use `.get()` with a default empty dict for missing lookups — the last record has `name=None`.

---

## Q16: WHERE IN — Filter Against a Set

**SQL:**
```sql
SELECT * FROM orders WHERE region IN ('US', 'EU');
```

**Python:**
```python
orders_region = [
    {"id": 1, "region": "US",   "amount": 100},
    {"id": 2, "region": "EU",   "amount": 200},
    {"id": 3, "region": "APAC", "amount": 300},
    {"id": 4, "region": "US",   "amount": 400},
]

allowed_regions = {"US", "EU"}   # set for O(1) lookup

filtered = [o for o in orders_region if o["region"] in allowed_regions]
print(filtered)
```

**Key insight:** Use a `set` for the `IN` list — O(1) lookup per check. Using a list gives O(k×n) instead of O(n).

---

## Q17: DISTINCT Values

**SQL:**
```sql
SELECT DISTINCT region FROM orders;
```

**Python:**
```python
regions = [o["region"] for o in orders_region]
distinct_regions = sorted(set(regions))
print(distinct_regions)
# ['APAC', 'EU', 'US']

# With Pandas
distinct_df = df["region"].unique()
```

**Key insight:** `set()` deduplicates automatically. Wrap in `sorted()` for deterministic output (sets are unordered).

---

## Q18: Window Function — Row Number per Partition

**SQL:**
```sql
SELECT *, ROW_NUMBER() OVER (PARTITION BY region ORDER BY amount DESC) AS row_num
FROM orders;
```

**Python:**
```python
from itertools import groupby

orders_sorted = sorted(orders_region, key=lambda x: (x["region"], -x["amount"]))
result = []
for region, group in groupby(orders_sorted, key=lambda x: x["region"]):
    for row_num, row in enumerate(group, start=1):
        result.append({**row, "row_num": row_num})

for r in sorted(result, key=lambda x: (x["region"], x["row_num"])):
    print(r)
```

**Python (Pandas — much cleaner):**
```python
import pandas as pd
df_orders = pd.DataFrame(orders_region)
df_orders["row_num"] = df_orders.sort_values("amount", ascending=False).groupby("region").cumcount() + 1
print(df_orders.sort_values(["region", "row_num"]))
```

**Key insight:** Window functions in Python are awkward without Pandas. For SQL window functions, Pandas is the right tool. Use `sort_values` + `groupby().cumcount()` for ROW_NUMBER equivalent.

---

## Q19: HAVING — Filter After Aggregation

**SQL:**
```sql
SELECT region, COUNT(*) AS cnt
FROM orders
GROUP BY region
HAVING COUNT(*) > 1;
```

**Python:**
```python
from collections import Counter

region_counts = Counter(o["region"] for o in orders_region)

# HAVING equivalent: filter after aggregation
having_result = {region: cnt for region, cnt in region_counts.items() if cnt > 1}
print(having_result)
# {'US': 2}

# Pandas equivalent
result = df_orders.groupby("region").size().reset_index(name="cnt")
result = result[result["cnt"] > 1]
print(result)
```

**Key insight:** `HAVING` = filter on aggregated results. In Python: aggregate first (Counter/groupby), then filter the result — never filter raw data before aggregation.

---

## Q20: UNION ALL vs UNION — Combine Two Datasets

**SQL:**
```sql
-- UNION ALL (keep duplicates)
SELECT name FROM table_a UNION ALL SELECT name FROM table_b;

-- UNION (remove duplicates)
SELECT name FROM table_a UNION SELECT name FROM table_b;
```

**Python:**
```python
table_a = [{"name": "Alice"}, {"name": "Bob"}, {"name": "Carol"}]
table_b = [{"name": "Bob"}, {"name": "Dave"}, {"name": "Alice"}]

# UNION ALL — keep all (including duplicates)
union_all = table_a + table_b
print("UNION ALL:", [r["name"] for r in union_all])
# ['Alice', 'Bob', 'Carol', 'Bob', 'Dave', 'Alice']

# UNION — deduplicate by name field
seen = set()
union_distinct = []
for r in union_all:
    if r["name"] not in seen:
        union_distinct.append(r)
        seen.add(r["name"])
print("UNION:", [r["name"] for r in union_distinct])
# ['Alice', 'Bob', 'Carol', 'Dave']

# Pandas equivalent
import pandas as pd
df_a = pd.DataFrame(table_a)
df_b = pd.DataFrame(table_b)
union_all_df = pd.concat([df_a, df_b], ignore_index=True)
union_df = union_all_df.drop_duplicates()
```

**Key insight:** `UNION ALL` = list concatenation (fast, O(n)). `UNION` = concatenate + deduplicate. In Pandas: `pd.concat` = UNION ALL, `drop_duplicates()` = UNION.

---

> **SQL → Python Quick Reference**
>
> | SQL Clause          | Python (Manual)               | Python (Pandas)                    |
> |---------------------|-------------------------------|------------------------------------|
> | `SELECT cols`       | dict comprehension            | `df[cols]`                         |
> | `WHERE condition`   | list comprehension with if    | `df[df.col == val]`                |
> | `GROUP BY + agg`    | defaultdict accumulation      | `df.groupby().agg()`               |
> | `ORDER BY`          | `sorted(key=, reverse=True)`  | `df.sort_values()`                 |
> | `LIMIT N`           | `[:N]` or `heapq.nlargest`    | `df.head(N)`                       |
> | `INNER JOIN`        | dict lookup + comprehension   | `pd.merge(how='inner')`            |
> | `LEFT JOIN`         | dict `.get()` with default    | `pd.merge(how='left')`             |
> | `DISTINCT`          | `set()` + `list()`            | `df.drop_duplicates()`             |
> | `HAVING`            | filter after aggregation      | filter after `groupby()`           |
> | `UNION ALL`         | `list1 + list2`               | `pd.concat([df1, df2])`            |
> | `ROW_NUMBER()`      | `groupby + enumerate`         | `groupby().cumcount() + 1`         |
>
> Return to: [09_amazon_de1_scenarios.md](09_amazon_de1_scenarios.md) | [../python_faq/06_decorators_functools_itertools.md](../python_faq/06_decorators_functools_itertools.md)
