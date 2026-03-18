# SAT & SMT Solver Experiments (Python / PySAT)

This repository contains a Google Colab notebook demonstrating how to solve various **SAT (Boolean Satisfiability)** problems using the **PySAT** library and custom resolution-based algorithms. It serves as an educational playground for understanding SAT solvers, CNF formulas, and how to encode problems such as graph coloring and small Sudoku variants.

> Original Colab:  
> [Open in Colab](https://colab.research.google.com/github/Vishvajitsinh-Chudasama/SAT-SMT_Solver/blob/main/SAT.ipynb)

---

## 1. Environment Setup

The notebook uses **Python 3** and the **PySAT** toolkit.

### Install dependencies (in Colab or local)

```bash
pip install python-sat
```

This provides:

- `pysat.solvers` — access to SAT solvers like `Glucose3`
- `pysat.formula` — CNF formula utilities (`CNF` class)
- `six` — compatibility dependency

---

## 2. Using the Glucose3 SAT Solver

The first part of the notebook shows how to use **Glucose3** (from PySAT) as an off‑the‑shelf SAT solver.

```python
from pysat.solvers import Glucose3

solver = Glucose3()
```

### 2.1 Basic SAT Example

Add clauses in **CNF** form:

```python
# (x1 ∨ x2) ∧ (x3 ∨ x4 ∨ x5)
solver.add_clause([1, 2])
solver.add_clause([3, 4, 5])

solved = solver.solve()
print(solved)          # True / False
print(solver.get_model())  # One satisfying assignment if SAT
```

The notebook then explores:

- Enforcing “**exactly one of three variables is true**”
- Retrieving models and interpreting them

---

## 3. Graph Coloring Encodings

The notebook shows how to encode **graph coloring** as SAT:

- Vertices and colors are encoded as Boolean variables.
- Constraints ensure:
  - Each vertex has at least one color.
  - Adjacent vertices do not share the same color.

### 3.1 Simple Graph `a -- b` with k = 2 Colors

```python
solver = Glucose3()

# a: (1 or 2), b: (3 or 4)
solver.add_clause([1, 2])
solver.add_clause([3, 4])

# a and b cannot share the same color
solver.add_clause([-1, -3])
solver.add_clause([-2, -4])

solved = solver.solve()
print(solved)
if solved:
    print(solver.get_model())
```

### 3.2 Triangle Graph `a -- b -- c -- a`

- First with **2 colors** → UNSAT (cannot color a triangle with 2 colors without conflict).
- Then with **3 colors** → SAT, showing a valid assignment.

---

## 4. Tiny Sudoku‑like Example (2×2)

The notebook encodes a toy **2×2 Sudoku**:

```
|---|---|
| b | c |
|---|---|
| a | d |
|---|---|
```

- Binary/one‑hot constraints ensure each “cell” has valid values.
- One of the cells (b) is pre‑assigned.
- The SAT solver finds a model consistent with all constraints.

Example structure:

```python
a, b, c, d = 1, 2, 3, 4

solver.add_clause([b, c])
solver.add_clause([-b, -c])

solver.add_clause([a, d])
solver.add_clause([-a, -d])
# ...
solver.add_clause([b])  # b is fixed

solved = solver.solve()
print(solved)
if solved:
    print(solver.get_model())
```

---

## 5. Custom SAT Solver via Resolution (DP‑style)

The final section implements a **custom SAT solver** using a resolution‑based approach (similar to the Davis–Putnam procedure) over CNF formulas.

### 5.1 CNF and Resolution

```python
from pysat.formula import CNF

def resolve(c1, c2, var):
    new_clause = set(c1) | set(c2)
    new_clause.remove(var)
    new_clause.remove(-var)

    # Skip clauses that become trivially true (x and ¬x)
    for i in new_clause:
        if -i in new_clause:
            return None
    return list(new_clause)
```

### 5.2 SAT_Solver(Formula)

```python
def SAT_Solver(Formula):
    clauses = Formula.clauses[:]
    all_var = set(abs(l) for clause in clauses for l in clause)

    for var in all_var:
        pos_clause = [clause for clause in clauses if var in clause]
        neg_clause = [clause for clause in clauses if -var in clause]
        other_clause = [clause for clause in clauses if var not in clause and -var not in clause]

        if not pos_clause or not neg_clause:
            clauses = other_clause
            continue

        new_clause = []
        for c1 in pos_clause:
            for c2 in neg_clause:
                ans = resolve(c1, c2, var)
                if ans is not None:
                    new_clause.append(ans)

        if [] in new_clause:
            return "UNSAT"

        clauses = other_clause + new_clause
        print("Remaining Clauses : ", clauses)

        if not clauses:
            return "SAT"

    if not clauses:
        return "SAT"
    else:
        return "UNSAT"
```

### 5.3 Example Formulas

```python
f = CNF()
f.append([1, 2, 3])
f.append([2, -3, -4])
f.append([-2, 4])
f.append([-1, -4])
print(SAT_Solver(f))   # SAT
```

```python
f1 = CNF()
f1.append([1, 2])
f1.append([1, -2])
f1.append([-1, 3])
f1.append([-1, -3])
print(SAT_Solver(f1))  # UNSAT
```

---

## 6. How to Run

### Option A: Google Colab

1. Open the notebook in Colab:

   - `SAT.ipynb` (or click the “Open in Colab” badge at the top of the notebook)

2. Run the first cell to install dependencies:

   ```python
   pip install python-sat
   ```

3. Execute cells sequentially to:
   - See SAT examples with `Glucose3`
   - Experiment with graph coloring encodings
   - Run the tiny Sudoku example
   - Try the custom `SAT_Solver` on different CNF formulas

### Option B: Local Jupyter

1. Create a virtual environment (optional but recommended):

   ```bash
   python -m venv venv
   source venv/bin/activate  # Linux/macOS
   venv\Scripts\activate     # Windows
   ```

2. Install requirements:

   ```bash
   pip install notebook python-sat
   ```

3. Launch Jupyter:

   ```bash
   jupyter notebook
   ```

4. Open `SAT.ipynb` and run the cells.

---

## 7. Learning Outcomes

Working through this notebook, you will:

- Understand how **CNF clauses** encode logical constraints.
- Learn to use a **modern SAT solver (Glucose3)** via PySAT.
- See how to model **“exactly one”**, **graph coloring**, and **mini-Sudoku** as SAT.
- Explore a **hand‑rolled resolution‑based SAT algorithm**, and observe how clause sets shrink or lead to contradictions (UNSAT).

This makes the notebook a good starting point for courses or self‑study on **SAT / SMT solving** and **constraint encoding** in Python.

