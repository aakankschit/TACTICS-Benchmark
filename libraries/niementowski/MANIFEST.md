# niementowski

ROCS shape-similarity benchmark library

## Reaction

Reaction id `niementowski` in the repository-root `reactions.csv`.

```
N[c:4][c:3]C(O)=O.[#6:1][NH2].[#6:2]C(=O)[OH]>>[C:2]c1n[c:4][c:3]c(=O)n1[C:1]
```

## Reagents

| Component | File | Reagents |
|---|---|---|
| 0 | `reagents/niementowski_reagent_0.smi` | 100 |
| 1 | `reagents/niementowski_reagent_1.smi` | 100 |
| 2 | `reagents/niementowski_reagent_2.smi` | 100 |

3 components, 100 x 100 x 100 = **1,000,000** theoretical products.

## Reference scores

| File | Rows | Queries | Score range | Objective |
|---|---|---|---|---|
| `scores/niementowski.parquet` | 1,000,000 | 109 | 0.051 to 1.221 | maximize |

**Wide schema.** One row per product, one column per query molecule:

```
Name         str      product code, underscore-joined reagent names
ShapeQuery   str      constant ('MOL001_1')
query_001    f64      TanimotoCombo similarity for query 1
   ...                109 query columns in total
query_109    f64      TanimotoCombo similarity for query 109
```

Scores are ROCS TanimotoCombo, so **higher is better** and the useful range is roughly 0 to 2. Each of the 109 queries is an independent optimisation problem over the same product set.

> The query molecules themselves are **not** distributed. The columns are identified only by index, and no structure file for them survives in the source tree. Precomputed scores are sufficient to run and reproduce the benchmark, but not to re-derive the scores from structures.

## Notes

- 3-component library; all 20 ROCS libraries in this set are sized to 1,000,000 theoretical products.
- The score set is complete: every theoretical product is present.

## Loading

```python
import polars as pl

df = pl.read_parquet("libraries/niementowski/scores/niementowski.parquet")
# one objective per query column; pick one to optimise
best = df.select(["Name", "query_001"]).sort("query_001", descending=True)
```
