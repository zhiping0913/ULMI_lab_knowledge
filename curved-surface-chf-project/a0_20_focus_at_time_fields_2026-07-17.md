# a0=20/2D focus-at-time field generation — 2026-07-17

## Request

Zhiping asked on Telegram (`main:6420513923:410`) to use

- Slurm wrapper: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_at_time.slurm`
- Python worker: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_at_time.py`
- input table: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/focus.tsv`

for every row of `focus.tsv`, mapping

```text
working_dir           = /scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/<focus.tsv working_dir>
nc_name               = {side}_clip.nc
propagation_time_T0   = propagation_time (laser_period)
output_name           = {side}_focus
```

to save the field at each row's focus time.

## Preflight and submission

Preflight found 32 rows, no missing `{side}_clip.nc` inputs, and no pre-existing `{side}_focus` output pairs. Before submission, `checkquota` passed on Tiger MIKHAILOVA scratch: used by ALL about `9.5 TiB / 15 TiB`, leaving about `5.5 TiB`, above the required `2 TiB` free threshold.

Original batch:

- jobs: `3540190–3540221`
- resources: `--mem=600G`, `--time=2:00:00`, 6 CPU
- logdir: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_at_time_logs/a0_20_2D_focus_fields_20260717_213228`
- manifest: `.../manifest.tsv`

Original result: `18/32` completed and wrote expected output pairs; `14/32` ended `OUT_OF_MEMORY` at `600G`.

Zhiping authorized rerunning OOM failures with larger memory (`main:6420513923:415`). After another quota check, only the failed rows were retried.

Retry batch:

- jobs: `3540403–3540416`
- resources: `--mem=900G`, `--time=2:00:00`, 6 CPU
- logdir: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_at_time_logs/a0_20_2D_focus_fields_retry_900G_20260717_221211`
- retry manifest: `.../retry_manifest.tsv`

## Final validation

Final Slurm status by `2026-07-17 23:13 EDT`:

- Retry jobs `3540404–3540416` completed, except row 1 retry `3540403` failed OOM.
- Row 1 retry `3540403` (`K=-0.002,ND_a0=0.10,L=0.00`, `reflection`, `T=20T0`) had `MaxRSS=943,715,876K` and Slurm `.err` reported:

```text
Killed ... propagate_2D_at_time.py ...
Detected 1 oom_kill event in StepId=3540403.batch. Some of the step tasks have been OOM Killed.
```

- Attempts made earlier to submit row 1 at `1000G` and at `--mem=0` all-memory were rejected by Slurm (`Requested node configuration is not available` / memory unavailable). Therefore row 1 should not be blindly resubmitted with larger memory.

Output validation used local helper script `artifacts/check_a0_20_focus_outputs.py`, copied to `tiger-vis:/tmp/check_a0_20_focus_outputs.py` and run in the `mpi_python` conda environment against the original and retry manifests. Results:

```text
TOTAL_ROWS          32
OK_ROWS             31
MISSING_OR_BAD_ROWS 1
OPEN_FAIL_ROWS      0
```

The `31/32` OK rows have both `{side}_focus.nc` and `{side}_focus_Bz.png`; the NetCDF files are xarray-openable and contain `Ex,Ey,Bz`. The only missing output pair is:

```text
row=1
case=K=-0.002,ND_a0=0.10,L=0.00
side=reflection
T0=20
orig_job=3540190
retry_job=3540403
expected_nc=/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/K=-0.002,ND_a0=0.10,L=0.00/reflection_focus.nc
```

A sample from the validator confirms rows like row 2 (`transmission`, retry job `3540404`) and row 3 (`reflection`, original job `3540192`) have openable outputs with `Ex,Ey,Bz`.

## Memory diagnosis for row 1

The row 1 `.out` tail showed a very large padded spectral array before propagation:

```text
shape: (2, 3, 12512, 30016, 1)
dtype: complex128
size: 33.6 GiB
NamedSharding: P('EM', 'channel')
```

This is only one visible array; the propagation path likely holds several similarly large complex128 intermediates (`spectrum`, k-grid components, transverse projection / transversality arrays, propagated fields), so peak resident memory can exceed `900G` even though one array is only `O(10–100 GiB)`. This failure is an algorithmic-memory issue, not a transient collective/JAX rendezvous failure.

If row 1 is required, next steps should be memory reduction rather than larger Slurm memory:

- reduce precision if physically acceptable (`complex64` / `float32` for propagation intermediates, with a comparison check against a smaller case);
- avoid keeping duplicate full-volume complex128 arrays alive;
- chunk/stream over spatial or k-space dimensions if the propagation formula allows it;
- write a row-1-specific diagnostic script to identify peak allocations and minimum necessary array set.

## Human report

Telegram report sent as `main:6420513923:419`: `31/32` focus fields generated and readable; row 1 remains unresolved due to true OOM at 900G and needs algorithmic memory reduction if required. The report also noted that the a0=50 fine scan was still running at that time.

## Status

- Done for 31 of 32 requested rows.
- Open issue: row 1 `K=-0.002,ND_a0=0.10,L=0.00/reflection_focus` missing due to OOM; do not resubmit blindly with larger memory.
