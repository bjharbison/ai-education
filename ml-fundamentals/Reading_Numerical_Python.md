# Reading Numerical Python

**Scope: reading numpy code well enough to follow a technical screen, a notebook, or a paper's reference implementation.**

**Not in scope: writing Python from a blank file.** That is the other track, on the other machine, on a 6–12 month clock. Nothing here asks you to produce syntax. Everything here is aimed at one failure mode: an ML person puts code on a shared screen, and you lose the thread on line 3 for reasons that have nothing to do with the concept being tested.

**Time budget: 40 minutes, once. Then use it as a lookup while running notebooks.**

Every example is from `01_numerical_stability_perplexity.ipynb`. Nothing invented.

---

## The one idea

A Python **list** is a container you walk through one item at a time. A numpy **array** is a single object that operations apply to *all at once*.

```python
p = [0.5, 0.2, 0.1]          # list
q = np.array([0.5, 0.2, 0.1]) # numpy array
```

They print almost identically. They behave completely differently:

```python
np.log(q)     # -> array([-0.69, -1.61, -2.30])   one call, three logs
```

There is no loop. `np.log` received an array and returned an array of the same shape. This is called **vectorization**, and it is why numpy code is short and why it is fast — the loop still happens, but inside compiled C rather than interpreted Python.

**Read this as the security-relevant fact it is:** when you see numerical ML code with no visible loops, the loops didn't disappear. They moved somewhere you can't inspect line by line. That's the same instinct you already have about anything running in a runtime you don't control.

---

## Reading a chained expression

This is the line that carries notebook 01:

```python
return float(np.exp(-np.log(p).mean()))
```

Five operations in one line. **Read it inside-out, not left to right.** Find the innermost parenthesis and work outward:

| Step | Fragment | What comes out |
|---|---|---|
| 1 | `p` | array of 500 probabilities, each in (0, 1) |
| 2 | `np.log(p)` | array of 500 negative numbers |
| 3 | `.mean()` | one number — the average of them |
| 4 | `-` (the minus in front) | that number, sign flipped, now positive |
| 5 | `np.exp(...)` | one number — e raised to it |
| 6 | `float(...)` | same number, as a plain Python float |

Step 3 is worth pausing on. `.mean()` attached to the end with a dot is a **method** — a function that belongs to the object in front of it. `np.log(p)` produced an array, and arrays know how to average themselves. You will see `.sum()`, `.max()`, `.shape`, `.size` used the same way.

Steps 2–4 are cross-entropy, `-mean(log p)`. Step 5 makes it perplexity. **The entire concept from Tier 1 item 5 is those three fragments**, which is worth knowing when someone writes it on a screen and expects recognition rather than derivation.

Step 6 is cosmetic — numpy hands back its own float type, and `float()` converts it to Python's. It changes nothing about the value. Don't spend attention there.

---

## Vocabulary actually used in notebook 01

Reading-level definitions only.

| Thing | What it does |
|---|---|
| `np.array([...])` | make an array from a list |
| `np.asarray(x, dtype=np.float64)` | make an array *if it isn't one already*, forcing the number type |
| `np.log(a)` | natural log of every element |
| `np.exp(a)` | e to the power of every element |
| `np.prod(a)` | multiply all elements together → one number |
| `a.mean()` | average of all elements → one number |
| `np.clip(a, lo, hi)` | force every element into the range [lo, hi] |
| `np.isclose(x, y)` | are these two floats equal within tolerance? |
| `np.linspace(0, 40, 60)` | 60 evenly spaced numbers from 0 to 40 |
| `len(x)` | how many items (works on lists and arrays) |
| `a.size` | how many elements in an array |

**Why `np.isclose` and not `==`:** floats accumulate rounding error, so two mathematically-equal computations rarely produce bit-identical results. `np.isclose(a, b)` asks whether they agree to about 8 significant figures. In notebook 01's Check 1, this is what makes the verification meaningful — `==` would have failed on a correct fix.

**Why `dtype=np.float64` is stated explicitly:** float64 is the type whose limit is `1e-308`. Saying it in the code makes the constraint the whole notebook is about visible in the function that respects it. float32 bottoms out around `1e-38` — the same bug, arriving roughly six times sooner. Worth knowing, because quantized models and GPU inference default to smaller types.

---

## `np.clip` — one line, one defense

```python
np.log(np.clip(pz, 1e-12, 1.0))
```

Inside-out: `np.clip` takes the array, replaces anything below `1e-12` with `1e-12`, replaces anything above `1.0` with `1.0`, hands back an array of the same shape. Then `np.log` runs on the result and never sees a zero, so it never returns `-inf`.

The upper bound `1.0` looks redundant — probabilities can't exceed 1. It's there because clip requires both bounds, and because asserting the range you *expect* is how you find out when a model hands you something you didn't expect. That is an input-validation habit, not a numerical one, and you already have it.

---

## Reading a class definition

```python
class MyLanguageModel:
    def __init__(self, seed=0):
        self.rng = np.random.default_rng(seed)

    def get_probabilities(self, seq):
        return list(self.rng.uniform(0.001, 0.3, len(seq)))
```

You need four things to read this, not to write it:

- **`class`** — a template. `MyLanguageModel(seed=1)` builds one instance from it.
- **`def`** — defines a function. Indented under a class, it's a method of that class.
- **`__init__`** — runs once, at construction. Here it stores a random number generator.
- **`self`** — the instance itself. `self.rng` means "this object's rng." It's the first parameter of every method and you never pass it in; Python fills it. It exists so an object can carry state between calls.

`seed=0` is a **default** — if the caller doesn't supply one, it's 0. `MyLanguageModel(seed=1)` overrides it.

**Why the seed matters more than it looks:** a seeded generator produces the same "random" numbers every run. That's what makes notebook 01's Check 1 valid — `sequence_score_naive` and `sequence_score_log` are handed identical inputs, so a difference in output is a difference in the *function*, not the data. Seeding is the difference between a comparison and a coincidence. When a vendor shows you a benchmark, "what was the seed, and how many seeds" is the same question wearing different clothes.

---

## Formatted printing

f-strings show up constantly in output-heavy code:

```python
print(f"n={n:4d}  product = {np.prod(p):.3e}")
```

The `f` before the quote means "evaluate anything in `{}`.'" After the colon is a format spec:

| Spec | Means | Example output |
|---|---|---|
| `:4d` | integer, padded to width 4 | `  50` |
| `:.4f` | 4 digits after the decimal | `3.7214` |
| `:.3e` | scientific notation, 3 digits | `2.317e-266` |
| `:.0%` | percent, no decimals | `73%` |

Purely cosmetic. Skim these; the value is to the left of the colon.

---

## Four things that will trip you

1. **Counting starts at zero.** `p[0]` is the first element. `range(len(p))` produces `0, 1, 2, ... n-1`.
2. **Indentation is syntax.** The indented block under `def` or `for` *is* the body. There are no braces. Misaligned indentation is a real error, not a style complaint.
3. **`%timeit` is not Python.** Lines starting with `%` are Jupyter magic commands. `%timeit f(x)` runs `f(x)` many times and reports the average. It will not work in a `.py` file.
4. **The notebook holds state, and state is per-run, not per-screen.** Already in your fundamentals doc. It's here too because it's the single most common source of "impossible" behavior. `Kernel → Restart & Run All`.

---

## Predict before you run

Three questions. Answer each out loud before checking. If you can't, that's the signal for where to re-read.

1. `p` is an array of 500 values between 0.001 and 0.3. What is the **sign** of every element of `np.log(p)`, and roughly what magnitude?
2. In `float(np.exp(-np.log(p).mean()))`, which single operation is the one that prevents underflow? What would break if you removed the minus sign?
3. `sequence_score_final` calls `np.clip(p, 1e-12, 1.0)` before `np.log`. If a model returned a probability of exactly `0.0`, what does that element become after clipping, and what does its log become?

<details>
<summary>Answers</summary>

1. All negative — log of anything below 1 is negative. Range roughly `-6.9` (for 0.001) to `-1.2` (for 0.3). Moderate numbers a float64 handles trivially, which is the entire point.
2. `np.log`. Once you're summing logs instead of multiplying probabilities, nothing tiny is ever computed. The minus sign flips the mean from negative to positive; without it you'd exponentiate a negative number and get a score below 1 — the reciprocal of the right answer. Not a crash, which makes it worse: silently wrong output.
3. It becomes `1e-12`. Its log becomes about `-27.6` — large, dominating the mean, and correctly signaling "the model found this token extremely surprising" instead of `-inf` poisoning the entire result.

</details>

---

## What you can now say out loud

- *"Vectorized numpy code has no visible loop because the loop moved into compiled C — `np.log(p)` on a 500-element array is 500 logs in one call."*
- *"Read a chained numerical expression inside-out. `exp(-log(p).mean())` is cross-entropy in the middle and perplexity on the outside."*
- *"Verify a float rewrite with `np.isclose`, not `==` — rounding error means two correct implementations rarely agree bit-for-bit."*
- *"A seeded generator is what makes a before/after comparison a comparison instead of a coincidence."*

---

*Doc 1 of 4. Next: `ATLAS_Schema_Notes.md`, which is the prerequisite for notebook 02. Created 30 Jul 2026.*
