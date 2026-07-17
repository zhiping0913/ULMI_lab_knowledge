# Fine propagation-time scan submission — 2026-07-08

Trigger: Telegram `main:6420513923:145`. Zhiping said the previous `propagate_2D_time_scan` run was a coarse focus locator and provided approximate focus centers in `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/focus.tsv`, column `propagation_time (laser_period)`, relative to each `reflection_clip`/`transmission_clip`.

Important interpretation: Zhiping mentioned `utility/plot_reflection_1D_scan.py`, but inspection showed that file is only a 1D summary plotting collector. The actual script controlling `times = np.linspace(...)` is `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan.py`, so the time-scan patch was made there.

Remote changes:
- Backup: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan.py.bak_fine_time_env_20260708_195159`
- Patched script: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan.py`
  - If no `propagation_time` env var is provided, preserves legacy quick scan `[-200, 200] T0` with 21 points.
  - If `propagation_time` is provided, defaults to `center ± 50 T0`, `101` points.
  - Supported env vars: `propagation_time`/`PROPAGATION_TIME`/`propagation_time_center`/`PROPAGATION_TIME_CENTER`, `time_scan_half_width_T0` (default 50), `time_scan_num_points` (default 101).
  - HDF attrs now record `time_scan_mode`, `propagation_time_center_over_T0`, `time_scan_half_width_over_T0`, `time_scan_num_points`, and scan min/max.
- Submission driver: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/submit_fine_time_scan_from_focus.sh`
  - Reads all rows of `focus.tsv`, including final row without relying on trailing newline.
  - Verifies `<side>_clip.nc` exists and is nonempty.
  - Uses env-var export rather than `--export=...` because `working_dir` contains commas.
  - Default `MEM=900G`, `TIME_LIMIT=12:00:00`, `HALF_WIDTH_T0=50`, `NUM_POINTS=101`, dry-run unless `SUBMIT=1`.

Validation before submission:
- Full dry-run succeeded: 32/32 rows, failed=0.
- `checkquota` before `sbatch`: Tiger MIKHAILOVA scratch `8.7/15 TiB`, so ~6.3 TiB free > 2 TiB threshold.

Submitted batch:
- Command used: `STAMP=20260708_1953_fine_focus SUBMIT=1 MEM=900G TIME_LIMIT=12:00:00 HALF_WIDTH_T0=50 NUM_POINTS=101 submit_fine_time_scan_from_focus.sh`
- Job IDs: `3441224–3441255` (32 jobs).
- Immediate `squeue`: all 32 `PENDING (Priority)`.
- Logdir: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_fine_20260708_1953_fine_focus/`
- Manifest: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_fine_20260708_1953_fine_focus/manifest.tsv`

Reported to Zhiping in Telegram `main:6420513923:147`.

Open follow-up:
- Check `squeue`/`sacct` later for jobs `3441224–3441255`.
- Validate outputs per row: `<side>_beam_width_strength.h5`, `<side>_beam_width_strength.png`, `<side>_peak_vs_x.nc`; inspect `.err` for OOM or warnings.
- A delayed self-reminder was scheduled for ~2 hours after submission.
