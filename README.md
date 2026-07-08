# Advanced Machine Learning — Assignment 4: The PC Algorithm

Implementation and application of the **PC algorithm** for causal structure
discovery, developed for the Advanced Machine Learning course.

**Authors:** Alessandro Coron (1138154) · Yazan Mousa (4971698)

---

## Overview

The project is split into two parts:

1. **Implementing the PC algorithm** — completing a Python implementation that
   recovers the Markov equivalence class (a PDAG) of a causal DAG from
   conditional-independence information.
2. **Applying PC to real data** — running the algorithm on the biological
   dataset of Sachs et al. (2005) to discover causal relations between proteins,
   and using interventional experiments to reason about edge directions.

Everything lives in a single Jupyter notebook: [`Coron_Mousa.ipynb`](Coron_Mousa.ipynb).

## Graph representation

Graphs (PDAGs) are represented as a boolean `numpy` array `G`:

| Condition | Meaning |
| --- | --- |
| `G[x,x] == False` | no self-loops |
| `G[x,y] == False` and `G[y,x] == False` | no edge between `x` and `y` |
| `G[x,y] == True` and `G[y,x] == True` | undirected edge `x — y` |
| `G[x,y] == True` and `G[y,x] == False` | directed edge `x → y` |

Graphs are visualized with [`graphviz`](https://graphviz.org/).

## The PC algorithm (`PC_algorithm` class)

The algorithm runs in four phases; the parts implemented in this assignment are
marked below:

1. **Initialization** — start from the fully connected undirected graph.
2. **Skeleton search** — for increasing conditioning-set size `k`, test each
   edge `x–y` against subsets `S ⊆ Adj(x)\{y}`; remove the edge and record the
   separating set on the first independence found. *(Task 1)*
3. **Orient v-structures** — for every unshielded triple `x – z – y`, orient
   `x → z ← y` when `z` is not in the separating set of `x` and `y`, taking care
   not to delete already-oriented edges. *(Task 2)*
4. **Orientation rules** — apply Meek's rules 1–3 repeatedly until no further
   edges can be oriented (a fixed-point loop, since some rules only fire after
   others have). *(Task 3)*

Independence information is supplied by a pluggable *independence tester*:

- **`IndependenceOracle`** — answers via `d`-separation on a known true DAG.
  Used for testing.
- **`IndependenceTester`** — a statistical test on real data based on partial
  (Spearman) correlations, comparing a p-value against a threshold `alpha`.

## Testing

The implementation is validated against five ground-truth DAGs (`G1`–`G5`) using
the oracle, checking that PC recovers the correct Markov equivalence class.
`G5` includes a **latent variable**, illustrating a case where no DAG over the
observed variables is Markov and faithful to the distribution (discussed in
Question 4).

## Application: the Sachs et al. (2005) dataset

`sachs2005_combined.csv` contains flow-cytometry measurements of **11 proteins**
across thousands of single human immune cells. A twelfth column, `experiment`,
labels how each cell was prepared:

- `experiment = 1` → observational data (853 cells),
- `experiment = 2…14` → various interventional datasets.

The assignment covers:

- **Task 5–6** — run PC on the observational data (`experiment = 1`) and display
  the discovered graph.
- **Question 7** — assess whether the structural equations plausibly match the
  linear / linear-Gaussian / additive-noise forms (they don't: the data are
  strongly non-Gaussian and heavy-tailed).
- **Task 8 – Question 10** — combine observational and interventional data
  (experiments 1+5 and 1+6) and use scatter/histogram plots to reason about
  which variables are intervened on and how edges should be oriented (e.g.
  `PIP2 → PIP3`, `praf → pmek`).

## Requirements

Install the dependencies (also listed in [`requirements.txt`](requirements.txt)):

```
graphviz==0.20.3
numpy==1.26.4
pandas==2.2.2
scipy==1.13.1
```

```bash
pip install -r requirements.txt
```

`matplotlib` is additionally required for the plots in Part 2, and the
[Graphviz system package](https://graphviz.org/download/) must be installed for
graph rendering.

## Usage

Open and run the notebook top to bottom:

```bash
jupyter notebook Coron_Mousa.ipynb
```

## References

- J. M. Mooij, S. Magliacane, and T. Claassen. *Joint Causal Inference from
  Multiple Contexts.* JMLR 21(99):1−108, 2020.
- J. Runge. *Conditional Independence Testing Based on a Nearest-Neighbor
  Estimator of Conditional Mutual Information.* AISTATS, 2018.
- K. Sachs, O. Perez, D. Pe'er, D. A. Lauffenburger, and G. P. Nolan. *Causal
  protein-signaling networks derived from multiparameter single-cell data.*
  Science, 308(5721):523–529, 2005.
