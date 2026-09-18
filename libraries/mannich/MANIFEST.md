# mannich

ROCS shape-similarity benchmark library

## Reaction

Reaction id `mannich` in the repository-root `reactions.csv`.

```
[#6:3]-[#7H]-[#6:4].[#6H](-[#6:5])=O.[#6:2]-[#6H2]-[#6](-[#6:1])=O>>[#6:1]-[#6]-[#6](=O)-[#6](-[#6:2])-[#6](-[#6:5])-[#7](-[#6:3])-[#6:4]
```

## Reagents

| Component | File | Reagents |
|---|---|---|
| 0 | `reagents/mannich_reagent_0.smi` | 100 |
| 1 | `reagents/mannich_reagent_1.smi` | 100 |
| 2 | `reagents/mannich_reagent_2.smi` | 100 |

3 components, 100 x 100 x 100 = **1,000,000** theoretical products.

## Reference scores

| File | Rows | Queries | Score range | Objective |
|---|---|---|---|---|
| `scores/mannich.parquet` | 941,150 | 109 | 0.053 to 1.27 | maximize |

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
- 58,850 products (5.9%) are absent from the score set, having failed enumeration or conformer generation.

## Loading

```python
import polars as pl

df = pl.read_parquet("libraries/mannich/scores/mannich.parquet")
# one objective per query column; pick one to optimise
best = df.select(["Name", "query_001"]).sort("query_001", descending=True)
```
