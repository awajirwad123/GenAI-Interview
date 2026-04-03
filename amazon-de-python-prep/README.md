# Amazon Data Engineer Python Prep

A one-stop Python preparation system for **Amazon DE1/DE2** interviews.

---

## Candidate Profile
- **Experience:** 4.8 years in Data Engineering
- **Current Python Level:** 1.5 / 5
- **Target Role:** Amazon Data Engineer (DE1/DE2)

---

## Repository Structure

```
amazon-de-python-prep/
│
├── python_coding_questions/
│   ├── 01_basics.md                    # Q1–Q10:  Variables, loops, functions, conditions
│   ├── 02_strings_lists.md             # Q11–Q20: String ops, list manipulation
│   ├── 03_dict_set.md                  # Q21–Q30: Dicts, sets, deduplication
│   ├── 04_data_engineering_patterns.md # Q31–Q42: ETL, logs, JSON, file I/O, aggregation
│   ├── 05_advanced_basics.md           # Q43–Q50: Generators, OOP, error handling
│   ├── 06_lists_tuples_dicts_strings.md# 50 questions: list/tuple/dict/string for DE2
│   ├── 07_classic_patterns.md          # 50 questions: palindrome, anagram, binary search, sort
│   ├── 08_two_pointer_sliding_window.md# 20 questions: two pointers + sliding window algorithms
│   ├── 09_amazon_de1_scenarios.md      # 15 real Amazon DE1 scenario problems
│   └── 10_boto3_sql_patterns.md        # 20 questions: boto3 hands-on + SQL-to-Python
│
├── python_faq/
│   ├── 01_core_python.md              # Q1–Q25:   Core Python fundamentals
│   ├── 02_memory_gil.md               # Q26–Q40:  Memory management & GIL
│   ├── 03_concurrency.md              # Q41–Q55:  Multithreading, multiprocessing, asyncio
│   ├── 04_file_json_api.md            # Q56–Q75:  File I/O, JSON, CSV, API/requests
│   ├── 05_data_engineering_python.md  # Q76–Q100: Pandas, boto3, AWS, pipelines
│   ├── 06_decorators_functools_itertools.md # Q101–Q125: Decorators, functools, itertools
│   └── 07_testing_typehints_logging.md     # Q126–Q150: pytest, type hints, logging, venv
│
└── README.md
```

---

## How to Use

### Step 1 — Warm Up (Week 1)
- Work through `01_basics.md` and `02_strings_lists.md`
- Read `01_core_python.md` FAQ

### Step 2 — Core Skills (Week 2)
- Work through `03_dict_set.md` and `06_lists_tuples_dicts_strings.md`
- Read `02_memory_gil.md` and `03_concurrency.md` FAQ

### Step 3 — DE Patterns (Week 3)
- Focus on `04_data_engineering_patterns.md` and `07_classic_patterns.md`
- Read `04_file_json_api.md` FAQ

### Step 4 — DE1 Production Skills (Week 4)
- Work through `08_two_pointer_sliding_window.md` and `09_amazon_de1_scenarios.md`
- Work through `10_boto3_sql_patterns.md` — boto3 hands-on and SQL↔Python translation
- Read `05_data_engineering_python.md` FAQ

### Step 5 — Polish & Pro Habits (Week 5)
- Complete `05_advanced_basics.md`
- Read `06_decorators_functools_itertools.md` FAQ (Q101–Q125)
- Read `07_testing_typehints_logging.md` FAQ (Q126–Q150) — testing, type hints, logging

---

## Difficulty Distribution

| Level        | % of Questions | Description                          |
|--------------|----------------|--------------------------------------|
| Basic        | 40%            | Pure fundamentals, beginner-safe     |
| Intermediate | 40%            | Amazon-typical, real-world patterns  |
| Challenging  | 20%            | Amazon DE2 level, scalability-aware  |

---

## Key Amazon DE Interview Themes Covered

- ETL pipeline logic and schema drift detection
- Data deduplication and CDC change tracking
- Log parsing and error spike detection
- JSON / CSV handling and nested JSON normalization
- Data cleaning, transformation, and pivot patterns
- Streaming simulation with generators
- Aggregations, groupBy, and window function patterns
- File handling at scale with chunking
- Error handling and exponential backoff retry in pipelines
- boto3: S3 pagination, multipart upload, DynamoDB batch write, Glue job polling
- SQL-to-Python translation (SELECT, JOIN, GROUP BY, window functions)
- Two-pointer and sliding window algorithms
- Classic patterns: palindrome, anagram, binary search, Kadane's, Dutch National Flag
- Decorators, `functools` (lru_cache, partial, reduce), `itertools` (chain, groupby, accumulate)
- pytest fixtures, parametrize, mocking boto3/requests
- Type hints with `typing` module and `mypy`
- Structured JSON logging and CloudWatch patterns
- Virtual environments, `requirements.txt`, `pyproject.toml`

---

> **Tip:** Don't just read — type every solution by hand. Muscle memory is critical for timed interviews.
