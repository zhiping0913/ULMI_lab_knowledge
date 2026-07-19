# a0=20 focus y=0 slices and aligned real-Bz comparison — 2026-07-18

## Request

Zhiping asked on Telegram (`main:6420513923:425`) to process existing Curved_surface `a0=20/2D` focus fields:

1. Use `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/slice_2D_at_y.py` to take `y=0` slices of `reflection_focus.nc` / `transmission_focus.nc`, producing `{side}_focus_at_y.nc`.
2. Run `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_1D.py` on those slices with `SIDE={side}_focus_at_y`, `THETA_DEGREE=0`, `NC_NAME={side}_focus_at_y.nc`, producing `{side}_focus_at_y_spectrum.nc` and HH pulse metadata.
3. Build a configurable notebook with editable `ND_A0` and `SIDE` that auto-collects the 1D reference and 2D y=0 focus slices, aligns by the `E_forward_hh_envelope` peak x, and plots real `Bz` only in a `±4 λ0` window.

The 1D reference must use the oblique moving-frame normalization for `θ=45°`: `λ_M=λ0/cos45°`, `B_{c,M}=B_c cos45°`; 2D uses `θ=0`, so `λ_M=λ0`, `B_{c,M}=B_c`.

## Script/code inspection

Code inspection found the following environment contracts:

- `slice_2D_at_y.py` reads `working_dir`, `nc_name`, and `y_lambda` (default 0). If input is `{stem}.nc`, it writes `{stem}_at_y.nc` and `{stem}_at_y_Bz.png`.
- `spectrum_1D.py` reads `WORKING_DIR`/`working_dir`, `SIDE`, `NC_NAME`/`nc_name`, `THETA_DEGREE`, `LASER_A0`, and harmonic-window env vars. `NC_NAME` is the input file. Output filenames are derived from `SIDE`: `{SIDE}_spectrum.nc`, `{SIDE}_Sx.nc`, `{SIDE}_per_harmonic_at_peak.tsv`, plus plots.
- Therefore the correct spectrum call for reflection y-slice is `SIDE=reflection_focus_at_y`, `NC_NAME=reflection_focus_at_y.nc`; this writes `reflection_focus_at_y_spectrum.nc`.
- `spectrum_1D.py` computes HH real/envelope arrays and prints the HH envelope peak x and FWHM, but it does not save `E_forward_hh_real`, `E_forward_hh_envelope`, or `pulse_len_T0` into the spectrum NetCDF. The batch workflow therefore parses stdout for peak/duration metadata.

Zhiping was notified of the `NC_NAME` correction in Telegram `main:6420513923:427`.

## Batch processing

Local driver source:

```text
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/process_focus_at_y_and_spectrum.py
```

Remote driver copy:

```text
/tmp/process_focus_at_y_and_spectrum.py
```

Run directory:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_at_y_spectrum_logs/a0_20_focus_at_y_spectrum_20260718_2023/
```

The driver was launched as async bash job `job-2e7e3a59` (not Slurm) on `tiger-vis`. It:

- processed all existing `a0=20/2D/K=...,ND_a0=...,L=.../{side}_focus.nc` files for `reflection` and `transmission`;
- generated `{side}_focus_at_y.nc`/PNG y=0 slices where needed;
- ran `spectrum_1D.py` with `THETA_DEGREE=0` on 2D y-slices;
- ran `spectrum_1D.py` with `THETA_DEGREE=45` on 1D reference files under `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/1D/45/ND_a0=...`;
- wrote metadata table:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_at_y_spectrum_logs/a0_20_focus_at_y_spectrum_20260718_2023/focus_at_y_spectrum_metadata.tsv
```

Final driver result:

```text
TOTAL_ROWS 92
ok 91
missing_focus_nc 1
```

The only non-OK row was the known missing focus field from the earlier OOM case:

```text
2D K=-0.002,ND_a0=0.10,L=0.00 reflection missing_focus_nc
```

For `K=-0.005,ND_a0=0.30,reflection`, an old pre-existing spectrum file caused the batch driver to skip re-running spectrum and initially left metadata blank. I reran just that spectrum command and patched the metadata table:

```text
x_peak = 57.3300 λ0
peak_env/Ec = 134.6729
pulse_len_T0 = 0.0504
```

## Plotting deliverables

Configurable plotting script:

- local: `artifacts/a0_20_focus_at_y_aligned_Bz_20260718/plot_focus_at_y_aligned_bz.py`
- remote: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/plot_focus_at_y_aligned_bz.py`

Configurable notebook:

- local: `artifacts/a0_20_focus_at_y_aligned_Bz_20260718/plot_focus_at_y_aligned_Bz.ipynb`
- remote: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/a0_20_focus_at_y_aligned_Bz_20260718/plot_focus_at_y_aligned_Bz.ipynb`

The notebook/script expose editable/env parameters:

```text
ND_A0=0.30
SIDE=reflection
K_LIST=+0.000,-0.002,-0.005,-0.008
WINDOW_LAMBDA=4.0
```

Default output directory:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/a0_20_focus_at_y_aligned_Bz_20260718/
```

Default local artifacts:

```text
artifacts/a0_20_focus_at_y_aligned_Bz_20260718/
```

Default plotted figure for `ND/a0=0.30`, `side=reflection`:

- `a0_20_ND_a0_0p30_reflection_focus_at_y_aligned_Bz.png`
- `a0_20_ND_a0_0p30_reflection_focus_at_y_aligned_Bz.pdf`
- `a0_20_ND_a0_0p30_reflection_focus_at_y_aligned_Bz_summary.tsv`
- `a0_20_ND_a0_0p30_reflection_focus_at_y_aligned_Bz_params.json`

Visual QA via the `vision` tool found the plot readable: curves/legend/axes/title rendered correctly, with the dashed vertical alignment marker at zero and the requested aligned real-`Bz` comparison visible.

## Default comparison values

For `ND/a0=0.30`, `side=reflection`, default rows were:

| case | HH envelope peak x | duration | window Bz range after normalization |
|---|---:|---:|---:|
| 1D | `7.727 λ_M` | `0.0129 T0` | `[-22.5, 22]` |
| 2D K=+0.000 | `282.342 λ0` | `0.0498 T0` | `[-62.2, 18.6]` |
| 2D K=-0.002 | `118.338 λ0` | `0.0521 T0` | `[-98.4, 27.3]` |
| 2D K=-0.005 | `57.330 λ0` | `0.0504 T0` | `[-144, 32.3]` |
| 2D K=-0.008 | `37.330 λ0` | `0.0487 T0` | `[-144, 29.7]` |

These values are metadata for alignment and plotting, not a new physical conclusion by themselves. The physics-consistent trend remains that stronger negative curvature brings the reflected focus closer to the target, while the real `Bz` waveform comparison shows the on-axis field structure around each aligned HH peak.

## Human report

Telegram report and attachments:

- text report: `main:6420513923:429`
- PNG: `main:6420513923:430`
- PDF: `main:6420513923:431`
- summary TSV: `main:6420513923:432`
- notebook: `main:6420513923:433`

## Status

Complete unless Zhiping asks for another `ND_A0`, `SIDE`, or `K_LIST` plot. The notebook/script are designed for reruns without changing the processing pipeline.
