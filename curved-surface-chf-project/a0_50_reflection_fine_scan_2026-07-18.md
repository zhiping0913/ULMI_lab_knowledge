# a0=50/2D ND_a0=0.30 reflection fine focus scan — 2026-07-18

## Request

Zhiping asked on Telegram (`main:6420513923:413`) to refine the completed a0=50, `ND_a0=0.30`, reflection focus scan. For each K case, take the coarse scan's on-axis peak time `time_T0_at_onaxis_peak` from

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/a0_50_focus_center50_hw100_step5_20260717/a0_50_focus_center50_hw100_step5_summary.tsv
```

and scan `±20T0` around that center with `step=1T0` (`41` points), reflection only.

## Submission

Quota check before submission passed on Tiger MIKHAILOVA scratch: used by ALL about `9.5 TiB / 15 TiB`, leaving about `5.5 TiB` (>2 TiB threshold).

Original fine-scan batch:

- jobs: `3540394–3540399`
- resources: `400G`, `2:00:00`, 6 CPU
- logdir: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_50_2D_reflection_fine_hw20_step1_20260717_221016`
- manifest: `.../manifest.tsv`

Centers used:

| K | center T0 | output subdir |
|---:|---:|---|
| -0.012 | -25 | `focus_scan_fine_onaxis_tm25_hw20_step1` |
| -0.010 | -30 | `focus_scan_fine_onaxis_tm30_hw20_step1` |
| -0.008 | -30 | `focus_scan_fine_onaxis_tm30_hw20_step1` |
| -0.005 | -45 | `focus_scan_fine_onaxis_tm45_hw20_step1` |
| -0.002 | -10 | `focus_scan_fine_onaxis_tm10_hw20_step1` |
| +0.000 | -10 | `focus_scan_fine_onaxis_tm10_hw20_step1` |

Original K=`-0.005` job `3540397` failed early with JAX/XLA CPU all-reduce rendezvous abort, not memory (`MaxRSS≈225,979,564K` / 400G). The `.err` contained rendezvous timeout / abort messages; no output was completed for that original run.

Retry:

- job: `3540500` for K=`-0.005`
- resources: same `400G`, `2:00:00`, 6 CPU
- retry logdir: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_50_2D_reflection_fine_hw20_step1_retry1_Km005_20260717_223203`
- retry manifest: `.../retry_manifest.tsv`
- retry result: `COMPLETED`, elapsed `01:12:19`, MaxRSS `215,165,648K`.

## Final Slurm status

By `2026-07-18 00:11 EDT`:

- `3540394` K=-0.012: COMPLETED, elapsed `01:11:33`, MaxRSS `220,817,852K`
- `3540395` K=-0.010: COMPLETED, elapsed `01:15:40`, MaxRSS `214,190,552K`
- `3540396` K=-0.008: COMPLETED, elapsed `01:08:48`, MaxRSS `221,206,048K`
- `3540397` K=-0.005 original: FAILED due XLA all-reduce rendezvous abort, elapsed `00:02:04`, MaxRSS `225,979,564K`
- `3540398` K=-0.002: COMPLETED, elapsed `01:12:17`, MaxRSS `214,944,124K`
- `3540399` K=+0.000: COMPLETED, elapsed `01:17:47`, MaxRSS `219,256,948K`
- `3540500` K=-0.005 retry: COMPLETED, elapsed `01:12:19`, MaxRSS `215,165,648K`

Successful `.err` files were small (~181 bytes); suspicious-line grep found only the original failed K=-0.005 rendezvous abort. Retry stderr was clean by the same check.

## Output validation and summary artifacts

Validation/summarization script:

- local copy: `artifacts/a0_50_reflection_fine_hw20_step1_20260718/summarize_a0_50_reflection_fine_hw20_step1.py`
- remote copy: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/a0_50_reflection_fine_hw20_step1_20260718/summarize_a0_50_reflection_fine_hw20_step1.py`

The script checked all output dirs and required files:

- `reflection_beam_width_strength.h5`
- `reflection_beam_width_strength.png`
- `reflection_peak_vs_x.nc`
- `reflection_env_total_peak_vs_x.nc`
- `reflection_env_band1_peak_vs_x.nc` through `reflection_env_band4_peak_vs_x.nc`
- `reflection_env_Epeak_vs_x_per_kband.png`
- `reflection_env_yLR_vs_x_per_kband.png`

Result:

```text
TOTAL_ROWS      6
OK_FILE_ROWS    6
BAD_OPEN_COUNT  0
```

All NetCDF summaries were openable and had 41 points.

Summary artifacts:

- remote directory: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/a0_50_reflection_fine_hw20_step1_20260718/`
- local artifacts: `artifacts/a0_50_reflection_fine_hw20_step1_20260718/`
- TSV: `a0_50_reflection_fine_hw20_step1_summary.tsv`
- PNG/PDF: `a0_50_reflection_fine_hw20_step1_summary.png/.pdf`
- notebook: `a0_50_reflection_fine_hw20_step1_summary.ipynb`

Summary values (`E` in `Ec`, `x` in `λ0`):

| K | raw `E_peak` max | raw x at max | env-total `E_peak` max | env-total x at max | note |
|---:|---:|---:|---:|---:|---|
| -0.012 | 329.107 | 17.5425 | 350.093 | 16.5325 | original completed |
| -0.010 | 307.297 | 19.5425 | 341.730 | 18.5325 | original completed |
| -0.008 | 285.794 | 22.5475 | 321.085 | 21.5375 | original completed |
| -0.005 | 236.269 | 26.5475 | 282.361 | 25.5375 | original failed, retry completed |
| -0.002 | 197.268 | 35.5525 | 243.271 | 32.5375 | original completed |
| +0.000 | 227.555 | 35.5525 | 281.557 | 37.5475 | original completed |

## Physical interpretation and caveats

The fine scan confirms the expected geometric trend: stronger negative curvature produces a more rapidly converging reflected wavefront, so the focal position is closer to the target and the peak field is higher. As `|K|` decreases, the focus moves downstream and the peak generally falls.

The `K=0` case still has a relatively high envelope-total peak at a farther x. This should not be interpreted as the same curvature-focusing mechanism; it likely reflects residual propagated beam / temporal-spatial structure in the flat case. Design comparisons should therefore keep at least three quantities together: peak field, focal distance/accessibility, and width/divergence/energy concentration.

## Human report

Telegram report and attachments sent as:

- text report: `main:6420513923:420`
- PNG: `main:6420513923:421`
- PDF: `main:6420513923:422`
- TSV: `main:6420513923:423`
- notebook: `main:6420513923:424`

## Status

Complete. No further monitoring is needed for this a0=50 fine reflection scan unless Zhiping asks for additional plotting or interpretation.
