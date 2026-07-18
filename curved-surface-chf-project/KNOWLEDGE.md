---
name: curved-surface-chf-project
description: >
  Project memory for Zhiping's Curved_surface ultrathin-foil coherent-harmonic-focusing work:
  goals, factorization, run/workflow registries, daily logs, figures, and open questions.
version: 0.1.0
last_updated: 2026-07-17
---

# Curved_surface / ultrathin-foil CHF project memory

This entry is the durable project ledger for Zhiping's ongoing Curved_surface work on field enhancement via harmonic focusing from laser-driven ultrathin plasma foils. Use it whenever the conversation resumes this project, especially when Zhiping reports simulations, post-processing, workflow details, or daily progress.

## How to use this entry

Keep this parent file as the compact project index and current-state summary. Put detailed daily work, simulation inventories, and workflow notes in the supporting files listed below.

Supporting files:

- `daily_log/2026-07-06.md` — first project log entry covering the 1D scan / all-a0 plotting / abstract discussion segment.
- `daily_log/2026-07-07.md` — propagate_and_clip validation plus direct-bash Spectrum_2D batch completion.
- `daily_log/2026-07-08.md` — 1D-vs-2D reflection spectrum comparison plot.
- `run_registry.md` — inventory of simulation/summary datasets, parameters, status, diagnostics, and caveats.
- `workflow_registry.md` — scripts, responsibilities, inputs/outputs, and gotchas.
- `figure_claim_map.md` — which figures support which scientific claims and what evidence is still missing.
- `propagate_clip_validation_2026-07-07.md` — clean validation record for the 31 propagate_and_clip jobs.
- `spectrum_2d_batch_2026-07-07.md` — direct tiger-vis Spectrum_2D batch over 32 clipped fields and validation.
- `spectrum_compare_plot_2026-07-08.md` — comparison plots of 1D reflection spectra with 2D reflection radial spectra across curvature `K`, first for `ND/a0=0.30` and then generalized to `ND/a0=0.10,0.30,0.50,1.00` with `D` computed from `a0,N`.
- `propagate_2d_time_scan_submission_2026-07-08.md` — submission record for 32 Slurm fast-propagation/focus-scan jobs over renamed `a0=20/2D/K=...,ND_a0=...,L=...` cases, both reflection and transmission.
- `order_focus_scan_2026-07-13.md` — order-resolved coarse focus scan (`center±100T0`, `10T0` step, 21 points) using updated `propagate_2D_time_scan.py` k-band outputs; includes script patch/driver, job IDs `3498134–3498166`, validation, argmax summary TSV path, and reflection/transmission strongest per band.
- `order_focus_plots_2026-07-13.md` — follow-up order-resolved reflection plots of `E_max/Ec` and `x_Emax/λ0` versus curvature `K` for `ND/a0=0.30,0.50,1.00` and bands `band1`, `band2`, `band3`; includes output PNG/PDF/notebook/CSV paths and Telegram attachment IDs.
- `order_focus_paraxial_overlay_2026-07-17.md` — added a paraxial Gaussian 1st-order overlay to the `a0=20` order-focus `x_Emax(K)` plot using `K_eff=K/cos45°` and reflected wavefront radius `R_i=1/(2K_eff)` for `w_i=12.5λ0`; records modified notebook/script, output paths, and formula caveats.
- `a0_20_focus_at_time_fields_2026-07-17.md` — focus-at-time field generation for all 32 rows of `a0=20/2D/focus.tsv` using `propagate_2D_at_time.py`; records original/retry job IDs, validation (`31/32` readable focus fields), and unresolved row1 900G OOM requiring algorithmic memory reduction rather than more memory.
- `tradeoff_objective_2026-07-13.md` — Zhiping's clarified design target: do not maximize near-target field alone; select curvature from a Pareto tradeoff between strong `E_peak`, focus distance/accessibility, and width/divergence. Includes quick candidates (`K=-0.005` for strong-but-not-too-close, `K=-0.002` for farther focus) from the current order-focus summary.
- `temporal_compression_estimate_2026-07-13.md` — 1D `reflection_Sx.nc` temporal-compression estimates: ultrathin `a0=20/1D/45/ND_a0=0.30` has `Sx_peak/(a0²Sc_M)≈7.79`, FWHM≈`0.013 T0_M`, `Sx_peak×FWHM`≈`0.102`, and actual `x_peak±0.5λ0_M` integral≈`0.612` of one-cycle incident fluence; thick `45,thick/L=0.10` has lower peak≈`3.43`, broader FWHM≈`0.163 T0_M`, and one-cycle integral≈`0.942`; records script/output paths and the `a0=20` vs `spectrum_1D.py` hardcoded `a0=200` caveat.
- `2d_band1_spatial_fluence_estimate_2026-07-13.md` — 2D `K=-0.005,ND_a0=0.30,L=0.00` order-focus `band1` focal estimate: at the `E_peak` row (`E≈34.68 Ec`, `x≈68.32λ0`, `width_x≈2.62λ0`, `width_y≈3.65λ0`), rectangular `E² width_x width_y / [a0² × 1λ0 × 2w0]≈1.15`; records Zhiping's correction that `width_y` is full width while `laser_w0=12.5λ0` is half-width.
- `k0_reflection_wide_order_scan_2026-07-14.md` — active Slurm rescan for four `K=+0.000` reflection cases because previous `±100T0` order scan clipped band1/band3 foci; submitted jobs `3512584–3512587` with `±300T0`, `61` points, `900G/12h`, output tag `order_focus_pm300_10T_K0_reflection`, logdir `utility/propagate_2D_time_scan_logs/a0_20_2D_order_focus_pm300_10T_K0_reflection_20260714_1944/`.
- `templates/update-template.md` — message-to-memory template for future updates from Zhiping.

Related knowledge:

- `../curved-surface-reflection-scan-plots/KNOWLEDGE.md` — reproducible plotting workflow for the current 1D L and ND/a0 reflection summary plots.
- `../zhiping-pic-tool-habits/KNOWLEDGE.md` — Zhiping's PIC/EPOCH/EM_analyzer coding and workflow conventions.

Relevant skills to load when needed:

- `wiki-laser-plasma-hhg` for ROM/CSE/CWE/CHF/plasma-mirror HHG physics and references.
- `research-software` for EPOCH/PIC simulation workflows.
- `knowledge-base-user` / `knowledge-base-lightrag` for local paper/book evidence retrieval.
- `princeton-hpc` before any Princeton cluster interaction. Always run `checkquota` before `sbatch`/`salloc`; stop on password/Duo prompts.

## Project goal and scientific storyline

Goal: explain and quantify how a curved ultrathin plasma foil can convert efficient local harmonic/reflected-field generation into a much larger reflected focal peak, especially for moderate-to-high but not necessarily maximal `a0`.

Working storyline:

1. Ultrathin foil harmonic/reflection efficiency is controlled strongly by areal-density matching / relativistic-transparency-window parameters such as `ND/a0`, not by `a0` alone.
2. 1D simulations without transverse focusing measure a local reflected peak amplification normalized to the incident intensity scale, roughly `Sx_peak/(a0^2 S_c,M)`.
3. Current 1D scan evidence suggests this normalized local gain can reach `O(10)` and is not strongly monotonic in `a0` after optimization. This is interpreted as a CSE/nanobunch short-time-window radiation gain, not simply ordinary long-pulse compression.
4. Curved foil / coherent harmonic focusing should add transverse spatial focusing and a wavefront/coherence quality factor. Therefore the full focal enhancement should be separated into three factors rather than quoted as one opaque number.

Core diagnostic factorization:

```text
S_focus^peak / (a0^2 S_c,M) = G_CSE/1D × G_spatial × C_wavefront
```

Definitions to preserve:

| Factor | Physical meaning | How to measure / evidence status |
|---|---|---|
| `G_CSE/1D` | Local short-time-window CSE/nanobunch reflected peak gain in flat/1D geometry. | From 1D scan near-field summaries, e.g. `Sx_peak/(a0^2 S_c,M)` vs `L` or `ND/a0`. Current evidence: O(10), weakly dependent on optimized `a0`. |
| `G_spatial` | Curved foil / CHF transverse focusing gain: concentration of reflected/harmonic energy from source aperture into focal spot. | Needs 2D/3D near-field plus propagation or direct focal diagnostics. Approximate scaling like `A_source/A_focus`, but harmonic order, divergence, and phase locking matter. |
| `C_wavefront` | Strehl-like coherence/wavefront-quality penalty relative to ideal coherent focusing with the same spectrum/energy. | Needs actual-vs-ideal focus comparison. Encodes aberrations, harmonic-dependent divergence, phase noise, foil deformation/breakup. |

## Current status snapshot — 2026-07-06

Completed / recorded:

- `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot_reflection_1D_scan.py` was updated to use `EM_analyzer.read_write.read_nc` for NetCDF summary reads.
- Default L scan over `a0=*/1D/45,thick/reflection.nc` now includes `a0=200` and writes under `utility/reflection_1D_scan_plots/`.
- `--scan-key ND_a0` compares existing all-a0 summaries under `a0=*/1D/45/reflection.nc`, writing under `utility/reflection_ND_a0_scan_plots/`.
- Corrected all-a0 ND/a0 artifacts were reported to Zhiping in Telegram `main:6420513923:60-63`:
  - `reflection_efficiency_high_3p5_1000_vs_ND_a0_all_a0.png`
  - `reflection_Sx_peak_over_incident_vs_ND_a0_all_a0.png`
  - `reflection_1D_ND_a0_scan_summary.csv`
- Important role boundary: `efficiency_1D_scan.py` owns generating/processing scan summaries; `plot_reflection_1D_scan.py` combines existing summaries and plots them.


Scientific state:

- 1D scan evidence supports `G_CSE/1D ~ O(10)` with weak optimized-a0 dependence.
- This supports the proportional/normalized statement: medium `a0` ultrathin foils can be comparable to very high `a0` in normalized reflected-peak gain, while absolute intensity still carries the `a0^2` base.
- Next major analysis direction is to separate and display `G_CSE/1D`, `G_spatial`, and `C_wavefront` using 2D/3D curved-foil data and/or propagation diagnostics.
- New 2D a0=20 inventory was inspected on tiger-vis under `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D`; see `inspection_2026-07-06_a0-20-2d.md`, `run_registry.md`, and `workflow_registry.md`. There are 17 K,D,L input-deck cases copied locally in `artifacts/curved_surface_2d_a020_inspection_20260706_204001/`; `K=-0.002,D=1.95,L=0.05` is the running thick-reference-like case, while the finished thin cases have reflection/transmission `.nc` outputs from `rotate_and_shift.py`.
- `propagate_and_clip.csv` / `propagate_and_clip.py` were inspected after Zhiping's explanation; see `propagate_clip_plan_2026-07-06.md`. CSV has 32 reflection/transmission rows; only `K=-0.002,D=0.01,L=0.00` transmission is marked done. Zhiping corrected that the Python script should **not** be linked to the CSV; instead, keep it generic and pass case parameters through environment variables exported by per-row Slurm scripts generated from an outer bash driver.
- Env-var driver dry run has been implemented on tiger-vis; see `propagate_clip_dryrun_2026-07-06.md`. Remote backup: `utility/propagate_and_clip.py.bak_env_20260706_214530`; modified env-var script: `utility/propagate_and_clip.py`; driver: `utility/generate_propagate_and_clip_slurms.sh`; generated 31 Slurm files under `utility/propagate_and_clip_slurms/a0_20_2D/`. Validation passed (`python` syntax, `bash -n` for 31 Slurms, would-sbatch count 31).
- After Zhiping authorized submission in Telegram `main:6420513923:94`, main re-ran `checkquota` (MIKHAILOVA used 8.7/15 TiB, ~6.3 TiB free) and submitted the 31 jobs with `SUBMIT=1 DRY_RUN=0`. Job IDs: `3382363–3382393`; immediate `squeue` state: all `PENDING`, reason `(None)`. Detailed mapping is in `propagate_clip_submission_2026-07-06.md`.
- Zhiping later reported the 31 jobs completed (`main:6420513923:99`). Validation is clean; see `propagate_clip_validation_2026-07-07.md`: `sacct` COMPLETED=31, bad_count=0; 31/31 expected `<side>_clip.nc/.png` output pairs present; no nonempty `.err`; no suspicious output logs by simple heuristic.
- Zhiping then requested direct-bash spectrum extraction from the clipped fields (`main:6420513923:104`, clarified no Slurm in `:106`). `Spectrum_2D.py` was run on tiger-vis for 32 clip inputs (16 reflection + 16 transmission), using env vars `working_dir` and `side`; validation found 32/32 OK and all 160 expected spectrum output files non-empty. See `spectrum_2d_batch_2026-07-07.md`.
- Zhiping requested a comparison plot (`main:6420513923:111`) of the 1D `a0=20/1D/45/ND_a0=0.30/reflection_spectrum.nc` curve with 2D `a0=20/2D/K=...,D=0.02,L=0.00/reflection_spectrum_kr.nc` curves. Plot outputs are under `utility/spectrum_compare_plots/`, attached in Telegram `main:6420513923:113`, and documented in `spectrum_compare_plot_2026-07-08.md`.
- Zhiping then clarified (`main:6420513923:124`) that the spectrum comparison should include `ND/a0=0.10,0.30,0.50,1.00`, with `D` computed from `D=ND_a0*a0/N` and directory formatted as `%3.2f`, not memorized. The canonical plot script now does this for `a0=20,N=350`, generated a 2×2 panel figure (`panel_count=4`, `curve_count=20`), and sent report/PNG/notebook in Telegram `main:6420513923:126-128`. New outputs are `utility/spectrum_compare_plots/reflection_spectrum_1D_vs_2D_ND_scan_a0_20.*`; source notebook is `utility/plot/reflection_spectrum_1D_vs_2D_ND_scan_a0_20.ipynb`.
- Zhiping then requested (`main:6420513923:129`) a more flexible notebook that accepts arbitrary result-directory lists and plots spectra together. Created `utility/plot/plot_reflection_spectrum_from_dir_list.ipynb` and `.py`; it originally parsed path plus `input.deck` metadata, finds `reflection_spectrum.nc` / `reflection_spectrum_kr.nc` / `reflection.nc`, and uses `plot_multiple_1D_fields`. Default example produced `utility/spectrum_compare_plots/reflection_spectrum_dirlist_example.*`; report/notebook/example PNG delivered in Telegram `main:6420513923:131-133`. After Zhiping clarified stale `input.deck` constants in `main:6420513923:134`, the notebook/script were revised to parse metadata only from directory names.
- Zhiping requested (`main:6420513923:134`) renaming `a0=20/2D` top-level case directories from `K=...,D=...,L=...` to `K=...,ND_a0=...,L=...`, excluding still-running `K=-0.002,D=1.95,L=0.05`. Completed 16 top-level directory renames with manifest `utility/plot/rename_2d_D_to_ND_a0_20260708_112525.tsv`; note that 32 in his wording refers to reflection+transmission sides.
- Zhiping requested (`main:6420513923:138`) fast propagation/focus scan over the renamed 16 directories × `reflection/transmission`. Submitted 32 Slurm jobs (`3438490–3438521`) after `checkquota` and input validation; log/manifest directory `utility/propagate_2D_time_scan_logs/a0_20_2D_20260708_152818/`; detailed record `propagate_2d_time_scan_submission_2026-07-08.md`. Initial queue: 3 running, 29 pending `(Priority)`. Zhiping then noted OOMs (`main:6420513923:141`); `sacct` found 8 OOM jobs (`3438490,3438492,3438493,3438494,3438496,3438498,3438511,3438513`), which were resubmitted with `--mem=900G` as jobs `3439194–3439201` under `rerun_900G_20260708_161740/`. Final validation found 32/32 final jobs completed and 64/64 final h5/png outputs nonempty; warning-only `.err` logs contain `All-NaN slice encountered` caveats for some beam-width/strength aggregates. Final report sent in Telegram `main:6420513923:144`.

## Memory update protocol for future messages

When Zhiping reports simulations, calculations, workflow, or daily progress:

1. Extract raw facts first: paths, dimensions, parameters, scripts, outputs, status, dates.
2. Update `run_registry.md` for simulation/dataset inventory.
3. Update `workflow_registry.md` if a script, diagnostic, or processing convention is new or changed.
4. Add a dated entry under `daily_log/YYYY-MM-DD.md` with completed work, observations, interpretation, caveats, and next steps.
5. Update `figure_claim_map.md` when a plot supports a claim or exposes a missing control.
6. Keep physics interpretation separate from raw fact. Mark uncertainties explicitly.
7. Reply to Zhiping with a short confirmation of what was recorded and what next action is implied.

Never store credential material. Do not treat simulation output as sufficient without physical interpretation and uncertainty discussion.

## a0=50/2D ND_a0=0.30 clip + focus scan (2026-07-17)

See supporting note `a0_50_clip_focus_scan_2026-07-17.md` for the completed clip jobs/retries, row6 one-node-two-task fix, focus-scan failure/retry recovery, and final combined summary artifacts/metrics for all six K values (center 50T0, half-width 100T0, step 5T0).

See supporting note `a0_50_reflection_fine_scan_2026-07-18.md` for the follow-up reflection-only fine scan using each K's coarse on-axis peak as center (`±20T0`, `step=1T0`, 41 points), including job/retry record, validation (`6/6` output dirs OK), summary TSV/PNG/PDF/notebook paths, and the trend that stronger negative curvature focuses closer and higher.
