# TACTICS-Benchmark

Reference data for benchmarking combinatorial-library search methods.

Twenty-four combinatorial libraries with exhaustively computed reference scores:
**20 ROCS** shape-similarity libraries and **4 docking** libraries, the latter scored
by two engines each. Twenty-eight score sets in all, covering roughly 87 million
scored products.

Every library folder is self-contained. It holds the reagents that build the
library, the reference scores, a `MANIFEST.md` describing both, and for docking
libraries the protein structures and design units used to produce the scores.

## Layout

```
reactions.csv              reaction SMARTS for all 24 libraries, keyed by folder name
libraries/<name>/
├── MANIFEST.md            components, schema, score ranges, provenance, caveats
├── reagents/              one .smi per component, "<SMILES> <name>" per line
├── scores/                reference scores as Parquet
└── receptors/             docking libraries only: .oedu design units + source .pdb
```

## Docking libraries

Four chemistries, each scored by both FRED and HYBRID. Chemgauss4 is a **minimize**
objective.

| Library | Components | Products | Target | Design units |
|---|---|---|---|---|
| [adenine](libraries/adenine/MANIFEST.md) | 30 x 688 x 294 | 6.07 M | Adenosine A2A receptor | 4EIY, Lenselink A2AR |
| [amide](libraries/amide/MANIFEST.md) | 10,000 x 1,000 | 10.0 M | JNK3 | 2ZDT |
| [quinazoline](libraries/quinazoline/MANIFEST.md) | 376 x 177 x 300 | 19.97 M | JNK3 | 2ZDT |
| [thrombin](libraries/thrombin/MANIFEST.md) | 130 x 3,844 | 0.50 M | Thrombin | 2ZFF |

| Score set | Rows | Range | Score set | Rows | Range |
|---|---|---|---|---|---|
| adenine (FRED) | 5,083,598 | −21.12 to 29.67 | adenine_hybrid | 5,203,413 | −19.46 to 24.52 |
| amide (FRED) | 9,199,275 | −15.41 to −4.29 | amide_hybrid | 9,609,696 | −14.23 to −2.63 |
| quinazoline (FRED) | 19,566,284 | −17.47 to 8.35 | quinazoline_hybrid | 19,566,280 | −16.52 to 6.50 |
| thrombin (FRED) | 499,720 | −17.83 to 10.03 | thrombin_hybrid | 492,982 | −15.28 to 17.63 |

FRED and HYBRID on the same chemistry are **two different ground truths, not a
comparison**. On adenine the two are scored against different receptor preparations
and their top-100 sets do not overlap at all.

## ROCS libraries

Twenty libraries, each sized to 1,000,000 theoretical products: ten two-component
(1,000 x 1,000) and ten three-component (100 x 100 x 100). Scores are ROCS
TanimotoCombo, a **maximize** objective, with **109 query molecules per library**
stored as one column each.

Two-component: [rxn101](libraries/rxn101/MANIFEST.md) ·
[rxn102a](libraries/rxn102a/MANIFEST.md) · [rxn108b](libraries/rxn108b/MANIFEST.md) ·
[rxn111b](libraries/rxn111b/MANIFEST.md) · [rxn114b](libraries/rxn114b/MANIFEST.md) ·
[rxn203](libraries/rxn203/MANIFEST.md) · [rxn205](libraries/rxn205/MANIFEST.md) ·
[rxn206](libraries/rxn206/MANIFEST.md) · [rxn207](libraries/rxn207/MANIFEST.md) ·
[rxn208](libraries/rxn208/MANIFEST.md)

Three-component: [passerini](libraries/passerini/MANIFEST.md) ·
[mannich](libraries/mannich/MANIFEST.md) ·
[niementowski](libraries/niementowski/MANIFEST.md) ·
[groebke-blackburn-bienayme](libraries/groebke-blackburn-bienayme/MANIFEST.md) ·
[dobener](libraries/dobener/MANIFEST.md) · [betti](libraries/betti/MANIFEST.md) ·
[petasis](libraries/petasis/MANIFEST.md) · [poparov](libraries/poparov/MANIFEST.md) ·
[orru](libraries/orru/MANIFEST.md) · [amide-suzuki](libraries/amide-suzuki/MANIFEST.md)

## Two schemas

Docking and ROCS scores are shaped differently. Check which one you have before
writing a loader.

```python
import polars as pl

# Docking: long, one objective
pl.read_parquet("libraries/thrombin/scores/thrombin.parquet")
# Product_Code: str, Scores: f64          -> minimize

# ROCS: wide, 109 independent objectives
pl.read_parquet("libraries/orru/scores/orru.parquet")
# Name: str, ShapeQuery: str, query_001..query_109: f64   -> maximize
```

Product codes in both are the reagent names joined by underscores, in component
order, so a code maps back to the exact reagents that built it.

## Large files

The score Parquets total about 3.3 GB and 24 of the 28 exceed GitHub's 100 MB
per-file limit, so they are tracked with [Git LFS](https://git-lfs.com). Install it
before cloning or the Parquets arrive as text pointers:

```bash
git lfs install
git clone https://github.com/aakankschit/TACTICS-Benchmark.git
```

To take the structure and manifests without the bulk data:

```bash
GIT_LFS_SKIP_SMUDGE=1 git clone https://github.com/aakankschit/TACTICS-Benchmark.git
```

## Known limitations

These are properties of the data, recorded so nobody rediscovers them the hard way.
Each library's `MANIFEST.md` carries the detail.

- **ROCS query molecules are not distributed.** The 109 columns are identified by
  index only, and no structure file for them survives. The precomputed scores are
  enough to run the benchmark, not to re-derive scores from structures.
- **Thrombin's conformers were built differently** from the other three docking
  libraries: `-maxconfs 2000`, `-strictStereo true`, no flipper, with one random
  stereoisomer assigned per product. Its scores are not directly comparable as a
  protocol replicate of the others.
- **No FRED design unit exists for thrombin.** That score set predates the
  design-unit pipeline and its receptor file was not preserved.
- **Adenine is missing two amidines.** `amidine_022` and `amidine_029` are not
  covered by the reaction SMARTS and are absent from both score sets by design.
- **ROCS score sets are incomplete by a few percent.** Products that failed
  enumeration or conformer generation are absent; each manifest states the count.

## Provenance

Scores were produced with OMEGA 6.1.1.1 and FRED/HYBRID 4.3.4.1. The docking
parameters are recorded in each docking library's manifest. The four reaction SMARTS
for the docking libraries were validated against their reagents when this repository
was assembled: all 24 reactions parse, match their component counts, and produce
products.
