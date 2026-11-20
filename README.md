<div align="center">

# 📍 Subsetting in pandas: Label-Based Selection

### Learn to pick exactly the rows and columns you want with `.loc`, and see how it differs from `.iloc`

![Python](https://img.shields.io/badge/Python-3.x-3776AB?logo=python&logoColor=white)
![pandas](https://img.shields.io/badge/pandas-DataFrame-150458?logo=pandas&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter&logoColor=white)
![Level](https://img.shields.io/badge/level-beginner-brightgreen)

</div>

---

## 📚 Table of Contents

1. [What you'll learn](#-what-youll-learn)
2. [Project structure](#-project-structure)
3. [Getting started](#-getting-started)
4. [The dataset](#-the-dataset)
5. [Core idea: labels vs. positions](#-core-idea-labels-vs-positions)
6. [Lesson walkthrough](#-lesson-walkthrough)
7. [`.loc` vs `.iloc` side by side](#-loc-vs-iloc-side-by-side)
8. [Common pitfalls](#%EF%B8%8F-common-pitfalls)
9. [Cheat sheet](#-cheat-sheet)
10. [Practice exercises](#-practice-exercises)
11. [Further reading](#-further-reading)

---

## 🎯 What you'll learn

By the end of the notebook you will be able to:

- ✅ Turn a column into the **index** of a DataFrame with `set_index()`
- ✅ Select one row or several rows **by their label** with `.loc`
- ✅ Select rows **and** columns at the same time with `.loc[rows, cols]`
- ✅ Explain why `.loc[2:4]` and `.iloc[2:4]` can return **different rows**
- ✅ Avoid the most common indexing errors beginners run into

---

## 🗂 Project structure

```
Subsetting based on Label/
├── 2. Subsetting- Label Based.ipynb   ← the lesson notebook
├── datasets/
│   └── big_mart_sales.csv             ← retail sales data used in the lesson
└── README.md                          ← you are here
```

---

## 🚀 Getting started

**1. Install the requirements**

```bash
pip install pandas jupyter
```

**2. Open the notebook from this folder.** The notebook reads `datasets/big_mart_sales.csv` with a relative path, so start Jupyter here:

```bash
jupyter notebook "2. Subsetting- Label Based.ipynb"
```

**3. Run the cells top to bottom** (`Shift + Enter`). Try to predict each output before you run the cell.

---

## 🛒 The dataset

`big_mart_sales.csv` holds sales records for products sold in a chain of retail outlets.

| Fact | Value |
|---|---|
| Rows | **8,523** |
| Columns | **12** |
| Unique products (`Item_Identifier`) | **1,559** |
| Unique outlets (`Outlet_Identifier`) | **10** |
| Columns with missing values | `Item_Weight` (1,463), `Outlet_Size` (2,410) |

<details>
<summary><b>📋 Column dictionary (click to expand)</b></summary>

| Column | Meaning |
|---|---|
| `Item_Identifier` | Product ID (for example `FDA15`). **Used as the index in this lesson.** |
| `Item_Weight` | Weight of the product |
| `Item_Fat_Content` | Low fat or regular |
| `Item_Visibility` | Share of the store's display area given to the product |
| `Item_Type` | Category (Dairy, Soft Drinks, Meat, …) |
| `Item_MRP` | Maximum retail price |
| `Outlet_Identifier` | Store ID |
| `Outlet_Establishment_Year` | Year the store opened |
| `Outlet_Size` | Small / Medium / High |
| `Outlet_Location_Type` | Tier 1 / Tier 2 / Tier 3 city |
| `Outlet_Type` | Grocery store or supermarket type |
| `Item_Outlet_Sales` | Sales of the product in that store (the target variable) |

</details>

> 💡 **Note:** The same product is sold in many outlets, so `Item_Identifier` is **not unique**. `FDA15` alone appears in 8 rows. This matters once it becomes the index (see [pitfalls](#%EF%B8%8F-common-pitfalls)).

---

## 🧠 Core idea: labels vs. positions

Every row in a DataFrame has two ways to be addressed:

```
                ┌─────────── label (the index value) ─── used by .loc
                │
   position →   │     gender  grade  marks
   ─────────────┼──────────────────────────
       0        A103     M      B      12
       1        A104     M      B      14
       2        A105     F      A      20
       3        A102     F      A      21
       4        A101     M      A      22
       ↑
       └── position (0, 1, 2, …) ─────────────────────── used by .iloc
```

| | `.loc` | `.iloc` |
|---|---|---|
| Selects by | **L**abel (the index value) | **I**nteger **loc**ation (position) |
| Slice end | **Included** | **Excluded** (like Python lists) |
| Accepts | labels, lists of labels, boolean masks | integers, lists of integers, boolean arrays |
| Affected by sorting? | No, a label stays with its row | **Yes**, positions change |

> 🧩 **Memory trick:** **`i`loc = `i`nteger. Plain `loc` = label.**

---

## 🧭 Lesson walkthrough

### Step 1: Load the data and set a meaningful index

```python
import pandas as pd

data = pd.read_csv('datasets/big_mart_sales.csv')
data.set_index('Item_Identifier', inplace=True, drop=True)
```

`set_index` moves `Item_Identifier` out of the columns and makes it the row labels. Rows can now be looked up by product ID instead of by row number.

### Step 2: Select rows by label

```python
data.loc['FDA15']              # every row whose label is FDA15
data.loc[['FDA15', 'FDA03']]   # pass a list to get several labels
```

### Step 3: Select rows **and** columns by label

The general form is **`df.loc[rows, columns]`**:

```python
# one column → returns a Series
data.loc[['FDA15', 'FDA03'], 'Item_Fat_Content']

# a list of columns → returns a DataFrame
data.loc[['FDA15', 'FDA03'], ['Item_Fat_Content', 'Item_Type', 'Item_MRP']]
```

> 🔎 **Spot it:** In the output, `Item_Fat_Content` contains both `Low Fat` and `LF`. The full column also has `low fat` and `reg`. They mean the same thing but are spelled differently, so this column needs cleaning before any analysis.

### Step 4: The experiment that shows `.loc` vs `.iloc`

A small DataFrame makes the difference easy to see:

```python
sample_df = pd.DataFrame({
    'gender': ['M', 'F', 'M', 'M', 'F'],
    'grade' : ['A', 'A', 'B', 'B', 'A'],
    'marks' : [ 22,  21,  12,  14,  20],
    'id'    : ['A101', 'A102', 'A103', 'A104', 'A105']
})
```

This happens in three rounds:

#### 🔹 Round 1: default index (`0, 1, 2, 3, 4`)

| Code | Rows returned | Why |
|---|---|---|
| `sample_df.loc[2:4]` | `2, 3, 4` | Labels 2 through 4, **end included** |
| `sample_df.iloc[2:4]` | `2, 3` | Positions 2 and 3, **end excluded** |

#### 🔹 Round 2: after `sort_values(by='marks')`

The index order becomes `2, 3, 4, 1, 0`, so labels and positions no longer line up.

| Code | Rows returned | Why |
|---|---|---|
| `sample_df.loc[2:4]` | labels `2, 3, 4` | From label `2` to label `4` in the current order |
| `sample_df.iloc[2:4]` | labels `4, 1` | Whatever sits at **positions** 2 and 3 now |

```
 position:   0     1     2     3     4
 label:     [2]   [3]   [4]   [1]   [0]
             └─── loc[2:4] ───┘
                        └ iloc[2:4] ┘
```

> ⚠️ **Heads-up:** The saved output of this cell in the notebook shows rows `2, 3`, which is outdated. Re-run the notebook and you'll get rows **`4` and `1`** (`A105`, `A102`). Positions follow the new order after sorting.

#### 🔹 Round 3: a text index (`set_index('id')`)

```python
sample_df.iloc[2:4]            # ✅ works: positions do not care about label types
sample_df.loc[2:4]             # ❌ TypeError: there is no label 2 or 4
sample_df.loc['A104':'A102']   # ✅ slices by label: A104, A105, A102
```

The last line slices **in the DataFrame's current order**, not alphabetically. `A105` is included because it sits between `A104` and `A102` in that order.

---

## ⚔️ `.loc` vs `.iloc` side by side

```python
df.loc['A104']            df.iloc[1]              # a single row
df.loc[['A104','A101']]   df.iloc[[1, 4]]         # several rows
df.loc['A104':'A102']     df.iloc[1:4]            # slice (loc includes the end)
df.loc[:, 'marks']        df.iloc[:, 2]           # one column, all rows
df.loc['A104', 'marks']   df.iloc[1, 2]           # a single cell
df.loc[df.marks > 15]     df.iloc[(df.marks > 15).values]  # boolean filter
```

---

## ⚠️ Common pitfalls

| Pitfall | What happens | Fix |
|---|---|---|
| **Expecting `.loc` slices to exclude the end** | You get one extra row | Remember: `.loc` includes the end, `.iloc` excludes it |
| **Using `.iloc` after sorting** | You get different rows than you expect | Use `.loc` when you mean a specific record |
| **Passing integers to `.loc` on a text index** | `TypeError` / `KeyError` | Use real labels, or switch to `.iloc` |
| **Non-unique index** | `.loc['FDA15']` returns 8 rows, not 1 | Check with `df.index.is_unique` |
| **Slicing an unsorted, non-unique index by label** | `KeyError` | Run `df.sort_index()` before slicing |
| **`df['a':'c']` vs `df[['a','c']]`** | Plain `[]` slices rows but selects columns | Prefer `.loc` / `.iloc` so your intent is explicit |

---

## 📝 Cheat sheet

```python
# ── setup ──────────────────────────────────────────────
df = df.set_index('col')            # make a column the index
df = df.reset_index()               # undo it
df.index.is_unique                  # are labels unique?

# ── .loc  (labels, end INCLUDED) ───────────────────────
df.loc['x']                         # row(s) labelled x
df.loc[['x', 'y']]                  # several labels
df.loc['x':'z']                     # label slice
df.loc['x', 'col']                  # single value
df.loc[['x', 'y'], ['c1', 'c2']]    # rows × columns
df.loc[df['col'] > 10, 'c1']        # boolean mask + column

# ── .iloc (positions, end EXCLUDED) ────────────────────
df.iloc[0]                          # first row
df.iloc[-1]                         # last row
df.iloc[2:5]                        # positions 2, 3, 4
df.iloc[:, 0:3]                     # first three columns
df.iloc[[0, 2], [1, 3]]             # specific rows × columns
```

---

## 🏋️ Practice exercises

Try these in a new cell at the end of the notebook. The answers are hidden below each one.

**1.** With `Item_Identifier` as the index, get the `Item_MRP` and `Item_Outlet_Sales` for product `DRC01`.

<details><summary>Show answer</summary>

```python
data.loc['DRC01', ['Item_MRP', 'Item_Outlet_Sales']]
```
</details>

**2.** Get the first 10 rows and the first 3 columns of `data` **by position**.

<details><summary>Show answer</summary>

```python
data.iloc[:10, :3]
```
</details>

**3.** Get every `Dairy` item that costs more than 200 (`Item_MRP`), showing only `Item_Type` and `Item_MRP`.

<details><summary>Show answer</summary>

```python
data.loc[(data['Item_Type'] == 'Dairy') & (data['Item_MRP'] > 200),
         ['Item_Type', 'Item_MRP']]
```
</details>

**4.** 🤔 *Predict first:* on a DataFrame with the default index `0…9`, how many rows does `df.loc[3:6]` return? How many does `df.iloc[3:6]` return?

<details><summary>Show answer</summary>

`loc` returns **4** rows (3, 4, 5, 6). `iloc` returns **3** rows (positions 3, 4, 5).
</details>

**5.** 🧹 *Bonus:* standardise `Item_Fat_Content` so that `LF` and `low fat` become `Low Fat`, and `reg` becomes `Regular`.

<details><summary>Show answer</summary>

```python
data['Item_Fat_Content'] = data['Item_Fat_Content'].replace(
    {'LF': 'Low Fat', 'low fat': 'Low Fat', 'reg': 'Regular'}
)
data['Item_Fat_Content'].value_counts()
```
</details>

---

## 📖 Further reading

- [pandas docs: Indexing and selecting data](https://pandas.pydata.org/docs/user_guide/indexing.html)
- [`DataFrame.loc` reference](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.loc.html)
- [`DataFrame.iloc` reference](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.iloc.html)
- **Next step:** boolean / condition-based subsetting (`df[df['col'] > x]`, `.query()`)

---

<div align="center">

**Labels for meaning, positions for order. Choose deliberately.** 📍

</div>
