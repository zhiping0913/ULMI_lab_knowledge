# Order-resolved coarse focus scan — 2026-07-13

## Context

Zhiping requested in Telegram `main:6420513923:217` that the updated tiger-vis script

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan.py
```

be used to scan every row in

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/focus.tsv
```

around the existing `propagation_time (laser_period)` center with `center ±100 T0`, step `10 T0`, total 21 points, to roughly locate the focal positions of separate k-bands / harmonic orders. Zhiping noted that the previous very large `K=+0.000,ND_a0=0.50,L=0.00` reflection clip had been re-cropped, so it should no longer require the previous 995 GB exceptional memory treatment.

## Script/interface changes

The script already had k-band filtering:

- `total`: full spectrum
- `band1`: `|k| in [0.5,1.5] k0`
- `band2`: `|k| in [1.5,2.5] k0`
- `band3`: `|k| in [2.5,40.5] k0`

For each band it writes per-timestep/per-peak diagnostics:

- `E_peak`
- `x_at_peak`, `y_at_peak`
- `x_left`, `x_right`, `y_left`, `y_right`
- `width_x`, `width_y`

I patched `propagate_2D_time_scan.py` to support a backward-compatible `output_dir` environment variable, so this scan writes into a subdirectory instead of overwriting prior fine-scan products in each case folder.

Backup:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan.py.bak_output_dir_20260712_212538
```

New driver:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/submit_order_time_scan_from_focus.sh
```

Status helper:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/check_order_scan.py
```

Summary extractor:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/summarize_order_focus_scan.py
```

## Submission and resources

Dry-run validated 32/32 focus rows and all input `*_clip.nc` files. Before submission, `checkquota` showed MIKHAILOVA scratch usage 8.7/15 TiB, above the 2 TiB-free safety threshold.

Submitted Slurm jobs:

```text
3498134–3498166  (Slurm skipped 3498143)
```

Logdir / manifest:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_order_focus_pm100_10T_20260712_2126_order_pm100_10T/
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_order_focus_pm100_10T_20260712_2126_order_pm100_10T/manifest.tsv
```

Resource policy:

- 27 standard jobs: `700G / 5h`
- 5 big-clip jobs (`clip.nc >= 1.75 GiB`): `900G / 12h`
- No 995 GB exceptional row; the re-cropped `K=+0.000,ND_a0=0.50,L=0.00` reflection row ran as a normal big-clip `900G/12h` job.

Per-case outputs are under:

```text
.../a0=20/2D/K=...,ND_a0=...,L=.../order_focus_pm100_10T/
```

## Final validation

Final state:

```text
COMPLETED = 32/32
output_ok = 32/32
FAILED/OOM/TIMEOUT = 0
```

All expected nonempty outputs were present for completed jobs, including:

- `{side}_beam_width_strength.h5`
- `{side}_peak_vs_x.nc`
- `{side}_env_total_peak_vs_x.nc`
- `{side}_env_band1_peak_vs_x.nc`
- `{side}_env_band2_peak_vs_x.nc`
- `{side}_env_band3_peak_vs_x.nc`
- `{side}_beam_width_strength.png`
- `{side}_env_yLR_vs_x_per_kband.png`
- `{side}_env_Epeak_vs_x_per_kband.png`

Warning caveat: nonempty `.err` files contained `RuntimeWarning: All-NaN slice encountered` in beam-width aggregation and one `PeakPropertyWarning` about zero peak prominence/width. These are warning-only and similar to previous focus-scan warnings; there was no traceback, OOM, timeout, or missing output.

Telegram reports:

- Submission report: `main:6420513923:219`
- First check: `main:6420513923:220`
- Second check: `main:6420513923:221`
- Final report: `main:6420513923:222`; attached summary TSV in `main:6420513923:223`.

## Per-band focus summary table

The argmax summary table is:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_order_focus_pm100_10T_20260712_2126_order_pm100_10T/order_focus_per_band_argmax_summary_latest.tsv
```

Local attached copy:

```text
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/curved_surface_order_scan_20260712/order_focus_per_band_argmax_summary_latest.tsv
```

Rows: `32 cases/sides × 4 bands = 128`, all status `ok`.

Selection rule: for each row and band, choose the scan point with maximum `E_peak` over the 21 sampled propagation times / x positions. This is a coarse sampled maximum, not a continuous optimization.

Important unit/width convention:

- `E_peak_max_Ec`: normalized to `Ec`
- `x_at_E_peak_lambda0`, `y_at_E_peak_lambda0`: peak position in `lambda0`
- `width_x_lambda0`: peak width along propagation axis, intensity-FWHM read from amplitude envelope convention (`rel_height=1-sqrt(2)/2`)
- `width_y_lambda0`: transverse 1/e width

## Global strongest rows from summary extractor

Reflection:

| band | strongest case | `E_peak/Ec` | `x/lambda0` | `width_y/lambda0` |
|---|---:|---:|---:|---:|
| total | `K=-0.005,ND_a0=0.30,L=0.00` | 157.961721 | 57.33 | 1.030309 |
| band1 | `K=-0.008,ND_a0=1.00,L=0.00` | 50.682704 | 39.814 | 3.047545 |
| band2 | `K=-0.008,ND_a0=0.30,L=0.00` | 25.959653 | 37.822 | 1.915984 |
| band3 | `K=-0.005,ND_a0=0.30,L=0.00` | 117.120054 | 57.326 | 0.635979 |

Transmission:

| band | strongest case | `E_peak/Ec` | `x/lambda0` | `width_y/lambda0` |
|---|---:|---:|---:|---:|
| total | `K=+0.000,ND_a0=0.15,L=0.00` | 39.499087 | 67.574 | 9.055111 |
| band1 | `K=+0.000,ND_a0=0.15,L=0.00` | 26.033846 | 57.63 | 12.403352 |
| band2 | `K=+0.000,ND_a0=0.15,L=0.00` | 7.642685 | 46.962 | 2.582411 |
| band3 | `K=-0.005,ND_a0=0.30,L=0.00` | 33.167908 | 85.154 | 1.427631 |

## Reflection band1/band2 scientific snapshot

Across reflection cases, tighter curvature (`K=-0.008`) moves band1/band2 foci closer to the target (`x≈38–44 lambda0`), while weaker curvature (`K=-0.002`) puts them around `x≈130–160 lambda0`; the flat cases (`K=0`) have broad transverse widths and focus farther away / weakly, especially for low orders. This is consistent with a spatial focusing interpretation: stronger curvature gives shorter focal length and smaller low-order transverse width, but the optimal `ND/a0` differs by band because the harmonic-order content and angular spectrum are not the same.

Caveats:

- These are 21-point coarse scans; if Zhiping wants precise focus locations, refine around each per-band argmax.
- The current summary takes max `E_peak` per band, not integrated energy or efficiency; high `E_peak` can be influenced by pulse train structure and band filtering.
- `band3` aggregates many orders `[2.5,40.5] k0`; it is not a single harmonic order.
- Do not overwrite `focus.tsv` automatically with these band-resolved values unless Zhiping explicitly asks; current `focus.tsv` is total-field oriented.
