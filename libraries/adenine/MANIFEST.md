# adenine

Structure-based docking benchmark library -- Adenosine A2A receptor

## Reaction

Reaction id `adenine` in the repository-root `reactions.csv`.

```
[#7:1][#6:2]:[#7:3].[*:4][N:5]=[#6:6].[*:7][#6H1:8]=O>>[*:7][#6X3:8]1[#7X2:1]=[#6X3:2][#7X3:3][#6X3:6]=1([NX3:5][*:4])
```

## Reagents

| Component | File | Reagents |
|---|---|---|
| 0 | `reagents/adenine_reagent_0.smi` | 30 |
| 1 | `reagents/adenine_reagent_1.smi` | 688 |
| 2 | `reagents/adenine_reagent_2.smi` | 294 |

3 components, 30 x 688 x 294 = **6,068,160** theoretical products.

## Reference scores

| File | Rows | Score range | Mean | Engine | Objective |
|---|---|---|---|---|---|
| `scores/adenine.parquet` | 5,083,598 | -21.12 to 29.665 | -12.092 | FRED | minimize |
| `scores/adenine_hybrid.parquet` | 5,203,413 | -19.458 to 24.523 | -8.157 | HYBRID | minimize |

**Long schema**, identical for both files:

```
Product_Code  str   underscore-joined reagent names
Scores        f64   Chemgauss4 score
```

Chemgauss4 is a **minimize** objective: lower is better. Where a product appears more than once in the raw output (stereoisomers from flipper), the lowest score was kept.

## Protein structures

| File | Role |
|---|---|
| `receptors/4EIY_A__DU__ZMA_A-2401.oedu` | design unit, FRED campaign |
| `receptors/a2ar-lenselink.oedu` | design unit, HYBRID campaign |
| `receptors/4eiy.pdb` | source structure for the FRED design unit |
| `receptors/lenselink_protein_ligand1.pdb` | source structure for the HYBRID design unit |
| `receptors/ligands_lenselink.sdf` | reference ligands used in the Lenselink preparation |

| PDB | Structure | Resolution |
|---|---|---|
| [4EIY](https://www.rcsb.org/structure/4EIY) | A2A receptor / BRIL chimera with ZM241385 | 1.8 A |

`.oedu` files are OpenEye design units and are what the docking tools consume directly. The `.pdb` files are the crystallographic structures they were prepared from, included so the preparation can be inspected or redone.

## Docking parameters

| Stage | Setting |
|---|---|
| conformers | `omega2 -ewindow 10 -maxconfs 1000 -strictstereo false -flipper true` |
| FRED | `fred -dock_resolution High -hitlist_size 0 -num_poses 1` |
| HYBRID | `hybrid -hitlist_size 0 -num_poses 1` (no resolution flag; pose reference is the bound ligand) |
| versions | OMEGA 6.1.1.1, FRED/HYBRID 4.3.4.1 |

## Notes

- Two campaigns against two different preparations of the same receptor. They are **separate ground truths, not a comparison** -- their top-100 sets do not overlap.
- `adenine.parquet` is the **post-merge** file (5,083,598 rows, 28 amidine prefixes, 174,000 `amidine_035` products). An earlier 4,909,597-row version with 27 prefixes and no `amidine_035` is superseded; check the row count if in doubt.
- Two of the 30 amidines, `amidine_022` and `amidine_029`, are not covered by the reaction SMARTS. Their enumerated SMILES are structurally invalid, conformer generation yields empty files, and they are absent from both score sets by design.
- The FRED and HYBRID design units differ in provenance: the FRED unit derives from PDB 4EIY directly, the HYBRID unit from a Lenselink-prepared structure supplied here as `lenselink_protein_ligand1.pdb`.

## Loading

```python
import polars as pl

df = pl.read_parquet("libraries/adenine/scores/adenine.parquet")
best = df.sort("Scores").head(100)   # minimize
```
