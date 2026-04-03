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
│   ├── 01_basics.md                  # Q1–Q10:  Variables, loops, functions, conditions
│   ├── 02_strings_lists.md           # Q11–Q20: String ops, list manipulation
│   ├── 03_dict_set.md                # Q21–Q30: Dicts, sets, deduplication
│   ├── 04_data_engineering_patterns.md # Q31–Q42: ETL, logs, JSON, file I/O, aggregation
│   └── 05_advanced_basics.md         # Q43–Q50: Generators, OOP, error handling
│
├── python_faq/
│   ├── 01_core_python.md             # Q1–Q25:  Core Python fundamentals
│   ├── 02_memory_gil.md              # Q26–Q40: Memory management & GIL
│   ├── 03_concurrency.md             # Q41–Q55: Multithreading, multiprocessing
│   ├── 04_file_json_api.md           # Q56–Q75: File I/O, JSON, CSV, API/requests
│   └── 05_data_engineering_python.md # Q76–Q100: Pandas, boto3, AWS, pipelines
│
└── README.md
```

---

## How to Use

### Step 1 — Warm Up (Week 1)
- Work through `01_basics.md` and `02_strings_lists.md`
- Read `01_core_python.md` FAQ

### Step 2 — Core Skills (Week 2)
- Work through `03_dict_set.md`
- Read `02_memory_gil.md` and `03_concurrency.md` FAQ

### Step 3 — DE Patterns (Week 3)
- Focus on `04_data_engineering_patterns.md` — closely mirrors Amazon interviews
- Read `04_file_json_api.md` FAQ

### Step 4 — Polish (Week 4)
- Complete `05_advanced_basics.md`
- Read `05_data_engineering_python.md` FAQ

---

## Difficulty Distribution

| Level        | % of Questions | Description                          |
|--------------|----------------|--------------------------------------|
| Basic        | 40%            | Pure fundamentals, beginner-safe     |
| Intermediate | 40%            | Amazon-typical, real-world patterns  |
| Challenging  | 20%            | Amazon DE2 level, scalability-aware  |

---

## Key Amazon DE Interview Themes Covered

- ETL pipeline logic
- Data deduplication
- Log parsing
- JSON / CSV handling
- Data cleaning & transformation
- Streaming simulation with generators
- Aggregations and groupBy patterns
- File handling at scale
- Error handling in pipelines
- boto3 + AWS S3 conceptual patterns
- Pandas conceptual understanding

---

> **Tip:** Don't just read — type every solution by hand. Muscle memory is critical for timed interviews.
