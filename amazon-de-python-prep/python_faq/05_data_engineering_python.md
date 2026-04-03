# Python FAQ: Data Engineering Python — Pandas, boto3, AWS & Pipelines (Q76–Q100)

---

**Q76: What is Pandas and why is it used in data engineering?**

A: Pandas is a Python library providing `DataFrame` (2D table) and `Series` (1D column) structures for data manipulation. In data engineering, it's used for data exploration, cleaning, transformation, and small-scale ETL. For large-scale pipelines, Spark or Dask replaces it, but Pandas is essential for local processing, prototyping, and glue scripts.

---

**Q77: What is a DataFrame?**

A: A DataFrame is a 2-dimensional, labeled data structure — like a table with rows and columns. Each column is a `Series`. DataFrames can be created from dicts, CSV files, JSON, or database queries. They support vectorized operations, making them much faster than plain Python loops for large datasets.

---

**Q78: What is the difference between `loc` and `iloc` in Pandas?**

A: `loc` accesses rows and columns by **label** (index name, column name). `iloc` accesses by **integer position** (0-based). Example: `df.loc[0, "name"]` selects row with index label 0, column named "name". `df.iloc[0, 0]` selects the first row, first column by position. Use `loc` for named access, `iloc` for positional.

---

**Q79: How do you filter rows in a Pandas DataFrame?**

A:
```python
# Keep only rows where age > 25
young = df[df["age"] > 25]

# Multiple conditions (use & not 'and')
filtered = df[(df["age"] > 25) & (df["city"] == "NYC")]
```
Conditions inside `[]` are called boolean indexing. Always use `&` (and), `|` (or), `~` (not) — not Python's `and`/`or` — because Pandas uses element-wise operations.

---

**Q80: What is `groupby()` in Pandas and how does it work?**

A: `groupby(column)` splits the DataFrame into groups based on unique values of the column, applies an aggregation function to each group, and combines results. Example:
```python
df.groupby("region")["sales"].sum()
df.groupby("region")["sales"].agg(["sum", "mean", "count"])
```
This is the Pandas equivalent of SQL `GROUP BY`. It's the most used operation in data analysis pipelines.

---

**Q81: What does `dropna()` do and when should you use it?**

A: `df.dropna()` removes rows (default) or columns containing `NaN` (missing values). Parameters: `axis=0` (rows), `axis=1` (columns), `subset=["col1"]` (only check specific columns), `how="all"` (drop only if all values are NaN vs any). Use after loading raw data to remove incomplete records.

---

**Q82: What is the difference between `fillna()` and `dropna()`?**

A: `dropna()` removes rows/columns with missing values. `fillna(value)` replaces missing values with a specified value — useful when dropping would lose too much data. Examples: `df.fillna(0)`, `df["salary"].fillna(df["salary"].mean())`. The right choice depends on your data quality requirements.

---

**Q83: What is `apply()` in Pandas and when should you use it?**

A: `apply(func)` applies a custom function to each row (`axis=1`) or each column (`axis=0`). Example: `df["full_name"] = df.apply(lambda row: row["first"] + " " + row["last"], axis=1)`. Use it for complex row-level transformations that can't be done with vectorized operations. Avoid it for simple column math — vectorized operations are 10-100x faster.

---

**Q84: What is `merge()` vs `concat()` in Pandas?**

A: `merge()` is like SQL JOIN — combines DataFrames on matching keys: `pd.merge(df1, df2, on="user_id", how="left")`. `concat()` stacks DataFrames vertically (rows) or horizontally (columns): `pd.concat([df1, df2], ignore_index=True)`. Use `merge` for relationship joins, `concat` for combining partitions of the same schema.

---

**Q85: How do you read a large CSV file in Pandas without loading it all into memory?**

A:
```python
import pandas as pd

chunk_size = 100_000
for chunk in pd.read_csv("large_file.csv", chunksize=chunk_size):
    # process each chunk as a DataFrame
    process(chunk)
```
`chunksize` returns a `TextFileReader` iterator. Each `chunk` is a DataFrame of `chunksize` rows. Critical for large files that exceed available RAM.

---

**Q86: What is `dtype` in Pandas and why does it matter for performance?**

A: `dtype` is the data type of a Series (column). Default behavior often infers `object` dtype for strings and `float64` for numbers, wasting memory. Explicitly specifying dtypes: `pd.read_csv(f, dtype={"id": "int32", "amount": "float32"})` can reduce memory usage by 50–75%. Using `category` dtype for low-cardinality string columns is especially efficient.

---

**Q87: What is boto3?**

A: `boto3` is the official AWS SDK for Python. It allows Python code to interact with AWS services: S3, DynamoDB, Glue, Lambda, Redshift, etc. You create a client or resource for a specific service and call its methods. For data engineering, it's used most heavily for S3 operations, Glue job management, and Redshift data loads.

---

**Q88: How do you create an S3 client using boto3?**

A:
```python
import boto3

# Using default credentials (from ~/.aws/credentials or IAM role)
s3 = boto3.client("s3", region_name="us-east-1")

# List objects in a bucket
response = s3.list_objects_v2(Bucket="my-data-lake", Prefix="raw/2024/")
for obj in response.get("Contents", []):
    print(obj["Key"])
```
In production, use IAM roles (not hardcoded credentials). `boto3.client()` creates a low-level client; `boto3.resource()` provides a higher-level object-oriented interface.

---

**Q89: How do you read a file from S3 using boto3?**

A:
```python
import boto3
import json

s3 = boto3.client("s3")

response = s3.get_object(Bucket="my-bucket", Key="data/records.json")
body = response["Body"].read().decode("utf-8")
records = json.loads(body)
print(f"Loaded {len(records)} records")
```
`get_object()` returns a response dict. `Body` is a streaming object — call `.read()` to get bytes, then decode to string.

---

**Q90: How do you upload a file to S3 using boto3?**

A:
```python
import boto3

s3 = boto3.client("s3")

# Upload local file
s3.upload_file("local_output.csv", "my-bucket", "processed/output.csv")

# Upload in-memory data
import json
data = json.dumps({"records": 1000}).encode("utf-8")
s3.put_object(Bucket="my-bucket", Key="metadata/run.json", Body=data)
```
`upload_file()` is for local files. `put_object()` is for in-memory data. Both support `ExtraArgs` for metadata and encryption settings.

---

**Q91: What is AWS Glue and how does Python relate to it?**

A: AWS Glue is a fully managed ETL service. Glue jobs run Python (PySpark) scripts to transform data between S3, RDS, Redshift, etc. The Glue Data Catalog stores metadata (schemas, table definitions). Python scripts in Glue use `awsglue` and `pyspark` libraries. Understanding Glue architecture is expected for Amazon DE roles.

---

**Q92: What is a Glue Job vs a Glue Crawler?**

A: A **Glue Crawler** scans data sources (S3, databases) and automatically infers schemas, populating the Glue Data Catalog. A **Glue Job** is a script (PySpark or Python shell) that performs the actual ETL transformation. The typical workflow: Crawler discovers schema → Catalog stores metadata → Job transforms data using that schema.

---

**Q93: What is the difference between `boto3.client` and `boto3.resource`?**

A: `client` provides a **low-level** 1:1 mapping to AWS API calls. Returns raw dicts. `resource` provides a **high-level** object-oriented interface with Python objects representing AWS resources (e.g., `s3.Bucket`, `s3.Object`). For complex workflows, `resource` is more readable. For full API access or batch operations, `client` is necessary.

---

**Q94: What is a Python decorator? Give a real data engineering use case.**

A: A decorator is a function that wraps another function to add behavior without modifying the original. Syntax: `@my_decorator` above the function definition. Real DE use case:
```python
def retry(func):
    def wrapper(*args, **kwargs):
        for attempt in range(3):
            try:
                return func(*args, **kwargs)
            except Exception as e:
                print(f"Attempt {attempt+1} failed: {e}")
        raise RuntimeError("All retries exhausted")
    return wrapper

@retry
def load_to_redshift(data):
    # might fail due to network issues
    pass
```
Retry decorators are standard in production pipeline code.

---

**Q95: What are Python iterators and generators? How do they benefit data pipelines?**

A: An **iterator** is an object with `__iter__()` and `__next__()` methods. A **generator** is a simpler way to create iterators using `yield`. Both produce values lazily — one at a time — without building the full collection in memory. In pipelines, this means you can process a 100GB file with constant memory by yielding rows one at a time.

---

**Q96: What is the difference between `yield` and `return` in Python?**

A: `return` exits the function and returns a single value. `yield` pauses the function and returns a value, but preserves the function's local state. The next call to `next()` resumes from where it left off. A function with `yield` is a generator function; calling it returns a generator object, not the result immediately.

---

**Q97: What is exception chaining in Python (the `raise ... from ...` pattern)?**

A: Exception chaining preserves the original cause when re-raising:
```python
try:
    data = json.loads(raw)
except json.JSONDecodeError as e:
    raise PipelineError("Invalid JSON in partition") from e
```
The `from e` attaches the original exception as `__cause__`. This gives full traceback context — you see both the pipeline error and the root JSON error. Essential for debugging production pipelines.

---

**Q98: What is `logging` in Python and why use it instead of `print()`?**

A: The `logging` module provides structured, configurable log output with levels (`DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`). Unlike `print()`, you can control what level gets output, write to files, add timestamps, and filter by module. Example:
```python
import logging
logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(message)s")
logging.info("Pipeline started")
logging.error("Load failed: %s", error_message)
```
In production pipelines, always use `logging` — `print()` is for scripts and debugging only.

---

**Q99: What is the `collections` module? Name 3 useful classes.**

A: `collections` extends Python's built-in data structures. Key classes: `Counter` — dict for counting hashable objects. `defaultdict` — dict that auto-creates a default value for missing keys. `OrderedDict` — dict that remembers insertion order (less relevant post Python 3.7+ since all dicts are ordered). `namedtuple` — tuple subclass with named fields, like a lightweight record class.

---

**Q100: What are the top Python performance best practices for data engineering?**

A: 
1. **Use generators** instead of lists for large data streams.
2. **Use vectorized Pandas operations** instead of `.apply()` row-by-row.
3. **Profile before optimizing** — use `cProfile` or `line_profiler`.
4. **Read files in chunks** (`chunksize` in Pandas, generators for text files).
5. **Use appropriate data types** — smaller dtypes (`int32`, `float32`) reduce memory.
6. **Use sets for membership checks** — O(1) vs O(n) for lists.
7. **Use `str.join()` for string concatenation** in loops — not `+=`.
8. **Use `multiprocessing`** for CPU-bound tasks, `threading`/`asyncio` for I/O-bound.
9. **Cache repeated computations** — compute once, store in a variable.
10. **Avoid loading entire files into memory** — stream or chunk-process instead.

---

> Congratulations! You've completed all 100 FAQ questions.
>
> Return to the coding questions: [../python_coding_questions/01_basics.md](../python_coding_questions/01_basics.md)
