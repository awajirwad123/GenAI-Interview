# Python FAQ: Testing, Type Hints, Logging & Dev Tools (Q126–Q150)

---

## PYTEST & UNIT TESTING (Q126–Q137)

---

**Q126: Why use pytest instead of unittest for Python testing?**

A: `pytest` requires less boilerplate — no class inheritance, no `self.assert*` methods. Test functions just use plain `assert`. It has better error messages, supports fixtures natively, has a rich plugin ecosystem (`pytest-mock`, `pytest-cov`), and auto-discovers tests. `unittest` is built-in but verbose. Amazon expects production code to be testable — `pytest` is the industry standard.

---

**Q127: What is the basic structure of a pytest test?**

A:
```python
# test_pipeline.py

def add(a, b):
    return a + b

def test_add_positive():
    assert add(2, 3) == 5

def test_add_negative():
    assert add(-1, -2) == -3

def test_add_zero():
    assert add(0, 5) == 5
```
Run with: `pytest test_pipeline.py -v`

Rules: Files must start/end with `test_`, functions must start with `test_`. No class needed. `assert` statements produce detailed failure messages.

---

**Q128: What is a pytest fixture and how do you use it?**

A: A fixture is a function that provides test data or setup/teardown logic. Declared with `@pytest.fixture`, injected into tests by name:
```python
import pytest

@pytest.fixture
def sample_records():
    return [
        {"id": 1, "region": "US", "amount": 100},
        {"id": 2, "region": "EU", "amount": 200},
        {"id": 3, "region": "US", "amount": 300},
    ]

def test_filter_us(sample_records):
    us_records = [r for r in sample_records if r["region"] == "US"]
    assert len(us_records) == 2
    assert all(r["region"] == "US" for r in us_records)
```
Fixtures can have scope: `function` (default, re-created per test), `module`, `session` (created once for entire test run). The `session` scope is used for expensive resources like database connections.

---

**Q129: What is parametrize and why is it useful?**

A: `@pytest.mark.parametrize` runs the same test with multiple input sets — avoids copy-pasting test functions:
```python
import pytest

def is_valid_region(region):
    return region in {"US", "EU", "APAC", "LATAM"}

@pytest.mark.parametrize("region,expected", [
    ("US",    True),
    ("EU",    True),
    ("APAC",  True),
    ("XX",    False),
    ("",      False),
    (None,    False),
])
def test_is_valid_region(region, expected):
    assert is_valid_region(region) == expected
```
Each parameter set is an independent test case. pytest names them clearly in output: `test_is_valid_region[US-True]`, `test_is_valid_region[XX-False]`, etc.

---

**Q130: How do you test that a function raises a specific exception?**

A:
```python
import pytest

def validate_row_count(actual, expected):
    if actual != expected:
        raise ValueError(f"Row count mismatch: {actual} != {expected}")

def test_raises_on_mismatch():
    with pytest.raises(ValueError) as exc_info:
        validate_row_count(900, 1000)
    assert "Row count mismatch" in str(exc_info.value)

def test_no_error_on_match():
    validate_row_count(1000, 1000)   # should not raise
```
`pytest.raises(ExceptionType)` is a context manager. `exc_info.value` holds the actual exception object for further assertions.

---

**Q131: How do you mock an external API call in pytest?**

A: Use `unittest.mock.patch` (or `pytest-mock`'s `mocker`) to replace the real function with a fake:
```python
from unittest.mock import patch, MagicMock
import pytest

def fetch_from_api(url):
    import requests
    response = requests.get(url, timeout=10)
    response.raise_for_status()
    return response.json()

def test_fetch_success():
    mock_response = MagicMock()
    mock_response.json.return_value = {"records": 100}
    mock_response.raise_for_status.return_value = None

    with patch("requests.get", return_value=mock_response):
        result = fetch_from_api("https://api.example.com/data")

    assert result == {"records": 100}

def test_fetch_timeout():
    import requests
    with patch("requests.get", side_effect=requests.exceptions.Timeout):
        with pytest.raises(requests.exceptions.Timeout):
            fetch_from_api("https://api.example.com/data")
```
`MagicMock` creates a fake object. `return_value` controls what the mock returns. `side_effect` raises an exception when called.

---

**Q132: How do you mock boto3 S3 calls?**

A:
```python
from unittest.mock import patch, MagicMock

def check_s3_file_exists(bucket, key):
    import boto3
    from botocore.exceptions import ClientError
    s3 = boto3.client("s3")
    try:
        s3.head_object(Bucket=bucket, Key=key)
        return True
    except ClientError:
        return False

def test_s3_file_exists():
    with patch("boto3.client") as mock_boto:
        mock_s3 = MagicMock()
        mock_boto.return_value = mock_s3
        mock_s3.head_object.return_value = {}   # success
        assert check_s3_file_exists("my-bucket", "data.csv") is True

def test_s3_file_missing():
    from botocore.exceptions import ClientError
    with patch("boto3.client") as mock_boto:
        mock_s3 = MagicMock()
        mock_boto.return_value = mock_s3
        mock_s3.head_object.side_effect = ClientError(
            {"Error": {"Code": "404", "Message": "Not Found"}}, "HeadObject"
        )
        assert check_s3_file_exists("my-bucket", "missing.csv") is False
```
Mocking boto3 prevents real AWS API calls in tests. The `moto` library (not shown) provides a more complete S3 mock that simulates the actual S3 API.

---

**Q133: What is `conftest.py` in pytest?**

A: `conftest.py` is a pytest plugin file automatically loaded by pytest. Fixtures defined there are available to all tests in the same directory (and subdirectories) without importing. Use it for shared fixtures, hooks, and plugins across an entire project:
```
tests/
  conftest.py         ← shared fixtures here
  test_ingest.py      ← can use fixtures from conftest.py
  test_transform.py   ← same
  etl/
    conftest.py       ← override/add fixtures for this subdirectory
    test_utils.py
```
Never import from `conftest.py` directly — pytest discovers it automatically.

---

**Q134: What does 100% code coverage mean and is it always the goal?**

A: Code coverage (`pytest --cov`) measures what percentage of code lines are executed by tests. 100% coverage means every line ran — but it doesn't mean every case is tested. You can have 100% line coverage with zero assertion coverage. Good targets: 80%+ for production code. Focus on testing boundary conditions, error paths, and critical transformations — not chasing 100% for trivial getters/setters.

---

**Q135: What is the AAA pattern in test writing?**

A: **Arrange → Act → Assert** is the standard test structure:
```python
def test_aggregate_by_region():
    # ARRANGE — set up test data and context
    transactions = [
        {"region": "US", "amount": 100},
        {"region": "EU", "amount": 200},
        {"region": "US", "amount": 300},
    ]

    # ACT — call the function under test
    result = aggregate_by_region(transactions)

    # ASSERT — verify the outcome
    assert result["US"] == 400
    assert result["EU"] == 200
    assert len(result) == 2
```
Keep each section clearly separated. Tests failing in the "Arrange" section indicate a fixture problem, not a code bug.

---

**Q136: What is test isolation and why does it matter in pipeline testing?**

A: Test isolation means each test is self-contained — it doesn't depend on the state left by other tests. In pipeline tests: reset shared state between tests (use `@pytest.fixture` with `yield` for setup/teardown), mock external dependencies (S3, databases, APIs), use fresh temporary directories (`tmp_path` fixture), and never share mutable module-level state. Non-isolated tests produce intermittent failures that are notoriously hard to debug.

---

**Q137: What is `pytest.ini` or `pyproject.toml` pytest configuration?**

A: Configure pytest behavior in `pytest.ini`:
```ini
[pytest]
testpaths = tests
python_files = test_*.py *_test.py
python_functions = test_*
addopts = -v --tb=short --strict-markers
markers =
    slow: marks test as slow (run with -m slow)
    integration: marks as integration test
```
`addopts` applies options automatically. `markers` declares custom marks. Use `@pytest.mark.slow` to tag expensive tests and skip them in fast CI: `pytest -m "not slow"`.

---

## TYPE HINTS (Q138–Q144)

---

**Q138: What are type hints in Python and why use them?**

A: Type hints are annotations (Python 3.5+) that declare expected types for variables, parameters, and return values. They're not enforced at runtime — they're documentation + tooling signals:
```python
from typing import List, Dict, Optional

def aggregate(records: List[Dict[str, float]], field: str) -> float:
    return sum(r[field] for r in records if field in r)
```
Benefits: IDE autocomplete, `mypy` static type checking catches bugs before runtime, better code readability. Amazon production codebases use type hints extensively.

---

**Q139: What are the most common types from the `typing` module?**

A:
```python
from typing import List, Dict, Tuple, Set, Optional, Union, Any, Callable, Generator

def process(
    items: List[str],                    # list of strings
    config: Dict[str, int],              # dict str→int
    threshold: Optional[float] = None,  # float or None
    callback: Optional[Callable] = None # callable or None
) -> Tuple[int, List[str]]:             # returns (count, list)
    ...

def stream() -> Generator[str, None, None]:
    yield "record"
```
Python 3.9+: use built-in `list[str]`, `dict[str, int]` directly without importing from `typing`.

---

**Q140: What is `Optional[T]`?**

A: `Optional[T]` is shorthand for `Union[T, None]` — the value is either type T or `None`. Always use `Optional` when `None` is a valid value:
```python
from typing import Optional

def find_record(record_id: str) -> Optional[dict]:
    # Returns a dict if found, None otherwise
    ...
```
A common mistake is annotating nullable values as `str` instead of `Optional[str]` — mypy catches this.

---

**Q141: What is `Union[A, B]` and when do you use it?**

A: `Union[A, B]` means the value can be type A or type B. Use when a function legitimately accepts or returns multiple types:
```python
from typing import Union

def parse_value(raw: str) -> Union[int, float, str]:
    try:
        return int(raw)
    except ValueError:
        try:
            return float(raw)
        except ValueError:
            return raw
```
Python 3.10+: use `int | float | str` syntax directly without `Union`.

---

**Q142: What is `TypedDict` and when is it useful for DE?**

A: `TypedDict` defines a dict with specific key names and types — used for structured records without creating a full class:
```python
from typing import TypedDict

class PipelineRecord(TypedDict):
    job_id: str
    status: str
    rows_loaded: int
    duration_sec: float

def process_run(run: PipelineRecord) -> bool:
    return run["status"] == "success" and run["rows_loaded"] > 0

run: PipelineRecord = {
    "job_id": "job_001",
    "status": "success",
    "rows_loaded": 50000,
    "duration_sec": 12.5
}
```
Great for JSON API response types and pipeline record schemas — gives IDE completion and mypy checking on dict access.

---

**Q143: What is `mypy` and how do you run it?**

A: `mypy` is a static type checker for Python — it catches type errors before running code:
```
pip install mypy
mypy pipeline.py          # check one file
mypy src/                 # check entire directory
mypy --strict pipeline.py # strict mode (no implicit Any)
```
Common mypy errors: `Argument has incompatible type "str"; expected "int"`, `Item "None" of "Optional[str]" has no attribute "upper"`. Fix: add None checks before accessing optional values. Integrates with VS Code, PyCharm, and CI pipelines.

---

**Q144: What is the difference between type hints and runtime type checking?**

A: Type hints are **static** — checked by mypy/editors, not enforced at runtime. Python ignores them during execution. `isinstance()` is **runtime** checking. For production pipelines, use type hints for documentation + mypy for development-time safety, and `isinstance()` / Pydantic at runtime boundaries (API inputs, file reads) where untrusted data enters the system.

---

## LOGGING & STRUCTURED LOGGING (Q145–Q150)

---

**Q145: What is the correct way to configure logging in Python?**

A:
```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s %(levelname)-8s %(name)-20s %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S"
)

logger = logging.getLogger(__name__)   # always use __name__

logger.debug("Debug detail — not shown at INFO level")
logger.info("Pipeline started")
logger.warning("Slow query detected: %s ms", 450)
logger.error("Failed to write to S3: %s", "permission denied")
logger.critical("Pipeline crashed — manual intervention required")
```
Always use `logging.getLogger(__name__)` — not the root logger `logging.info()`. `__name__` gives module-qualified names in log output, making it easy to trace which module logged each message.

---

**Q146: What are Python's logging levels and when to use each?**

A: Five standard levels (lowest to highest):
- `DEBUG` — detailed diagnostic info during development. Disabled in production.
- `INFO` — normal pipeline progress: "job started", "1000 records loaded".
- `WARNING` — unexpected but non-fatal: slow query, missing optional field, retrying.
- `ERROR` — a step failed but the pipeline may continue: "failed to load partition 3".
- `CRITICAL` — catastrophic failure requiring immediate attention: data corruption, credential expiry.

Production pipelines: `INFO` level. CI/debugging: `DEBUG`. Set level per handler to send `ERROR+` to an alert channel and `INFO+` to CloudWatch.

---

**Q147: What is structured logging and why does it matter for DE?**

A: Structured logging emits log entries as JSON dicts instead of plain strings — making them machine-parseable by CloudWatch Logs, Elasticsearch, Datadog:
```python
import logging
import json

class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_obj = {
            "timestamp": self.formatTime(record),
            "level": record.levelname,
            "logger": record.name,
            "message": record.getMessage(),
            "job_id": getattr(record, "job_id", None),
        }
        if record.exc_info:
            log_obj["exception"] = self.formatException(record.exc_info)
        return json.dumps(log_obj)

handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logger = logging.getLogger("pipeline")
logger.addHandler(handler)
logger.setLevel(logging.INFO)

# Add context via extra
logger.info("Batch complete", extra={"job_id": "job_001", "rows": 50000})
```
Structured logs enable: `"filter logs where job_id == 'job_001'"`, SLA dashboards, automated alerting on error patterns.

---

**Q148: What is a logging handler and how do you write to multiple destinations?**

A: A handler determines where log records go. Add multiple handlers to send logs to different destinations:
```python
import logging

logger = logging.getLogger("etl_pipeline")
logger.setLevel(logging.DEBUG)

# Console handler (INFO and above)
console = logging.StreamHandler()
console.setLevel(logging.INFO)
console.setFormatter(logging.Formatter("%(levelname)s %(message)s"))

# File handler (all levels, DEBUG and above)
file_handler = logging.FileHandler("pipeline.log")
file_handler.setLevel(logging.DEBUG)
file_handler.setFormatter(logging.Formatter("%(asctime)s %(levelname)s %(message)s"))

logger.addHandler(console)
logger.addHandler(file_handler)

# INFO → both console and file; DEBUG → only file
logger.debug("Detailed: processing row 42")
logger.info("Batch started")
logger.error("Partition 3 failed")
```
`RotatingFileHandler` automatically rotates log files when they exceed a size limit — use in production to prevent disk fill.

---

**Q149: What are virtual environments and why are they essential?**

A: A virtual environment is an isolated Python installation with its own packages, independent from the system Python. Essential for:
- Preventing package version conflicts between projects
- Reproducible builds (`pip freeze > requirements.txt`)
- Clean testing environments

```bash
# Create
python -m venv .venv

# Activate
.venv\Scripts\activate        # Windows
source .venv/bin/activate     # Linux/macOS

# Install packages
pip install pandas boto3 pytest requests

# Freeze dependencies
pip freeze > requirements.txt

# Install from requirements
pip install -r requirements.txt

# Deactivate
deactivate
```
In production: use `requirements.txt` with pinned versions (`boto3==1.34.0`). For AWS Lambda: build zipped deployment packages from a virtual environment.

---

**Q150: What is the difference between `requirements.txt` and `pyproject.toml`?**

A: `requirements.txt` lists exact package versions for reproducible installs — used in most production Python projects:
```
boto3==1.34.0
pandas==2.1.4
requests==2.31.0
pytest==7.4.3
```

`pyproject.toml` is the modern standard (PEP 517/518) for defining project metadata, build system, and dependencies together. Supports version ranges for libraries:
```toml
[project]
name = "my-pipeline"
version = "1.0.0"
requires-python = ">=3.9"
dependencies = [
    "boto3>=1.34,<2.0",
    "pandas>=2.0",
    "requests>=2.28",
]

[project.optional-dependencies]
dev = ["pytest>=7.0", "mypy>=1.0"]

[tool.pytest.ini_options]
testpaths = ["tests"]
```
Use `requirements.txt` for deployments (exact pins). Use `pyproject.toml` for libraries and modern project structure.

---

> **You have now completed all 150 FAQ questions.**
>
> | Section                        | Questions  | Count |
> |--------------------------------|------------|-------|
> | Core Python                    | Q1–Q25     | 25    |
> | Memory & GIL                   | Q26–Q40    | 15    |
> | Concurrency                    | Q41–Q55    | 15    |
> | File, JSON & API               | Q56–Q75    | 20    |
> | DE Python (Pandas/boto3/AWS)   | Q76–Q100   | 25    |
> | Decorators, functools, itertools | Q101–Q125 | 25    |
> | Testing, Type Hints, Logging   | Q126–Q150  | 25    |
>
> Return to coding questions: [../python_coding_questions/09_amazon_de1_scenarios.md](../python_coding_questions/09_amazon_de1_scenarios.md)
