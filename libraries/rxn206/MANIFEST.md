# rxn206

ROCS shape-similarity benchmark library

## Reaction

Reaction id `rxn206` in the repository-root `reactions.csv`.

```
[#6;A;D3$(C([#6])[#6]),D2$(C[#6]):1][#8;A;D1].[#7;A;D2;$(N([#6])S(=O)=O):2]>>[#6:1]-[#7:2]
```

## Reagents

| Component | File | Reagents |
|---|---|---|
| 0 | `reagents/rxn206_reagent_0.smi` | 1,000 |
| 1 | `reagents/rxn206_reagent_1.smi` | 1,000 |

2 components, 1,000 x 1,000 = **1,000,000** theoretical products.

## Reference scores

| File | Rows | Queries | Score range | Objective |
|---|---|---|---|---|
| `scores/rxn206.parquet` | 973,130 | 109 | 0.052 to 1.52 | maximize |

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

- 2-component library; all 20 ROCS libraries in this set are sized to 1,000,000 theoretical products.
- 26,870 products (2.7%) are absent from the score set, having failed enumeration or conformer generation.

## Loading

```python
import polars as pl

df = pl.read_parquet("libraries/rxn206/scores/rxn206.parquet")
# one objective per query column; pick one to optimise
best = df.select(["Name", "query_001"]).sort("query_001", descending=True)
```
