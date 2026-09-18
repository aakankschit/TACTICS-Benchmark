# quinazoline

Structure-based docking benchmark library -- JNK3 (c-Jun N-terminal kinase 3)

## Reaction

Reaction id `quinazoline` in the repository-root `reactions.csv`.

```
N[c:4][c:3]C(O)=O.[#6:1][NH2].[#6:2]C(=O)[OH]>>[C:2]c1n[c:4][c:3]c(=O)n1[C:1]
```

## Reagents

| Component | File | Reagents |
|---|---|---|
| 0 | `reagents/quinazoline_reagent_0.smi` | 376 |
| 1 | `reagents/quinazoline_reagent_1.smi` | 177 |
| 2 | `reagents/quinazoline_reagent_2.smi` | 300 |

3 components, 376 x 177 x 300 = **19,965,600** theoretical products.

## Reference scores

| File | Rows | Score range | Mean | Engine | Objective |
|---|---|---|---|---|---|
| `scores/quinazoline.parquet` | 19,566,284 | -17.467 to 8.354 | -9.518 | FRED | minimize |
| `scores/quinazoline_hybrid.parquet` | 19,566,280 | -16.519 to 6.498 | -8.456 | HYBRID | minimize |

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

- The largest library in the set at ~19.6M products.
- Both campaigns share one design unit, so the two score sets differ only by scoring function. The two score sets differ by only 4 products.

## Loading

```python
import polars as pl

df = pl.read_parquet("libraries/quinazoline/scores/quinazoline.parquet")
best = df.sort("Scores").head(100)   # minimize
```
