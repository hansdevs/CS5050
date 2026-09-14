# Big-O Quick Notes

## 1. Main idea
Big-O asks:

> **How fast does this grow when `n` gets very large?**

Ignore:
- constant multipliers
- smaller/slower-growing terms

Example:

```text
7^n + n^n + n!
```

The fastest-growing term is `n^n`, so:

```text
O(n^n)
```

---

## 2. For sums: keep the biggest term

Examples:

```text
n^2 + n + 5       -> O(n^2)
n^3 + 100n^2      -> O(n^3)
2^n + n^10        -> O(2^n)
7^n + n^n + n!    -> O(n^n)
```

---

## 3. Useful growth order

From smaller/faster to bigger/slower:

```text
1
log n
n
n^2
n^3
...
n^k
n^(log n)
c^n
n!
n^n
```

Here, `k` is a constant and `c > 1`.

---

## 4. Logs

Different constant log bases only differ by a constant factor:

```text
log_2(n) = Theta(log_10(n)) = Theta(ln n)
```

Useful identity:

```text
a^(log_b n) = n^(log_b a)
```

Example:

```text
2^(lg n) = n
3^(lg n) = n^(lg 3)
```

So:

```text
2^(lg n) < 3^(lg n)
```

---

## 5. Comparing exponentials

For:

```text
2^n
e^n
374^n
```

the bigger base grows faster:

```text
2^n < e^n < 374^n
```

A polynomial factor usually does not beat a larger exponential base:

```text
n * 2^n < e^n
```

for large enough `n`.

---

## 6. Factorial vs powers

```text
n! = 1 * 2 * 3 * ... * n
```

Compare with:

```text
n^n = n * n * n * ... * n
```

Every factor in `n!` is at most `n`, so:

```text
n! <= n^n
```

Thus:

```text
n! = O(n^n)
```

---

## 7. Quick strategy

When you see a Big-O problem:

1. Split the expression into terms.
2. Identify each term's growth type.
3. Find the fastest-growing term.
4. Drop constants and slower terms.
5. Keep only the dominant term.

Example:

```text
4n^3 + 2n + 100
```

Dominant term:

```text
n^3
```

Answer:

```text
O(n^3)
```
