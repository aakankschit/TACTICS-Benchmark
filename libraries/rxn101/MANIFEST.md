# rxn101

ROCS shape-similarity benchmark library

## Reaction

Reaction id `rxn101` in the repository-root `reactions.csv`.

```
[N;D1$(N-[#6]),D2$(N(-[#6])-[#6]);$(N-[#6])!$(N-C=[O,N,S]):1].[C;D1,$(C[#6]):2](=[OD1:3])[OD1,Cl]>>[N:1][C:2](=[O:3])
```

## Reagents

| Component | File | Reagents |
|---|---|---|
| 0 | `reagents/rxn101_reagent_0.smi` | 1,000 |
| 1 | `reagents/rxn101_reagent_1.smi` | 1,000 |

2 components, 1,000 x 1,000 = **1,000,000** theoretical products.

## Reference scores

| File | Rows | Queries | Score range | Objective |
|---|---|---|---|---|
| `scores/rxn101.parquet` | 962,160 | 109 | 0.043 to 1.527 | maximize |

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
- 37,840 products (3.8%) are absent from the score set, having failed enumeration or conformer generation.

## Loading

```python
import polars as pl

df = pl.read_parquet("libraries/rxn101/scores/rxn101.parquet")
# one objective per query column; pick one to optimise
best = df.select(["Name", "query_001"]).sort("query_001", descending=True)
```
