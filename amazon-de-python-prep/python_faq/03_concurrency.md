# Python FAQ: Concurrency — Multithreading vs Multiprocessing (Q41–Q55)

---

**Q41: What is the difference between concurrency and parallelism?**

A: **Concurrency** means multiple tasks are in progress at the same time but may not run simultaneously (e.g., one switches to the other when waiting). **Parallelism** means tasks literally run at the same instant on multiple CPU cores. Python's `threading` provides concurrency; `multiprocessing` provides true parallelism.

---

**Q42: What is a thread in Python?**

A: A thread is a lightweight unit of execution within a process, sharing the same memory space. Python's `threading` module allows you to run functions concurrently. Threads are ideal for I/O-bound tasks (waiting on disk, network, database) because the GIL releases during I/O waits.

---

**Q43: What is a process in Python?**

A: A process is an independent program instance with its own memory space and Python interpreter (and its own GIL). Multiple processes can run Python code truly in parallel. Created via the `multiprocessing` module. Processes are heavier to start than threads but bypass the GIL.

---

**Q44: When should you use threading vs multiprocessing?**

A: Use **threading** for I/O-bound tasks: reading files, calling APIs, querying databases — tasks that spend most time waiting. Use **multiprocessing** for CPU-bound tasks: heavy transformations, compression, cryptographic operations — tasks that spend most time computing. In data engineering, most tasks are I/O-bound, so threading or async is preferred.

---

**Q45: What is `concurrent.futures` and why is it useful?**

A: `concurrent.futures` provides a high-level API with `ThreadPoolExecutor` (for I/O tasks) and `ProcessPoolExecutor` (for CPU tasks). It abstracts away thread/process management and provides a `Future` object to track results. It's the modern, recommended way to do parallelism in Python for data pipelines.

---

**Q46: What is a ThreadPoolExecutor? Give a simple example.**

A:
```python
from concurrent.futures import ThreadPoolExecutor

def fetch_data(url):
    # simulate API call
    return f"data from {url}"

urls = ["url1", "url2", "url3"]

with ThreadPoolExecutor(max_workers=3) as executor:
    results = list(executor.map(fetch_data, urls))

print(results)
```
The pool manages thread creation and cleanup. `executor.map()` applies the function to all items concurrently and collects results in order.

---

**Q47: What is a race condition and how do you prevent it?**

A: A race condition occurs when two threads read and write shared data simultaneously, producing unpredictable results. Example: two threads both increment a counter — they both read `5`, both write `6`, losing one increment. Prevention: use `threading.Lock()` to ensure only one thread can access shared data at a time.

---

**Q48: What is a Lock in Python threading?**

A:
```python
import threading

counter = 0
lock = threading.Lock()

def increment():
    global counter
    with lock:
        counter += 1
```
`with lock:` acquires the lock, executes the block, then releases it. Only one thread can hold the lock at a time. Other threads wait. This prevents race conditions on shared mutable state.

---

**Q49: What is a deadlock?**

A: A deadlock occurs when two threads are each waiting for a lock held by the other, so neither can proceed. Thread A holds Lock 1 and waits for Lock 2. Thread B holds Lock 2 and waits for Lock 1. Avoid by always acquiring locks in the same order, using timeouts, or redesigning to avoid shared mutable state.

---

**Q50: What is asyncio and how does it differ from threading?**

A: `asyncio` is Python's async/await framework for single-threaded concurrency. Instead of using multiple threads, a single thread switches between coroutines during `await` points (I/O waits). It's more efficient than threading for very high numbers of concurrent I/O operations (thousands of API calls) because it avoids thread-switching overhead.

---

**Q51: What is a coroutine in Python?**

A: A coroutine is a function defined with `async def` that can be paused with `await`. It doesn't block — when it hits an `await`, control returns to the event loop, which runs other coroutines. Coroutines only run inside an event loop (`asyncio.run()`). They are not threads — just cooperative multitasking within one thread.

---

**Q52: What is `multiprocessing.Pool` and when should you use it?**

A: `Pool(n)` creates a pool of `n` worker processes. `pool.map(func, iterable)` splits the iterable across processes and returns results. Use it for CPU-bound parallelism like transforming large datasets, running heavy computations on partitions, or parallel file processing.

---

**Q53: What data can you share between processes?**

A: Processes don't share memory directly (unlike threads). To share data you can use: `multiprocessing.Queue` (message passing), `multiprocessing.Pipe` (two-way channel), `multiprocessing.Value`/`Array` (shared memory), or return values via `Pool.map()`. Queues and return values are the most common in data pipelines.

---

**Q54: What is the `Queue` module used for?**

A: `queue.Queue` (thread-safe) is used to safely pass data between threads in producer-consumer patterns. For example, one thread reads from S3, puts records in a queue, and another thread writes to a database. `Queue` handles synchronization internally.

---

**Q55: What is the difference between daemon threads and normal threads?**

A: A daemon thread runs in the background and is automatically killed when the main program exits, without waiting for it to finish. Normal threads must complete before the program can exit. In pipelines, background monitoring or heartbeat threads are often set as daemon threads with `thread.daemon = True`.

---

> Next: [04_file_json_api.md](04_file_json_api.md)
