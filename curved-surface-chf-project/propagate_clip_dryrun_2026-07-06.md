# propagate_and_clip env-var Slurm dry run — 2026-07-06

Triggered by Telegram `main:6420513923:90`: Zhiping authorized writing the driver in tiger-vis utility and dry-running generation of 31 Slurm scripts. No real job submission was authorized.

## Remote paths changed / created

Modified existing Python script:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_and_clip.py
```

Backup before modification:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_and_clip.py.bak_env_20260706_214530
```

New driver script:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/generate_propagate_and_clip_slurms.sh
```

Generated Slurm directory:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_and_clip_slurms/a0_20_2D/
```

Local exact copies / helper patch scripts:

```text
artifacts/curved_surface_2d_a020_inspection_20260706_204001/propagate_and_clip.env_patched.py
artifacts/curved_surface_2d_a020_inspection_20260706_204001/generate_propagate_and_clip_slurms.sh
artifacts/curved_surface_2d_a020_inspection_20260706_204001/patch_propclip_env_driver.py
artifacts/curved_surface_2d_a020_inspection_20260706_204001/fix_generator_final_newline.py
```

## Python patch

`propagate_and_clip.py` now keeps old defaults but reads case-specific parameters from environment variables when they are supplied:

```text
WORKING_DIR
SIDE
X_MIN_CLIP_0
X_MAX_CLIP_0
Y_MIN_CLIP_0
Y_MAX_CLIP_0
X_MIN_CLIP_1
X_MAX_CLIP_1
Y_MIN_CLIP_1
Y_MAX_CLIP_1
PROPAGATION_TIME
```

Coordinate env vars are in `laser_lambda` units. `PROPAGATION_TIME` is in `laser_period` units. If an env var is absent/empty, the old hard-coded default is used. `SIDE` is validated to be `reflection` or `transmission`.

Python syntax check passed via `compile(...)`; no actual propagation run was performed.

## Driver behavior

`generate_propagate_and_clip_slurms.sh` reads the tab-delimited CSV:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/propagate_and_clip.csv
```

Default behavior:

- `DRY_RUN=1`
- `SUBMIT=0`
- creates Slurm files but does not call `sbatch`.

It skips rows with `done=Y`; for each `done=N` row it writes one executable Slurm script exporting the row values as environment variables and then running:

```text
srun /home/zl8336/.conda/envs/mpi_python/bin/python3.13 /scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_and_clip.py
```

The generated scripts preserve the original resource/environment pattern:

- account `mikhailova`
- nodes `2`
- ntasks-per-node `1`
- cpus-per-task `3`
- mem `600G`
- time `2:00:00`
- QOS `standard`
- anaconda3/2024.10 + `/home/zl8336/.conda/envs/mpi_python`
- JAX CPU/distributed env including `JAX_COORDINATOR`.

## Dry-run validation results

First generator run produced 30 scripts, because the TSV's final line lacked a newline and the bash `read` loop dropped it. This was fixed by changing the loop condition to handle a non-newline-terminated final record:

```bash
while IFS=$'\t' read -r ... || [[ -n "${case_rel:-}" ]]; do
```

After the fix:

```text
summary: generated=31, skipped_done=1, out_dir=/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_and_clip_slurms/a0_20_2D, dry_run=1, submit=0
SLURM_COUNT=31
MISSING_CHECK=present
SLURM_SYNTAX_OK=31
WOULD_SBATCH_COUNT=31
summary: generated=31, skipped_done=1, out_dir=/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_and_clip_slurms/a0_20_2D, dry_run=1, submit=1
```

Meaning:

- 31 `done=N` rows have corresponding Slurm files.
- 1 `done=Y` row was skipped: `K=-0.002,D=0.01,L=0.00 transmission`.
- All 31 generated Slurm files pass `bash -n`.
- `SUBMIT=1 DRY_RUN=1` would submit 31 jobs, but no actual `sbatch` was executed.

## Submission boundary

No real Slurm jobs were submitted. Before any future submission:

1. Run `checkquota`.
2. Block if Tiger scratch has <2 TiB free.
3. Get Zhiping's explicit confirmation.
4. Then submit with:

```bash
SUBMIT=1 DRY_RUN=0 /scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/generate_propagate_and_clip_slurms.sh
```
