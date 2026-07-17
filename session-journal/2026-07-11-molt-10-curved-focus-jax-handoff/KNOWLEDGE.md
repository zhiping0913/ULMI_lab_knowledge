---
name: 2026-07-11-molt-10-curved-focus-jax-handoff
description: >-
  Session segment covering Curved_surface focus.tsv x-coordinate completion, Emax/x plots, kernel-update nudges, and handoff into corrected JAX-in-Cell/PyPIC3D a0=20 reproduction tests.
type: session-journal
molt_count: 10
created_at: 2026-07-11T20:47:00-04:00
---

# 2026-07-11 — molt 10 — Curved_surface focus plots and JAX/PIC handoff

Timestamp: 2026-07-11 20:47 EDT. TL;DR: Finished Zhiping's follow-up focus.tsv augmentation and summary plots, then received a corrected input-deck update that opens the next JAX-in-Cell/PyPIC3D test task. Cache-miss budget is near the 1,000,000 soft cap, so this journal prepares a deliberate molt before the heavy adroit-vis work.

## What the segment was about

The segment began after a proactive cache-efficiency molt. The active work was Zhiping's Curved_surface fine focus scan finalization and follow-up analysis. Later, a kernel-update nudge appeared twice. At the end of the segment, Zhiping sent a new JAX/PIC instruction correcting the earlier `laser_a0=3` caveat and authorizing adroit-vis tests from the corrected a0=20 1D input deck.

## Accomplishments

### Runtime refresh / nudge handling
- Loaded installed LingTai kernel update 0.16.2 via `system(action='refresh')` when running 0.16.1 and installed 0.16.2 differed.
- Inspected and dismissed a stale worker-recovery notification; artifact only concerned a system refresh-success turn, not a human instruction.
- On 2026-07-11 UTC, read `runtime-update-checks` for package nudge `running=0.16.2`, `installed=0.16.2`, `latest=0.16.3`, `source=pypi-json`.
- Asked Zhiping in Telegram `main:6420513923:197` whether to update through his normal LingTai runtime/TUI path; did not download/install/refresh.
- On 2026-07-12 UTC, the same daily nudge recurred; dismissed without re-pinging because the explicit confirmation request was already pending.

### Curved_surface focus.tsv x-coordinate update
- Zhiping asked in Telegram `main:6420513923:180` to add the x position corresponding to each maximum `|centerline|_max`.
- Determined `.out` files do not print x; `propagate_2D_time_scan.py` stores per-timestep `x_at_peak_per_t` and `peak_amp_per_t`, and writes `<side>_peak_vs_x.nc` with coordinate `x_at_peak = x_at_peak_per_t / lambda0` and variable `E_peak = peak_amp_per_t / Ec`.
- Wrote and ran reproducible script:
  - local: `/home/zhiping/ULMI_lab/.lingtai/main/artifacts/update_focus_with_x_at_emax_20260710.py`
  - remote: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/update_focus_with_x_at_emax_20260710.py`
- Updated `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/focus.tsv` with new column `x_at_E_max (lambda0)`.
- Backup: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/focus.tsv.bak_20260710_105035_before_x_at_Emax_update`.
- Provenance table: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_fine_20260708_2014_fine_focus_tiered/focus_x_at_Emax_summary_20260710_105035.tsv`.
- Validation: 32 rows; no missing x; x range `32.178..332.342 lambda0`; max discrepancy between rounded `E_max(Ec)` and NetCDF max `E_peak` was `4.51e-4 Ec`.
- Reported to Zhiping in Telegram `main:6420513923:182`.

### Curved_surface summary plots
- Zhiping asked in Telegram `main:6420513923:183` / `:185` to plot `side=reflection`, x-axis `K`, y-axis `E_max`, with three fixed `ND/a0=0.30,0.50,1.00` curves on one figure.
- Generated plot and reproducible notebook/script:
  - PNG: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_summary_plots/Emax_vs_K_reflection_ND_scan_a0_20.png`
  - PDF: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_summary_plots/Emax_vs_K_reflection_ND_scan_a0_20.pdf`
  - CSV: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_summary_plots/Emax_vs_K_reflection_ND_scan_a0_20_long.csv`
  - Notebook: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/plot_Emax_vs_K_reflection_ND_scan_a0_20.ipynb`
  - Script: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/plot_Emax_vs_K_reflection_ND_scan_a0_20.py`
- Attached PNG/PDF/notebook to Telegram `main:6420513923:188-190`.
- Zhiping then asked in Telegram `main:6420513923:191` to use the same notebook and add `x_at_E_max` vs `K`, same three curves.
- Updated the same notebook/script to contain both plots and generated:
  - PNG: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_summary_plots/x_at_Emax_vs_K_reflection_ND_scan_a0_20.png`
  - PDF: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_summary_plots/x_at_Emax_vs_K_reflection_ND_scan_a0_20.pdf`
- Attached new PNG/PDF plus updated notebook to Telegram `main:6420513923:194-196`.
- Visual validation was performed on both figures.

### New JAX/PIC handoff
- Zhiping wrote in Telegram `main:6420513923:198`:
  - The earlier `laser_a0=3` in `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/1D/45/ND_a0=0.30/input.deck` was an accidental overwrite by an a0=3 scan input.
  - He has corrected all `input.deck` files under `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/1D/45`.
  - The existing simulation outputs are still correct a0=20 outputs and were not affected.
  - The corrected `ND_a0=0.30/input.deck` can now be used as the initial-condition reference for adroit-vis tests.
  - He already tested initial field insertion and Yee-grid offsets in JAX-in-Cell and PyPIC3D; see `/scratch/network/zl8336/try_PIC_GPU_JAX/pypic3d_Bz_offset_sweep.ipynb`.
- I acknowledged in Telegram `main:6420513923:199`, promising a minimal no-Slurm adroit-vis test workflow.
- Durable JAX note updated: `knowledge/jax-pic-survey/epoch1d_reproduction_route_2026-07-09.md` now includes the correction and new active test direction.
- Pad updated with this as the new active task.

## Decisions and reasoning

- For focus x-coordinate, used `*_peak_vs_x.nc` rather than rerunning scans, because `propagate_2D_time_scan.py` already stores the exact per-timestep centerline peak x coordinate and E peak. The `.out` logs were insufficient because they lacked x.
- For plots, used `focus.tsv` as the single source of truth after adding `E_max(Ec)` and `x_at_E_max (lambda0)`, and stored plot source under the Curved_surface convention `/utility/plot/`.
- Did not update LingTai package to 0.16.3 because the nudge source was `pypi-json`; runtime rules require explicit human confirmation before package update side effects.
- Decided to molt before the JAX/PIC work because cache-miss budget is near exhaustion; the next task will involve remote code inspection and possibly GPU tests, so a fresh context is more efficient and safer.

## Artifacts and paths

Key remote project paths:
- Focus TSV: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/focus.tsv`
- Focus x provenance: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_fine_20260708_2014_fine_focus_tiered/focus_x_at_Emax_summary_20260710_105035.tsv`
- Plot output dir: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_summary_plots/`
- Plot source dir: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/`
- JAX/PyPIC workdir: `/scratch/network/zl8336/try_PIC_GPU_JAX/`
- Corrected EPOCH reference deck: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/1D/45/ND_a0=0.30/input.deck`
- Zhiping offset notebook: `/scratch/network/zl8336/try_PIC_GPU_JAX/pypic3d_Bz_offset_sweep.ipynb`

Local artifacts:
- `/home/zhiping/ULMI_lab/.lingtai/main/artifacts/update_focus_with_x_at_emax_20260710.py`
- `/home/zhiping/ULMI_lab/.lingtai/main/artifacts/curved_surface_focus_plots_20260710/`
- `/home/zhiping/ULMI_lab/.lingtai/main/artifacts/plot_emax_vs_k_reflection_nd_scan_20260710.py`

Durable notes updated:
- `knowledge/curved-surface-chf-project/fine_time_scan_tiered_resubmission_2026-07-08.md`
- `knowledge/jax-pic-survey/epoch1d_reproduction_route_2026-07-09.md`
- `system/pad.md`

## Open tasks

1. Continue from Telegram `main:6420513923:198` / ack `:199`:
   - SSH to `adroit-vis` with `BatchMode=yes` and short timeout. If password/Duo prompt or auth failure, stop and report.
   - Inspect corrected tiger-vis deck and copy/parse relevant parameters; do not assume old `laser_a0=3` caveat still holds.
   - Inspect `/scratch/network/zl8336/try_PIC_GPU_JAX/pypic3d_Bz_offset_sweep.ipynb` and existing scripts/reports in `/scratch/network/zl8336/try_PIC_GPU_JAX`.
   - Run a minimal no-Slurm sanity benchmark in that workdir: shared initializer, vacuum pulse propagation, then short collisionless slab test only if field offsets are sound.
   - Preserve outputs and report progress/blockers to Zhiping via Telegram.
2. If Zhiping replies explicitly to the LingTai 0.16.3 update question (`main:6420513923:197`), follow runtime-update protocol. Without explicit confirmation, do not update/download/install/refresh.

## Collaborators

- Zhiping via Telegram chat `6420513923`; reply on Telegram.
- No persistent avatars were created. No active daemons/background jobs are running.

## Gotchas and lessons

- The previous JAX/PIC note said the target deck had `laser_a0=3`; this is now superseded by Zhiping's correction. Preserve the history, but use the corrected deck for new tests.
- In Bourdier/moving-frame mapping, when APIs expect velocity, use `v_y=-c sin(theta)` rather than raw EPOCH momentum-like `drift_y=-m_s c tan(theta)`.
- JAX-in-Cell and PyPIC3D do not currently include Nanbu collisions; first benchmark should be collisionless or compared only to collision-off EPOCH if such a reference is authorized.
- Do not submit Slurm jobs unless Zhiping explicitly asks and `checkquota` passes. The immediate adroit-vis task is interactive/no-Slurm.
- For generated figure requests, deliver both rendered plots and reproducible notebook; this was followed.
