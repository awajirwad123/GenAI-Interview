# 50 Frequently Asked Coding Questions: Classic Patterns

> **Level:** Easy to Moderate | **Target:** Amazon DE2
> **Topics:** Palindrome · Anagram · Linear & Binary Search · Max/Min · Two Pointers · Sorting · Classic Patterns
> **Format:** Problem → Code → Explanation

---

# PALINDROME (Q1–Q7)

---

## Q1: Check if a String is a Palindrome

**Scenario:** Validate that a generated token reads the same forwards and backwards.

```python
def is_palindrome(s):
    s = s.lower().replace(" ", "")
    return s == s[::-1]

print(is_palindrome("racecar"))       # True
print(is_palindrome("A man a plan a canal Panama"))  # True
print(is_palindrome("hello"))         # False
```

**Explanation:** Normalize to lowercase and strip spaces first to handle case and word boundaries. Then compare the string with its reverse using slicing `[::-1]`. O(n) time, O(n) space.

---

## Q2: Check if a Number is a Palindrome

**Scenario:** Check whether a numeric batch ID reads the same forwards and backwards.

```python
def is_palindrome_number(n):
    if n < 0:
        return False   # negatives can't be palindromes
    s = str(n)
    return s == s[::-1]

print(is_palindrome_number(121))    # True
print(is_palindrome_number(1221))   # True
print(is_palindrome_number(-121))   # False
print(is_palindrome_number(123))    # False
```

**Explanation:** Convert to string and reverse-compare — the simplest approach. Negative numbers are immediately `False` (the `-` breaks the palindrome). Interviewers may ask for a math-only solution as a follow-up.

---

## Q3: Count All Palindromic Substrings in a String

**Scenario:** In a data tag string, count how many substrings are palindromes.

```python
def count_palindromic_substrings(s):
    count = 0
    for i in range(len(s)):
        for j in range(i + 1, len(s) + 1):
            sub = s[i:j]
            if sub == sub[::-1]:
                count += 1
    return count

print(count_palindromic_substrings("abc"))    # 3  (a, b, c)
print(count_palindromic_substrings("aaa"))    # 6  (a,a,a,aa,aa,aaa)
print(count_palindromic_substrings("racecar")) # 10
```

**Explanation:** Check every substring using two nested loops. O(n³) — acceptable for small strings at this level. For large inputs, the "expand around center" technique is O(n²).

---

## Q4: Find the Longest Palindromic Substring

**Scenario:** Given a string column value, find its longest palindrome substring (useful for pattern detection).

```python
def longest_palindrome(s):
    if not s:
        return ""

    best = s[0]

    def expand(left, right):
        nonlocal best
        while left >= 0 and right < len(s) and s[left] == s[right]:
            if right - left + 1 > len(best):
                best = s[left:right + 1]
            left -= 1
            right += 1

    for i in range(len(s)):
        expand(i, i)       # odd-length palindromes
        expand(i, i + 1)   # even-length palindromes

    return best

print(longest_palindrome("babad"))     # "bab" or "aba"
print(longest_palindrome("cbbd"))      # "bb"
print(longest_palindrome("racecar"))   # "racecar"
```

**Explanation:** Expand from each character outward as long as characters match. Two calls per center handle odd-length (`aba`) and even-length (`abba`) palindromes. O(n²) time.

---

## Q5: Check if a List is a Palindrome

**Scenario:** Verify that a pipeline stage sequence is symmetric (stage 1 = last stage, etc.).

```python
def is_list_palindrome(lst):
    return lst == lst[::-1]

print(is_list_palindrome([1, 2, 3, 2, 1]))      # True
print(is_list_palindrome(["a", "b", "a"]))      # True
print(is_list_palindrome([1, 2, 3]))             # False
```

**Explanation:** `[::-1]` reverses any sequence — works on lists, tuples, and strings. For memory-sensitive code, use two-pointer comparison without creating a reversed copy.

---

## Q6: Check if Any Permutation of a String is a Palindrome

**Scenario:** Can a string's characters be rearranged to form a palindrome?

```python
from collections import Counter

def can_form_palindrome(s):
    counts = Counter(s.replace(" ", "").lower())
    odd_count = sum(1 for c in counts.values() if c % 2 != 0)
    return odd_count <= 1

print(can_form_palindrome("civic"))       # True
print(can_form_palindrome("ivicc"))       # True  (can rearrange to "civic")
print(can_form_palindrome("hello"))       # False
print(can_form_palindrome("aab"))         # True  → "aba"
```

**Explanation:** A palindrome can have at most one character with an odd frequency (the middle character). Count character frequencies and check the rule. O(n) time.

---

## Q7: Remove Minimum Characters to Make a String a Palindrome

**Scenario:** Data cleaning — find the minimum deletions to make a string palindromic.

```python
def min_deletions_palindrome(s):
    # Length of longest palindromic subsequence (LPS)
    # Min deletions = len(s) - LPS
    n = len(s)
    dp = [[0] * n for _ in range(n)]

    for i in range(n):
        dp[i][i] = 1

    for length in range(2, n + 1):
        for i in range(n - length + 1):
            j = i + length - 1
            if s[i] == s[j]:
                dp[i][j] = dp[i+1][j-1] + 2 if length > 2 else 2
            else:
                dp[i][j] = max(dp[i+1][j], dp[i][j-1])

    lps = dp[0][n - 1]
    return n - lps

print(min_deletions_palindrome("abcba"))   # 0 — already palindrome
print(min_deletions_palindrome("abcbd"))   # 1 — remove 'd' → "abcba"
print(min_deletions_palindrome("abcd"))    # 3
```

**Explanation:** Classic dynamic programming. The minimum deletions equals `len(s) - longest palindromic subsequence`. The DP table builds up LPS length bottom-up. This is the one "slightly challenging" question in this section — focus on understanding the concept.

---

# ANAGRAM (Q8–Q14)

---

## Q8: Check if Two Strings Are Anagrams

**Scenario:** Check if two column aliases are anagrams (reorganized versions of the same characters).

```python
from collections import Counter

def are_anagrams(s1, s2):
    return Counter(s1.lower()) == Counter(s2.lower())

print(are_anagrams("listen", "silent"))    # True
print(are_anagrams("triangle", "integral")) # True
print(are_anagrams("hello", "world"))      # False
print(are_anagrams("Astronomer", "Moon starer".replace(" ", "")))  # True
```

**Explanation:** `Counter` counts character frequencies. Two strings are anagrams if and only if their character frequency maps are equal. O(n) time. Alternative: `sorted(s1) == sorted(s2)` works but is O(n log n).

---

## Q9: Group a List of Words Into Anagram Groups

**Scenario:** Given a list of column name candidates, group all anagram variants together.

```python
from collections import defaultdict

def group_anagrams(words):
    groups = defaultdict(list)
    for word in words:
        key = tuple(sorted(word.lower()))
        groups[key].append(word)
    return list(groups.values())

words = ["eat", "tea", "tan", "ate", "nat", "bat"]
groups = group_anagrams(words)
for g in sorted(groups, key=len, reverse=True):
    print(g)
# ['eat', 'tea', 'ate']
# ['tan', 'nat']
# ['bat']
```

**Explanation:** The canonical key for all anagrams is their sorted character tuple. Words with the same key belong to the same anagram group. `defaultdict(list)` avoids manual key initialization.

---

## Q10: Find the First Non-Repeated Character in a String

**Scenario:** Find the first character in a log tag string that doesn't repeat.

```python
from collections import Counter

def first_non_repeating(s):
    counts = Counter(s)
    for char in s:
        if counts[char] == 1:
            return char
    return None

print(first_non_repeating("aabbcde"))    # 'c'
print(first_non_repeating("aabb"))       # None
print(first_non_repeating("leetcode"))   # 'l'
```

**Explanation:** First pass: build frequency map. Second pass: iterate original string in order to find the first character with count 1. Two O(n) passes — cannot do it in one pass while maintaining "first" correctly.

---

## Q11: Check if One String is a Rotation of Another

**Scenario:** Check if a pipeline stage sequence is a rotation of a reference sequence.

```python
def is_rotation(s1, s2):
    if len(s1) != len(s2):
        return False
    return s2 in (s1 + s1)

print(is_rotation("abcde", "cdeab"))   # True
print(is_rotation("abcde", "abced"))   # False
print(is_rotation("hello", "llohe"))   # True
```

**Explanation:** Any rotation of `s1` will be a substring of `s1 + s1`. Double the first string and check containment. O(n) time using Python's efficient substring search.

---

## Q12: Find All Anagrams of a Pattern in a String

**Scenario:** Find all starting positions where an anagram of pattern "abc" appears in the text.

```python
from collections import Counter

def find_anagram_positions(text, pattern):
    p_len = len(pattern)
    p_count = Counter(pattern)
    w_count = Counter(text[:p_len])
    result = []

    if w_count == p_count:
        result.append(0)

    for i in range(p_len, len(text)):
        # Add new character
        w_count[text[i]] += 1
        # Remove character that slid out
        old_char = text[i - p_len]
        w_count[old_char] -= 1
        if w_count[old_char] == 0:
            del w_count[old_char]
        if w_count == p_count:
            result.append(i - p_len + 1)

    return result

print(find_anagram_positions("cbaebabacd", "abc"))   # [0, 6]
print(find_anagram_positions("abab", "ab"))          # [0, 1, 2]
```

**Explanation:** Sliding window + Counter comparison. Move a window of `len(pattern)` across the string — add incoming character, remove outgoing character. Compare counters each step. O(n) time.

---

## Q13: Count Minimum Character Swaps to Make Two Strings Anagrams

**Scenario:** Measure how "different" two schema column name lists are via anagram distance.

```python
from collections import Counter

def min_swaps_to_anagram(s1, s2):
    if len(s1) != len(s2):
        return -1  # impossible
    count1 = Counter(s1)
    count2 = Counter(s2)
    # Characters missing in s2 that need to be swapped in
    missing = sum((count1 - count2).values())
    return missing   # each swap fixes two mismatches

print(min_swaps_to_anagram("hello", "world"))   # 3
print(min_swaps_to_anagram("leetcode", "practice"))  # 5
print(min_swaps_to_anagram("listen", "silent"))  # 0
```

**Explanation:** `Counter subtraction` gives only positive differences (characters in excess in s1). The number of excess characters equals the number of swaps needed — each swap inserts one missing character.

---

## Q14: Longest Common Prefix in a List of Strings

**Scenario:** Find the common path prefix across a list of S3 file keys.

```python
def longest_common_prefix(strings):
    if not strings:
        return ""
    prefix = strings[0]
    for s in strings[1:]:
        while not s.startswith(prefix):
            prefix = prefix[:-1]
            if not prefix:
                return ""
    return prefix

paths = [
    "s3://data-lake/raw/2024/01/events.csv",
    "s3://data-lake/raw/2024/02/events.csv",
    "s3://data-lake/raw/2024/01/users.csv",
]

print(longest_common_prefix(paths))
# s3://data-lake/raw/2024/
```

**Explanation:** Start with the first string as the candidate prefix. For each subsequent string, shrink the prefix until it matches. Worst case O(n × m) where m is prefix length.

---

# LINEAR SEARCH (Q15–Q19)

---

## Q15: Linear Search — Find the First Occurrence

**Scenario:** Search an unsorted list of job names for a specific job.

```python
def linear_search(lst, target):
    for i, val in enumerate(lst):
        if val == target:
            return i
    return -1

jobs = ["ingest", "validate", "transform", "load", "report"]

idx = linear_search(jobs, "transform")
print(f"Found at index: {idx}")   # 2

idx = linear_search(jobs, "archive")
print(f"Found at index: {idx}")   # -1
```

**Explanation:** Scan each element sequentially. Return the index at first match, `-1` if not found. O(n) worst case. Use when the list is unsorted or small.

---

## Q16: Linear Search — Find All Occurrences

**Scenario:** Find all positions where a specific error code appears in a log list.

```python
def find_all_occurrences(lst, target):
    return [i for i, val in enumerate(lst) if val == target]

error_codes = [200, 404, 200, 500, 200, 403, 500, 200]

positions = find_all_occurrences(error_codes, 200)
print("200 at positions:", positions)
# [0, 2, 4, 7]

positions = find_all_occurrences(error_codes, 503)
print("503 at positions:", positions)
# []
```

**Explanation:** Unlike the first-occurrence version, this doesn't stop at the first match. List comprehension with `enumerate` collects all matching indexes cleanly.

---

## Q17: Linear Search — Find Maximum and Minimum Simultaneously

**Scenario:** Find the largest and smallest latency values in one pass through an unsorted list.

```python
def find_max_min(lst):
    if not lst:
        return None, None
    max_val = min_val = lst[0]
    for val in lst[1:]:
        if val > max_val:
            max_val = val
        if val < min_val:
            min_val = val
    return max_val, min_val

latencies = [150, 300, 95, 420, 180, 60, 310]
high, low = find_max_min(latencies)
print(f"Max: {high}ms, Min: {low}ms")
# Max: 420ms, Min: 60ms
```

**Explanation:** Initialize both `max` and `min` to the first element. A single pass updates both. This is O(n) with approximately n comparisons — more efficient than calling `max()` and `min()` separately (which would be 2n comparisons).

---

## Q18: Search for a Record in a List of Dicts by Field Value

**Scenario:** Find the first pipeline run record where `status == "failed"`.

```python
def search_by_field(records, field, value):
    for rec in records:
        if rec.get(field) == value:
            return rec
    return None

runs = [
    {"job": "ingest",    "status": "success", "rows": 50000},
    {"job": "validate",  "status": "success", "rows": 50000},
    {"job": "transform", "status": "failed",  "rows": 0},
    {"job": "load",      "status": "skipped", "rows": 0},
]

result = search_by_field(runs, "status", "failed")
print(result)
# {'job': 'transform', 'status': 'failed', 'rows': 0}
```

**Explanation:** Generic linear search over dicts using `.get()` for safe field access. Returns the full dict, not just the index. Extend this to return all matches with a list comprehension.

---

## Q19: Find the Index of the Closest Value in an Unsorted List

**Scenario:** Find which batch size option in a config is closest to the desired 750.

```python
def find_closest(lst, target):
    closest_idx = 0
    closest_diff = abs(lst[0] - target)
    for i, val in enumerate(lst[1:], start=1):
        diff = abs(val - target)
        if diff < closest_diff:
            closest_diff = diff
            closest_idx = i
    return closest_idx, lst[closest_idx]

batch_sizes = [100, 256, 512, 1024, 2048]
idx, val = find_closest(batch_sizes, 750)
print(f"Closest to 750: {val} at index {idx}")
# Closest to 750: 512 at index 2
```

**Explanation:** Track the minimum absolute difference seen so far. Update when a closer value is found. Linear scan is necessary since the list is unordered (even though this one happens to be sorted).

---

# BINARY SEARCH (Q20–Q26)

---

## Q20: Binary Search — Find an Element in a Sorted List

**Scenario:** Search for a specific record ID in a sorted master list. O(log n) instead of O(n).

```python
def binary_search(lst, target):
    left, right = 0, len(lst) - 1
    while left <= right:
        mid = (left + right) // 2
        if lst[mid] == target:
            return mid
        elif lst[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return -1

record_ids = [101, 203, 305, 412, 518, 624, 730]

print(binary_search(record_ids, 412))   # 3
print(binary_search(record_ids, 999))   # -1
```

**Explanation:** Divide the search space in half each step. Compare the midpoint: if target is larger, search right half; if smaller, search left half. O(log n) — a 1 million-element list takes at most 20 comparisons.

---

## Q21: Binary Search — Find the First Occurrence of a Repeated Value

**Scenario:** A sorted log has repeated timestamps. Find the index of the very first occurrence.

```python
def first_occurrence(lst, target):
    left, right = 0, len(lst) - 1
    result = -1
    while left <= right:
        mid = (left + right) // 2
        if lst[mid] == target:
            result = mid        # record this match
            right = mid - 1    # but keep searching LEFT for earlier occurrence
        elif lst[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return result

timestamps = [1, 2, 2, 2, 3, 3, 4]
print(first_occurrence(timestamps, 2))   # 1
print(first_occurrence(timestamps, 3))   # 4
print(first_occurrence(timestamps, 5))   # -1
```

**Explanation:** When a match is found, save the index but continue searching the LEFT half (set `right = mid - 1`). This pushes toward earlier occurrences. For last occurrence, search the right half instead.

---

## Q22: Binary Search — Find the Last Occurrence

**Scenario:** Same sorted log — find the last timestamp matching a target value.

```python
def last_occurrence(lst, target):
    left, right = 0, len(lst) - 1
    result = -1
    while left <= right:
        mid = (left + right) // 2
        if lst[mid] == target:
            result = mid        # record this match
            left = mid + 1     # keep searching RIGHT for later occurrence
        elif lst[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return result

timestamps = [1, 2, 2, 2, 3, 3, 4]
print(last_occurrence(timestamps, 2))   # 3
print(last_occurrence(timestamps, 3))   # 5
```

**Explanation:** Same as first occurrence but search the RIGHT side when a match is found (`left = mid + 1`). The `result` holds the last match seen.

---

## Q23: Count Occurrences of a Value in a Sorted List Using Binary Search

**Scenario:** Efficiently count how many records share the same timestamp in a sorted event log.

```python
def count_occurrences(lst, target):
    first = first_occurrence(lst, target)
    if first == -1:
        return 0
    last = last_occurrence(lst, target)
    return last - first + 1

def first_occurrence(lst, target):
    left, right, result = 0, len(lst) - 1, -1
    while left <= right:
        mid = (left + right) // 2
        if lst[mid] == target: result = mid; right = mid - 1
        elif lst[mid] < target: left = mid + 1
        else: right = mid - 1
    return result

def last_occurrence(lst, target):
    left, right, result = 0, len(lst) - 1, -1
    while left <= right:
        mid = (left + right) // 2
        if lst[mid] == target: result = mid; left = mid + 1
        elif lst[mid] < target: left = mid + 1
        else: right = mid - 1
    return result

events = [1, 2, 2, 2, 2, 3, 4, 4, 5]
print(count_occurrences(events, 2))   # 4
print(count_occurrences(events, 4))   # 2
print(count_occurrences(events, 9))   # 0
```

**Explanation:** Two binary searches: find first and last positions. Count = `last - first + 1`. Total O(log n) — far better than O(n) linear scan for large sorted datasets.

---

## Q24: Find the Smallest Element Greater Than Target (Upper Bound)

**Scenario:** Find the smallest batch size option strictly greater than the current load of 600.

```python
def upper_bound(lst, target):
    left, right = 0, len(lst) - 1
    result = -1
    while left <= right:
        mid = (left + right) // 2
        if lst[mid] > target:
            result = mid
            right = mid - 1   # look for smaller qualifying value
        else:
            left = mid + 1
    return result if result == -1 else lst[result]

batch_sizes = [100, 256, 512, 768, 1024, 2048]
print(upper_bound(batch_sizes, 600))    # 768
print(upper_bound(batch_sizes, 1024))   # 2048
print(upper_bound(batch_sizes, 3000))   # -1
```

**Explanation:** Search for the leftmost value that is strictly greater than `target`. When `lst[mid] > target`, record it and search left for a smaller qualifying value. This is the "upper bound" pattern from C++ STL — used in range queries on sorted data.

---

## Q25: Binary Search on a Sorted 2D Matrix

**Scenario:** A sorted partitioned dataset is stored as a 2D grid. Search for a target value.

```python
def search_matrix(matrix, target):
    if not matrix:
        return False
    rows, cols = len(matrix), len(matrix[0])
    left, right = 0, rows * cols - 1
    while left <= right:
        mid = (left + right) // 2
        row, col = divmod(mid, cols)
        val = matrix[row][col]
        if val == target:
            return True
        elif val < target:
            left = mid + 1
        else:
            right = mid - 1
    return False

matrix = [
    [1,  3,  5,  7],
    [10, 11, 16, 20],
    [23, 30, 34, 60]
]

print(search_matrix(matrix, 3))    # True
print(search_matrix(matrix, 13))   # False
```

**Explanation:** Treat the 2D matrix as a flattened sorted array. Use `divmod(mid, cols)` to convert a 1D index back to `(row, col)`. Standard binary search runs unchanged. O(log(m×n)).

---

## Q26: Find the Square Root of a Number Using Binary Search

**Scenario:** Classic binary search on answer space — find the integer square root of N.

```python
def integer_sqrt(n):
    if n < 0:
        return -1
    if n == 0:
        return 0
    left, right = 1, n
    result = 0
    while left <= right:
        mid = (left + right) // 2
        if mid * mid == n:
            return mid
        elif mid * mid < n:
            result = mid    # mid might be the floor sqrt
            left = mid + 1
        else:
            right = mid - 1
    return result

print(integer_sqrt(25))    # 5
print(integer_sqrt(30))    # 5  (floor)
print(integer_sqrt(100))   # 10
print(integer_sqrt(2))     # 1
```

**Explanation:** Binary search on the answer space `[1, n]`. When `mid² < n`, record `mid` as a candidate floor and search higher. O(log n). Interviewers use this to test "binary search on answer space" thinking.

---

# MAX / MIN PATTERNS (Q27–Q35)

---

## Q27: Find Maximum Profit from Stock Prices (Single Buy/Sell)

**Scenario:** Given daily prices of a data product subscription, find the max profit from one buy-sell.

```python
def max_profit(prices):
    if len(prices) < 2:
        return 0
    min_price = prices[0]
    max_profit = 0
    for price in prices[1:]:
        profit = price - min_price
        if profit > max_profit:
            max_profit = profit
        if price < min_price:
            min_price = price
    return max_profit

prices = [7, 1, 5, 3, 6, 4]
print(max_profit(prices))   # 5  (buy at 1, sell at 6)

prices = [7, 6, 4, 3, 1]
print(max_profit(prices))   # 0  (prices only fall)
```

**Explanation:** Track the minimum price seen so far. At each step, compute potential profit if sold today. One pass, O(n). A common interview question that tests whether you avoid the O(n²) nested loop approach.

---

## Q28: Find the Kth Largest Element

**Scenario:** Find the 3rd highest revenue day from a list without fully sorting it.

```python
def kth_largest(lst, k):
    if k > len(lst):
        return None
    return sorted(lst, reverse=True)[k - 1]

revenues = [3200, 1500, 4100, 2800, 3700, 900, 4500]
print(kth_largest(revenues, 1))   # 4500
print(kth_largest(revenues, 3))   # 3700
```

**Explanation:** `sorted(..., reverse=True)[k-1]` is the simplest approach. O(n log n). The optimal solution uses a min-heap of size k for O(n log k) — mention this as a follow-up. For large n and small k, the heap approach is significantly faster.

---

## Q29: Maximum Sum Subarray (Kadane's Algorithm)

**Scenario:** Given daily delta (gain/loss) in record counts, find the period with the maximum cumulative gain.

```python
def max_subarray_sum(nums):
    max_sum = current_sum = nums[0]
    for num in nums[1:]:
        current_sum = max(num, current_sum + num)
        max_sum = max(max_sum, current_sum)
    return max_sum

deltas = [-2, 1, -3, 4, -1, 2, 1, -5, 4]
print(max_subarray_sum(deltas))   # 6  (subarray [4, -1, 2, 1])
```

**Explanation:** Kadane's algorithm: at each position, decide whether to extend the current subarray or start fresh from the current element. O(n) time, O(1) space. One of the most famous greedy/DP interview problems.

---

## Q30: Find the Running Maximum of a List

**Scenario:** Track the highest revenue seen so far as each daily record arrives.

```python
def running_maximum(lst):
    result = []
    current_max = lst[0]
    for val in lst:
        if val > current_max:
            current_max = val
        result.append(current_max)
    return result

revenues = [100, 250, 180, 320, 290, 400, 350]
print(running_maximum(revenues))
# [100, 250, 250, 320, 320, 400, 400]
```

**Explanation:** Keep a running `current_max`. Each step, update it if the current value exceeds it, then record the current max. Creates a non-decreasing sequence — used in monitoring dashboards to track "all-time high" metrics.

---

## Q31: Find the Minimum Difference Between Any Two Elements

**Scenario:** Find the closest pair of batch sizes in a config list (to detect near-duplicates).

```python
def min_difference(lst):
    sorted_lst = sorted(lst)
    min_diff = float("inf")
    pair = (None, None)
    for i in range(len(sorted_lst) - 1):
        diff = sorted_lst[i + 1] - sorted_lst[i]
        if diff < min_diff:
            min_diff = diff
            pair = (sorted_lst[i], sorted_lst[i + 1])
    return min_diff, pair

values = [100, 500, 256, 512, 1024, 300]
diff, pair = min_difference(values)
print(f"Min difference: {diff} between {pair}")
# Min difference: 44 between (256, 300)
```

**Explanation:** Sort first — the minimum difference in any unsorted array always occurs between adjacent elements in the sorted version. Sort once (O(n log n)), then one linear scan.

---

## Q32: Find the Maximum Product of Two Numbers in a List

**Scenario:** Maximize the product of two distinct elements — could be two large positives or two large negatives.

```python
def max_product_two(lst):
    sorted_lst = sorted(lst)
    # Either the two largest positives or two most negative numbers
    option1 = sorted_lst[-1] * sorted_lst[-2]   # two largest
    option2 = sorted_lst[0] * sorted_lst[1]      # two most negative
    return max(option1, option2)

nums = [3, -10, -20, 5, 7]
print(max_product_two(nums))   # 200  (-10 × -20)

nums = [1, 9, 3, 7]
print(max_product_two(nums))   # 63  (9 × 7)
```

**Explanation:** Two candidates: product of the two largest, or product of the two smallest (which may both be very negative, yielding a large positive). Take the max. Sorting makes this clean and O(n log n).

---

## Q33: Find Peak Element in a List

**Scenario:** A peak element is greater than both its neighbors. Find any peak in the array.

```python
def find_peak(lst):
    n = len(lst)
    if n == 1:
        return 0
    if lst[0] >= lst[1]:
        return 0
    if lst[-1] >= lst[-2]:
        return n - 1
    for i in range(1, n - 1):
        if lst[i] >= lst[i-1] and lst[i] >= lst[i+1]:
            return i
    return -1

nums = [1, 3, 20, 4, 1, 0]
idx = find_peak(nums)
print(f"Peak: {nums[idx]} at index {idx}")   # Peak: 20 at index 2
```

**Explanation:** Check edges first (they only have one neighbor). Then scan for any element greater than both neighbors. O(n). Binary search can find a peak in O(log n) — mention as a follow-up.

---

## Q34: Find the Majority Element (Appears > n/2 Times)

**Scenario:** Detect the dominant status code in a list of pipeline outcomes.

```python
def majority_element(lst):
    candidate = None
    count = 0
    for val in lst:
        if count == 0:
            candidate = val
        count += (1 if val == candidate else -1)
    return candidate

statuses = ["ok", "ok", "fail", "ok", "ok", "fail", "ok"]
print(majority_element(statuses))   # "ok"  (appears 5/7 times)
```

**Explanation:** Boyer-Moore Voting Algorithm. Maintain a candidate and count. When count hits 0, switch candidate. Works in O(n) time and O(1) space. Only valid when a majority element is guaranteed to exist.

---

## Q35: Find the Two Numbers That Sum to a Target (Two Sum)

**Scenario:** Find two transaction amounts that together equal the budget target.

```python
def two_sum(nums, target):
    seen = {}   # value → index
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return (seen[complement], i)
        seen[num] = i
    return None

amounts = [150, 400, 250, 600, 350]
result = two_sum(amounts, 550)
if result:
    i, j = result
    print(f"Indices {i} and {j}: {amounts[i]} + {amounts[j]} = 550")
# Indices 0 and 2: 150 + 400... wait, 150+400=550 — index 0 and 1
```

**Explanation:** Hash map approach: for each number, check if its complement (`target - num`) was seen before. Store each number with its index. O(n) time, O(n) space — far better than O(n²) nested loops.

---

# SORTING PATTERNS (Q36–Q42)

---

## Q36: Bubble Sort — Understand the Basics

**Scenario:** Implement bubble sort to understand the O(n²) baseline that faster sorts improve upon.

```python
def bubble_sort(lst):
    arr = lst.copy()
    n = len(arr)
    for i in range(n):
        swapped = False
        for j in range(0, n - i - 1):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
                swapped = True
        if not swapped:
            break   # already sorted
    return arr

numbers = [64, 34, 25, 12, 22, 11, 90]
print(bubble_sort(numbers))
# [11, 12, 22, 25, 34, 64, 90]
```

**Explanation:** Each pass "bubbles" the largest unsorted element to its correct position. The `swapped` flag makes it O(n) best-case on already-sorted input. Know this to explain WHY Python's `sorted()` (Timsort) is much better.

---

## Q37: Sort by Frequency (Most Frequent First)

**Scenario:** Sort a list of error codes so the most frequent errors appear first.

```python
from collections import Counter

def sort_by_frequency(lst):
    counts = Counter(lst)
    return sorted(lst, key=lambda x: counts[x], reverse=True)

errors = [500, 404, 500, 200, 404, 500, 200, 404, 404]
print(sort_by_frequency(errors))
# [404, 404, 404, 404, 500, 500, 500, 200, 200]
```

**Explanation:** `Counter` builds the frequency map. `sorted()` with `key=lambda x: counts[x]` sorts by count descending. Elements with equal frequency maintain their relative original order (stable sort).

---

## Q38: Sort 0s, 1s, and 2s (Dutch National Flag)

**Scenario:** Categorize pipeline records by status code `0=skip`, `1=process`, `2=priority` — in-place in one pass.

```python
def sort_012(lst):
    arr = lst.copy()
    low, mid, high = 0, 0, len(arr) - 1
    while mid <= high:
        if arr[mid] == 0:
            arr[low], arr[mid] = arr[mid], arr[low]
            low += 1
            mid += 1
        elif arr[mid] == 1:
            mid += 1
        else:  # arr[mid] == 2
            arr[mid], arr[high] = arr[high], arr[mid]
            high -= 1
    return arr

flags = [2, 0, 2, 1, 1, 0, 1, 2, 0]
print(sort_012(flags))
# [0, 0, 0, 1, 1, 1, 2, 2, 2]
```

**Explanation:** Dutch National Flag algorithm (Dijkstra). Three pointers: `low` (next 0 position), `mid` (current), `high` (next 2 position). One pass, O(n) time, O(1) extra space.

---

## Q39: Find the Kth Smallest Element Using Partial Sort

**Scenario:** Find the 3rd cheapest data transfer cost without fully sorting.

```python
import heapq

def kth_smallest(lst, k):
    return heapq.nsmallest(k, lst)[-1]

costs = [12.5, 3.2, 8.9, 1.5, 15.0, 6.3, 9.8]
print(kth_smallest(costs, 1))   # 1.5
print(kth_smallest(costs, 3))   # 6.3
```

**Explanation:** `heapq.nsmallest(k, lst)` returns the k smallest elements using a heap internally. O(n log k) — more efficient than full sort O(n log n) when k << n. The last element of the result is the kth smallest.

---

## Q40: Sort a Dictionary by Keys

**Scenario:** A `date → record_count` dict arrived in random key order. Sort it chronologically.

```python
daily_counts = {
    "2024-03": 4500,
    "2024-01": 3200,
    "2024-05": 5100,
    "2024-02": 2800,
    "2024-04": 4900,
}

sorted_counts = dict(sorted(daily_counts.items()))
for date, count in sorted_counts.items():
    print(f"{date}: {count:,}")
# 2024-01: 3,200
# 2024-02: 2,800
# 2024-03: 4,500
# 2024-04: 4,900
# 2024-05: 5,100
```

**Explanation:** `sorted(dict.items())` sorts by key by default (lexicographic for strings, numeric for numbers). ISO date format `YYYY-MM` sorts correctly as strings. Wrap in `dict()` to get back an ordered dict.

---

## Q41: Merge Two Sorted Lists Into One Sorted List

**Scenario:** Two sorted partition outputs arrive. Merge them without resorting from scratch.

```python
def merge_sorted(lst1, lst2):
    result = []
    i = j = 0
    while i < len(lst1) and j < len(lst2):
        if lst1[i] <= lst2[j]:
            result.append(lst1[i])
            i += 1
        else:
            result.append(lst2[j])
            j += 1
    result.extend(lst1[i:])
    result.extend(lst2[j:])
    return result

a = [1, 3, 5, 7, 9]
b = [2, 4, 6, 8, 10]
print(merge_sorted(a, b))
# [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

**Explanation:** Two pointers step through both lists simultaneously. Always pick the smaller of the two current elements. After one list is exhausted, append the remainder of the other. O(n + m). This is the merge step from merge sort.

---

# CLASSIC INTERVIEW PATTERNS (Q42–Q50)

---

## Q42: Check for Valid Balanced Brackets

**Scenario:** Validate that all brackets/parentheses in a dynamically built SQL query string are balanced.

```python
def is_balanced(s):
    stack = []
    pairs = {")": "(", "]": "[", "}": "{"}
    for char in s:
        if char in "([{":
            stack.append(char)
        elif char in ")]}":
            if not stack or stack[-1] != pairs[char]:
                return False
            stack.pop()
    return len(stack) == 0

print(is_balanced("SELECT (col + (val)) FROM t"))   # True
print(is_balanced("array[index}"))                   # False
print(is_balanced("((())"))                          # False
```

**Explanation:** Push opening brackets onto a stack. For each closing bracket, check if the stack top is its matching opener. If not, or if the stack is empty, it's unbalanced. After scanning, the stack must be empty.

---

## Q43: Fibonacci — Iterative (Memory Efficient)

**Scenario:** Generate Fibonacci sequence up to n terms for a test data generator.

```python
def fibonacci(n):
    if n <= 0:
        return []
    if n == 1:
        return [0]
    fibs = [0, 1]
    for _ in range(2, n):
        fibs.append(fibs[-1] + fibs[-2])
    return fibs

print(fibonacci(10))
# [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

**Explanation:** Iterative approach is O(n) time and O(n) space. Recursive without memoization is O(2ⁿ) — exponentially worse. Always prefer iterative or memoized recursive for Fibonacci.

---

## Q44: Check if a Number is Prime

**Scenario:** Use prime-number batches as unique partition keys in test pipeline scenarios.

```python
def is_prime(n):
    if n < 2:
        return False
    if n == 2:
        return True
    if n % 2 == 0:
        return False
    for i in range(3, int(n**0.5) + 1, 2):
        if n % i == 0:
            return False
    return True

for n in [2, 7, 10, 13, 25, 97]:
    print(f"{n:3}: {is_prime(n)}")
# 2: True  7: True  10: False  13: True  25: False  97: True
```

**Explanation:** Only check odd divisors up to √n — if n has a factor larger than √n, it has a corresponding factor smaller than √n. Checking even numbers after 2 is redundant. O(√n) instead of O(n).

---

## Q45: Find GCD and LCM of Two Numbers

**Scenario:** Compute GCD for synchronizing two pipeline schedules (runs every 12h and 18h → sync every GCD=6h).

```python
def gcd(a, b):
    while b:
        a, b = b, a % b
    return a

def lcm(a, b):
    return (a * b) // gcd(a, b)

print(gcd(12, 18))   # 6
print(gcd(48, 64))   # 16
print(lcm(4, 6))     # 12
print(lcm(12, 18))   # 36
```

**Explanation:** Euclidean algorithm for GCD: replace (a, b) with (b, a mod b) until b is 0. O(log(min(a,b))). LCM is derived from GCD using the identity `lcm(a,b) = a*b / gcd(a,b)`. Python 3.5+ has `math.gcd()` built-in.

---

## Q46: Reverse an Integer

**Scenario:** A legacy system returns inverted numeric IDs. Reverse the digits.

```python
def reverse_integer(n):
    sign = -1 if n < 0 else 1
    reversed_str = str(abs(n))[::-1]
    return sign * int(reversed_str)

print(reverse_integer(12345))    # 54321
print(reverse_integer(-678))     # -876
print(reverse_integer(100))      # 1  (leading zeros dropped)
```

**Explanation:** Handle sign separately. Reverse the string of absolute digits. `int()` automatically strips leading zeros. Edge case: `100` → `"001"` → `1`. The math-only solution (using `% 10` and `//= 10`) is O(digits) but string reversal is cleaner.

---

## Q47: Find All Pairs That Sum to a Target

**Scenario:** Find all pairs of data amounts in a list whose combined size equals a target threshold.

```python
def find_all_pairs(lst, target):
    seen = set()
    pairs = []
    for num in lst:
        complement = target - num
        if complement in seen:
            pairs.append((complement, num))
        seen.add(num)
    return pairs

amounts = [1, 5, 3, 7, 4, 8, 2, 6]
print(find_all_pairs(amounts, 9))
# [(1, 8), (5, 4), (7, 2), (3, 6)]
```

**Explanation:** Similar to Two Sum but collects all pairs. The `seen` set grows as we iterate — we only find each pair once (when the second element is encountered). O(n) time and O(n) space.

---

## Q48: Count Set Bits (Number of 1s in Binary Representation)

**Scenario:** A bitmask encodes pipeline feature flags. Count how many features are enabled.

```python
def count_set_bits(n):
    count = 0
    while n:
        count += n & 1   # check last bit
        n >>= 1          # shift right
    return count

# Alternative: Brian Kernighan's algorithm
def count_set_bits_fast(n):
    count = 0
    while n:
        n &= (n - 1)   # clears the lowest set bit
        count += 1
    return count

print(count_set_bits(13))       # 3  (1101 → three 1s)
print(count_set_bits_fast(13))  # 3
print(bin(13))                  # 0b1101
```

**Explanation:** Method 1: check each bit, shift right. Method 2 (Brian Kernighan): `n & (n-1)` clears the lowest set bit in one operation — runs as many iterations as there are set bits. Useful for permission/flag systems.

---

## Q49: Check if a Number is a Power of Two

**Scenario:** Validate that a partition count is a power of 2 (required by some systems for even distribution).

```python
def is_power_of_two(n):
    if n <= 0:
        return False
    return (n & (n - 1)) == 0

for n in [1, 2, 3, 4, 8, 12, 16, 100, 1024]:
    print(f"{n:4}: {is_power_of_two(n)}")
# 1: True  2: True  3: False  4: True  8: True  12: False  16: True ...
```

**Explanation:** Powers of 2 in binary are `1000...0`. Subtracting 1 gives `0111...1`. Their bitwise AND is `0`. Any non-power-of-two has multiple set bits, so the AND is non-zero. O(1) — the fastest possible check.

---

## Q50: Find the Missing Number in a Range 1 to N

**Scenario:** A batch should have records numbered 1–100. Find the missing record ID.

```python
def find_missing(lst, n):
    expected_sum = n * (n + 1) // 2
    actual_sum = sum(lst)
    return expected_sum - actual_sum

records = list(range(1, 101))
records.remove(47)   # simulate missing record

missing = find_missing(records, 100)
print("Missing record ID:", missing)
# Missing record ID: 47
```

**Explanation:** Sum formula for 1 to n is `n(n+1)/2`. Subtract the actual sum to find the missing value. O(n) time, O(1) space — no extra data structures needed. Interviewers love this for its elegance.

---

> **Total: 50 Questions**
>
> | Topic                    | Questions | Count |
> |--------------------------|-----------|-------|
> | Palindrome               | Q1–Q7     | 7     |
> | Anagram                  | Q8–Q14    | 7     |
> | Linear Search            | Q15–Q19   | 5     |
> | Binary Search            | Q20–Q26   | 7     |
> | Max / Min Patterns       | Q27–Q35   | 9     |
> | Sorting Patterns         | Q36–Q41   | 6     |
> | Classic Interview Patterns | Q42–Q50 | 9     |
>
> Return to: [06_lists_tuples_dicts_strings.md](06_lists_tuples_dicts_strings.md) | [04_data_engineering_patterns.md](04_data_engineering_patterns.md)
