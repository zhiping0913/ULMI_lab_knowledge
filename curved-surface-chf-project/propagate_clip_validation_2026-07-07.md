# propagate_and_clip validation — 2026-07-07

Triggered by Telegram `main:6420513923:99`: Zhiping reported that the submitted batch of 31 jobs completed. Main performed a read-only validation pass on tiger-vis and reported clean results in Telegram `main:6420513923:102`.

## Validation scope

Jobs:

```text
3382363–3382393
```

Expected outputs:

For each of the 31 `done=N` rows in:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/propagate_and_clip.csv
```

check that the corresponding case directory contains:

```text
<side>_clip.nc
<side>_clip.png
```

where `side` is `reflection` or `transmission`.

Log directory:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_and_clip_slurms/a0_20_2D/
```

## Validation script

Local validation helper:

```text
artifacts/curved_surface_2d_a020_inspection_20260706_204001/validate_propclip_jobs.py
```

It was copied to tiger-vis `/tmp/validate_propclip_jobs.py` and run read-only. The first attempt to validate failed because an ssh heredoc quoting error stripped Python string quotes; no remote files or jobs were affected. The second attempt used `scp` for the validation script and succeeded.

## Results

Validation output:

```text
# propagate_and_clip validation
time 2026-07-07T11:42:39.078302
job_ids 3382363-3382393 count 31

## sacct
state_counts {'COMPLETED': 31}
bad_count 0

## output files
expected_rows 31
present_nc_png 31
missing_count 0
small_file_count 0

## logs
out_count 32 err_count 32 nonempty_err_count 0
out_potential_bad_count 0
```

Interpretation:

- All 31 submitted jobs completed with no bad Slurm accounting state.
- All 31 expected `<side>_clip.nc` and `<side>_clip.png` output pairs exist.
- No missing outputs.
- No trivially tiny output files.
- No nonempty `.err` files.
- No suspicious `.out` logs by the simple marker heuristic.

## Notes

The log directory has 32 `.out` / `.err` files because it includes the one previously completed/done row or prior dry/run artifact in addition to the 31 submitted jobs. The validation of expected outputs was based on the 31 `done=N` CSV rows and passed.

Next scientific step, if Zhiping asks: use these clipped fields for downstream propagation/focusing or diagnostic analysis; map outputs to `G_spatial` and eventually `C_wavefront`.
