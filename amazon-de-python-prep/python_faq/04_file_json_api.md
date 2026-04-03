# Python FAQ: File Handling, JSON, CSV & APIs (Q56–Q75)

---

**Q56: How do you open and read a file in Python?**

A: Use `open(filepath, mode)` in a `with` block:
```python
with open("data.txt", "r") as f:
    content = f.read()        # read entire file as a string
    # OR
    lines = f.readlines()     # read all lines as a list
    # OR
    for line in f:            # read one line at a time (memory-efficient)
        process(line)
```
The `with` block ensures the file is closed automatically, even if an error occurs.

---

**Q57: What are the different file open modes?**

A: Common modes: `"r"` (read, default), `"w"` (write, creates/overwrites), `"a"` (append), `"x"` (create new, fails if exists), `"rb"`/`"wb"` (binary read/write). Always use `"r"` and `"w"` for text files and binary modes for images, Parquet, or compressed files.

---

**Q58: What is the difference between `read()`, `readline()`, and `readlines()`?**

A: `read()` returns the entire file as one string. `readline()` returns one line per call (including the `\n`). `readlines()` returns a list of all lines. For large files, iterating with `for line in f:` is best — it reads one line at a time without loading everything into memory.

---

**Q59: Why should you always use `with open(...)` instead of `f = open(...)`?**

A: `with` is a context manager that guarantees the file is closed when the block exits — even if an exception is raised. Without `with`, you must manually call `f.close()` and risk leaving file handles open on errors, causing resource leaks and potential data corruption.

---

**Q60: How do you write to a file in Python?**

A:
```python
lines = ["Alice,Engineering\n", "Bob,Marketing\n"]

with open("output.csv", "w") as f:
    f.write("name,department\n")     # write header
    f.writelines(lines)              # write many lines at once
```
`write()` writes a single string. `writelines()` takes an iterable of strings. Neither adds newlines automatically — include `\n` yourself.

---

**Q61: What is the `csv` module and what are its main classes?**

A: The `csv` module provides `csv.reader` (reads rows as lists), `csv.writer` (writes rows as lists), `csv.DictReader` (reads rows as dicts using the header row), and `csv.DictWriter` (writes dicts as rows). Prefer `DictReader`/`DictWriter` — they make code more readable and self-documenting.

---

**Q62: How do you handle CSV files with a non-comma delimiter?**

A:
```python
import csv

with open("data.tsv", "r") as f:
    reader = csv.reader(f, delimiter="\t")
    for row in reader:
        print(row)
```
Pass `delimiter="\t"` for tab-separated, `delimiter="|"` for pipe-separated. The `csv` module handles quoted fields and escape characters correctly.

---

**Q63: What is `json.loads()` vs `json.load()`?**

A: `json.loads(string)` parses a JSON **string** and returns a Python object. `json.load(file_object)` reads from an open **file** and parses it. Similarly, `json.dumps(obj)` serializes to a string, and `json.dump(obj, file)` writes to a file. The "s" stands for "string".

---

**Q64: How do you pretty-print JSON output?**

A:
```python
import json

data = {"name": "Alice", "orders": [1, 2, 3]}
print(json.dumps(data, indent=2, sort_keys=True))
```
`indent=2` adds readable indentation. `sort_keys=True` alphabetizes keys. Useful for readable logs, config files, and API responses.

---

**Q65: How do you handle JSON with non-serializable types (like datetime)?**

A: By default, `json.dumps()` raises `TypeError` for objects like `datetime`. Solutions: (1) convert to string before serializing: `dt.isoformat()`, or (2) provide a custom `default` function:
```python
import json
from datetime import datetime

def json_serializer(obj):
    if isinstance(obj, datetime):
        return obj.isoformat()
    raise TypeError(f"Type {type(obj)} not serializable")

json.dumps({"ts": datetime.now()}, default=json_serializer)
```

---

**Q66: What is `requests` library and what is it used for?**

A: `requests` is a third-party library for making HTTP requests. It simplifies GET, POST, PUT, DELETE calls to REST APIs. It handles encoding, headers, authentication, and response parsing automatically. It's the standard choice for API integration in Python data pipelines.

---

**Q67: How do you make a GET request and parse the JSON response?**

A:
```python
import requests

response = requests.get("https://api.example.com/data", params={"limit": 100})
response.raise_for_status()   # raises HTTPError if 4xx/5xx

data = response.json()        # parse JSON body as dict/list
print(data)
```
`raise_for_status()` is critical — always call it before processing, so errors don't silently fail.

---

**Q68: How do you send a POST request with a JSON body?**

A:
```python
import requests

payload = {"user_id": 42, "event": "purchase", "amount": 99.99}

response = requests.post(
    "https://api.example.com/events",
    json=payload,                    # automatically sets Content-Type: application/json
    headers={"Authorization": "Bearer my-token"}
)
response.raise_for_status()
print(response.status_code)
```
Using `json=payload` (not `data=`) automatically serializes the dict and sets the correct content type.

---

**Q69: What is a request timeout and why is it important?**

A: A timeout limits how long `requests` will wait for a response before raising a `Timeout` exception. Without a timeout, your pipeline could hang indefinitely waiting for a slow or unresponsive API. Always set timeouts:
```python
response = requests.get(url, timeout=10)  # 10 seconds
```
Pass a tuple `(connect_timeout, read_timeout)` for fine-grained control.

---

**Q70: How do you handle errors in API calls?**

A:
```python
import requests

try:
    response = requests.get("https://api.example.com/data", timeout=10)
    response.raise_for_status()
    return response.json()
except requests.exceptions.Timeout:
    print("Request timed out")
except requests.exceptions.HTTPError as e:
    print(f"HTTP error: {e.response.status_code}")
except requests.exceptions.RequestException as e:
    print(f"Request failed: {e}")
```
`RequestException` is the base class for all requests errors — use it as a catch-all fallback.

---

**Q71: What are HTTP status codes you must know for API handling?**

A: `200 OK` — success. `201 Created` — resource created. `400 Bad Request` — invalid input. `401 Unauthorized` — missing/invalid auth. `403 Forbidden` — valid auth but no permission. `404 Not Found` — resource doesn't exist. `429 Too Many Requests` — rate limited. `500 Internal Server Error` — server-side failure. `503 Service Unavailable` — server overloaded/down.

---

**Q72: What is pagination in API calls and how do you handle it?**

A: Many APIs return results in pages to avoid huge responses. You handle pagination by looping, passing a page/cursor parameter until no more data is returned:
```python
all_records = []
page = 1
while True:
    resp = requests.get(url, params={"page": page, "limit": 100}).json()
    if not resp["records"]:
        break
    all_records.extend(resp["records"])
    page += 1
```
Always check for a "next page" indicator from the API rather than blindly incrementing.

---

**Q73: What is the `os` module? Name 3 useful functions for file handling.**

A: The `os` module provides OS-level operations. Key functions: `os.path.exists(path)` — check if file/dir exists. `os.listdir(path)` — list directory contents. `os.makedirs(path, exist_ok=True)` — create directories recursively. `os.path.join()` — build paths safely across OS. `os.path.getsize(path)` — get file size in bytes.

---

**Q74: What is `pathlib` and how does it improve on `os.path`?**

A: `pathlib.Path` is the modern object-oriented approach to file paths (Python 3.4+). Instead of string manipulation, you use path objects:
```python
from pathlib import Path

p = Path("data") / "output" / "results.csv"
print(p.exists())
print(p.suffix)     # ".csv"
print(p.stem)       # "results"
```
It's cleaner, cross-platform, and has methods like `.read_text()`, `.write_text()`, `.glob()` built in.

---

**Q75: How do you read all JSON files from a directory?**

A:
```python
import json
from pathlib import Path

data_dir = Path("data/partitions")
all_records = []

for json_file in data_dir.glob("*.json"):
    with open(json_file, "r") as f:
        records = json.load(f)
        all_records.extend(records)

print(f"Total records: {len(all_records)}")
```
`Path.glob("*.json")` returns all matching files lazily. This is how you process partitioned data from S3 downloads or local staging areas.

---

> Next: [05_data_engineering_python.md](05_data_engineering_python.md)
