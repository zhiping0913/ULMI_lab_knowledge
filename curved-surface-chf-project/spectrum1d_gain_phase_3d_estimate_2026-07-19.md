# spectrum_1D.py harmonic gain / phase / ideal 3D focusing estimate

Date: 2026-07-19
Telegram request: Zhiping corrected the previous custom local-window projection and asked in `main:6420513923:444` to use `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_1D.py`'s own spectrum normalization and each-order phase to compare 1D and 2D focus centerline spectra, then estimate 3D focusing gain from the 1D→2D gain.

## Delivered artifacts

Telegram delivery:

- Text/report: `main:6420513923:446`
- Main PNG/PDF: `main:6420513923:447-448`
- Phase-at-peak auxiliary PNG: `main:6420513923:449`
- Full data TSV: `main:6420513923:450`
- Phase/gain band statistics TSV: `main:6420513923:451`
- Notebook: `main:6420513923:452`

Remote output directory:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/a0_20_focus_at_y_spectrum1d_gain_3d_20260719/
```

Local copy:

```text
artifacts/a0_20_focus_at_y_spectrum1d_gain_3d_20260719/
```

Key files:

- `plot_spectrum1d_harmonic_gain_phase_3d_estimate.py`
- `plot_spectrum1d_harmonic_gain_phase_3d_estimate.ipynb`
- `a0_20_ND_a0_0p30_reflection_spectrum1D_gain_phase_n01_30_ref_1D.png/.pdf/.tsv`
- `a0_20_ND_a0_0p30_reflection_spectrum1D_phase_at_peak_gain_n01_30.png/.pdf`
- `a0_20_ND_a0_0p30_reflection_spectrum1D_phase_gain_band_stats.tsv`

## Method

Default parameters:

- `a0=20`, `ND/a0=0.30`, `side=reflection`
- Cases: 1D reference plus 2D `K=+0.000`, `K=-0.002`, `K=-0.005`, `K=-0.008`
- Harmonic orders `n=1..30`

Inputs:

- 1D: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/1D/45/ND_a0=0.30/reflection_per_harmonic_at_peak.tsv`
- 2D: each case's `{side}_focus_at_y_per_harmonic_at_peak.tsv`, e.g. `reflection_focus_at_y_per_harmonic_at_peak.tsv`

Field-amplitude column:

```text
envelope_at_peak_over_a0EcM
```

Phase column:

```text
phase_at_peak_rad
```

Gain definitions:

```text
G_2D,E(n,K) = envelope_2D(n,K) / envelope_1D(n)
G_3D,E,ideal(n,K) ≈ [G_2D,E(n,K)]^2
G_3D,I,ideal(n,K) ≈ [G_2D,E(n,K)]^4
```

The ideal 3D estimate assumes the 2D centerline gain represents one transverse focusing dimension, and a symmetric/separable 3D paraboloidal/spherical focus supplies a comparable second transverse field-amplitude gain. Real 3D gain should be reduced by finite aperture, aberration, wavefront/Strehl, and harmonic-coherence penalties.

Phase diagnostics saved:

```text
phase_at_peak_rad / π
wrapped phase residual: wrap(phase_n - n phase_1) / π
phase residual minus 1D reference residual
```

## Main numerical result

1D reference per-order amplitudes (`envelope_at_peak_over_a0EcM`):

```text
n=1: 0.59301, n=5: 0.13988, n=10: 0.05359, n=20: 0.02681, n=30: 0.02118
```

2D/1D field-amplitude gain samples:

```text
K=+0.000: n1=1.0175, n5=1.8446, n10=2.5845, n20=1.7986, n30=1.0970
K=-0.002: n1=1.5854, n5=3.0658, n10=4.3854, n20=1.2974, n30=2.1473
K=-0.005: n1=2.0552, n5=4.1899, n10=6.2422, n20=2.0103, n30=2.2865
K=-0.008: n1=1.8320, n5=4.1597, n10=6.3177, n20=1.2844, n30=1.0488
```

Band means/maxima for `G_2D,E`:

- `K=+0.000`: mean `1–10 = 1.94`, `11–20 = 2.51`, `21–30 = 1.37`; max `5.25` at `n=15`.
- `K=-0.002`: mean `1–10 = 3.24`, `11–20 = 3.20`, `21–30 = 1.58`; max `6.48` at `n=15`.
- `K=-0.005`: mean `1–10 = 4.41`, `11–20 = 5.24`, `21–30 = 2.35`; max `11.13` at `n=15`.
- `K=-0.008`: mean `1–10 = 4.40`, `11–20 = 4.86`, `21–30 = 2.55`; max `10.07` at `n=15`.

Ideal 3D field-amplitude estimate (`G_3D,E≈G_2D,E²`):

- `K=-0.005`: `n=10 ≈39`, `n=15 ≈124`, band mean `11–20 ≈38`.
- `K=-0.008`: `n=10 ≈40`, `n=15 ≈101`, band mean `11–20 ≈33`.

Corresponding intensity/Poynting gains would be squared again (`G_3D,I≈G_2D,E⁴`) under the same ideal assumptions, so `n=15` for `K=-0.005` is order `1.5e4` in intensity. This is an upper estimate, not a simulated 3D result.

## Phase result

The raw residual `wrap(φ_n - nφ_1)` over all `n=1..30` remains visually busy. More useful is `spectrum_1D.py`'s direct `phase_at_peak` in the dominant band. For `n=5..15`, 2D focus phases cluster tightly compared with 1D:

```text
1D:       circular mean -0.409π, std 0.146π
K=+0.000: mean -0.817π, std 0.021π
K=-0.002: mean -0.763π, std 0.023π
K=-0.005: mean -0.869π, std 0.022π
K=-0.008: mean -0.816π, std 0.026π
```

Interpretation: in the main gain band (`n≈5–15`), the 2D focus centerline shows both enhanced harmonic amplitudes and strong phase clustering. Higher orders `n≈20–30` show wrapping/scatter, likely because amplitudes are smaller and phase becomes sensitive/noisy; do not claim all `1–30` are equally phase locked.

## Implementation notes

- The revised analysis reads spectrum_1D.py per-harmonic TSVs rather than redoing an independent local Fourier projection.
- A separate auxiliary phase plot was created because `phase_at_peak` clustering is more informative than the wrapped `φ_n-nφ_1` residual for this question.
- The notebook has editable defaults for `ND_A0`, `SIDE`, `K_LIST`, harmonic range, and amplitude/phase columns.
