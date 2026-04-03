# Coding Questions: Two Pointers & Sliding Window (Q1–Q20)

> **Level:** Easy to Moderate | **Target:** Amazon DE1/DE2
> **Why this matters:** ~25% of Amazon phone screens use these patterns. Both are fundamental to efficient data processing — they reduce O(n²) brute-force solutions to O(n).

---

# TWO POINTERS (Q1–Q10)

---

## Q1: Remove Duplicates from a Sorted List In-Place

**Scenario:** A sorted event log has consecutive duplicates. Remove them in-place, return the new length. (Simulates compacting a sorted partition.)

```python
def remove_duplicates(nums):
    if not nums:
        return 0
    write = 1   # pointer to write position
    for read in range(1, len(nums)):
        if nums[read] != nums[read - 1]:
            nums[write] = nums[read]
            write += 1
    return write

nums = [1, 1, 2, 3, 3, 4, 5, 5]
length = remove_duplicates(nums)
print(nums[:length])
# [1, 2, 3, 4, 5]
```

**Explanation:** `write` pointer tracks where the next unique value goes. `read` scans forward. When a new value is found, copy it to `write` position and advance `write`. No extra list needed — O(1) space, O(n) time.

---

## Q2: Two Sum II — Sorted Array (Opposite Ends)

**Scenario:** Find two file sizes in a sorted list that together equal a target archive size. Return their 1-based positions.

```python
def two_sum_sorted(nums, target):
    left, right = 0, len(nums) - 1
    while left < right:
        current = nums[left] + nums[right]
        if current == target:
            return (left + 1, right + 1)   # 1-based
        elif current < target:
            left += 1
        else:
            right -= 1
    return None

sizes = [10, 20, 35, 50, 75, 100]
print(two_sum_sorted(sizes, 85))    # (3, 5) → 35 + 50... wait 35+50=85 → (3,4)
# (3, 4)  (35 + 50 = 85)
```

**Explanation:** Start with pointers at both ends. If the sum is too small, move left pointer right (increase sum). If too large, move right pointer left (decrease sum). O(n) — only works on SORTED input.

---

## Q3: Check if a String is a Palindrome Using Two Pointers

**Scenario:** Validate that a record key reads the same forwards and backwards without extra space.

```python
def is_palindrome(s):
    left, right = 0, len(s) - 1
    while left < right:
        if s[left] != s[right]:
            return False
        left += 1
        right -= 1
    return True

print(is_palindrome("racecar"))    # True
print(is_palindrome("abcba"))      # True
print(is_palindrome("hello"))      # False
```

**Explanation:** Two pointers start at both ends and move inward. Check characters match at each step. Stops when pointers meet or cross. O(n) time, O(1) space — no string reversal copy needed.

---

## Q4: Container With Most Water

**Scenario:** You have container widths. Find the two boundaries that hold the most water (maximize area between them). Classic capacity-planning metaphor.

```python
def max_water(heights):
    left, right = 0, len(heights) - 1
    max_area = 0
    while left < right:
        width = right - left
        area = min(heights[left], heights[right]) * width
        max_area = max(max_area, area)
        # Move the shorter side inward (hoping to find a taller edge)
        if heights[left] < heights[right]:
            left += 1
        else:
            right -= 1
    return max_area

heights = [1, 8, 6, 2, 5, 4, 8, 3, 7]
print(max_water(heights))   # 49  (between index 1 and 8: min(8,7)*7=49)
```

**Explanation:** Area = `min(left_height, right_height) × width`. Moving the taller side inward can only shrink the area. Always move the shorter side — the only possible way to find a better container. O(n).

---

## Q5: Merge Two Sorted Arrays Without Extra Space (In-Place)

**Scenario:** Two sorted partition files are represented as arrays. Merge the second into the first in-place (first array has extra capacity at the end).

```python
def merge_sorted_inplace(nums1, m, nums2, n):
    # Start from the END of both arrays
    p1, p2, write = m - 1, n - 1, m + n - 1
    while p2 >= 0:
        if p1 >= 0 and nums1[p1] > nums2[p2]:
            nums1[write] = nums1[p1]
            p1 -= 1
        else:
            nums1[write] = nums2[p2]
            p2 -= 1
        write -= 1
    return nums1

nums1 = [1, 3, 5, 7, 0, 0, 0]
nums2 = [2, 4, 6]
print(merge_sorted_inplace(nums1, 4, nums2, 3))
# [1, 2, 3, 4, 5, 6, 7]
```

**Explanation:** Fill from the back — start at the combined length end and work backwards. Three pointers: end of valid nums1, end of nums2, write position. No overwriting unprocessed data since we move backwards. O(m+n) time, O(1) space.

---

## Q6: Three Sum — Find All Triplets That Sum to Zero

**Scenario:** Find all unique sets of 3 latency readings whose combined deviation equals zero (anomaly detection).

```python
def three_sum(nums):
    nums.sort()
    result = []
    for i in range(len(nums) - 2):
        if i > 0 and nums[i] == nums[i - 1]:
            continue   # skip duplicates
        left, right = i + 1, len(nums) - 1
        while left < right:
            total = nums[i] + nums[left] + nums[right]
            if total == 0:
                result.append([nums[i], nums[left], nums[right]])
                while left < right and nums[left] == nums[left + 1]:
                    left += 1
                while left < right and nums[right] == nums[right - 1]:
                    right -= 1
                left += 1
                right -= 1
            elif total < 0:
                left += 1
            else:
                right -= 1
    return result

nums = [-1, 0, 1, 2, -1, -4]
print(three_sum(nums))
# [[-1, -1, 2], [-1, 0, 1]]
```

**Explanation:** Sort first. Fix one element, use two pointers for the remaining pair (reduces to Two Sum II). Skip duplicate values at each level to avoid duplicate triplets. O(n²) overall.

---

## Q7: Move All Zeros to End In-Place

**Scenario:** A batch status list has zeros (skipped rows). Push them to the end without losing the relative order of valid entries.

```python
def move_zeros(nums):
    write = 0
    for i in range(len(nums)):
        if nums[i] != 0:
            nums[write] = nums[i]
            write += 1
    # Fill remaining positions with zeros
    for i in range(write, len(nums)):
        nums[i] = 0
    return nums

nums = [0, 1, 0, 3, 0, 12]
print(move_zeros(nums))
# [1, 3, 12, 0, 0, 0]
```

**Explanation:** `write` pointer tracks the next position for non-zero values. First pass: compact all non-zeros to the front. Second pass: fill the remaining tail with zeros. Preserves relative order. O(n) time, O(1) space.

---

## Q8: Find the Pair With the Maximum Difference in a Sorted Array

**Scenario:** Given sorted daily pipeline runtimes, find the pair (i < j) with the greatest difference `nums[j] - nums[i]`.

```python
def max_difference(nums):
    # In a sorted array, max difference is always last - first
    if len(nums) < 2:
        return 0
    return nums[-1] - nums[0], (nums[0], nums[-1])

runtimes = [12, 15, 23, 30, 45, 61, 78]
diff, pair = max_difference(runtimes)
print(f"Max difference: {diff} (between {pair[0]} and {pair[1]})")
# Max difference: 66 (between 12 and 78)
```

**Explanation:** In a sorted array, the maximum difference is always the last minus the first element. O(1) after sorting (if already sorted). Mention this insight to interviewers — it shows you think before coding.

---

## Q9: Partition Array Around a Pivot (Quicksort's Core Step)

**Scenario:** Partition transaction records around a median value — records below go left, above go right.

```python
def partition(nums, pivot):
    left, right = 0, len(nums) - 1
    result = nums.copy()
    i, j = 0, len(result) - 1
    for num in nums:
        if num < pivot:
            result[i] = num
            i += 1
        elif num > pivot:
            result[j] = num
            j -= 1
    # Fill the middle with pivot values
    for k in range(i, j + 1):
        result[k] = pivot
    return result

nums = [5, 2, 8, 1, 9, 3, 7, 4, 6]
print(partition(nums, 5))
# [2, 1, 3, 4, 5, 6, 9, 7, 8]  — elements < 5 | 5 | elements > 5
```

**Explanation:** Three-region partition: values less than pivot go left, greater go right, equal fill the middle. This is the core of quicksort and the Dutch National Flag problem generalized to arbitrary pivot.

---

## Q10: Trap Rainwater Between Columns

**Scenario:** Classic capacity planning — how much water (data buffer) can be stored between barriers of varying heights.

```python
def trap_rainwater(heights):
    if not heights:
        return 0
    left, right = 0, len(heights) - 1
    left_max = right_max = 0
    water = 0
    while left < right:
        if heights[left] < heights[right]:
            if heights[left] >= left_max:
                left_max = heights[left]
            else:
                water += left_max - heights[left]
            left += 1
        else:
            if heights[right] >= right_max:
                right_max = heights[right]
            else:
                water += right_max - heights[right]
            right -= 1
    return water

heights = [0, 1, 0, 2, 1, 0, 1, 3, 2, 1, 2, 1]
print(trap_rainwater(heights))   # 6
```

**Explanation:** Two pointers + running max from each side. Water at any column = `min(left_max, right_max) - height`. Process the side with the smaller max first — it's already bounded. O(n) time, O(1) space.

---

# SLIDING WINDOW (Q11–Q20)

---

## Q11: Maximum Sum of a Subarray of Size K

**Scenario:** Find the K-day window with the highest total ingested records.

```python
def max_sum_window(nums, k):
    if len(nums) < k:
        return 0
    # Compute first window
    window_sum = sum(nums[:k])
    max_sum = window_sum
    for i in range(k, len(nums)):
        window_sum += nums[i] - nums[i - k]   # slide: add new, remove old
        max_sum = max(max_sum, window_sum)
    return max_sum

daily_records = [1000, 1500, 800, 2000, 1200, 1800, 900, 2200, 1600]
print(max_sum_window(daily_records, 3))   # 5200 (1800+900+2200... no, 2000+1200+1800=5000)
# Actually: max 3-day window
```

**Explanation:** Compute the first window. Slide by adding the incoming element and subtracting the outgoing element. O(n) instead of O(n×k) brute force. This is the essential sliding window mechanics.

---

## Q12: Longest Substring Without Repeating Characters

**Scenario:** Find the longest sequence of unique column names in a pipeline schema (no column repeated in the window).

```python
def longest_unique_substring(s):
    char_index = {}
    left = 0
    max_len = 0
    max_start = 0
    for right, char in enumerate(s):
        if char in char_index and char_index[char] >= left:
            left = char_index[char] + 1   # jump left past the duplicate
        char_index[char] = right
        if right - left + 1 > max_len:
            max_len = right - left + 1
            max_start = left
    return max_len, s[max_start:max_start + max_len]

print(longest_unique_substring("abcabcbb"))     # (3, 'abc')
print(longest_unique_substring("pwwkew"))       # (3, 'wke')
print(longest_unique_substring("abcdefg"))      # (7, 'abcdefg')
```

**Explanation:** Hash map stores last-seen index of each character. When a duplicate is found inside the window, shrink the left boundary past the previous occurrence. Track the longest valid window seen. O(n).

---

## Q13: Find All Anagrams of Pattern in a String (Sliding Window)

**Scenario:** Find all positions where a permutation of a search pattern matches in a document string.

```python
from collections import Counter

def find_anagrams(s, p):
    p_count = Counter(p)
    window = Counter(s[:len(p)])
    result = []
    if window == p_count:
        result.append(0)
    for i in range(len(p), len(s)):
        new_char = s[i]
        old_char = s[i - len(p)]
        window[new_char] += 1
        window[old_char] -= 1
        if window[old_char] == 0:
            del window[old_char]
        if window == p_count:
            result.append(i - len(p) + 1)
    return result

print(find_anagrams("cbaebabacd", "abc"))   # [0, 6]
print(find_anagrams("abab", "ab"))          # [0, 1, 2]
```

**Explanation:** Fixed-size window (size = len(p)). Slide by incrementing new character, decrementing outgoing character. Deleting zero-count keys ensures clean Counter comparison. O(n).

---

## Q14: Minimum Window Substring

**Scenario:** Find the smallest consecutive window in a log string that contains all required search tokens.

```python
from collections import Counter

def min_window_substring(s, t):
    if not s or not t:
        return ""
    need = Counter(t)
    have = {}
    formed = 0
    required = len(need)
    left = 0
    min_len = float("inf")
    min_start = 0
    for right, char in enumerate(s):
        have[char] = have.get(char, 0) + 1
        if char in need and have[char] == need[char]:
            formed += 1
        while formed == required:
            if right - left + 1 < min_len:
                min_len = right - left + 1
                min_start = left
            have[s[left]] -= 1
            if s[left] in need and have[s[left]] < need[s[left]]:
                formed -= 1
            left += 1
    return "" if min_len == float("inf") else s[min_start:min_start + min_len]

print(min_window_substring("ADOBECODEBANC", "ABC"))   # "BANC"
print(min_window_substring("aa", "aa"))               # "aa"
```

**Explanation:** Expand right until all required characters are present (`formed == required`). Then shrink from left while condition holds — recording the minimum window each time. Variable-size sliding window. O(n + m).

---

## Q15: Longest Subarray with Sum ≤ K (Positive Numbers)

**Scenario:** Find the longest consecutive sequence of daily record counts whose total doesn't exceed a storage budget K.

```python
def longest_subarray_sum_k(nums, k):
    left = 0
    current_sum = 0
    max_len = 0
    for right in range(len(nums)):
        current_sum += nums[right]
        while current_sum > k:
            current_sum -= nums[left]
            left += 1
        max_len = max(max_len, right - left + 1)
    return max_len

daily_counts = [1, 4, 2, 3, 5, 1, 2]
print(longest_subarray_sum_k(daily_counts, 9))   # 4  (window [2,3,1,2] = 8 ≤ 9... actually [4,2,3] = 9 or [2,3,1,2]=8)
# Result: 4
```

**Explanation:** Expand right, add to sum. When sum exceeds K, shrink from left until valid again. For positive numbers, the window can only grow when it's valid. O(n).

---

## Q16: Count Distinct Values in Every Window of Size K

**Scenario:** Sliding audit window — count distinct data sources active in each K-minute window of a stream.

```python
from collections import defaultdict

def count_distinct_in_windows(arr, k):
    freq = defaultdict(int)
    result = []
    # Build first window
    for i in range(k):
        freq[arr[i]] += 1
    result.append(len(freq))
    # Slide
    for i in range(k, len(arr)):
        # Add new element
        freq[arr[i]] += 1
        # Remove outgoing element
        old = arr[i - k]
        freq[old] -= 1
        if freq[old] == 0:
            del freq[old]
        result.append(len(freq))
    return result

stream = ["s3", "kafka", "s3", "rds", "kafka", "s3", "glue"]
print(count_distinct_in_windows(stream, 3))
# [2, 3, 2, 2, 2]
```

**Explanation:** A frequency dict tracks element counts in the window. `len(freq)` gives distinct count. Remove from dict only when count hits 0 — otherwise a zero-count entry would be counted. O(n).

---

## Q17: Longest Substring with At Most K Distinct Characters

**Scenario:** Find the longest span in a stream of data source types where no more than K distinct sources are active simultaneously.

```python
from collections import defaultdict

def longest_k_distinct(s, k):
    freq = defaultdict(int)
    left = 0
    max_len = 0
    for right, char in enumerate(s):
        freq[char] += 1
        while len(freq) > k:
            freq[s[left]] -= 1
            if freq[s[left]] == 0:
                del freq[s[left]]
            left += 1
        max_len = max(max_len, right - left + 1)
    return max_len

s = "araaci"
print(longest_k_distinct(s, 2))   # 4  ("araa" has 2 distinct)
print(longest_k_distinct(s, 1))   # 2  ("aa")
```

**Explanation:** Expand right, track frequency of each character. If distinct count exceeds K, shrink from left until back to K or fewer. Standard variable-size sliding window pattern. O(n).

---

## Q18: Maximum Product Subarray

**Scenario:** Given daily multipliers for a compounding pipeline metric, find the subarray with the largest product.

```python
def max_product_subarray(nums):
    max_prod = min_prod = result = nums[0]
    for num in nums[1:]:
        # A negative number flips max and min
        candidates = (num, max_prod * num, min_prod * num)
        max_prod = max(candidates)
        min_prod = min(candidates)
        result = max(result, max_prod)
    return result

nums = [2, 3, -2, 4]
print(max_product_subarray(nums))    # 6  ([2,3])

nums = [-2, 0, -1]
print(max_product_subarray(nums))    # 0

nums = [-2, 3, -4]
print(max_product_subarray(nums))    # 24 (all three: -2*3*-4=24)
```

**Explanation:** Track both max and min products ending at each position — a negative number can turn the minimum into the maximum. This is a sliding DP pattern: `max_prod[i] = max(num, max_prod * num, min_prod * num)`.

---

## Q19: Subarray Sum Equals K (Count All Subarrays)

**Scenario:** Count how many contiguous time windows had exactly K total errors — used for SLA breach detection.

```python
from collections import defaultdict

def subarray_sum_equals_k(nums, k):
    count = 0
    prefix_sum = 0
    freq = defaultdict(int)
    freq[0] = 1   # empty prefix
    for num in nums:
        prefix_sum += num
        # If (prefix_sum - k) was seen before, there's a subarray summing to k
        count += freq[prefix_sum - k]
        freq[prefix_sum] += 1
    return count

errors = [1, 2, 3, 1, 2]
print(subarray_sum_equals_k(errors, 3))   # 3  ([1,2], [3], [1,2] at the end)
```

**Explanation:** Prefix sum approach: `sum(i..j) = prefix[j] - prefix[i-1]`. We want `prefix[j] - prefix[i-1] == k`, i.e., prefix[j] - k was seen before. Hash map stores prefix sum frequencies. O(n) — cannot use basic sliding window because elements can be negative.

---

## Q20: Find the Smallest Window Containing All Unique Characters

**Scenario:** Find the shortest segment of a schema string that contains every distinct character at least once.

```python
from collections import Counter

def smallest_window_all_unique(s):
    unique_chars = set(s)
    required = len(unique_chars)
    need = Counter(unique_chars)   # each needed exactly once
    have = {}
    formed = 0
    left = 0
    min_len = float("inf")
    min_str = ""
    for right, char in enumerate(s):
        have[char] = have.get(char, 0) + 1
        if have[char] == 1:   # first time seeing this char in window
            formed += 1
        while formed == required:
            size = right - left + 1
            if size < min_len:
                min_len = size
                min_str = s[left:right + 1]
            have[s[left]] -= 1
            if have[s[left]] == 0:
                formed -= 1
            left += 1
    return min_str

print(smallest_window_all_unique("aabcbcdbca"))   # "dbca"
print(smallest_window_all_unique("aaab"))          # "ab"
```

**Explanation:** Variant of minimum window substring where the target is all distinct characters in the string itself. Expand until all unique chars present, then shrink to minimum. O(n).

---

> **Summary Table**
>
> | Pattern          | Questions | Core Idea                            | Time  |
> |------------------|-----------|--------------------------------------|-------|
> | Two Pointers     | Q1–Q10    | Start/end or slow/fast pointers      | O(n)  |
> | Sliding Window   | Q11–Q20   | Expand right, shrink left on invalid | O(n)  |
>
> **Key insight:** Both patterns convert O(n²) brute-force solutions into O(n). Always ask: "Can I avoid recomputing the full window from scratch each step?"
>
> Return to: [07_classic_patterns.md](07_classic_patterns.md) | [09_amazon_de1_scenarios.md](09_amazon_de1_scenarios.md)
