# thrombin

Structure-based docking benchmark library -- Thrombin

## Reaction

Reaction id `thrombin` in the repository-root `reactions.csv`.

```
[#6:1](=[O:2])[OH].[#7X3;H1,H2;!$(N[!#6]);!$(N[#6]=[O]);!$(N[#6]~[!#6;!#16]):3]>>[#6:1](=[O:2])[#7:3]
```

## Reagents

| Component | File | Reagents |
|---|---|---|
| 0 | `reagents/thrombin_reagent_0.smi` | 130 |
| 1 | `reagents/thrombin_reagent_1.smi` | 3,844 |

2 components, 130 x 3,844 = **499,720** theoretical products.

## Reference scores

| File | Rows | Score range | Mean | Engine | Objective |
|---|---|---|---|---|---|
| `scores/thrombin.parquet` | 499,720 | -17.829 to 10.028 | -10.707 | FRED | minimize |
| `scores/thrombin_hybrid.parquet` | 492,982 | -15.279 to 17.629 | -6.732 | HYBRID | minimize |

**Long schema**, identical for both files:

```
Product_Code  str   underscore-joined reagent names
Scores        f64   Chemgauss4 score
```

Chemgauss4 is a **minimize** objective: lower is better. Where a product appears more than once in the raw output (stereoisomers from flipper), the lowest score was kept.

## Protein structures

| File | Role |
|---|---|
| `receptors/2ZFF_HIL__DU__53U_H-2001.oedu` | design unit, HYBRID campaign |
| `receptors/2zff.pdb` | source structure |

| PDB | Structure | Resolution |
|---|---|---|
| [2ZFF](https://www.rcsb.org/structure/2ZFF) | Thrombin S1-pocket complex | 1.47 A |

`.oedu` files are OpenEye design units and are what the docking tools consume directly. The `.pdb` files are the crystallographic structures they were prepared from, included so the preparation can be inspected or redone.

## Docking parameters

**Thrombin does not use the settings the other three docking libraries use.** Its
conformers came from an earlier campaign with different parameters:

| Stage | Setting |
|---|---|
| conformers | `omega2 -ewindow 10 -maxconfs 2000 -strictStereo true` (**no flipper**) |
| stereochemistry | one random stereoisomer per product, assigned rather than enumerated |
| HYBRID | `hybrid -hitlist_size 0 -num_poses 1` |
| versions | OMEGA 6.1.1.1, HYBRID 4.3.4.1 |

For contrast, the settings used by `adenine`, `amide` and `quinazoline`:

| Stage | Setting |
|---|---|
| conformers | `omega2 -ewindow 10 -maxconfs 1000 -strictstereo false -flipper true` |
| FRED | `fred -dock_resolution High -hitlist_size 0 -num_poses 1` |
| HYBRID | `hybrid -hitlist_size 0 -num_poses 1` |

## Notes

- ~30:1 component-size imbalance (130 x 3,844). This is the imbalanced-library case in the benchmark set: at typical budgets each dipeptide receives only a handful of observations.
- **The conformers for this library were built differently from every other docking library.** The originating campaign used `-maxconfs 2000` and `-strictStereo true` with no flipper, assigning a single random stereoisomer per product rather than enumerating them. That is why the FRED set is exactly 130 x 3,844 = 499,720 rows with no duplicates, while the other libraries lose products to SMARTS failures and gain duplicates from flipper.
- No FRED design unit is distributed here: the FRED score set predates the design-unit based pipeline and its receptor file was not preserved. The 2ZFF unit shipped here was used for the HYBRID campaign.

## Loading

```python
import polars as pl

df = pl.read_parquet("libraries/thrombin/scores/thrombin.parquet")
best = df.sort("Scores").head(100)   # minimize
```
