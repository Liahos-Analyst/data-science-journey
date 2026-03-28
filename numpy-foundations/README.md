# NumPy Foundations
**Timeline** *MArch 13, 2026 - March 27, 2026

Documenting my NumPy learning — concepts, problems hit, and how I solved them.
Each chapter covers a core concept cluster, practiced through code, tasks built on Pakistani business scenarios, and problems encountered and resolved.

### 📚 Primary Resources
* **Core Text:** *Python for Data Analysis* by Wes McKinney.
* **Technical Guide:** *Python Data Science Handbook* by Jake VanderPlas.
* **Documentation:** [Official NumPy Documentation](https://numpy.org/doc/stable/).

##  Phase 1: Professional Growth Summary

This table represents my shift from "standard Python thinking" to "Data Engineering thinking" over the course of Phase 1.

| Feature | Before NumPy (Day 0) | Post-NumPy (Day 15) |
| :--- | :--- | :--- |
| **Logic** | **Iterative:** "How do I loop through these values?" | **Vectorized:** "How do I align these shapes for the CPU?" |
| **Memory** | **Abstract:** Lists are just "buckets" for items. | **Physical:** Memory is a contiguous buffer with strides and offsets. |
| **Data Integrity** | **Implicit:** Trusting the data or using `if/else`. | **Explicit:** Masking NaNs, checking condition numbers, and handling views vs. copies. |
| **Performance** | **Blind:** "It runs." | **Profiled:** Identifying bottlenecks and choosing `f4` over `f8` to save RAM. |
| **Problem Solving** | **Surface-Level:** Fixing syntax errors. | **First-Principles:** Fixing mathematical instability (e.g., `solve` vs `inv`). |

---

###  Phase 1 Reflection: The Hardware Mindset

> Before starting Phase 1 (NumPy), I saw data as a simple Python List—a collection of objects. Today, I see it as a **memory address**. 
>
> I have learned that the difference between a 10-second script and a 10-millisecond pipeline isn't just the code—it's the **alignment of shapes**, the **choice of dtypes**, and the **avoidance of intermediate copies**. 
> 
> I am moving into Phase 2 (Pandas) not as a spreadsheet user, but as a Data Engineer who knows exactly what is happening under the hood. I am no longer just "using libraries"; I am managing resources.

# Chapter Progress Log

## Chapter 1 — Arrays, dtypes, creation functions
**Concept:** NumPy arrays are contiguous memory buffers.
Python lists are scattered pointers. That single design
difference explains why NumPy is roughly 100x faster for
numerical operations.

**Built:** Visitor analysis — total, average, max, min,
and 6-month compound growth projection. Zero loops.

**Problems I hit:**

**Problem 1 — Compound growth formula**
- What I tried: Used simple multiplication to project growth per month
- What went wrong: Simple multiplication applies the same base each time. Compound growth applies the percentage on top of already-grown values
- Fix: Use exponent — `weekly_avg * (1.10 ** months)` — which collapses multiple multiplications into one operation

**Problem 2 — dtype range vs precision**
- What I tried: Thought float dtypes were only about decimal precision, not memory
- What went wrong: Mental model was incomplete. For floats, bytes and precision move together — more bytes means more precision means more memory. They are the same decision
- Fix: Corrected mental model — choosing float32 vs float64 is simultaneously a precision decision and a memory decision

**Problem 3 — int32 vs int64 on Windows**
- What I tried: Expected int64 as default integer dtype based on the worked example
- What went wrong: Windows NumPy defaults to int32. Linux and Mac default to int64. Got different nbytes than expected
- Fix: Always specify dtype explicitly at array creation so behaviour is consistent across all systems

**Notebook:** [day_01_numpy_basics.ipynb](https://github.com/Liahos-Analyst/data-science-journey/blob/main/numpy-foundations/day_01_numpy_basics%20.ipynb)

---

## Chapter 2 — Arithmetic, silent upcasting, boolean filtering
**Concept:** NumPy silently upcasts dtypes in mixed-type
operations. No warning. No error. Invisible memory cost
that compounds at scale.

**Built:** Monthly profit analysis, silent upcast diagnosis,
boolean filter for high-profit months, and a 1-million-row
memory comparison test.

**Problems I hit:**

**Problem 1 — Intermediate float64 allocation**
- What I tried: Used `(revenue * tax_rate).astype(np.float32)` to get a float32 result
- What went wrong: This silently creates a full float64 buffer first, then casts it. Peak RAM usage is higher than expected on large datasets
- Fix: Cast the input array before the operation — `np.float32(revenue) * np.float32(tax_rate)` — so float64 buffer is never allocated at all

**Problem 2 — int32 * float32 upcasts to float64**
- What I tried: Multiplied int32 revenue array by np.float32 scalar expecting float32 result
- What went wrong: NumPy type promotion means int32 * float32 = float64 on most systems — not float32
- Fix: Cast both operands to float32 first so NumPy sees float32 * float32 and stays in float32 throughout

**Problem 3 — b'' prefix on string arrays**
- What I tried: Used np.string_ to create a string array
- What went wrong: Got b'1.0' instead of '1.0' — byte strings, not Unicode text
- Fix: Use .astype(str) to convert back to readable Unicode, or avoid np.string_ entirely and use np.str_ instead

**Problem 4 — String to int conversion fails on decimals**
- What I tried: Called .astype(np.int64) on an array of strings like '1.0' and '9.6'
- What went wrong: Integer parsing does not accept decimal points — got ValueError
- Fix: Two-step conversion — .astype(np.float64) first to parse the decimal, then .astype(np.int64) to drop it

**Notebook:** [day_02_dtype_arithmetic.ipynb](https://github.com/Liahos-Analyst/data-science-journey/blob/main/numpy-foundations/day_02_dtype_arithmetic.ipynb)

---

## Chapter 3 — Indexing, Views vs Copies, Reshaping

**Concept:** Every extraction from a NumPy array is either
a view (same memory) or a copy (new memory). Getting this
wrong silently corrupts data or wastes RAM with no error
and no warning.

**Built:** 3D retail sales analysis — indexed a (4,3,4)
array across products, regions and quarters. Reshaped to
(12,4) for ML input format. Applied boolean filters and
dtype audits throughout. Proved data corruption live in
the notebook.

**The rule learned today:**
- Slice       → VIEW   (fast, but silent mutation risk)
- Boolean     → COPY   (safe, costs memory)
- Fancy index → COPY   (safe, costs memory)
- .copy()     → COPY   (explicit, always clear)

**Problems I hit:**

**Problem 1 — Boolean indexing vs direct indexing**
- What I tried: Used `sales[product_names == 'Bags']` to extract product data
- What went wrong: Boolean indexing always returns a copy. When you know the index, direct indexing `sales[2]` returns a view and is faster
- Fix: Use direct indexing when the position is known. Use boolean indexing only when filtering by condition

**Problem 2 — Fixed the corruption at the wrong place**
- What I tried: Copied the array at the call site before passing it into the normalize function
- What went wrong: This relies on every caller remembering to copy. One caller who forgets = silent corruption
- Fix: Copy inside the function itself — defensive copying. The function protects its own input regardless of how it is called

**Problem 3 — Filtered on wrong array in Task 4**
- What I tried: Applied boolean filter on full 3D sales array instead of the reshaped sales_2d
- What went wrong: Got all values above 2000 across every dimension instead of Q1 column specifically
- Fix: `sales_2d[:, 0] > 2000` — column 0 is Q1 in the reshaped 2D format. Always filter on the right shape.

**Notebook:** [day_03_indexing_views_reshaping.ipynb](https://github.com/Liahos-Analyst/data-science-journey/blob/main/numpy-foundations/day_03_indexing_views_reshaping%20.ipynb)

---

## Chapter 4 — Universal Functions, Aggregations, Performance

**Concept:** UFuncs are pre-compiled C functions that bypass
Python's interpreter entirely. Aggregations collapse arrays
along specific axes. Performance must be proved with numbers,
not assumed.

**Built:** Logistics delivery analysis across 5 cities, 4 weeks,
3 metrics. Vectorized a nested loop scoring function. Timed
Python vs NumPy — 86x speedup measured directly at scale.

**Key discoveries:**
- `arr.sum()` is faster than `np.sum(arr)` — fewer dispatch steps before hitting C code
- Vectorization speedup is only visible at scale — small arrays show no difference
- `np.argmax` / `np.argmin` return indices, not values — use alongside name arrays for readable output

**Problems I hit:**

**Problem 1 — Speedup showed 1x on small data**
- What I tried: Timed loop vs vectorized on a (5,4,3) array — 60 elements total
- What went wrong: Dataset too small. NumPy function call overhead equals loop overhead at this scale. Both finish in microseconds
- Fix: Benchmark on large arrays — (500,400,3) or 1M+ elements minimum to see the real difference

**Problem 2 — np.log on ratings without a safety check**
- What I tried: Applied np.log(ratings) directly to the array
- What went wrong: Any value of 0.0 would produce -inf silently — no error, no warning, silent propagation through the pipeline
- Fix: `np.log(np.clip(ratings, 1e-10, None))` always. Never assume values are safe for log.

**Problem 3 — axis confusion on 3D arrays (carry-forward gap)**
- What I tried: Applied axis=0 and axis=1 on a 3D array expecting row/column behaviour
- What went wrong: The mental model of axis=0 as rows and axis=1 as columns only holds for 2D. In 3D each axis collapses a full dimension — the "axis = the direction that disappears" rule breaks down without the right mental picture
- Fix: Partially resolved — needs dedicated drill on 3D axis collapse before Phase 2

**Notebook:** [day_04_ufuncs_aggregations_performance.ipynb](https://github.com/Liahos-Analyst/data-science-journey/blob/main/numpy-foundations/day_04_ufuncs_aggregations_performance%20.ipynb)

---

## Chapter 5 — Broadcasting

**Concept:** Broadcasting is NumPy's rule system for operating
on arrays of different shapes without copying data. It works
by virtually stretching the smaller array to match the larger
one — no memory is allocated for the stretch.

**Built:** City delivery performance analysis — applied per-city
multipliers to a (5,4) matrix using broadcasting. Normalized
metrics across weeks. Compared shape alignment using the
right-align rule.

**Key discoveries:**
- `np.newaxis` and `reshape(-1, 1)` both achieve the same result — newaxis is the cleaner syntax
- Broadcasting rule: right-align dimensions, pad missing axes with 1s, then check compatibility
- The dtype you set at creation propagates through every broadcasting operation

**Problems I hit:**

**Problem 1 — City multiplier shape mismatch**
- What I tried: Multiplied delivery array of shape (5,4) directly by city_multipliers of shape (5,)
- What went wrong: Right-aligning gives 4 vs 5 — incompatible. NumPy raised an error
- Fix: Reshape city_multipliers to (5,1) first, then (5,4) broadcasts correctly against (5,1)

**Problem 2 — Boolean result returned values, not city names**
- What I tried: Used `normalized.mean(1) > 1` to find above-average cities, printed the boolean result values
- What went wrong: Got the float values of above-average cities, not their names
- Fix: `np.array(cities)[boll]` — use the boolean mask to index the names array directly

**Notebook:** [day_05_broadcasting.ipynb](https://github.com/Liahos-Analyst/data-science-journey/blob/main/numpy-foundations/day_05_broadcasting%20.ipynb)

---

## Chapter 6 — Linear Algebra, Performance Profiling

**Concept:** Matrix multiplication is a transformation — you
are projecting data through a change of space. When sklearn's
LinearRegression fits a model, it solves the normal equation
using exactly the operations built today.

**Built:** Full normal equation implementation from scratch —
constructed feature matrix, computed Gram matrix (X.T @ X),
solved for weights using np.linalg.solve, and recovered
true weights from noisy data within 2% accuracy. Timed
three prediction implementations (loop, matmul, einsum).

**Key discoveries:**
- Determinant = the "squashing factor" of a transformation. det=0 means your space collapsed flat.
- Condition number check prevents hours of debugging mysterious weight values in real ML pipelines
- einsum beats matmul on small matrices (BLAS overhead not worth it at small scale); matmul wins at 100K+ rows
- `x * w` creates an invisible intermediate (1000,5) array before summing — `x @ w` goes straight to (1000,) with no intermediate

**Problems I hit:**

**Problem 1 — Variable name mismatch (stale Jupyter variables)**
- What I tried: Computed `left_side` and `right_side` in one cell, then printed `left_weight` and `right_weight` in another
- What went wrong: Code ran without error because Jupyter kept old variables from a previous cell run. In a fresh kernel this crashes with NameError
- Fix: Always use the exact same variable names you defined in the current session. Never rely on stale Jupyter state.

**Problem 2 — predict_v3 intermediate allocation not measured**
- What I tried: Measured output shape and memory for predict_v3 (element-wise multiply then sum)
- What went wrong: Missed the invisible intermediate array — `x * w` creates a full (1000,5) buffer before summing it to (1000,). That intermediate never shows in the output shape
- Fix: Track peak allocation, not just output allocation. predict_v2 (`x @ w`) skips the intermediate entirely

**Notebook:** [day_06_linear_algebra_profiling.ipynb](https://github.com/Liahos-Analyst/data-science-journey/blob/main/numpy-foundations/day_06_linear_algebra_profiling%20.ipynb)

---

## Chapter 7 — Sorting, Searching, Set Logic, NaN Handling

**Concept:** Sorting in NumPy is not about ordering data —
it is about finding rank and position without moving the
original array. argsort returns indices, not values.
NaN values are contagious — one NaN poisons any regular
aggregation that touches it.

**Built:** Marketing campaign intelligence system for a fictional company — deduplicated participant lists across two campaigns using set operations, filtered blacklisted members using boolean mask inversion, identified employees eligible for special rewards via intersection logic, and ranked products by total sales using argsort. All without a single loop.

**Key discoveries:**
- `np.sort()` returns a copy. `.sort()` in-place modifies the original. Know which you are calling.
- `argsort` gives you the rank order as indices — pair with a names array to get sorted labels
- `np.nanmean` ignores NaN; `np.mean` returns NaN if any NaN exists. Always use nan-safe versions on real data
- `np.where(condition, x, y)` evaluates BOTH x and y before applying the condition — if x or y is expensive, use boolean indexing assignment instead

**Problems I hit:**

**Problem 1 — np.sort vs .sort() behaviour**
- What I tried: Called `np.sort(arr)` and expected arr to be modified in place
- What went wrong: `np.sort()` returns a new sorted array (copy). The original arr was unchanged
- Fix: Use `arr.sort()` for in-place modification, `np.sort(arr)` when you need to keep the original

**Problem 2 — np.where evaluates both branches always**
- What I tried: Used `np.where(mask, expensive_op(a), b)` expecting only the True branch to run
- What went wrong: np.where computes both x and y before applying the condition — expensive_op ran on the full array, not just the masked subset
- Fix: For expensive operations, use boolean indexing assignment: `result = b.copy(); result[mask] = expensive_op(a[mask])`

**Notebook:** [day_07_sorting_structured_arrays.ipynb](https://github.com/Liahos-Analyst/data-science-journey/blob/main/numpy-foundations/day_07_sorting_structured_arrays%20.ipynb)

---

# 🏆 Phase 1 Capstone: Pakistani E-Commerce Sales Pipeline
**Timeline:** Day 8 to Day 15 (8 Sessions of Engineering & Refactoring)
**Focus:** Numerical Stability, Memory Optimization, and ML-Ready Handoff.
**Status:** Completed ✅ | **Final Audit:** March 27, 2026

## 🎯 Project Mission
Transform 6 months of "noisy" raw sales data for 500 products into a **High-Fidelity ML-Ready Handoff**. This project moves beyond simple cleaning into **Numerical Engineering**, ensuring the data is mathematically stable for future predictive modeling.

### 🏗️ Technical Architecture
The pipeline is built on three core pillars of Numerical Engineering:

| Pillar | NumPy Implementation | Business Impact |
| :--- | :--- | :--- |
| **Data Recovery** | `np.nanmean` + Broadcasting | Recovered 5% missing data (logistics errors) without data leakage. |
| **Trend Spotting** | Category-wise Masking | Identified "Category Kings" by normalizing performance against niche averages. |
| **Stability Audit** | `np.linalg.cond()` | Verified Gram Matrix stability (Cond # 1.19) to prevent gradient explosion in ML. |

---

## 🛠️ The Technical Tasks
-  **Synthetic Simulation:** Generated a (500, 6) matrix with a 5% NaN injection to simulate real-world logistics gaps.
-  **Missing Value Imputation:** Used axis-aware broadcasting to fill NaNs with product-specific historical means.
-  **Feature Engineering:** Created a standardized "Efficiency Index" to compare high-value items (Phones) with high-volume items (Soap).
- **ML Handoff ($X_{final}$):** Standardized features (Zero-mean, Unit-variance) and appended a Bias Column using `np.hstack`.
- **Stability Testing:** Computed the **Gram Matrix ($X^T X$)** and verified its condition number was well below the $10^{10}$ threshold.

---

## 🧠 Problems I Hit & First-Principles Solutions

### **Problem: The "Apples to Oranges" Scale Issue**
* **The Conflict:** A 10,000 PKR product dominates the variance, making a 500 PKR high-growth product invisible to the model.
* **The Fix:** Applied Z-score normalization `(x - mean) / std`. 
* **First-Principles Win:** By centering data at 0 with a spread of 1, I ensured the CPU treats every feature with equal mathematical importance, regardless of the currency value.

### **Problem: Silent Numerical Instability**
* **The Conflict:** If the features are too highly correlated, the Gram Matrix becomes "singular," causing ML models to crash or give infinite weights.
* **The Fix:** Checked `np.linalg.cond(gram_matrix)`. 
* **Result:** Achieved a condition number of **1.19**. In production, anything near 1.0 is "perfectly healthy," while $10^{15}$ is a "system failure" waiting to happen.

---

## 📊 Final Memory & Stability Report
* **Total RAM Usage:** ~76.39 KB (Optimized via efficient dtypes)
* **Matrix Shape ($X_{final}$):** (500, 7) — [1 Bias + 6 Monthly Sales]
* **Numerical Health:** 1.197 (Ultra-stable)

**Capstone Deliverable:** [day_15_numpy_capstone_project.ipynb](https://github.com/Liahos-Analyst/data-science-journey/blob/main/numpy-foundations/numpy_capstone_project%20.ipynb)

---

## 🏁 Phase 1 Conclusion
Phase 1 (NumPy) is now complete. I have transitioned from standard Python to **Numerical Computing**. 
- **Total Time:** 15 Days
- **Key Deliverable:** High-Fidelity Data Pipeline (Condition Number: 1.19)
- **Status:** [MOVING TO PHASE 2: PANDAS]