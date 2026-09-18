# poparov

ROCS shape-similarity benchmark library

## Reaction

Reaction id `poparov` in the repository-root `reactions.csv`.

```
[#7H2]-c1[cH:2][c:3][c:4][c:5][cH:6]1.[#6H](-[#6:7])=[#6H]-[#6:8].[#6H](-[#6:1])=O>>[#6:8]-[#6]-1-[#7]-c2[c:6][c:5][c:4][c:3][c:2]2-[#6](-[#6:1])-[#6]-1-[#6:7]
```

## Reagents

| Component | File | Reagents |
|---|---|---|
| 0 | `reagents/poparov_reagent_0.smi` | 100 |
| 1 | `reagents/poparov_reagent_1.smi` | 100 |
| 2 | `reagents/poparov_reagent_2.smi` | 100 |

3 components, 100 x 100 x 100 = **1,000,000** theoretical products.

## Reference scores

| File | Rows | Queries | Score range | Objective |
|---|---|---|---|---|
| `scores/poparov.parquet` | 870,013 | 109 | 0.052 to 1.151 | maximize |

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
- 129,987 products (13.0%) are absent from the score set, having failed enumeration or conformer generation.

## Loading

```python
import polars as pl

df = pl.read_parquet("libraries/poparov/scores/poparov.parquet")
# one objective per query column; pick one to optimise
best = df.select(["Name", "query_001"]).sort("query_001", descending=True)
```
