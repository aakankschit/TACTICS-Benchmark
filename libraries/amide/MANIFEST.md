# amide

Structure-based docking benchmark library -- JNK3 (c-Jun N-terminal kinase 3)

## Reaction

Reaction id `amide` in the repository-root `reactions.csv`.

```
[NH2:2][#6:1].[#6:4][C:3]([OH])=O>>[NH:2]([#6:1])[C:3]([#6:4])=O
```

## Reagents

| Component | File | Reagents |
|---|---|---|
| 0 | `reagents/amide_reagent_0.smi` | 10,000 |
| 1 | `reagents/amide_reagent_1.smi` | 1,000 |

2 components, 10,000 x 1,000 = **10,000,000** theoretical products.

## Reference scores

| File | Rows | Score range | Mean | Engine | Objective |
|---|---|---|---|---|---|
| `scores/amide.parquet` | 9,199,275 | -15.408 to -4.288 | -8.897 | FRED | minimize |
| `scores/amide_hybrid.parquet` | 9,609,696 | -14.227 to -2.627 | -7.766 | HYBRID | minimize |

**Long schema**, identical for both files:

```
Product_Code  str   underscore-joined reagent names
Scores        f64   Chemgauss4 score
```

Chemgauss4 is a **minimize** objective: lower is better. Where a product appears more than once in the raw output (stereoisomers from flipper), the lowest score was kept.

## Protein structures

| File | Role |
|---|---|
| `receptors/2zdt_receptor.oedu` | design unit, both campaigns |
| `receptors/2zdt.pdb` | source structure |

| PDB | Structure | Resolution |
|---|---|---|
| [2ZDT](https://www.rcsb.org/structure/2ZDT) | Human JNK3 with an isoquinolone inhibitor | 2.0 A |

`.oedu` files are OpenEye design units and are what the docking tools consume directly. The `.pdb` files are the crystallographic structures they were prepared from, included so the preparation can be inspected or redone.

## Docking parameters

| Stage | Setting |
|---|---|
| conformers | `omega2 -ewindow 10 -maxconfs 1000 -strictstereo false -flipper true` |
| FRED | `fred -dock_resolution High -hitlist_size 0 -num_poses 1` |
| HYBRID | `hybrid -hitlist_size 0 -num_poses 1` (no resolution flag; pose reference is the bound ligand) |
| versions | OMEGA 6.1.1.1, FRED/HYBRID 4.3.4.1 |

## Notes

- 10:1 component-size imbalance (10,000 x 1,000).
- Both campaigns share one design unit, so the two score sets differ only by scoring function.

## Loading

```python
import polars as pl

df = pl.read_parquet("libraries/amide/scores/amide.parquet")
best = df.sort("Scores").head(100)   # minimize
```
