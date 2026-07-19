# Local raw-Bz harmonic spectrum diagnostic at focus_at_y

Date: 2026-07-19
Telegram request: Zhiping asked in `main:6420513923:437` to analyze `1D raw` and `2D focus_at_y raw` within the focus window, plotting harmonic-order gain

```text
G_n(K) = |B_n(x_focus)| / |B_n(reference)|
```

and phase residual

```text
φ_n - n φ_1
```

for harmonic orders 1–30.

## Delivered artifacts

Telegram delivery:

- Text/report: `main:6420513923:439`
- PNG: `main:6420513923:440`
- PDF: `main:6420513923:441`
- TSV: `main:6420513923:442`
- Notebook: `main:6420513923:443`

Remote output directory:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/a0_20_focus_at_y_local_spectrum_20260719/
```

Local copy:

```text
artifacts/a0_20_focus_at_y_local_spectrum_20260719/
```

Files:

- `plot_focus_at_y_local_harmonic_spectrum.py`
- `plot_focus_at_y_local_harmonic_spectrum.ipynb`
- `a0_20_ND_a0_0p30_reflection_local_harmonics_n01_30_W4_ref_1D.png`
- `a0_20_ND_a0_0p30_reflection_local_harmonics_n01_30_W4_ref_1D.pdf`
- `a0_20_ND_a0_0p30_reflection_local_harmonics_n01_30_W4_ref_1D.tsv`
- `a0_20_ND_a0_0p30_reflection_local_harmonics_n01_30_W4_ref_1D_params.json`

## Default analysis parameters

- `a0=20`
- `ND/a0=0.30`
- `side=reflection`
- Cases: `1D raw` plus 2D `K=+0.000`, `K=-0.002`, `K=-0.005`, `K=-0.008`
- Reference case for `G_n`: `1D raw`
- Harmonic range: `n=1..30`
- Local window: `±4 λ` around each case’s HH-envelope peak location from `focus_at_y_spectrum_metadata.tsv`
- Taper: Hann window
- Coefficient definition:

```text
u = x / lambda_norm - x_peak
c_n = ∫ w(u) [Bz(u)/B_norm - local weighted mean] exp(-i 2π n u) du / ∫ w(u) du
|B_n| = 2 |c_n|
G_n(case) = |B_n(case)| / |B_n(1D reference)|
phase_residual_n = wrapped_angle(arg(c_n) - n arg(c_1))
```

Normalization follows Zhiping’s moving-frame convention:

- 1D 45°: `lambda_norm=lambda_M=lambda0/cos45°`, `B_norm=Bc_M=Bc cos45°`
- 2D θ=0: `lambda_norm=lambda0`, `B_norm=Bc`

## Numerical summary

Selected script stdout:

```text
1D gains {1: 1.0, 5: 1.0, 10: 1.0, 20: 1.0, 30: 1.0} phase_resid_pi_rms 0.564 window_points 45255
K=+0.000 gains {1: 1.417, 5: 2.0538, 10: 2.0202, 20: 1.4647, 30: 0.8719} phase_resid_pi_rms 0.5522 window_points 2000
K=-0.002 gains {1: 2.1892, 5: 3.3919, 10: 3.3774, 20: 1.6913, 30: 2.4127} phase_resid_pi_rms 0.5611 window_points 2000
K=-0.005 gains {1: 2.8692, 5: 4.0293, 10: 4.9396, 20: 1.8278, 30: 3.2259} phase_resid_pi_rms 0.5616 window_points 2001
K=-0.008 gains {1: 2.5998, 5: 3.691, 10: 6.408, 20: 2.1021, 30: 1.0566} phase_resid_pi_rms 0.5809 window_points 2000
```

Band-mean trends from the TSV:

- `K=+0.000`: mean gain `n=1–10 ≈4.8`, `11–20 ≈1.21`, `21–30 ≈2.01`; strongest order `n=9` gain `≈16.6`, `n=24` gain `≈9.05`.
- `K=-0.002`: mean gain `n=1–10 ≈7.94`, `11–20 ≈1.73`, `21–30 ≈3.15`; strongest order `n=9` gain `≈24.8`.
- `K=-0.005`: mean gain `n=1–10 ≈9.82`, `11–20 ≈3.27`, `21–30 ≈5.03`; strongest order `n=9` gain `≈33.6`, `n=24` gain `≈14.1`.
- `K=-0.008`: mean gain `n=1–10 ≈9.2`, `11–20 ≈3.59`, `21–30 ≈5.89`; strongest order `n=9` gain `≈35.8`, `n=24` gain `≈20.0`.

## Interpretation and caveats

This first diagnostic supports the spatial-gain/effective-filter part of the coherent-focusing story: 2D focus_at_y raw fields show order-dependent gain relative to the 1D raw reference, especially for `K=-0.005` and `K=-0.008`, with strong peaks around harmonic orders `n≈9` and `n≈24`.

However, the simple wrapped phase residual `φ_n - n φ_1` is jagged and not obviously flat over all `n=1..30`; amplitude-weighted RMS is `~0.5π` for both 1D and 2D curves. This should not yet be reported as proof of full-band phase locking. It may be affected by finite spatial windowing, using the fundamental phase as the reference, small-denominator/noisy orders, and the fact that this is a fixed-time spatial slice rather than a time-domain local field at a fixed point.

Recommended next refinement if needed: fit a linear spectral phase over high-amplitude/high-gain harmonic bands (e.g. `5–15` and `20–28`) and plot fit residuals, or reconstruct the time-domain field cumulatively while varying low-order cutoff. That would more directly diagnose whether the focusing-selected harmonic band is phase locked into an attosecond spike.

## Implementation notes

- The first remote run failed because the tiger-vis `mpi_python` NumPy no longer exposes `np.trapz`; script was patched to use `np.trapezoid`.
- The notebook is generated from the same script and has editable defaults near the top.
