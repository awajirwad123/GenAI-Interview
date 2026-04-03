# Amazon DE1 Real Interview Scenarios (Q1–Q15)

> **Level:** Intermediate to Amazon DE1 | **Target:** Amazon Data Engineer 1
> **These questions mirror actual Amazon phone screen and virtual onsite patterns.**
> Each problem is framed as a real DE task and requires clean, efficient Python.

---

## Q1: Identify Hourly Error Spikes From CloudTrail-Style Logs

**Scenario:** You receive a list of log entries, each with a timestamp and log level. Find all hours where the error count exceeds a threshold (spike detection).

```python
from collections import defaultdict

logs = [
    {"timestamp": "2024-01-15 08:03:22", "level": "ERROR"},
    {"timestamp": "2024-01-15 08:45:11", "level": "INFO"},
    {"timestamp": "2024-01-15 08:59:55", "level": "ERROR"},
    {"timestamp": "2024-01-15 09:01:00", "level": "ERROR"},
    {"timestamp": "2024-01-15 08:12:00", "level": "ERROR"},
    {"timestamp": "2024-01-15 09:22:00", "level": "ERROR"},
    {"timestamp": "2024-01-15 09:33:00", "level": "ERROR"},
    {"timestamp": "2024-01-15 10:00:00", "level": "INFO"},
]

THRESHOLD = 2   # alert if > 2 errors in an hour

def find_error_spikes(logs, threshold):
    hourly_errors = defaultdict(int)
    for entry in logs:
        if entry["level"] == "ERROR":
            hour = entry["timestamp"][:13]   # "2024-01-15 08"
            hourly_errors[hour] += 1

    spikes = {hour: count for hour, count in hourly_errors.items() if count > threshold}
    return spikes

spikes = find_error_spikes(logs, THRESHOLD)
for hour, count in sorted(spikes.items()):
    print(f"SPIKE at {hour}:xx — {count} errors")
# SPIKE at 2024-01-15 08:xx — 3 errors
# SPIKE at 2024-01-15 09:xx — 3 errors
```

**Explanation:** Extract hour prefix by slicing the timestamp string (first 13 chars = `YYYY-MM-DD HH`). Count errors per hour using `defaultdict`. Filter for counts exceeding threshold. This exact pattern appears in CloudWatch metric anomaly detection jobs.

---

## Q2: Efficiently Join Two Large Datasets Using a Hash Join

**Scenario:** You have two lists of dicts: `orders` and `customers`. Join them on `customer_id` (like a SQL LEFT JOIN). The customer list is small enough to fit in memory; orders are streamed.

```python
orders = [
    {"order_id": "o1", "customer_id": "c1", "amount": 500},
    {"order_id": "o2", "customer_id": "c3", "amount": 200},
    {"order_id": "o3", "customer_id": "c1", "amount": 150},
    {"order_id": "o4", "customer_id": "c9", "amount": 900},  # no matching customer
]

customers = [
    {"customer_id": "c1", "name": "Alice", "region": "US"},
    {"customer_id": "c2", "name": "Bob",   "region": "EU"},
    {"customer_id": "c3", "name": "Carol", "region": "APAC"},
]

def hash_join(orders, customers, key="customer_id"):
    # Build hash table from smaller dataset (customers)
    lookup = {c[key]: c for c in customers}

    result = []
    for order in orders:
        customer = lookup.get(order[key])
        if customer:
            merged = {**order, **customer}   # customer fields extend order
        else:
            merged = {**order, "name": None, "region": None}   # LEFT JOIN: keep order
        result.append(merged)
    return result

joined = hash_join(orders, customers)
for r in joined:
    print(r)
```

**Explanation:** Build a lookup dict from the smaller table (customers) — O(n). Stream the larger table (orders) and do O(1) lookups per row — O(m). Total O(n+m) vs O(n×m) for nested loops. This is exactly how Spark and most query engines implement hash joins. Always build the hash table on the smaller side.

---

## Q3: Detect and Report Schema Drift Between Two Pipeline Runs

**Scenario:** Your pipeline writes JSON records. Compare today's schema against yesterday's and report any new, removed, or type-changed fields.

```python
def get_schema(records):
    """Infer schema: field name → set of observed types."""
    schema = {}
    for rec in records:
        for key, val in rec.items():
            type_name = type(val).__name__
            schema.setdefault(key, set()).add(type_name)
    return schema

yesterday = [
    {"user_id": 1, "name": "Alice", "amount": 100.0, "active": True},
    {"user_id": 2, "name": "Bob",   "amount": 200.0, "active": False},
]

today = [
    {"user_id": 1, "name": "Alice", "amount": "100.00", "region": "US"},  # amount is now str!
    {"user_id": 2, "name": "Bob",   "amount": 200.0,    "region": "EU"},
]

def detect_schema_drift(old_records, new_records):
    old_schema = get_schema(old_records)
    new_schema = get_schema(new_records)

    old_keys = set(old_schema)
    new_keys = set(new_schema)

    report = []
    for field in sorted(new_keys - old_keys):
        report.append(f"NEW FIELD:     '{field}' — types: {new_schema[field]}")
    for field in sorted(old_keys - new_keys):
        report.append(f"DROPPED FIELD: '{field}' — was: {old_schema[field]}")
    for field in sorted(old_keys & new_keys):
        if old_schema[field] != new_schema[field]:
            report.append(f"TYPE CHANGE:   '{field}' — {old_schema[field]} → {new_schema[field]}")
    return report

for line in detect_schema_drift(yesterday, today):
    print(line)
# DROPPED FIELD: 'active'
# NEW FIELD:     'region'
# TYPE CHANGE:   'amount' — {'float'} → {'float', 'str'}
```

**Explanation:** Infer schema by sampling all records — collect all field names and their Python types. Set operations compare schemas. Type inference from actual data catches real drift that schema-only comparison misses. This is a simplified version of AWS Glue Crawler logic.

---

## Q4: Implement an Idempotent Upsert Into an In-Memory Store

**Scenario:** A pipeline may replay events. Implement an upsert that inserts new records and updates existing ones by primary key — safely rerunnable (idempotent).

```python
class IdempotentStore:
    def __init__(self):
        self._store = {}   # primary_key → record

    def upsert(self, record, pk_field="id"):
        pk = record[pk_field]
        if pk in self._store:
            # UPDATE: merge fields, newer record wins
            self._store[pk] = {**self._store[pk], **record}
            return "updated"
        else:
            # INSERT
            self._store[pk] = record
            return "inserted"

    def get_all(self):
        return list(self._store.values())

store = IdempotentStore()

events = [
    {"id": 1, "name": "Alice", "score": 80},
    {"id": 2, "name": "Bob",   "score": 70},
    {"id": 1, "name": "Alice", "score": 95},   # replay — update score
    {"id": 1, "name": "Alice", "score": 95},   # exact replay — no change
    {"id": 3, "name": "Carol", "score": 88},
]

for event in events:
    action = store.upsert(event)
    print(f"ID {event['id']}: {action}")

print("\nFinal store:")
for rec in store.get_all():
    print(rec)
```

**Explanation:** Dict keyed by primary key naturally implements upsert — key collision = update, new key = insert. Merging with `{**old, **new}` means new values overwrite old ones. Replaying the same event produces the same result — that's idempotency. Critical concept for exactly-once pipeline guarantees.

---

## Q5: Parse and Aggregate a Multi-Source Event Stream

**Scenario:** Events arrive from multiple sources. Compute per-source stats (count, total, min, max) in a single pass.

```python
events = [
    {"source": "app", "metric": "latency", "value": 120},
    {"source": "db",  "metric": "query_time", "value": 45},
    {"source": "app", "metric": "latency", "value": 200},
    {"source": "app", "metric": "latency", "value": 95},
    {"source": "db",  "metric": "query_time", "value": 300},
    {"source": "s3",  "metric": "download", "value": 500},
]

def aggregate_stream(events):
    stats = {}
    for evt in events:
        key = (evt["source"], evt["metric"])
        v = evt["value"]
        if key not in stats:
            stats[key] = {"count": 0, "total": 0, "min": v, "max": v}
        s = stats[key]
        s["count"] += 1
        s["total"] += v
        s["min"] = min(s["min"], v)
        s["max"] = max(s["max"], v)
    # Compute average at output time
    for key, s in sorted(stats.items()):
        s["avg"] = round(s["total"] / s["count"], 2)
        print(f"{key[0]}/{key[1]}: count={s['count']}, avg={s['avg']}, min={s['min']}, max={s['max']}")

aggregate_stream(events)
# app/latency:       count=3, avg=138.33, min=95, max=200
# db/query_time:     count=2, avg=172.5,  min=45, max=300
# s3/download:       count=1, avg=500.0,  min=500, max=500
```

**Explanation:** Composite key `(source, metric)` groups events. Accumulate count, total, min, max in one pass. Compute avg at the end (dividing accumulated values). Single-pass aggregation is how streaming systems (Flink, Kinesis) process events.

---

## Q6: Write a Pipeline Config Validator with Detailed Error Messages

**Scenario:** Before a pipeline starts, validate its config dict and collect ALL validation errors (don't stop at first failure).

```python
def validate_pipeline_config(config):
    errors = []
    required_fields = ["job_name", "source_bucket", "target_bucket", "schedule"]

    # Rule 1: Required fields
    for field in required_fields:
        if field not in config or not config[field]:
            errors.append(f"MISSING: '{field}' is required")

    # Rule 2: Bucket names must start with s3://
    for bucket_field in ["source_bucket", "target_bucket"]:
        val = config.get(bucket_field, "")
        if val and not val.startswith("s3://"):
            errors.append(f"INVALID: '{bucket_field}' must start with 's3://', got: '{val}'")

    # Rule 3: Parallelism must be positive integer
    if "parallelism" in config:
        if not isinstance(config["parallelism"], int) or config["parallelism"] < 1:
            errors.append(f"INVALID: 'parallelism' must be a positive integer, got: {config['parallelism']!r}")

    # Rule 4: Schedule format basic check
    schedule = config.get("schedule", "")
    valid_schedules = ["daily", "hourly", "weekly"]
    if schedule and schedule not in valid_schedules:
        errors.append(f"INVALID: 'schedule' must be one of {valid_schedules}, got: '{schedule}'")

    return errors

config = {
    "job_name": "ingest_orders",
    "source_bucket": "my-bucket/raw",     # missing s3://
    "target_bucket": "s3://data-lake",
    "schedule": "biweekly",               # invalid
    "parallelism": -2,                    # invalid
}

errors = validate_pipeline_config(config)
if errors:
    print("Config validation FAILED:")
    for e in errors:
        print(f"  ✗ {e}")
else:
    print("Config valid — starting pipeline")
```

**Explanation:** Collect ALL errors before returning — don't fail on first error. This is the "accumulator" validation pattern. Validating before pipeline start prevents partial runs. Amazon DE interviews often ask: "how would you validate inputs before a long-running job?"

---

## Q7: Implement Exponential Backoff Retry Logic

**Scenario:** An S3 API call may throttle. Implement retry with exponential backoff and jitter.

```python
import time
import random

def with_exponential_backoff(func, max_retries=5, base_delay=1.0, max_delay=32.0):
    """
    Retry a function with exponential backoff + jitter.
    Returns the result if successful, raises the last exception if exhausted.
    """
    last_error = None
    for attempt in range(max_retries):
        try:
            return func()
        except Exception as e:
            last_error = e
            if attempt == max_retries - 1:
                break   # last attempt — don't sleep
            delay = min(base_delay * (2 ** attempt), max_delay)
            jitter = random.uniform(0, delay * 0.1)   # 10% jitter
            total_delay = delay + jitter
            print(f"Attempt {attempt + 1} failed: {e}. Retrying in {total_delay:.2f}s...")
            time.sleep(total_delay)
    raise RuntimeError(f"All {max_retries} retries exhausted. Last error: {last_error}")

# Simulation
call_count = 0
def flaky_api_call():
    global call_count
    call_count += 1
    if call_count < 4:
        raise ConnectionError("Throttled by S3")
    return {"status": "success", "records": 5000}

result = with_exponential_backoff(flaky_api_call)
print("Result:", result)
# Attempt 1 failed: Throttled by S3. Retrying in 1.05s...
# Attempt 2 failed: Throttled by S3. Retrying in 2.09s...
# Attempt 3 failed: Throttled by S3. Retrying in 4.17s...
# Result: {'status': 'success', 'records': 5000}
```

**Explanation:** Delay doubles each retry (`base * 2^attempt`). Capped at `max_delay` to prevent unreasonably long waits. Jitter (random offset) prevents "thundering herd" — all retrying clients hitting at exactly the same time. AWS SDK uses this same pattern internally.

---

## Q8: Compute Percentile Statistics on Pipeline Metrics

**Scenario:** You have a list of job run times. Compute P50, P90, P99 percentiles for SLA reporting.

```python
import math

def percentile(data, p):
    """Return the p-th percentile of data (0-100)."""
    if not data:
        return None
    sorted_data = sorted(data)
    index = (p / 100) * (len(sorted_data) - 1)
    lower = int(index)
    upper = lower + 1
    if upper >= len(sorted_data):
        return sorted_data[-1]
    fraction = index - lower
    return round(sorted_data[lower] + fraction * (sorted_data[upper] - sorted_data[lower]), 2)

def pipeline_sla_report(run_times_ms):
    print(f"Count:   {len(run_times_ms)}")
    print(f"Min:     {min(run_times_ms)} ms")
    print(f"Max:     {max(run_times_ms)} ms")
    print(f"Mean:    {round(sum(run_times_ms)/len(run_times_ms), 2)} ms")
    print(f"P50:     {percentile(run_times_ms, 50)} ms")
    print(f"P90:     {percentile(run_times_ms, 90)} ms")
    print(f"P99:     {percentile(run_times_ms, 99)} ms")

run_times = [120, 135, 98, 210, 145, 320, 88, 175, 195, 430,
             102, 115, 188, 165, 240, 95, 155, 510, 130, 160]
pipeline_sla_report(run_times)
```

**Explanation:** Linear interpolation between adjacent sorted values gives smooth percentile estimates. P99 is the value below which 99% of observations fall — the key SLA metric. Amazon DE teams use percentiles constantly for pipeline health monitoring.

---

## Q9: Build a Simple In-Memory LRU Cache

**Scenario:** Cache S3 metadata lookups — keep the last N accessed items, evict the least recently used.

```python
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity):
        self.capacity = capacity
        self.cache = OrderedDict()   # maintains insertion order

    def get(self, key):
        if key not in self.cache:
            return None
        self.cache.move_to_end(key)   # mark as recently used
        return self.cache[key]

    def put(self, key, value):
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        if len(self.cache) > self.capacity:
            self.cache.popitem(last=False)   # evict least recently used (front)

# Usage: cache S3 object metadata (size, last_modified)
cache = LRUCache(3)
cache.put("s3://bucket/a", {"size": 100})
cache.put("s3://bucket/b", {"size": 200})
cache.put("s3://bucket/c", {"size": 300})
print(cache.get("s3://bucket/a"))   # hit — moves 'a' to end
cache.put("s3://bucket/d", {"size": 400})   # evicts 'b' (LRU)
print(cache.get("s3://bucket/b"))   # None — evicted
print(list(cache.cache.keys()))     # ['s3://bucket/c', 's3://bucket/a', 's3://bucket/d']
```

**Explanation:** `OrderedDict` maintains insertion order. `move_to_end()` marks recently accessed items. `popitem(last=False)` removes the front (oldest/least used). LRU cache design is a classic Amazon interview problem and maps directly to metadata caching in DE pipelines.

---

## Q10: Detect the First Duplicate Record in a Data Stream

**Scenario:** Records arrive in a stream. Return the first record ID that has been seen before (duplicate detection in real-time).

```python
def first_duplicate_in_stream(stream):
    seen = set()
    for record in stream:
        rid = record["id"]
        if rid in seen:
            return record   # first duplicate found
        seen.add(rid)
    return None   # no duplicates

stream = [
    {"id": "r1", "payload": "data_a"},
    {"id": "r2", "payload": "data_b"},
    {"id": "r3", "payload": "data_c"},
    {"id": "r2", "payload": "data_b"},   # duplicate!
    {"id": "r4", "payload": "data_d"},
]

dup = first_duplicate_in_stream(stream)
if dup:
    print(f"First duplicate: ID={dup['id']}")
else:
    print("No duplicates found")
# First duplicate: ID=r2
```

**Explanation:** `set` provides O(1) average-case membership test. Stop at the very first duplicate — don't scan the whole stream. This is O(n) time but O(n) space. For memory-constrained streaming, a Bloom filter provides O(1) space with a small false positive rate — mention this as a follow-up.

---

## Q11: Normalize Nested JSON Records for a Relational Target

**Scenario:** An API returns deeply nested customer records. Flatten them to a tabular format for loading into Redshift.

```python
def normalize_record(record, prefix=""):
    """Recursively flatten nested dict. Lists become pipe-delimited strings."""
    flat = {}
    for key, val in record.items():
        full_key = f"{prefix}_{key}" if prefix else key
        if isinstance(val, dict):
            flat.update(normalize_record(val, full_key))
        elif isinstance(val, list):
            flat[full_key] = "|".join(str(v) for v in val)
        else:
            flat[full_key] = val
    return flat

record = {
    "id": 1,
    "user": {
        "name": "Alice",
        "address": {
            "city": "Seattle",
            "zip": "98101"
        }
    },
    "tags": ["prime", "enterprise"],
    "score": 95.5
}

normalized = normalize_record(record)
for k, v in normalized.items():
    print(f"{k:<30} = {v}")
# id                             = 1
# user_name                      = Alice
# user_address_city              = Seattle
# user_address_zip               = 98101
# tags                           = prime|enterprise
# score                          = 95.5
```

**Explanation:** Recursive flattening with underscore-joined keys. Lists become pipe-delimited (a common convention for array columns in relational targets). This mirrors AWS Glue DynamicFrame's `relationalize()` operation.

---

## Q12: Compute a Sliding SLA Compliance Rate

**Scenario:** Keep a rolling 5-job window. For each new job result, report the SLA compliance rate in that window.

```python
from collections import deque

def sliding_sla_compliance(results, window_size, threshold_ms):
    """
    results: list of (job_name, duration_ms)
    Returns compliance rate for each window position.
    """
    window = deque()
    compliance_history = []

    for job, duration in results:
        window.append(duration <= threshold_ms)   # True = within SLA
        if len(window) > window_size:
            window.popleft()   # evict oldest

        compliant = sum(window)
        rate = compliant / len(window) * 100
        compliance_history.append({
            "job": job,
            "within_sla": duration <= threshold_ms,
            "window_compliance": f"{rate:.0f}%"
        })

    return compliance_history

SLA_MS = 200
WINDOW = 5

results = [
    ("j1", 150), ("j2", 250), ("j3", 180), ("j4", 310),
    ("j5", 190), ("j6", 170), ("j7", 230), ("j8", 160),
]

for r in sliding_sla_compliance(results, WINDOW, SLA_MS):
    flag = "✓" if r["within_sla"] else "✗"
    print(f"{flag} {r['job']:4}  |  Rolling SLA: {r['window_compliance']}")
```

**Explanation:** `deque` with `maxlen` is perfect for sliding windows — `popleft()` is O(1). At each step, count `True` values in window for compliance rate. `deque` is more efficient than slicing a list for window operations.

---

## Q13: Partition and Write Records to Multiple Output Files by Key

**Scenario:** Write pipeline output records to separate files partitioned by `region` — simulating S3 prefix partitioning.

```python
import json
import os

def partition_and_write(records, output_dir, partition_key):
    os.makedirs(output_dir, exist_ok=True)
    partitions = {}

    # Group records by partition key
    for rec in records:
        key = str(rec.get(partition_key, "unknown"))
        partitions.setdefault(key, []).append(rec)

    # Write each partition to its own file
    written = {}
    for key, recs in partitions.items():
        filename = os.path.join(output_dir, f"partition_{key}.jsonl")
        with open(filename, "w") as f:
            for rec in recs:
                f.write(json.dumps(rec) + "\n")
        written[key] = len(recs)
        print(f"Written: {filename}  ({len(recs)} records)")

    return written

records = [
    {"id": 1, "region": "US",   "amount": 100},
    {"id": 2, "region": "EU",   "amount": 200},
    {"id": 3, "region": "US",   "amount": 300},
    {"id": 4, "region": "APAC", "amount": 150},
    {"id": 5, "region": "EU",   "amount": 250},
]

summary = partition_and_write(records, "output/by_region", "region")
print("\nSummary:", summary)
```

**Explanation:** Group records by partition key into a dict, then write each group to its own file using JSON Lines format (one JSON object per line — common for streaming-compatible files). This mirrors how Spark and Glue write partitioned Parquet: `s3://bucket/data/region=US/`.

---

## Q14: Build a Pipeline Lineage Tracker

**Scenario:** Track which input files contributed to which output files in a pipeline run (data lineage).

```python
class LineageTracker:
    def __init__(self):
        self.lineage = {}   # output → set of inputs

    def record(self, inputs, output):
        """Record that `output` was produced from `inputs`."""
        if output not in self.lineage:
            self.lineage[output] = set()
        for inp in inputs:
            self.lineage[output].add(inp)

    def get_sources(self, output):
        """Return all direct inputs for an output."""
        return self.lineage.get(output, set())

    def full_lineage(self, output, visited=None):
        """Recursively resolve all upstream sources."""
        if visited is None:
            visited = set()
        if output in visited:
            return set()
        visited.add(output)
        sources = self.get_sources(output)
        all_sources = set(sources)
        for source in sources:
            all_sources |= self.full_lineage(source, visited)
        return all_sources

tracker = LineageTracker()
tracker.record(["s3://raw/orders_jan.csv", "s3://raw/orders_feb.csv"], "s3://staging/orders_combined.parquet")
tracker.record(["s3://staging/orders_combined.parquet", "s3://dim/customers.csv"], "s3://analytics/order_summary.parquet")

print("Direct sources of order_summary:")
print(tracker.get_sources("s3://analytics/order_summary.parquet"))

print("\nFull lineage of order_summary:")
for src in sorted(tracker.full_lineage("s3://analytics/order_summary.parquet")):
    print(f"  {src}")
```

**Explanation:** A dict maps each output to its direct inputs. Recursive `full_lineage()` follows the chain back to raw sources. A `visited` set prevents infinite loops (circular lineage). Data lineage is a core DE2 concept — interviewers ask "how do you know where your data came from?"

---

## Q15: Simulate a Rate-Limited API Caller (Token Bucket)

**Scenario:** Call a downstream API that allows max 10 requests per second. Implement a rate limiter in Python.

```python
import time

class TokenBucketRateLimiter:
    def __init__(self, rate_per_second, burst_limit):
        self.rate = rate_per_second      # tokens added per second
        self.burst = burst_limit         # max tokens in bucket
        self.tokens = burst_limit        # start full
        self.last_refill = time.time()

    def _refill(self):
        now = time.time()
        elapsed = now - self.last_refill
        new_tokens = elapsed * self.rate
        self.tokens = min(self.burst, self.tokens + new_tokens)
        self.last_refill = now

    def acquire(self):
        """Try to consume one token. Block if necessary."""
        while True:
            self._refill()
            if self.tokens >= 1:
                self.tokens -= 1
                return True
            # Not enough tokens — wait for refill
            wait_time = (1 - self.tokens) / self.rate
            time.sleep(wait_time)

def call_api(url):
    return {"url": url, "status": 200}

limiter = TokenBucketRateLimiter(rate_per_second=5, burst_limit=5)

urls = [f"https://api.example.com/record/{i}" for i in range(8)]
for url in urls:
    limiter.acquire()   # blocks if rate exceeded
    result = call_api(url)
    print(f"{time.strftime('%H:%M:%S')} - Fetched {url}")
```

**Explanation:** Token bucket algorithm: tokens refill at `rate/second`, capped at `burst`. Each API call consumes one token. If no tokens available, sleep until enough accumulate. The burst allows a short surge above steady-state rate. Amazon APIs (S3, DynamoDB) enforce rate limits — this pattern prevents `ThrottlingException`.

---

> **Key Amazon DE1 Patterns Across These Questions:**
>
> | Scenario                    | Core Pattern                     |
> |-----------------------------|----------------------------------|
> | Error spike detection       | Time-bucketing aggregation       |
> | Hash join                   | Build hash table on smaller side |
> | Schema drift                | Set operations on key sets       |
> | Idempotent upsert           | Dict-as-primary-key store        |
> | Exponential backoff         | `base * 2^attempt` + jitter      |
> | Percentile computation      | Sorted interpolation             |
> | LRU cache                   | OrderedDict + move_to_end        |
> | Normalize JSON              | Recursive flatten + prefix keys  |
> | Sliding SLA window          | deque for O(1) window operations |
> | Rate limiting               | Token bucket algorithm           |
>
> Return to: [08_two_pointer_sliding_window.md](08_two_pointer_sliding_window.md) | [10_boto3_sql_patterns.md](10_boto3_sql_patterns.md)
