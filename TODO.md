# TODO

Items in **Priority** below must be addressed before anything in Later.
Covers all three repos, all under `C:\Users\user\Documents\GitHub\`:
`OSAR-Nicole-version`, `CLOSAR`, `DrosoClimb`.

---

## ⚠ READ BEFORE TOUCHING ANYTHING — Dropbox rule

Data lives in `D:\ACC Lab Dropbox\ACC Lab\Nicole Lee\` and files there are **online-only**.
Anything that reads a file or enumerates a directory makes Dropbox hydrate (download) it, and
**the download cannot be cancelled**. This has already happened twice by accident.

**Never, against any Dropbox path:**

- `os.listdir`, `glob`, `Path.iterdir`, `os.walk` — no directory enumeration in Python
- Git Bash `ls` — Cygwin `ls` opens every entry to test for symlinks, which hydrates it
- executing notebook cells, or any script whose code touches those paths
- reading a file to "verify" something

**Verify notebooks by parsing and compiling only** (`json.load`, `compile`) — never by running
them. If a directory listing is genuinely unavoidable, use PowerShell
`Get-ChildItem <path> -Directory -Name`, which reads directory metadata natively and was
verified not to hydrate. `Rename-Item` is metadata-only and safe. Ask the user first.

Everything below can be done **without touching Dropbox at all** — it is all repo-local file
edits.

---

## Priority 0 — START HERE. Nothing else until this is done.

### DrosoClimb `Dev` branch is broken right now

**Steps**

1. `git -C DrosoClimb checkout Dev`
2. In the 8 notebooks below, replace `Falling_New` with `Climbing_New`.
3. In the 6 that use it, repoint the font path to `fonts` (`font_dirs = ["fonts"]` /
   `font_path = "fonts"`).
4. Copy in `fonts/` (18 Inter TTFs), `NLProcessing.py`, `MBONlist.csv`, `vortexmap.py` —
   all present on `Dev_AsOPN3` and in the other two repos.
5. Commit and push `Dev`.
6. Then `Dev_AsOPN3` pulls from `Dev`. **Check the merge carefully** — see the divergence
   warning below.

**Why it is broken**

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

### 1f. OSAR `Vortex Maps.ipynb` — broken by the `vortexmap.py` cleanup (2026-08-15)

`load_bootstrap_data` lost its `METRICS` module-global fallback, so `metrics` is now a required
argument. One call site was never updated:

- **cell 5**: `load_bootstrap_data(data_path, responders=responders)` →
  `load_bootstrap_data(data_path, METRICS, responders=responders)`.
  `METRICS` is already defined in cell 1 (the 7 OSAR metrics), so that is the whole fix.
  Cells 7–9 already pass metrics explicitly and need nothing.

Found while there, not yet fixed:

- **cell 6** calls `NLProcessing.generate_lobelocation(mbons, ...)` but `mbons` isn't bound until
  cell 7. `NameError` on a fresh kernel — same class as the `dfreg_a` bug in 1e.
- **cell 6** sets `mbon_csv_path = "MBONlist.csv"` then passes the literal again instead of the
  variable.

Also dead now: in CLOSAR `Climbing-OSAR vortexmaps.ipynb` cell 1, the line
`METRICS = OSAR_METRICS  # default used by the reused load_bootstrap_data`. That fallback never
worked (`from vortexmap import *` leaves the function reading vortexmap's globals, not the
notebook's). It already passes `metrics=OSAR_METRICS` explicitly, so the line can just go.

### 1f-2. Merge `NLMATH` + `NLGRAPHS` into `NLCLIMB` (DrosoClimb) — ON HOLD

Decided to do this, then paused. **Do not delete the dead functions yet** — leave them in place.

Measured on `Dev_AsOPN3`: 54 functions total, **32 live, 22 dead** (liveness = reachable from a
notebook, following internal calls).

| Module | live | dead |
|---|---|---|
| `NLCLIMB.py` | 16 | 0 |
| `NLMATH.py` | 16 | 14 |
| `NLGRAPHS.py` | 0 | 8 — nothing calls any of it |

Three name collisions between `NLCLIMB` and `NLMATH`:

- `frames` — identical, NLMATH's copy dead. Keep NLCLIMB's.
- `speedcalc` — differ, but NLMATH's is dead (only reachable via `avgmean`, itself dead).
  Keep NLCLIMB's.
- `separation` — **both live and genuinely different functions.** `NLCLIMB.separation(df, phase)`
  filters one phase; `NLMATH.separation(dfexpt, dfwt, phrase)` splits expt+wt into
  Dark/Full/Recovery. **RESOLVED 2026-08-15**: NLMATH's renamed to `refine` (def + its 2 callers
  `fallingocc`, `totalheight`); `NLCLIMB.separation` kept as-is. No notebook edits were needed —
  neither was ever called directly from a notebook.

Remaining work when resumed: rewrite `NLMATH.x(...)` → `NLCLIMB.x(...)` across the 10 notebooks
importing `NLMATH` and the 2 importing `NLGRAPHS`, then concatenate. `Manuscript related/` has its
own byte-identical `NLCLIMB.py`/`NLMATH.py` copies that must move in step.

Note: `rastergraph` is commented `#obsolete` in `NLMATH.py` but IS used by
`Generic line plots - figures.ipynb`. The comment is wrong — do not delete it.

CLOSAR is a separate case: its `NLCLIMB` (16 fns) and `NLMATH` (39 fns) are **100% dead** — 1,106
lines, imported by 5 notebooks each, zero calls. But CLOSAR's `NLMATH` holds 9 functions
DrosoClimb's lacks (`maxheight`, `timespentabovemeanline`, `displacementbetweenpauses`,
`boutdisplacement`, `countval`, `behavior`, `boutanalysis`, `pausecomp`, `timetype`,
`deltaversion_deltag`, `deltaversion_meandiff`, `log2speedratio`, `singledelta`). Check those
before touching CLOSAR's copies.

### 1f-3. `Dev` pruned (2026-08-16) — what AsOPN3 needs when `Dev` lands on it

`Dev` is the authoritative version. Removed 19 functions that were dead on **both** branches:
all 8 of `NLGRAPHS` (file deleted, plus its 3 dead imports in `Dabest_six_plots`,
`Generic line plots`, `wt comparisons`), and `NLMATH`'s `avgmean`, `boutheight`, `disptravel`,
`frames`, `maxheight`, `positional_arguments`, `speedcalc`, `timespentabovemeanline`,
`timetoreach`, `timetype`, `totaldisp`. Verified none was live on AsOPN3 first. 1,125 lines gone.
`NLMATH` is now 31 functions, `NLCLIMB` 16.

**DECIDED 2026-08-16:**

- **`rastergraph` is gone for good.** Not on `Dev`, not being reinstated. AsOPN3's
  `Generic line plots - figures.ipynb` still calls it, and `Dev`'s own
  `Generic line plots.ipynb` has one unresolved `NLCLIMB.rastergraph` call left in place —
  **Nicole is handling those two notebooks separately** (only one cell in each is actually
  wanted). Left deliberately unfixed, not an oversight.
- **`timerule` is the canonical name**, not `fivesecondrule`. `Dev`'s 3 notebooks
  (`Dabest_six_plots`, `PCA`, `wt comparisons`) were updated to call `NLCLIMB.timerule`.
  AsOPN3 renamed it to `fivesecondrule` — **that rename must be reverted on AsOPN3**.
- **`separation` → `refine`** kept, matching AsOPN3. `NLCLIMB.separation` (the phase filter)
  and `NLCLIMB.refine` (the Dark/Full/Recovery splitter) now coexist in the merged module.

**STILL MUST FIX ON AsOPN3 when `Dev` lands:**

- AsOPN3's commit `0febd95` ("REmoved obsolete stuff") cut its `NLMATH` from 42 → 30 functions.
  8 of those removals are live on `Dev`: `bheight`, `disppersec`, `distpersec`, `falldbest`,
  `pauseheight`, `pausenumber`, `sectioneddispchunks`, `straightnessindexmeter`. `Dev`'s version
  must win that merge conflict.
- AsOPN3 still has separate `NLMATH.py` / `NLGRAPHS.py`; `Dev` has neither. AsOPN3's notebooks
  need the same `NLMATH.x` → `NLCLIMB.x` rewrite `Dev` just got.

### 1f-4. Does AsOPN3 actually need MBON code? — CHECK

Working assumption is no: DrosoClimb/AsOPN3 is climbing, not MBON. Scan found MBON only as a
leftover **column label** in `3. AsOPN3_Multi-file processing.ipynb` and
`4. AsOPN3_multistate_processing.ipynb` (`dftotal['MBON'] = n`, `dftotal.set_index("MBON")`) —
no MBON analysis, no `MBONlist.csv` use, no `generate_lobelocation`. So the assumption holds, but
the column name should probably be renamed to something climbing-appropriate. Confirm the
column isn't consumed downstream before renaming.

### 1f-5. `LinReg.ipynb` `matchinglobesets` — DEFERRED, has a latent bug

Nicole's call: leave it, the notebook works. Do not touch without a reason.

`matchinglobesets` + its helper `matchingdfs` are defined in `LinReg` cell 5 and used 3× in
cell 26. After cleanup they exist nowhere else in DrosoClimb — the 4 other notebooks that
carried byte-identical copies never called them and have been stripped.

**The latent bug:** `matchinglobesets` filters `lobelocation` to the MBONs common to both
frames, but assigns `Lobe_location` / `MBON number` / `Neurotransmitter` onto the *unfiltered*
`dfreg`. Because both are `reset_index(drop=True)`, pandas aligns by position. Demonstrated:

    dfreg A,B,C  +  lobelocation A,C,D   ->   B gets C's lobe, C gets NaN

It is only correct when both frames hold exactly the same MBON set — which they may always do
here, since `lobelocation` is built by `generate_lobelocation` from the MBON list. **Unverified:
checking needs the Dropbox data.** If the sets ever diverge, cell 26 builds `spare` from three
separate calls, so the ACR and Chrimson2 columns could be offset against each other.

The fix when wanted (what the other repos already moved to):

    dfreg.merge(lobelocation[["MBON","Lobe_location","MBON number","Neurotransmitter"]],
                on="MBON", how="left")

Identical output when the MBON sets match, correct when they don't, and it makes `matchingdfs`
redundant.

### 1f-6. TEST NEEDED — `vortex_map` annotation text colour (DrosoClimb only)

`_get_text_color` was rewritten on 2026-08-16 and **has not been tested against real data or
eyeballed on a real figure.** Nicole to verify before trusting any figure it produces.

It no longer recomputes the colour from `cmap`/`vmin`/`vmax`. It now reads seaborn's own
QuadMesh (`mesh = ax.collections[0]`) and asks `mesh.cmap(mesh.norm(value))` — the colour
actually painted, after seaborn has applied `center`, resolved `None`, and handled `robust`.
Signature changed to `_get_text_color(spirals, row_idx, col_idx, n, mesh)`.

Why it changed: the old linear normalisation only agreed with seaborn while limits were a fixed
symmetric ±1. Once `vmin`/`vmax` became data-driven (usually asymmetric), the estimate drifted —
measured 0.742 actual vs 0.865 predicted at `vmin=-0.5, vmax=2.0`, enough to flip the black/white
choice. It also crashed on an explicit `{'vmin': None}` and gave wrong colours under `robust=True`.

Passes a synthetic smoke test across: defaults, symmetric ±1, asymmetric −0.5…2, one-tone
`rocket` + `center=None`, explicit `vmin: None`, `robust=True`, user `xticklabels`. **Synthetic
only — no real bootstrap data, no visual check.**

Watch for: text unreadable against mid-tone cells (the 0.5 luminance cut-off is arbitrary);
behaviour if `ax.collections` is empty or reordered by a future seaborn; one-tone palettes where
every cell resolves to the same colour.

### 1f-7. `vortexmap.py` aligned to DABEST `whorlmap` — DrosoClimb ONLY, NOT YET TESTED

**Rule set 2026-08-16: all vortexmap edits happen in DrosoClimb first. OSAR and CLOSAR are for
comparison only — do not edit their `vortexmap.py` until Nicole says so.** Baseline for "correct"
is `Preloaded Vortexmap.ipynb`'s `fast_vortexmap`, which is byte-equivalent to DABEST's
`whorlmap` helpers; the goal is for `vortexmap.py` to reproduce those figures.

Seven differences were identified between `vortexmap.py`, `fast_vortexmap` and DABEST. All seven
are now closed in DrosoClimb's copy:

| # | Item | Resolution |
|---|---|---|
| 1 | Centre value | **Kept Hedges' g** (not DABEST's `mean(long_ranks)`) — deliberate: the bootstrap is pre-computed, so the observed effect size is the wanted number |
| 2 | `chop_tail` | default `0` in the function, notebooks pass `2.5` at the call site |
| 3 | `reverse_neg` | added as kwarg, default `True` (never varied from `True` at any call site in any repo) |
| 4 | `abs_rank` | added as kwarg, default `False` |
| 5 | Renderer | `imshow` → `sns.heatmap`, via a `heatmap_kwargs` dict merged with `setdefault`, matching DABEST's `merge_two_dicts` pattern. `vmin`/`vmax` now default to the data range, not ±1 |
| 6 | Tie-break | `len(bs)//2` → `len(bs)/2`, matching DABEST. Only differs at an exact 50/50 split, but there it inverts the whole spiral |
| 7 | Ordering | `sort_by` kwarg — `None` = sorted, `"mbon_number"` = MBON-number order, or pass an index list / label list |

Also added: `effect_col="Hedges_g"` kwarg on `load_bootstrap_data`, so the column can be switched
to `mean_diff` if the schema changes. `CI_low`/`CI_high`/`Bootstrap` are still hardcoded.

**NEXT STEP once the notebook below checks out: port all of this to OSAR and CLOSAR.** Their
`vortexmap.py` is still the pre-2026-08-16 version (`chop_tail=2.5` default, `imshow`, no
`sort_by`/`reverse_neg`/`abs_rank`/`effect_col`, `//2` tie-break). OSAR's `Vortex Maps.ipynb` and
CLOSAR's `Climbing-OSAR vortexmaps.ipynb` will both need call-site updates when that happens —
including OSAR's still-outstanding `METRICS` argument (see 1f).

### 1f-8. `Testing vortexmaps.ipynb` (DrosoClimb) — WRITTEN BUT NEVER RUN

Created 2026-08-16 to verify the rewritten `vortexmap.py` against real data. **Claude did not
execute it — the Dropbox rule forbids it, since `load_bootstrap_data` calls `os.listdir` on the
data folder.** Nicole to run.

9 cells: read one OSAR bootstrap CSV and print its schema → `load_bootstrap_data` →
`build_vortex_df` → `vortex_map` → repeat for the climbing folder → a one-tone check with
`heatmap_kwargs={"cmap": "rocket", "center": None}`.

Paths used, temporary and repo-local for now:
- OSAR: `...\Data Compilation\osar_compiled\Bootstrapped stats\` (from `MB011B x Chrimson2_bootstrap.csv`)
- Climbing: `...\Data Compilation\Climbing_New\Compilation with delta\2025deltagcollection\`

**Known unknowns — run cells 1-3 first, they read a single file and tell you the rest:**

- The OSAR path given was one file, but `load_bootstrap_data` takes a **folder** and enumerates
  it, so it will hydrate everything in `Bootstrapped stats\`, not just that one file.
- The climbing folder's schema is unverified. `load_bootstrap_data` needs filenames shaped
  `<MBON> x <responder>_bootstrap.csv` and columns `Light_Intensity`, `Metric`, `Bootstrap`,
  `Hedges_g`, `CI_low`, `CI_high`. A delta-g collection folder may well use none of those, in
  which case cells 6-8 come back empty or raise.
- `Light_Intensity == "Full"` is hardcoded in the filter.
- `lobelocation=None` is passed, so MBONs sort alphabetically. Use `sort_by="mbon_number"` with a
  real `lobelocation` for the intended order.

### 1g. `load_bootstrap_data` enumerates its data directory

`os.listdir(data_path)` — with `data_path` pointing into Dropbox, calling it hydrates the folder.
See the Dropbox rule above. Consider accepting an explicit file list instead.

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

## Done (2026-08-16/17) — vortexmap alignment, DrosoClimb only, uncommitted

- `vortexmap.py` (DrosoClimb) aligned to DABEST `whorlmap` across all 7 identified differences —
  see 1f-7 for the table and 1f-6 for the untested text-colour rewrite.
- `Testing vortexmaps.ipynb` written to verify it. **Never run** — see 1f-8.
- OSAR and CLOSAR `vortexmap.py` deliberately left on the older version; porting is the next step.

## Done (2026-08-16) — DrosoClimb `Dev`, all uncommitted

- Paths: `Falling_New` → `Climbing_New` (22 refs, 14 notebooks); font blocks → `["fonts"]`
  (7 notebooks); added `fonts/`, `NLProcessing.py`, `MBONlist.csv`, `vortexmap.py`.
- Modules: `NLGRAPHS.py` deleted (8/8 dead), `NLMATH.py` merged into `NLCLIMB.py` (47 functions)
  and deleted, 19 dead functions dropped, `NLMATH.separation` → `refine`,
  `fivesecondrule` → `timerule` in 3 notebooks.
- Machine paths standardised to `workcomp` / `laptop` / `officecomp` / `homecomp` →
  `specifiedpath` across 16 notebooks (was 11 aliases for 4 machines). Trailing backslashes left
  exactly as they were — they are intentional.
- `Countingfiles`: `filefalling` → `filedir`.
- `wrap_labels`: 6 inline copies removed, 3 notebooks repointed to `NLProcessing.wrap_labels`.
- `find_number`: 7 inline copies removed, 29 call sites repointed to `NLProcessing.find_number`.
  **Output changes** — now sorted and prefix-compressed (`MBON01, 03`) and NaN-safe.
- Imports: 175 unused + 44 duplicate lines removed across 19 notebooks. 14 multi-name lines kept
  where at least one name is still used.
- `matchingdfs` / `matchinglobesets` deleted from the 4 notebooks that never called them.

## Done (2026-08-15)

- DrosoClimb `Dev_AsOPN3`: font path repointed to local `fonts/` in the 6 notebooks the earlier
  pass missed (`AsOPN3_forestplot_vertical_streamlined`, `Brain stuff/Brain volume diff` — uses
  `../fonts`, `Dabest_three_plots_recovery_streamlined2`, `Generic line plots - figures`,
  `Propplotfecundity-new`, `Propplotfecundity`). Not committed yet.
- DrosoClimb `NLMATH.separation` renamed to `refine` (root + `Manuscript related/` copy), clearing
  the only blocking collision with `NLCLIMB.separation`. See 1f-2.
- `vortexmap.py` (all three repos, byte-identical): dropped unused `seaborn`/`defaultdict`, added
  the missing `os`, made `metrics` a required argument of `load_bootstrap_data`, and replaced
  `find_number` with the sorting/compressing version so it works without `NLProcessing`.
  **Breaks one OSAR call site — see 1f.**

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
