# Fri Sept 18 — Week 3 Discussion + GPS2.1 Induction

Two separate things landed today:

1. **Week 3 discussion** — three design problems: stooge sort's runtime, a majority
   keycard algorithm, and finding a local minimum in a complete binary tree.
2. **GPS2.1** — a drag-and-drop proof by induction on the recurrence
   `f(n) = 4f(n-2) - 3f(n-1)`.

Part 1 below is the dumbed-down version: what each thing *is*, in the fewest words
that are still true. Part 2 is the version that survives an exam. Read Part 1 first
even if you already get it, because the exam traps live in the gap between the two.

---
---

# PART 1 — DUMMY NOTES

## The one mental model for this whole week

Every algorithm this week runs on the same move:

> **Cut the problem into smaller copies of itself, solve those, glue the answers
> back together.** That is divide and conquer.

When an algorithm calls itself, you cannot write down its cost as a normal formula,
because the cost refers to itself:

```text
T(n) = (how many calls) * T(size of each call) + (cost of the glue)
```

That self-referencing formula is a **recurrence**. "Analyzing the algorithm" means
turning the recurrence into a plain answer like `n^2` or `n log n`.

## The three tools, one sentence each

| Tool | What it actually does |
| --- | --- |
| Recursion tree | Draw the calls as a tree, add up the work one level at a time. |
| Master theorem | A lookup table, so you do not have to draw the tree. |
| Induction | You already have a guessed answer; this is how you prove it is right. |

They are not competitors. Tree and master theorem *find* the answer. Induction
*confirms* it. GPS2.1 is the third one in isolation.

## How the master theorem works, in dummy terms

You have a recurrence shaped like this:

```text
T(n) = a * T(n/b) + f(n)
         ^      ^      ^
         |      |      +--- glue work at this level
         |      +---------- each call is b times smaller
         +----------------- you make a calls
```

Compute one magic number. It is the **number of leaves** in the recursion tree:

```text
magic = n^(log_b(a))
```

Then it is just a fight between two numbers. Compare the glue `f(n)` to `magic`:

| Situation | Who wins | Answer |
| --- | --- | --- |
| glue is smaller than magic | the leaves | `magic` |
| glue is the same size as magic | tie | `magic * log n` |
| glue is bigger than magic | the glue | `f(n)` |

That is it. Almost every recurrence you meet is one of those three rows.

Sanity check with merge sort: `T(n) = 2T(n/2) + n`. Here `a=2`, `b=2`, so
`magic = n^(log_2 2) = n^1 = n`. The glue is also `n`. Tie. Answer `n log n`. Correct.

> **Dummy rule:** many calls on barely-smaller pieces means the leaves win and the
> answer is ugly. Few calls on much-smaller pieces means the glue wins and the
> answer is clean.

---

## 1. Stooge sort, explained like you are five

The dumbest sorting algorithm that still actually works.

```text
1. Make sure the first and last item are in the right order (swap if not).
2. Sort the first two-thirds.
3. Sort the last two-thirds.
4. Sort the first two-thirds AGAIN.
```

Step 4 is the joke. It looks pointless. It is not.

**Why it works.** After step 2 and step 3, the biggest third of the array is parked
at the very end, in final sorted position, permanently. Step 4 exists to clean up the
front, which got scrambled when step 3 pushed the big stuff backward.

**Why it is slow.** Each call spawns *three* calls, and each of those is only *a
third* smaller. Three calls that are barely smaller is an explosion. Compare:

| Algorithm | Calls per level | Shrink per call | Result |
| --- | --- | --- | --- |
| Binary search | 1 | half | `log n` |
| Merge sort | 2 | half | `n log n` |
| **Stooge sort** | **3** | **only to two-thirds** | **about `n^2.71`** |

**The punchline:** stooge sort is *worse than bubble sort*. Bubble sort is `n^2`.
Stooge sort is `n^2.71`. It is correct and useless, which is exactly why it is a
teaching problem.

Memorize the shape, not the number:

> **`3` calls of size `2n/3` gives `n^(log base 1.5 of 3)`, which is about `n^2.71`.**

---

## 2. The keycard problem, explained like you are five

You have `n` hotel keycards. Each card belongs to some account. You **cannot read**
the account off a card. The only machine you have takes two cards and says
"same" or "different." That is the whole API.

**Question:** does one single account own *more than half* the cards? If so, hand
one back.

**The reason this is hard.** You can only test equality, never "less than." So:

| Normal trick | Why it is dead here |
| --- | --- |
| Sort the cards and look for a run | Sorting needs "less than." You do not have it. |
| Hash map of counts | Hashing needs to read the ID. You cannot read the ID. |
| Bucket them | Same problem. Nothing to bucket by. |

**The trick that does work.** Cut the pile in half. Ask each half "who is *your*
majority?" The whole algorithm rests on one fact:

> If somebody owns more than half of the **whole pile**, they must own more than
> half of **at least one of the two halves.**

Think about why that has to be true: if you are at most half of the left pile and
at most half of the right pile, then you are at most half of the total. You cannot
be over half the total while being under half of both halves. There is no room.

So each half hands you at most one suspect. You end up with at most two suspects
total. Walk the entire pile once per suspect, counting matches. If a suspect clears
`n/2`, that is your answer. If neither does, there is no majority card.

**Cost:** splitting in half gives `log n` levels, and at every level you do one
counting pass over `n` cards. `n log n`.

---

## 3. Local minimum in a tree, explained like you are five

You have a tree. Every node has a number on it. You want to find **any** node whose
number is less than or equal to every node it touches (its parent and its children).

Important: you do **not** want the smallest number in the tree. You want a spot where
you cannot walk downhill anymore. There can be many of them. Any one counts.

**The algorithm is literally "roll downhill":**

```text
stand on the root
loop:
    if every child is >= you:  stop, you are standing on a local minimum
    else:                      step onto a child that is smaller than you
```

**Why it cannot fail.** The moment you step down onto a child, that child is smaller
than its parent. So the parent can never be what disqualifies you. The only thing
that can stop you is a child being smaller. And if no child ever stops you, you walk
all the way to the bottom, where a leaf has no children at all. A leaf that is
smaller than its parent is automatically a local minimum.

**Why it is fast.** You only ever move *down*. Never sideways, never back up. A
complete binary tree with `n` nodes is only about `log2(n)` levels tall, so you take
at most that many steps, doing two comparisons at each.

> **The word "complete" is doing real work in the problem.** It is what guarantees
> the tree is short. In a stringy, lopsided tree the same algorithm could walk `n`
> steps straight down.

---

## 4. Induction (GPS2.1), explained like you are five

You are given a **slow rule**:

```text
f(1) = 5
f(2) = -5
f(n) = 4*f(n-2) - 3*f(n-1)
```

To get `f(100)` with that rule, you have to compute 98 earlier values first. Painful.

Somebody claims a **shortcut**:

```text
f(n) = 2 * (-4)^(n-1) + 3
```

Plug in `n`, get the answer instantly. Induction is the ritual for proving the
shortcut really is the same function as the slow rule, for *every* `n`, forever.

**It is a domino argument:**

| Step | Dominoes | Proof |
| --- | --- | --- |
| 1 | Push the first domino over by hand | **Base case:** check the shortcut by hand on the starting values |
| 2 | Show each domino knocks the next one | **Inductive step:** assume the shortcut works below `k`, prove it works at `k` |
| 3 | Conclude they all fall | The statement holds for all `n` |

**The one thing everybody gets wrong on this problem:** the rule reaches **two steps
back** (`n-2` and `n-1`). So you have to push over **two** dominoes by hand, `n=1`
and `n=2`. A single base case leaves a hole.

> **Dummy rule: how far the recurrence reaches back is how many base cases you buy.**

---

## Dummy cheat sheet

| Thing | The one-liner |
| --- | --- |
| Recurrence | A cost formula that mentions itself |
| Master theorem | Compare glue work against `n^(log_b a)`; bigger one wins |
| Stooge sort | Sort first 2/3, last 2/3, first 2/3 again. `n^2.71`. Worse than bubble sort |
| Majority card | Majority overall must be majority in a half. 2 suspects, count both. `n log n` |
| Local minimum | Roll downhill from the root. You only go down, so `log n` |
| Induction | Push two dominoes by hand, show each one pushes the next |
| Base case count | However many steps back the recurrence reaches |

---
---

# PART 2 — THE REAL NOTES

## 0. Toolbox

### 0.1 Master theorem, stated properly

For `T(n) = a*T(n/b) + f(n)` with `a >= 1` and `b > 1`, let `c = log_b(a)`.

| Case | Condition | Conclusion |
| --- | --- | --- |
| 1 | `f(n) = O(n^(c-e))` for some `e > 0` | `T(n) = Theta(n^c)` |
| 2 | `f(n) = Theta(n^c)` | `T(n) = Theta(n^c * log n)` |
| 3 | `f(n) = Omega(n^(c+e))` for some `e > 0`, plus regularity | `T(n) = Theta(f(n))` |

`n^c` is the number of leaves in the recursion tree. The three cases are "leaves
dominate," "every level costs the same," and "the root dominates."

Note that `b` does not have to be an integer, and `c` does not have to be an integer
either. Stooge sort is the standard example where both are ugly.

### 0.2 Recursion tree, stated properly

```text
depth of the tree      = number of times you divide by b to reach 1 = log_b(n)
nodes at depth d       = a^d
cost at depth d        = a^d * f(n / b^d)
number of leaves       = a^(log_b n) = n^(log_b a)
T(n)                   = sum over all levels + leaf cost
```

The identity `a^(log_b n) = n^(log_b a)` is the one worth memorizing. It is why the
magic number in the master theorem looks the way it does.

### 0.3 Substitution (guess and verify)

Guess a closed form, then prove it by induction on `n`. The trap: if the recurrence
has an additive constant, a naive guess of `T(n) <= c*n^x` will not close, because
the constant survives. The fix is a **subtractive lower-order term**, guessing
`T(n) <= c*n^x - d`, so the extra copies of `d` eat the constant. This is used in
problem 1 below.

---

## 1. Stooge sort

### 1.1 The code

```text
function stoogesort(A[], lo, hi) {
    if (A[lo] > A[hi])
        swap(A[lo], A[hi]);
    if (hi - lo + 1 >= 3) {
        oneThird = floor((hi - lo + 1) / 3);
        stoogesort(A, lo,            hi - oneThird);   // first two-thirds
        stoogesort(A, lo + oneThird, hi);              // last two-thirds
        stoogesort(A, lo,            hi - oneThird);   // first two-thirds again
    }
    return A;
}
```

### 1.2 Get the subproblem size exactly right

Let `n = hi - lo + 1` and `t = floor(n/3)`.

```text
first call  spans lo .. hi - t        length = n - t
second call spans lo + t .. hi        length = n - t
third call  spans lo .. hi - t        length = n - t
```

So every one of the three calls has length `n - floor(n/3) = ceil(2n/3)`.

The important part: it is **two-thirds**, not one-third. A very common misread is
`T(n) = 3T(n/3) + O(1)`, which would give `Theta(n)` and is wrong by a mile. The
subarrays overlap, which is exactly why this algorithm is expensive.

### 1.3 The recurrence

```text
T(n) = 3 * T(2n/3) + Theta(1)
T(1) = T(2) = Theta(1)
```

The `Theta(1)` glue is just the compare, the swap, and the arithmetic. No loop, no
merge, no partition. Everything expensive is in the three recursive calls.

### 1.4 Solve it — master theorem

```text
a = 3
b = 3/2        (because 2n/3 means n divided by 3/2)
f(n) = Theta(1)

c = log_b(a) = log_(3/2)(3) = ln(3)/ln(1.5) = 1.0986/0.4055 = 2.7095...
```

`f(n) = Theta(1) = O(n^(c - e))` for basically any small `e`, so this is **case 1**,
the leaves dominate:

```text
T(n) = Theta( n^(log_(3/2) 3) ) = Theta( n^2.7095... ) ~ Theta(n^2.71)
```

### 1.5 Solve it — recursion tree (same answer, no theorem)

```text
level 0:  1   call   of size n
level 1:  3   calls  of size (2/3)n
level 2:  9   calls  of size (2/3)^2 n
level d:  3^d calls  of size (2/3)^d n
```

Work per node is constant, so the cost of a level is just the node count, and node
counts triple every level. That means the bottom level dominates the entire sum.
Depth is where `(2/3)^d * n = 1`, so `d = log_(3/2)(n)`, and

```text
leaves = 3^(log_(3/2) n) = n^(log_(3/2) 3) = n^2.7095...
```

Same answer. When the branching factor beats the shrink factor, you can skip the sum
entirely and just count leaves.

### 1.6 Solve it — substitution (if asked to prove it)

Let `x = log_(3/2)(3)`, so that `(3/2)^x = 3`, and therefore `(2/3)^x = 1/3`.

Guess `T(n) <= c*n^x - d`, with the subtractive term from section 0.3.

```text
T(n) <= 3 * ( c*(2n/3)^x - d ) + e
      = 3c*n^x*(2/3)^x - 3d + e
      = 3c*n^x*(1/3)  - 3d + e
      = c*n^x - 3d + e
      <= c*n^x - d        whenever 2d >= e
```

It closes. Notice that without the `-d` term the induction would stall at exactly
`c*n^x + e`, which is not `<= c*n^x`. That is the entire reason the subtractive trick
exists.

### 1.7 Why it is correct (not asked, but it is the obvious follow-up)

Write `t = floor(n/3)` again. After the first call, the prefix `A[lo .. hi-t]` is
sorted.

Claim: **no element of the largest third can still be sitting in the first third.**
Take any `x` in the first `t` positions of the sorted prefix. The prefix has length
`n - t`, so at least `(n - t) - t = n - 2t >= t` elements sit after `x` in the prefix,
and all of them are `>= x`. If at least `t` elements are `>= x`, then `x` is not in
the top `t`. So every top-third element is somewhere in the last two-thirds.

The second call sorts the last two-thirds, which now contains all of the top third,
so the top third lands in the final `t` slots in correct order and is finished
forever. The third call sorts the first two-thirds, which holds everything that is
left. Array sorted.

The pre-swap of `A[lo]` and `A[hi]` is what handles `n = 2`, where the body is
skipped entirely.

### 1.8 Where it lands

| Algorithm | Runtime | Notes |
| --- | --- | --- |
| Merge sort | `n log n` | comparison-sort optimum |
| Insertion / bubble / selection | `n^2` | the "bad" sorts |
| **Stooge sort** | **`n^2.71`** | **worse than the bad sorts** |
| Bogosort | unbounded | not in the same conversation |

---

## 2. Majority keycard in `O(n log n)`

### 2.1 What the constraints actually forbid

Given: `n` cards, each carrying an account ID. The **only** primitive is
`match(i, j)` returning true or false in `Theta(1)`.

| Forbidden | Reason |
| --- | --- |
| Sorting | needs a total order; you only have equality |
| Hashing / dictionary | needs to read or hash the value |
| Counting sort / bucketing | needs to index by value |

Definition: card `x` is a **majority card** if strictly more than `n/2` cards match it.

There is at most one majority value, since two disjoint groups each over `n/2` would
need more than `n` cards.

### 2.2 The lemma the whole algorithm stands on

> **Lemma.** If `x` is a majority of array `A`, and `A` is split into `L` and `R`,
> then `x` is a majority of `L` or a majority of `R` (possibly both).

**Proof by contradiction.** Suppose `x` is a majority of neither. Then

```text
count_L(x) <= |L| / 2
count_R(x) <= |R| / 2
```

Adding them:

```text
count_A(x) = count_L(x) + count_R(x) <= (|L| + |R|)/2 = n/2
```

which contradicts `count_A(x) > n/2`. Done.

This is the standard "a global majority has to be a local majority somewhere" shape.
Recognize it. It reappears any time you divide and conquer on counts.

### 2.3 The algorithm

```text
majority(A, lo, hi):
    if lo == hi:
        return A[lo]                          # one card is its own majority

    mid = floor((lo + hi) / 2)
    candL = majority(A, lo, mid)
    candR = majority(A, mid + 1, hi)

    n = hi - lo + 1

    if candL != NONE and countMatches(A, lo, hi, candL) > n/2:
        return candL
    if candR != NONE and countMatches(A, lo, hi, candR) > n/2:
        return candR
    return NONE

countMatches(A, lo, hi, c):
    k = 0
    for i = lo to hi:
        if match(A[i], c): k = k + 1
    return k
```

`countMatches` is the only place equality is used, and it uses `hi - lo + 1` of them.

### 2.4 Correctness

By induction on the size of the range.

- **Base.** `lo == hi`: a single card matches itself once, and `1 > 1/2`, so returning
  it is correct.
- **Step.** Assume both recursive calls are correct. If the range has a majority `x`,
  the lemma says `x` is a majority of `L` or `R`, so by the inductive hypothesis at
  least one of `candL`, `candR` equals `x`. Both are then verified by a full count, so
  `x` is returned. If the range has no majority, then neither candidate can pass the
  full count, so `NONE` is returned. No false positives, because **every** returned
  value has been verified by an explicit count.

The verification step is not optional. The candidates are only suspects; a half that
has a majority does not mean the parent does.

### 2.5 Runtime

```text
T(n) = 2*T(n/2) + Theta(n)
```

Two recursive calls, and the glue is at most two full counting passes, which is
`Theta(n)`. Master theorem: `a = 2`, `b = 2`, `c = log_2(2) = 1`, and
`f(n) = Theta(n) = Theta(n^1)`, so this is case 2:

```text
T(n) = Theta(n log n)
```

which is the bound the problem asked for. Same shape as merge sort.

### 2.6 It can be done in `O(n)` (bonus, uses only equality)

Worth knowing, because the natural exam follow-up is "can you do better."

**Boyer-Moore majority vote.** One pass, one candidate slot, one counter.

```text
cand = NONE; count = 0
for each card x:
    if count == 0:            cand = x; count = 1
    else if match(x, cand):   count = count + 1
    else:                     count = count - 1
# then one more pass to verify cand really exceeds n/2
```

Every decrement pairs off one candidate card with one non-matching card. A value
appearing more than `n/2` times cannot be fully cancelled by the rest of the array,
so it survives as the candidate. Uses equality only. `Theta(n)` time, `Theta(1)`
extra space. The verification pass is still mandatory.

**Pairing / halving.** Pair up the cards. Throw away both cards of any mismatched
pair, keep one representative from each matched pair (and set aside the odd card out
if `n` is odd, to be checked at the end). A majority element stays a majority of what
is kept, so recurse:

```text
T(n) = T(n/2) + Theta(n) = Theta(n)
```

---

## 3. Local minimum in a complete binary tree in `O(log n)`

### 3.1 Restating the problem precisely

- Complete binary tree, `n` vertices, each holding a number. Not a BST, so values are
  in no particular arrangement.
- Neighbors of a vertex are its parent and its children only.
- `v` is a **local minimum** if `value(v) <= value(u)` for every neighbor `u`.
- The root's only neighbors are its children. A leaf's only neighbor is its parent.

A local minimum always exists, since the global minimum trivially qualifies. So the
algorithm never has to return "none."

Brute force is `Theta(n)`: check every vertex. The problem wants `Theta(log n)`, which
means you are allowed to look at only one root-to-leaf path.

### 3.2 The algorithm

```text
localMin(root):
    v = root
    loop:
        c = the child of v with the smaller value      # only over children that exist
        if v has no children:      return v
        if value(v) <= value(c):   return v
        v = c
```

### 3.3 The invariant, which is the actual proof

> **Invariant:** every time the loop repeats, `value(v) < value(parent(v))`.

It holds the first time vacuously, because the root has no parent. It is preserved
because the only way to move from `v` to `c` is `value(c) < value(v)`, and `v` is
exactly `c`'s parent.

Now look at the two ways the loop can exit:

| Exit | Children condition | Parent condition | Verdict |
| --- | --- | --- | --- |
| `v` has no children (leaf) | vacuous | `value(v) < value(parent)` by the invariant | local minimum |
| `value(v) <= value(c)` | holds for the smaller child, therefore for both | invariant, or `v` is the root and has no parent | local minimum |

Both exits return a local minimum, and the loop always reaches one of them because
the depth strictly increases each pass and the tree is finite.

### 3.4 Runtime

One step descends exactly one level, and each step does a constant number of
comparisons. A complete binary tree with `n` vertices has height `floor(log2 n)`, so

```text
T(n) = O(log n)
```

### 3.5 Things that will show up as exam traps

- **"Complete" is a load-bearing word.** It is what caps the height at `log2 n`. On a
  path-shaped tree the identical algorithm is `Theta(n)`.
- **This finds *a* local minimum, not *the* global minimum.** Rolling downhill lands in
  whichever basin you happened to fall into. Do not claim more than you proved.
- **Handle the partial bottom layer.** In a complete tree the last level may be partly
  filled, so a node can have exactly one child. Compare against the children that
  exist, not against a hardcoded two.
- **Descend only on a strict decrease.** Definition uses `<=`, so ties count as a stop.
  If you descended on ties you could wander a long plateau and lose the bound.
- **Same family as binary search.** You get `log n` because you can throw away a whole
  subtree from one comparison, exactly like the array local-minimum problem. It stops
  working on a general graph, where there is no "downward" to commit to.

---

## 4. GPS2.1 — the induction proof, assembled

### 4.1 The statement

```text
f : N -> R
f(1) = 5
f(2) = -5
f(n) = 4*f(n-2) - 3*f(n-1)

Prove:  f(n) = 2*(-4)^(n-1) + 3
```

Note the problem text says the recurrence holds "for `n >= 2`," but `f(2)` is given
outright and `f(0)` does not exist, so the recurrence can only actually be applied
from `n = 3` upward. That detail is not a nitpick, it is the reason the inductive
hypothesis has to say `k >= 3`.

### 4.2 Sanity table (always do this before proving anything)

```text
n :   1     2      3      4      5
rec:  5    -5     35   -125    515
cls:  5    -5     35   -125    515

f(3) = 4*f(1) - 3*f(2) = 20 + 15  = 35        2*(-4)^2 + 3 = 32 + 3   = 35
f(4) = 4*f(2) - 3*f(3) = -20 - 105 = -125     2*(-4)^3 + 3 = -128 + 3 = -125
f(5) = 4*f(3) - 3*f(4) = 140 + 375 = 515      2*(-4)^4 + 3 = 512 + 3  = 515
```

The formula is right. Now prove it.

### 4.3 The correct assembly, in order

```text
1.  Proof by induction on n.

2.  Inductive Predicate P(n) : f(n) = 2*(-4)^(n-1) + 3

3.  Base Case(s): For n = 1, 2*(-4)^(n-1) + 3 = 2*(-4)^0 = 2*1 + 3 = 5,
    which is equal to f(1). For n = 2, 2*(-4)^(n-1) + 3 = 2*(-4)^1
    = 2*(-4) + 3 = -5, which is equal to f(2). So both base cases P(1)
    and P(2) hold.

4.  Inductive Hypothesis: Suppose that P(n) : f(n) = 2*(-4)^(n-1) + 3 holds
    for n = 1 ..., k-1 for some integer k >= 3.

5.  Inductive Step: we need to show that P(k) : f(k) = 2*(-4)^(k-1) + 3 holds

6.  Using the definition of f, we get f(k) = 4f(k-2) - 3f(k-1)

7.  Then using the inductive hypothesis, we get
    4f(k-2) - 3f(k-1) = 4(2*(-4)^(k-3) + 3) - 3(2*(-4)^(k-2) + 3)

8.  Simplifying the algebra,
    4(2*(-4)^(k-3) + 3) - 3(2*(-4)^(k-2) + 3)
      = 8*(-4)^(k-3) + 12 - 6*(-4)^(k-2) - 9
      = -2*(-4)^(k-2) - 6*(-4)^(k-2) + 3
      = -8*(-4)^(k-2) + 3
      = 2*(-4)^(k-1) + 3

9.  So f(k) = 2*(-4)^(k-1) + 3, which is what we needed to prove.
```

### 4.4 The two algebra moves that are easy to lose

```text
8*(-4)^(k-3) = 8 * (-4)^(k-2) / (-4) = -2*(-4)^(k-2)      # step down one exponent
-8*(-4)^(k-2) = 2*(-4) * (-4)^(k-2)  = 2*(-4)^(k-1)       # step up one exponent
```

Everything in step 8 is those two rewrites plus `12 - 9 = 3`. The goal of the whole
step is to reach an expression whose exponent is `k-1`, because that is the shape
`P(k)` demands.

### 4.5 Every "pick one," and why the other option is wrong

**Base case: `n = 1` and `n = 2`, or just `n = 3`?**

Take both. The recurrence reaches two steps back, so the inductive step at `k` leans
on `k-1` and `k-2`. Nothing below `n = 3` is ever produced by the recurrence, so `P(1)`
and `P(2)` have to be established by direct computation or they are never established
at all. The `n = 3` block is a true statement that proves the wrong thing: it verifies
`P(3)`, while leaving `P(1)` and `P(2)` unproven, and the step at `k = 4` would then be
standing on hypotheses nobody ever checked.

> **Rule: the number of base cases equals how many steps back the recurrence reaches.**
> Reaches back one, one base case. Reaches back two, two base cases.

**Inductive hypothesis: `k >= 3` or `k >= 2`?**

`k >= 3`. At `k = 2` the step would invoke `f(k-2) = f(0)`, which is outside the
domain. And `P(2)` is a base case already, so proving it again inside the step is
circular. Pick the smallest `k` for which every index the recurrence touches is both
legal and already covered.

> **Rule: the smallest `k` is the first index at which the recurrence is actually
> defined, which is one past the last base case.**

**Justification: "using the definition of `f`" or "using the inductive hypothesis"?**

The expansion `f(k) = 4f(k-2) - 3f(k-1)` is **the definition**. It is what `f` *is*.
Nothing has been assumed at that point. The inductive hypothesis enters one line
later, when `f(k-2)` and `f(k-1)` are swapped for their closed forms. Both blocks are
in the puzzle and only differ by that word, so this is a pure "do you know which
justification you are using" check.

> **Rule: definition unrolls the recurrence. Hypothesis replaces smaller values with
> the closed form. That order never changes.**

**Inductive step: show `P(k)` or `P(k+1)`?**

`P(k)`. The hypothesis was stated as "holds for `n = 1 ..., k-1`," so the first
unproven index is `k`. Showing `P(k+1)` would skip `k` entirely. The two legal pairings
are hypothesis-through-`k` with step-at-`k+1`, or hypothesis-through-`k-1` with
step-at-`k`. Never mix them.

> **Rule: the index in the step is exactly one past the top index in the hypothesis.**

### 4.6 Why the submission came back "invalid, not gradable"

The construction box only contained `Proof by induction on n.` A partial assembly is
not a wrong proof, it is not a proof, so the grader refuses to score it. Every
required block has to be dragged in, in order, before hitting **Save & Grade**.

### 4.7 Where the closed form came from (nice for exams, not required here)

You are handed `2*(-4)^(n-1) + 3` out of nowhere and asked to verify it. You can also
derive it. Rewrite the recurrence in standard homogeneous form:

```text
f(n) + 3*f(n-1) - 4*f(n-2) = 0
```

Guess `f(n) = r^n` and divide by `r^(n-2)`, giving the **characteristic equation**:

```text
r^2 + 3r - 4 = 0
(r + 4)(r - 1) = 0
r = -4  or  r = 1
```

Two distinct roots means the general solution is a blend of both:

```text
f(n) = A*(-4)^n + B*(1)^n = A*(-4)^n + B
```

Pin `A` and `B` with the given values:

```text
n = 1:   -4A + B =  5
n = 2:   16A + B = -5
subtract: 20A = -10   ->  A = -1/2  ->  B = 3

f(n) = -(1/2)*(-4)^n + 3 = 2*(-4)^(n-1) + 3
```

because `-(1/2)*(-4)^n = -(1/2)*(-4)*(-4)^(n-1) = 2*(-4)^(n-1)`. Matches exactly.

Also notice this explains the `+3`: the root `r = 1` contributes a constant term that
never decays, so `f(n)` is a constant `3` plus a term that flips sign and quadruples
every step. That is precisely the `5, -5, 35, -125, 515` behavior in the table.

---

## 5. Exam trap list

| Trap | The correction |
| --- | --- |
| Reading stooge sort as `3T(n/3)` | It is `3T(2n/3)`. The subarrays overlap. `n^2.71`, not linear |
| Assuming `b` in the master theorem must be an integer | `b = 3/2` here, and `c = log_(3/2) 3` is irrational. Fine |
| Substitution guess without a subtractive term | `c*n^x` alone will not close against a `+Theta(1)` glue. Guess `c*n^x - d` |
| Trying to sort the keycards | Equality only. No order, so no sort, no hash, no buckets |
| Returning a candidate without verifying it | A majority of a half is only a suspect. Always do the full counting pass |
| Thinking a local minimum is the global minimum | It is any downhill dead end. There can be many |
| Forgetting the partial bottom layer of a complete tree | A node can have exactly one child. Compare only against children that exist |
| Descending on ties in the local-minimum walk | Definition is `<=`, so a tie is a stop. Descend only on a strict decrease |
| One base case for a two-step recurrence | Two base cases. Reach back two, prove two |
| Starting the hypothesis at `k >= 2` here | `k >= 3`. `f(0)` does not exist and `P(2)` is already a base case |
| Calling the first expansion "by the inductive hypothesis" | The first expansion is the definition. The hypothesis comes one line later |
| Hypothesis through `k-1` paired with a step at `k+1` | Step index is exactly one past the top hypothesis index |
| Submitting a partially assembled GPS proof | Grader marks it not gradable. All blocks or nothing |
