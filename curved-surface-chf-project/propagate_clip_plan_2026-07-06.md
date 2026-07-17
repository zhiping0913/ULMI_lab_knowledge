# propagate_and_clip plan and CSV mapping — 2026-07-06

Triggered by Telegram `main:6420513923:81`.

Remote files:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_and_clip.slurm
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_and_clip.py
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/propagate_and_clip.csv
```

Local read-only copies / parsed summary:

```text
artifacts/curved_surface_2d_a020_inspection_20260706_204001/propagate_and_clip.slurm
artifacts/curved_surface_2d_a020_inspection_20260706_204001/propagate_and_clip.py
artifacts/curved_surface_2d_a020_inspection_20260706_204001/propagate_and_clip.csv
artifacts/curved_surface_2d_a020_inspection_20260706_204001/propagate_and_clip_plan_summary.md
```

## Zhiping's explanation

`propagate_and_clip.slurm` calls `propagate_and_clip.py`. The intended operation is:

1. Start from the rotated reflection or transmission field produced by `rotate_and_shift.py`.
2. First crop the true reflected/transmitted pulse region: `clip_0` / `# Clip region for propagation input`.
3. Propagate the cropped field for a case-specific time so the pulse moves away from target noise.
4. Crop again after propagation: `clip_1` / `# Clip region for propagation output`.
5. The values `clip_0`, `propagation_time`, and `clip_1` are case-by-case. Zhiping already made low-resolution previews and saved the tab-separated table in `propagate_and_clip.csv`.

## CSV structure

The CSV is tab-delimited, 33 lines total: one header + 32 data rows = 16 thin cases × 2 sides. It does not include the still-running thick case `K=-0.002,D=1.95,L=0.05`.

Header columns:

```text
working_dir (/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D)
side
x_min_clip_0 (laser_lambda)
x_max_clip_0 (laser_lambda)
y_min_clip_0 (laser_lambda)
y_max_clip_0 (laser_lambda)
x_min_clip_1 (laser_lambda)
x_max_clip_1 (laser_lambda)
y_min_clip_1 (laser_lambda)
y_max_clip_1 (laser_lambda)
propagation_time (laser_period)
done
```

Column-to-script mapping:

| CSV column | `propagate_and_clip.py` variable | Unit/action |
|---|---|---|
| first column | `working_dir` | case dir under base `/scratch/gpfs/.../a0=20/2D` |
| `side` | `side` | selects `fields0001_250cpl_<side>.nc`; output is `<side>_clip.nc` |
| `x_min_clip_0`, `x_max_clip_0` | `x_min_clip_0`, `x_max_clip_0` | multiply by `laser_lambda` |
| `y_min_clip_0`, `y_max_clip_0` | `y_min_clip_0`, `y_max_clip_0` | multiply by `laser_lambda` |
| `propagation_time` | `propagation_time` | multiply by `laser_period` |
| `x_min_clip_1`, `x_max_clip_1` | `x_min_clip_1`, `x_max_clip_1` | multiply by `laser_lambda` |
| `y_min_clip_1`, `y_max_clip_1` | `y_min_clip_1`, `y_max_clip_1` | multiply by `laser_lambda` |
| `done` | not currently used by script | should skip `Y` rows if batching |

Current `done` status in CSV:

- `Y`: only `K=-0.002,D=0.01,L=0.00` `transmission`.
- `N`: remaining 31 rows.

## Critical implementation observation

The current `propagate_and_clip.py` does **not** read `propagate_and_clip.csv`. It is a single-case script with hard-coded values:

```text
working_dir = '/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/K=-0.002,D=0.01,L=0.00'
side = 'transmission'
x_min_clip_0 = 30λ0, x_max_clip_0 = 50λ0, y_min_clip_0=-35λ0, y_max_clip_0=35λ0
propagation_time = 140 T0
x_min_clip_1 = 170λ0, x_max_clip_1 = 190λ0, y_min_clip_1=-25λ0, y_max_clip_1=25λ0
```

These exactly match the CSV row currently marked `done=Y`. Therefore, running `propagate_and_clip.slurm` as-is would likely rerun only that already-done transmission row, not process the 31 `done=N` rows.

Do not submit as-is unless Zhiping explicitly wants to rerun that one row.

## Script workflow

`propagate_and_clip.py`:

1. Initializes JAX distributed when running under Slurm with `SLURM_NTASKS > 1`.
2. Reads `fields0001_250cpl_<side>.nc` from `working_dir` using `read_nc` with keys `Ex`, `Ey`, `Bz`.
3. Clips input grid to `clip_0` region.
4. Initializes `Spectral_Maxwell_Solver_2D` with clipped fields, zero `Ez/Bx/By`.
5. Propagates with `window_shift_velocity=(c,0)` for `propagation_time`.
6. Gathers sharded outputs across processes.
7. On main process, clips output grid to `clip_1`, writes `<side>_clip.nc`, and plots `<side>_clip.png` using normalized `Bz/Bc`.

## Corrected safe next step if Zhiping asks to run batch

Zhiping corrected in Telegram `main:6420513923:85`: do **not** make `propagate_and_clip.py` consume `propagate_and_clip.csv` directly. The Python script is a generic propagation/clipping kernel and should remain reusable for future cases such as `a0=50`.

Preferred architecture:

1. Modify `propagate_and_clip.py` only to replace hard-coded case parameters with environment-variable reads, e.g. `os.environ.get('WORKING_DIR', default)`, `os.environ.get('SIDE', default)`, `os.environ.get('X_MIN_CLIP_0', default)`, etc. Keep hard-coded defaults for VSCode/manual/debug fallback.
2. Keep `propagate_and_clip.csv` as an input to an outer driver for this specific `a0=20/2D` batch, not as a Python dependency.
3. Write a bash driver that reads the tab-delimited CSV, selects rows with `done != Y`, and for each row generates/submits a Slurm job exporting the row values as environment variables.
4. Possible exported variables: `WORKING_DIR`, `SIDE`, `X_MIN_CLIP_0`, `X_MAX_CLIP_0`, `Y_MIN_CLIP_0`, `Y_MAX_CLIP_0`, `PROPAGATION_TIME`, `X_MIN_CLIP_1`, `X_MAX_CLIP_1`, `Y_MIN_CLIP_1`, `Y_MAX_CLIP_1`.
5. Prefer dry-run first: generate scripts or print `sbatch` commands without submitting, then inspect.
6. Before any actual `sbatch`, run `checkquota`; block if Tiger scratch has <2 TiB free.
7. Update `done` only after verifying output files exist and are sensible.

No editing/submission has been done yet.
