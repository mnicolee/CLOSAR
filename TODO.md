# TODO

Items in **Priority** below must be addressed before anything in Later.
Covers all three repos: `OSAR-Nicole-version`, `CLOSAR`, `DrosoClimb`.

---

## Priority 0 — do this first, before anything else

### DrosoClimb `Dev` branch is broken

The Dropbox folder `Data Compilation\Falling_New\` was renamed to `Climbing_New\` on
2026-08-05, and the font path moved to a local `fonts\` folder. Those fixes were applied to
`Dev_AsOPN3` only. **`Dev` still points at paths that no longer exist.**

8 notebooks on `Dev` need it:

| Notebook | `Falling_New` refs | old font path |
|---|---|---|
| `Countingfiles.ipynb` | 1 | — |
| `Dabest_six_plots.ipynb` | 5 | 1 |
| `Forestplot.ipynb` | 1 | 1 |
| `Heatmap2_withnoclassifications.ipynb` | 1 | 1 |
| `Heatmappotofu_1.ipynb` | 1 | 1 |
| `LinReg.ipynb` | 1 | 1 |
| `LinReg_2024.ipynb` | 1 | 1 |
| `PCA.ipynb` | 1 | — |

Also missing from `Dev`: `fonts/` (18 Inter TTFs), `NLProcessing.py`, `MBONlist.csv`,
`vortexmap.py`.

Note `3. AsOPN3_Multi-file processing.ipynb` and `4. AsOPN3_multistate_processing.ipynb` exist
**only** on `Dev_AsOPN3`, so the existing commit there cannot simply be moved to `Dev`.

Intended flow: fix and push `Dev`, then `Dev_AsOPN3` pulls from `Dev`. Beware — the branches
have diverged by 42 (Dev) and 56 (Dev_AsOPN3) commits across 51 files, so that merge is not
trivial.

---

## Priority 1 — functions still unresolved

### 1a. `multiresponder_osar` — 4 copies, versions not yet compared

In `Linregonly` (OSAR), `Multi and single dabest OSAR plots` (OSAR),
`Climbing-OSAR classification`, `Climbing-OSAR comparison`. Used by 3 notebooks, so it
qualifies for `NLProcessing` — but the copies differ and one has already caused a bug.

Known so far: `Linregonly` now holds the 7-metric version producing `Δ LAI`; the
`Multi and single dabest` copy produces 4 metrics; the CLOSAR copies are unverified. Compare
all four before moving anything.

### 1b. `singleplottingdabest` — 2 copies, 2 versions

In `Multi and single dabest OSAR plots` and `Single dabest plot`. Used by 2 notebooks, so it
qualifies for `NLProcessing`, but the versions differ. Compare first.

### 1c. `rename_labels` — movable only with a signature change

In `Osar Phenomics` and `Climbing-OSAR phenomics`, identical. Reads the notebook global
`mbon_csv_path`, so moving it to `NLProcessing` means
`rename_labels(mbons, mbon_csv_path, use_mbon_numbers=False)`. Decide whether that's wanted.

### 1d. Unused functions, ~272 lines (OSAR repo)

| Notebook | Functions | Lines |
|---|---|---|
| `Light Intensity Summary` | `create_forest_plot`, `create_heatmap`, `create_multi_forest_plot`, `create_multi_heatmap`, `get_available_metrics` | 186 |
| `Linregonly` | `meandiffchart` | 20 |
| `Multi and single dabest OSAR plots` | `forestplot_clusterplot`, `multiresponder_osar` (dead copy) | 45 |
| `3. Complete file generation` | `forestplot_multiplot` (dead copy) | 11 |
| `Asovalencevsme` | `forestplot_multiplot` (dead copy) | 10 |

`forestplot_multiplot` is live in `Multi and single dabest`, and `multiresponder_osar` is live
in `Linregonly` + 2 CLOSAR notebooks — delete only the dead copies.

### 1e. Known defects, not fixed

- **`chartplottingdabest_truncated`** (`Climbing-OSAR classification`, `Climbing-OSAR
  comparison`) truncates the offspring group to the control's n, because assigning a Series
  into an empty DataFrame fixes the index from the first column. Confirmed intended, so left
  alone — but it means Hedges' g there is computed on fewer offspring flies than exist.
- **`Climbing-OSAR comparison` cell 33** references `dfreg_a` / `dfreg_c` / `dfreg_ao`, which
  nothing has built since 2025-03-13. Cell will `NameError` on a fresh kernel. Data still
  exists as `dfreg_ACR` / `dfreg_Chrimson2` (renamed deliberately).
- **`Climbing-OSAR comparison` cell 9**: `MBONList = []666` — stray `666` introduced in commit
  `652f4791` (2025-10-04). Should be `MBONList = []`.
- **`Fly Track Demo` cell 12** (OSAR): `for` loop whose only body line is commented out.

---

## Later

- `REORG_PLAN.md` (in `OSAR-Nicole-version`) — remaining open items in its decision log.
- Fuse the two loops in `4. Appendix and bootstrap values` into a single dabest pass
  (~3,000 fits currently run twice).
- `Light Intensity Summary` carries its own drifted copy of the vortex code, separate from
  `vortexmap.py`.
- Machine-path block (`officecomp`/`labcomp`/`homecomp`) still copy-pasted across ~15
  notebooks.
- `2. Multi-OSAR file processing.ipynb` still sits alongside the "new" one.
- README for each repo.

---

## Done (2026-08-05/06)

- Falling → Climbing rename: Dropbox folders, 4 xlsx files, `_Falling` column headers in 5
  workbooks, notebook filenames, ~150 comments/labels.
- Split climbing work out of OSAR into CLOSAR; `NLCLIMB`/`NLGRAPHS`/`NLMATH` moved to CLOSAR.
- `NLProcessing` now holds `find_number`, `generate_lobelocation`, `wrap_labels`,
  `parse_lobes`, `parse_nt` — copied to all three repos.
- 63+ duplicate function definitions removed; `find_number` fixed to sort and compress
  (`MBON01, 03`); Greek-letter corruption of lobe names removed.
- `fonts/` + `MBONlist.csv` added to all three repos; paths made repo-relative.
- `matchingcomp`/`matchinglobesets`/`matchingdfs` replaced by `pd.merge`.
- All four `omission` versions now drop `R76B09` and `VT999036`.
