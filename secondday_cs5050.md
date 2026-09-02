# Day 2

- Homework 1 Due next Wednesday

- written homework due next Friday


# Time Complexity

![complexity](https://assets.insightmediagroup.io/media/wp-content/uploads/2021/03/1WBYUz6Lh2Z21DQnEk-MWFQ.png)


# Python Examples
##  O(1) — Constant

You cannot sort N items in constant time; you have to at least look at each one. Constant time is only reachable when **N is fixed at compile time** — a sorting network with a hard-coded number of comparisons.

```python
def sort_pair(a, b):
    """O(1): one comparison, always."""
    return (a, b) if a <= b else (b, a)


def sort_three(a, b, c):
    """O(1): a 3-element sorting network. Three comparisons, no loop, no recursion."""
    if a > b:
        a, b = b, a
    if b > c:
        b, c = c, b
    if a > b:
        a, b = b, a
    return a, b, c
```

Real use: sorting networks are how GPUs and SIMD code sort small fixed blocks, and how `median-of-3` pivot selection works inside quicksort.

---

##  O(log N) — Logarithmic

Also not a full sort — it's the *lookup* inside one. Binary search finds where a new element belongs in an already-sorted list.

```python
def insertion_point(sorted_nums, target):
    """O(log N): where does `target` belong in an already-sorted list?

    This is what the standard library's bisect.bisect_left does.
    """
    lo, hi = 0, len(sorted_nums)
    while lo < hi:
        mid = (lo + hi) // 2
        if sorted_nums[mid] < target:
            lo = mid + 1
        else:
            hi = mid
    return lo


def binary_insertion_sort(nums):
    """O(N log N) comparisons, but still O(N^2) total — the shifting dominates."""
    result = []
    for n in nums:
        i = insertion_point(result, n)
        result.insert(i, n)   # the O(N) that ruins it
    return result
```

Worth noticing: cutting *comparisons* to `log N` doesn't help if you still move `O(N)` memory per insert. Big-O tracks the dominant cost, not the clever part.

---

##  O(N) — Linear

Achievable only by **not comparing elements**. Counting sort uses each value as an array index, so it runs in `O(N + k)` where `k` is the key range.

```python
def counting_sort(nums):
    """O(N + k) time, O(N + k) space. Integers only, and k must stay small."""
    if not nums:
        return []

    lo, hi = min(nums), max(nums)
    counts = [0] * (hi - lo + 1)

    for n in nums:              # O(N)
        counts[n - lo] += 1

    out = []
    for offset, count in enumerate(counts):   # O(k)
        out.extend([offset + lo] * count)
    return out


def radix_sort(nums):
    """O(d * (N + 10)) for d-digit non-negative integers — linear when d is fixed."""
    if not nums:
        return []

    out = list(nums)
    place = 1
    while max(out) // place > 0:
        buckets = [[] for _ in range(10)]
        for n in out:
            buckets[(n // place) % 10].append(n)
        out = [n for bucket in buckets for n in bucket]
        place *= 10
    return out


def is_sorted(nums):
    """O(N): the cheapest useful sorting-adjacent operation there is."""
    return all(nums[i] <= nums[i + 1] for i in range(len(nums) - 1))
```

The catch: sort a million values spread across a range of `10**12` and counting sort tries to allocate a trillion-slot array. `k` is not free.

---

##  O(N log N) — Linearithmic

The best a general comparison sort can do, and where every production sort lives.

```python
def merge_sort(nums):
    """O(N log N) always — log N levels of splitting, O(N) merge work per level.
    Stable. Costs O(N) extra space."""
    if len(nums) <= 1:
        return list(nums)

    mid = len(nums) // 2
    left = merge_sort(nums[:mid])
    right = merge_sort(nums[mid:])

    merged, i, j = [], 0, 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            merged.append(left[i])
            i += 1
        else:
            merged.append(right[j])
            j += 1
    merged.extend(left[i:])
    merged.extend(right[j:])
    return merged


def quick_sort(nums):
    """O(N log N) average, O(N^2) worst case (already-sorted input with a bad pivot).
    Fastest constant factors of the three in practice."""
    if len(nums) <= 1:
        return list(nums)

    pivot = nums[len(nums) // 2]
    smaller = [n for n in nums if n < pivot]
    equal = [n for n in nums if n == pivot]
    larger = [n for n in nums if n > pivot]
    return quick_sort(smaller) + equal + quick_sort(larger)


def heap_sort(nums):
    """O(N log N) guaranteed, no worst case to worry about. Not stable."""
    import heapq

    heap = list(nums)
    heapq.heapify(heap)                                # O(N)
    return [heapq.heappop(heap) for _ in range(len(heap))]   # N * O(log N)


def timsort(nums):
    """What you should actually call. CPython's sorted() is Timsort:
    O(N log N) worst case, O(N) on already-sorted or nearly-sorted input, stable."""
    return sorted(nums)
```

---

##  O(N²) — Quadratic

Two nested loops over the input. Slow asymptotically, but the constants are tiny — CPython's Timsort drops to **binary insertion sort** for runs under 64 elements.

```python
def insertion_sort(nums):
    """O(N^2) worst case, O(N) on already-sorted input. Stable, in-place.
    The best of the quadratic sorts by a wide margin."""
    a = list(nums)
    for i in range(1, len(a)):
        key = a[i]
        j = i - 1
        while j >= 0 and a[j] > key:
            a[j + 1] = a[j]
            j -= 1
        a[j + 1] = key
    return a


def selection_sort(nums):
    """O(N^2) always — even on sorted input. But it makes only N swaps,
    which matters when writes are expensive (flash memory, say)."""
    a = list(nums)
    for i in range(len(a)):
        lowest = i
        for j in range(i + 1, len(a)):
            if a[j] < a[lowest]:
                lowest = j
        a[i], a[lowest] = a[lowest], a[i]
    return a


def bubble_sort(nums):
    """O(N^2). Taught everywhere, used nowhere. The early exit makes it
    O(N) on already-sorted input, which is its only redeeming feature."""
    a = list(nums)
    for end in range(len(a) - 1, 0, -1):
        swapped = False
        for i in range(end):
            if a[i] > a[i + 1]:
                a[i], a[i + 1] = a[i + 1], a[i]
                swapped = True
        if not swapped:
            break
    return a
```

---

##  O(N³) — Cubic

Nobody designs a cubic sort. You get one by accident: an `O(N)` operation hiding inside two nested loops.

```python
def accidentally_cubic_sort(nums):
    """O(N^3) — bubble sort where every step copies the whole list.

    This is the classic cubic bug. Slicing, `.index()`, `in` on a list, and
    string concatenation all look like single statements but cost O(N).
    Put one inside a doubly-nested loop and your quadratic sort is cubic.
    """
    a = list(nums)
    n = len(a)
    for _ in range(n):                                  # N passes
        for i in range(n - 1):                          # N comparisons each
            if a[i] > a[i + 1]:
                a = a[:i] + [a[i + 1], a[i]] + a[i + 2:]   # O(N) "swap"
            else:
                a = a[:]                                    # O(N) copy anyway
    return a


def stooge_sort(nums):
    """O(N^2.71) — the real named algorithm that lives up here.

    Sort the first 2/3, then the last 2/3, then the first 2/3 again.
    It is correct, which is somehow the most surprising part.
    """
    a = list(nums)

    def _stooge(i, j):
        if a[i] > a[j]:
            a[i], a[j] = a[j], a[i]
        if j - i + 1 > 2:
            third = (j - i + 1) // 3
            _stooge(i, j - third)
            _stooge(i + third, j)
            _stooge(i, j - third)

    if a:
        _stooge(0, len(a) - 1)
    return a
```

---

##  O(2^N) — Exponential

Doubling the work for every element added. There is no reason to build one of these except to see what it looks like.

```python
def exponential_sort(nums):
    """T(N) = 2*T(N-1) + O(N)  ->  O(2^N).

    A correct insertion sort that solves every subproblem twice and throws
    one answer away. The duplicated recursive call is the whole cost.
    """
    if len(nums) <= 1:
        return list(nums)

    head, tail = nums[0], nums[1:]
    exponential_sort(tail)          # branch 1: computed, discarded
    rest = exponential_sort(tail)   # branch 2: the one we keep

    i = 0
    while i < len(rest) and rest[i] < head:
        i += 1
    return rest[:i] + [head] + rest[i:]


def slowsort(nums):
    """'Multiply and surrender.' T(N) = 2*T(N/2) + T(N-1) + 1.

    Runs in N^(Theta(log N)) — superpolynomial: worse than any O(N^k),
    still short of true 2^N. Designed in 1986 as a parody of divide-and-conquer.
    """
    a = list(nums)

    def _slow(i, j):
        if i >= j:
            return
        mid = (i + j) // 2
        _slow(i, mid)          # sort the first half
        _slow(mid + 1, j)      # sort the second half
        if a[j] < a[mid]:      # the largest element is now at i..mid or j
            a[j], a[mid] = a[mid], a[j]
        _slow(i, j - 1)        # ...and re-sort everything except it

    if a:
        _slow(0, len(a) - 1)
    return a
```

---

## ⬛ O(N!) — Factorial

Try every possible arrangement until one happens to be sorted. 20 elements is `2.4 × 10^18` permutations.

```python
from itertools import permutations
import random


def permutation_sort(nums):
    """O(N! * N): generate every ordering, return the first sorted one.
    Deterministic, and completely useless past about N = 10."""
    for candidate in permutations(nums):
        if all(candidate[i] <= candidate[i + 1] for i in range(len(candidate) - 1)):
            return list(candidate)
    return list(nums)


def bogosort(nums):
    """Expected O(N * N!), worst case unbounded — it might never terminate.

    Shuffle. Check. Repeat. There is a quantum variant that destroys the
    universe if the list isn't sorted, which is at least O(1).
    """
    a = list(nums)
    while not all(a[i] <= a[i + 1] for i in range(len(a) - 1)):
        random.shuffle(a)
    return a
```



# Visual Doc
![notes](https://miro.medium.com/1*U4dZWeXgNNrYaedRCuzTIg.png)


# Other Visuals
![notes](https://media.licdn.com/dms/image/v2/D5612AQGq4yBs2Mvsyg/article-cover_image-shrink_720_1280/B56ZV65Cb5GoAM-/0/1741523529159?e=2147483647&v=beta&t=skWKRXATbX0Qz-m1uDyq-KaccUoid0PkpedfERK1ccU)


# Programming examples

![java bubblesort](https://viterbi-web.usc.edu/~adamchik/15-121/lectures/Sorting%20Algorithms/pix/bubbleSort.bmp)



# Extras

![Array Sorting](https://miro.medium.com/v2/resize:fit:1400/1*X1hZCxNdfgZ0sT_2tynPKA.png)