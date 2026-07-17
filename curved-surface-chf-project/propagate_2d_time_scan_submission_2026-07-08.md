---
name: propagate_2d_time_scan_submission_2026-07-08
description: Submission record for a0=20/2D renamed K,ND_a0,L reflection/transmission fast propagation time-scan jobs.
type: project-note
---

# a0=20/2D propagate_2D_time_scan submission — 2026-07-08

Triggered by Telegram `main:6420513923:138` from Zhiping:

> Use `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan.slurm` calling `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan.py` to run fast propagation / focus-position scan for every `K=...,ND_a0=...,L=...` directory under `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D`, both `reflection` and `transmission`, total 32 jobs.

## Pre-submission checks

- Loaded Princeton HPC guidance before Slurm submission.
- `checkquota` was run immediately before submission and saved in the log directory.
- Quota result: MIKHAILOVA scratch `/scratch/gpfs/MIKHAILOVA` used by all = `9.0 TiB` of `15 TiB`, so free ≈ `6 TiB`, above the Tiger scratch 2 TiB submission threshold.
- Found 16 renamed case directories matching `K=...,ND_a0=...,L=...`.
- Confirmed all 16 directories contain nonempty `reflection_clip.nc` and `transmission_clip.nc`; total inputs = 32.
- Explicit still-running excluded directory from prior rename instruction remains untouched: `K=-0.002,D=1.95,L=0.05`.

## Scripts used

Requested Slurm wrapper:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan.slurm
```

Requested Python script:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan.py
```

`propagate_2D_time_scan.py` reads environment variables:

```text
working_dir=<case directory>
side=reflection|transmission
```

Important Slurm invocation detail: because case directory names contain commas, the submission script did **not** pass `working_dir` inside `sbatch --export=...` (comma-separated parsing would corrupt the value). Instead it set `working_dir` and `side` in the environment of each `sbatch` process and relied on the script/default `--export=ALL` behavior.

## Submission artifact

Local helper used to submit:

```text
artifacts/curved_surface_2d_a020_inspection_20260706_204001/submit_propagate_2d_time_scan_a0_20.py
```

Remote log directory:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_20260708_152818/
```

Manifest:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_20260708_152818/manifest.tsv
```

Quota evidence:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_20260708_152818/checkquota.txt
```

## Submitted jobs

Submitted job IDs: `3438490–3438521` (32 jobs).

Cases × sides:

```text
K=+0.000,ND_a0=0.15,L=0.00  reflection/transmission  3438490/3438491
K=+0.000,ND_a0=0.30,L=0.00  reflection/transmission  3438492/3438493
K=+0.000,ND_a0=0.50,L=0.00  reflection/transmission  3438494/3438495
K=+0.000,ND_a0=1.00,L=0.00  reflection/transmission  3438496/3438497
K=-0.002,ND_a0=0.10,L=0.00  reflection/transmission  3438498/3438499
K=-0.002,ND_a0=0.30,L=0.00  reflection/transmission  3438500/3438501
K=-0.002,ND_a0=0.50,L=0.00  reflection/transmission  3438502/3438503
K=-0.002,ND_a0=1.00,L=0.00  reflection/transmission  3438504/3438505
K=-0.005,ND_a0=0.10,L=0.00  reflection/transmission  3438506/3438507
K=-0.005,ND_a0=0.30,L=0.00  reflection/transmission  3438508/3438509
K=-0.005,ND_a0=0.50,L=0.00  reflection/transmission  3438510/3438511
K=-0.005,ND_a0=1.00,L=0.00  reflection/transmission  3438512/3438513
K=-0.008,ND_a0=0.10,L=0.00  reflection/transmission  3438514/3438515
K=-0.008,ND_a0=0.30,L=0.00  reflection/transmission  3438516/3438517
K=-0.008,ND_a0=0.50,L=0.00  reflection/transmission  3438518/3438519
K=-0.008,ND_a0=1.00,L=0.00  reflection/transmission  3438520/3438521
```

Initial `squeue` after submission:

- 3 jobs RUNNING: `3438490`, `3438491`, `3438492`.
- 29 jobs PENDING with reason `(Priority)`.

## Expected outputs per case/side

Each job writes into its case directory:

```text
<side>_beam_width_strength.h5
<side>_beam_width_strength.png
<side>_Ey_t=...T0.png
```

The HDF stores time scan arrays including width/strength vs fixed absolute x grid, per-timestep on-axis peak amplitude, and `x_at_peak` diagnostics. The summary PNG contains transverse 1/e half-width and on-axis peak vs propagation position.

## Telegram report

Reported to Zhiping in Telegram `main:6420513923:140`.

## OOM rerun with higher memory

Zhiping noticed a few jobs were OOM killed and asked in Telegram `main:6420513923:141` to increase memory and rerun only those cases.

`sacct` status after the first wave showed 8 top-level jobs in `OUT_OF_MEMORY (0:125)` with `MaxRSS` at about `629,143,xxx K`, i.e. right at the original 600G request. Most other jobs completed; one original job was still running at the time of rerun.

OOM jobs:

```text
3438490  K=+0.000,ND_a0=0.15,L=0.00  reflection
3438492  K=+0.000,ND_a0=0.30,L=0.00  reflection
3438493  K=+0.000,ND_a0=0.30,L=0.00  transmission
3438494  K=+0.000,ND_a0=0.50,L=0.00  reflection
3438496  K=+0.000,ND_a0=1.00,L=0.00  reflection
3438498  K=-0.002,ND_a0=0.10,L=0.00 reflection
3438511  K=-0.005,ND_a0=0.50,L=0.00 transmission
3438513  K=-0.005,ND_a0=1.00,L=0.00 transmission
```

Pre-rerun `checkquota` again passed: MIKHAILOVA scratch used by all = `9.0 TiB` of `15 TiB`.

Rerun used the same Slurm/Python but command-line `--mem=900G` to override the original `#SBATCH --mem=600G` request.

Rerun logdir:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_20260708_152818/rerun_900G_20260708_161740/
```

Rerun manifest:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_20260708_152818/rerun_900G_20260708_161740/manifest.tsv
```

Rerun job IDs:

```text
3439194,3439195,3439196,3439197,3439198,3439199,3439200,3439201
```

Initial rerun `squeue`: all 8 pending with reason `(None)`; original job `3438507` still running. Reported to Zhiping in Telegram `main:6420513923:143`.

## Final validation

Triggered by delayed self-reminders. First validation at ~18:02 EDT found 31/32 final jobs complete and only rerun `3439197` still running; a short reminder was scheduled.

Final validation at ~18:43 EDT:

```text
FINAL_32_COUNTS {'COMPLETED': 32}
OUTPUTS_PRESENT 64 OF 64 MISSING 0
```

All 8 reruns completed successfully:

```text
3439194  COMPLETED  00:56:23
3439195  COMPLETED  00:52:36
3439196  COMPLETED  00:42:36
3439197  COMPLETED  01:45:07  # last; K=+0.000,ND_a0=0.50,L=0.00 reflection
3439198  COMPLETED  00:53:43
3439199  COMPLETED  01:03:07
3439200  COMPLETED  00:45:07
3439201  COMPLETED  00:44:46
```

Final output check: all 32 case/side combinations have nonempty:

```text
<side>_beam_width_strength.h5
<side>_beam_width_strength.png
```

Error-log caveat:

- Original logdir has 8 OOM `.err` records from the first 600G wave and 24 warning-only `.err` logs.
- Rerun logdir has 8 nonempty `.err` logs, all warning-only; no OOM in reruns.
- The common warning is `RuntimeWarning: All-NaN slice encountered` from `beam_widths_min = np.nanmin(widths_2d, axis=0)` and sometimes `beam_strength_max`. This is not a job failure but matters for interpretation: some x regions have no valid above-threshold transverse-width samples, so aggregated width/strength arrays may contain NaN regions.

Reported final success and caveat to Zhiping in Telegram `main:6420513923:144`.
