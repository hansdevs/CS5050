# Day 5 — Big-O vs Omega vs Theta

Day 4 was "find the dominant term." This one is "which of the three symbols do I
actually write, and how do I tell?"

---

## 1. The one-line version

All three compare two functions, f(n) and g(n), for large values of n. The only
thing that changes between them is which direction the bound points.

| Notation | Reads as | Think of it as |
| --- | --- | --- |
| Big-O of g | f grows **at most** as fast as g | a ceiling |
| Omega of g | f grows **at least** as fast as g | a floor |
| Theta of g | f grows at the **same rate** as g | an exact match |

Theta is not a third separate idea. It is just both of the others at once:

> **Theta of g means Big-O of g *and* Omega of g.**

So any time you prove an upper bound and a lower bound using the same g, you get
Theta for free.

---

## 2. The definitions, in words

Pick some positive constant c, and some starting point n0. Everything below only
has to be true once n gets past n0.

| Notation | What has to be true for every n past n0 |
| --- | --- |
| Big-O | f(n) stays at or below c times g(n) |
| Omega | f(n) stays at or above c times g(n) |
| Theta | f(n) is sandwiched between two multiples of g(n) |

That "once n gets past n0" clause is why small-n weirdness never matters. A
function that costs 1000n is still smaller than one that costs n squared, because
eventually is the only thing being measured.

---

## 3. How to detect which one you have

Take f(n) divided by g(n), simplify it, and watch what it does as n grows.

| f divided by g | Meaning | What is true |
| --- | --- | --- |
| shrinks toward zero | f is smaller than g | **Big-O only** |
| settles on a constant | f is the same size as g | **all three** |
| grows without bound | f is bigger than g | **Omega only** |

The whole process:

1. Write f over g.
2. Cancel everything that cancels.
3. Ask one question: does it die off, settle, or blow up?
4. Die off means Big-O. Settle means all three. Blow up means Omega.

Watch the middle row. When two functions are the same size, the answer is *all
three are true*, not "only Theta." Theta is the strongest of the three claims, but
being strong does not cancel the two weaker ones.

---

## 4. Worked detections

### Problem 1 — f is 5 to the n, g is 2 to the n

```text
5^n / 2^n = (5/2)^n = 2.5^n
```

Raising 2.5 to a growing power blows up.

**Omega only.** Not Big-O, not Theta.

### Problem 2 — f is log base 7 of n, g is log base 4 of n

Put both over the same base:

```text
log_7(n) = ln(n) / ln(7)
log_4(n) = ln(n) / ln(4)

ratio    = ln(4) / ln(7)
         = about 0.712
```

The ln(n) cancels completely and a plain constant is left.

**All three are true.**

This is the Day 4 rule restated: changing log base only multiplies by a constant,
so every constant log base is Theta of every other one. That is exactly why nobody
ever writes the base inside a Big-O.

### Problem 3 — f is n to the 2.1, g is n squared

```text
n^2.1 / n^2 = n^0.1
```

Growth of n to the 0.1 is slow, but it never stops. Slow growth is still growth.

**Omega only.**

---

## 5. Trap one: bounds do not have to be tight

Big-O and Omega say "at most" and "at least." Neither one promises you picked the
closest possible g. Every row below is true at the same time for a function that
is exactly Theta of n squared:

| Claim | True? | Why |
| --- | --- | --- |
| Big-O of n squared | yes | tight |
| Big-O of n cubed | yes | sloppy but legal |
| Big-O of 2 to the n | yes | ridiculous but legal |
| Omega of n squared | yes | tight |
| Omega of n | yes | sloppy but legal |
| Omega of 1 | yes | always true of anything |
| Theta of n squared | yes | the exact match |
| Theta of n cubed | **no** | wrong size |

Only Theta can be wrong for being the wrong size. A Big-O claim can only fail by
being too small, and an Omega claim can only fail by being too big.

---

## 6. Trap two: "worst case" and "Big-O" are different axes

These two choices are completely independent:

| Axis | Options |
| --- | --- |
| Which input | best case, worst case, average case |
| Which bound | Big-O, Omega, Theta |

Any bound can be attached to any case. "The worst-case runtime is Omega of n" is a
normal, correct sentence. It says that even on its worst input, the algorithm
needs at least on the order of n steps.

The habit of reading "Big-O" as "worst case" comes from the fact that people
usually bound the worst case from above. That is a convention, not a definition.

Safe homework approach: nail down the exact cost of each case, state each one in
Theta, then answer any Big-O or Omega question by reading off the table in
section 5.

---

## 7. Reading it off code

The job is counting how many times the innermost line runs, then checking whether
anything lets a loop quit early.

### Selection-Sort

```text
for i = 1 to n-1
    jMin = i
    for j = i to n
        if A[j] < A[jMin]
            jMin = j
    swap A[i] and A[jMin]
```

The inner loop has no break. It scans from i all the way to n no matter what the
data looks like. Sorted input, reversed input, random input, identical cost.

```text
total = (n-1) + (n-2) + ... + 1
      = n(n-1)/2
```

| Case | Runtime |
| --- | --- |
| best | Theta of n squared |
| worst | Theta of n squared |

When best and worst match, you can drop the qualifier and just call the algorithm
Theta of n squared.

### Insertion-Sort

```text
for i = 2 to n
    for j = i down to 2
        if A[j] < A[j-1]
            swap A[j] and A[j-1]
        else
            break
```

Now there is a break, so the data matters.

**Already sorted.** The very first comparison fails, the break fires, and the
inner loop does one step. The outer loop runs n-1 times.

**Reverse sorted.** Nothing ever breaks, so element i walks all the way down.

```text
total = 1 + 2 + ... + (n-1)
      = n(n-1)/2
```

| Case | Runtime |
| --- | --- |
| best | Theta of n |
| worst | Theta of n squared |

> **Detection rule:** a break or an early return inside a loop is the signal that
> best case and worst case will differ. No early exit means one single Theta
> covers every input.

### The true/false parts

**(b) Insertion-Sort always runs at least as fast as Selection-Sort — false.**
The word "always" kills it. On a reverse-sorted array both do roughly half of n
squared comparisons, but Insertion-Sort also performs roughly half of n squared
swaps, while Selection-Sort performs only n-1 swaps. Swaps are writes, and writes
are not free, so Selection-Sort wins that input.

**(c) Worst-case Selection-Sort is Omega of n — true.**
It is Theta of n squared, and n squared grows at least as fast as n. The bound is
loose, but loose lower bounds are still valid. See section 5.

**(d) Worst-case Insertion-Sort is Omega of n squared — true.**
Its worst case really is Theta of n squared, and Theta hands you the Omega half
for free.

Both (c) and (d) are true even though they feel like opposite claims. (c) is true
because the bound is loose, (d) is true because the bound is tight. Both are legal
ways for an Omega statement to hold.

---

## 8. Recurrences

Method: unroll a few levels, spot the pattern, then sanity-check by induction.

### Problem 4 — T(n) = T(n-1) + n, with T(1) = 1

Unroll it:

```text
T(n) = n + T(n-1)
     = n + (n-1) + T(n-2)
     = n + (n-1) + (n-2) + ... + T(1)
     = n + (n-1) + ... + 2 + 1
     = n(n+1)/2
```

Half of n squared plus half of n stays at or below n squared once n is at least 1,
so with c set to 1 this is Big-O of n squared. It is really Theta of n squared,
since the sum also stays at or above half of n squared.

> **Shape to memorize:** subtract 1 each level and pay n, and you get n levels of
> roughly n work. That is n squared.

### Problem 5 — T(n) = T(n/2) + constant, with T(1) = 1

Each level does a fixed amount of work and halves the problem. The only question
is how many halvings it takes to reach 1.

```text
n  ->  n/2  ->  n/4  ->  ...  ->  1
```

That is log base 2 of n steps, so the total is a constant times log n.

**Big-O of log n.** This is binary search.

> **Shape to memorize:** divide by 2 and pay a constant, and you get log n.

### Problem 7 — T(n) = 2T(n/2) + n, with T(1) = 1

Draw the recursion tree and total each level.

| Level | Problems | Size of each | Work at that level |
| --- | --- | --- | --- |
| 0 | 1 | n | n |
| 1 | 2 | n/2 | n |
| 2 | 4 | n/4 | n |
| k | 2 to the k | n over 2 to the k | n |

Every level costs exactly n, and the tree is log base 2 of n levels deep.

**Big-O of n log n.** This is merge sort.

> **Shape to memorize:** two halves plus a linear merge gives n log n.

---

## 9. Cheat sheet

| Situation | Answer |
| --- | --- |
| f over g shrinks toward zero | Big-O only |
| f over g settles on a constant | Big-O, Omega, and Theta all hold |
| f over g grows without bound | Omega only |
| loop with no early exit | best equals worst, one Theta |
| loop with a break or early return | best and worst differ |
| is it Omega of something smaller | yes, loose lower bounds are fine |
| is it Big-O of something bigger | yes, loose upper bounds are fine |
| is it Theta of the wrong size | no, Theta has to be exact |
| T(n) = T(n-1) + n | Theta of n squared |
| T(n) = T(n/2) + constant | Theta of log n |
| T(n) = 2T(n/2) + n | Theta of n log n |
