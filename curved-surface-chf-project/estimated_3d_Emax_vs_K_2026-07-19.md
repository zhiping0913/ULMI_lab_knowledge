# Estimated 3D focus Emax(K) from 2D Curved_surface focus data

Date: 2026-07-19
Telegram request: Zhiping asked in `main:6420513923:453` for a more conservative but realistic 2D→3D focus estimate, and specifically a plot of estimated 3D focus `E_max` versus curvature `K`, based on the existing 2D plot notebook `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/plot_Emax_vs_K_reflection_ND_scan_a0_20.ipynb`.

## Delivered artifacts

Telegram delivery:

- Text/report: `main:6420513923:455`
- PNG: `main:6420513923:456`
- PDF: `main:6420513923:457`
- TSV: `main:6420513923:458`
- Notebook: `main:6420513923:459`

Remote output directory:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/a0_20_estimated_3d_Emax_vs_K_20260719/
```

Local copy:

```text
artifacts/a0_20_estimated_3d_Emax_vs_K_20260719/
```

Key files:

- `plot_estimated_3d_Emax_vs_K_reflection_a0_20.py`
- `plot_estimated_3d_Emax_vs_K_reflection_a0_20.ipynb`
- `a0_20_reflection_estimated_3D_Emax_vs_K_locked_n01_20_alpha0p5.png/.pdf/.tsv`
- `a0_20_reflection_estimated_3D_Emax_vs_K_locked_n01_20_alpha0p5_params.json`

## Data and model

The existing 2D focus data come from:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/focus.tsv
```

using the same columns and definitions as the existing 2D notebook:

```text
E2D_max_Ec = E_max(Ec)
x_at_E_max_lambda0 = x_at_E_max (lambda0)
```

The missing 2D→3D focusing multiplier is estimated from `spectrum_1D.py` per-harmonic TSVs:

```text
1D: /scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/1D/45/ND_a0=<ND>/reflection_per_harmonic_at_peak.tsv
2D: /scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/<case>/reflection_focus_at_y_per_harmonic_at_peak.tsv
```

Definitions:

```text
G_2D,E(n,K,ND) = A_2D(n,K,ND) / A_1D(n,ND)
```

where `A_n` is `spectrum_1D.py`'s `envelope_at_peak_over_a0EcM`.

Locked band:

```text
n = 1..20
```

This follows Zhiping's interpretation: harmonics `1–20` are visibly phase locked at the focus; `n>20` may appear detuned mainly because the current `reflection.nc` / focus-at-y data were stored at `250 cpl`, making spatial resolution insufficient for higher harmonics in subsequent post-processing.

The locked-band scalar gain is an amplitude-weighted mean over the 2D focal per-order amplitudes:

```text
G_locked(K,ND) = average_n[G_2D,E(n,K,ND); weights=A_2D(n,K,ND)], n=1..20
```

3D field estimates from the 2D simulated focus field:

```text
E3D_ideal = E2D * G_locked
E3D_conservative = E2D * sqrt(G_locked)
```

Interpretation:

- `E3D_ideal` assumes the missing transverse dimension focuses as well as the simulated 2D transverse dimension (separable/symmetric focusing upper bound).
- `E3D_conservative` is a midpoint between no extra focusing and ideal extra focusing. It can be viewed as applying an effective Strehl/aberration/finite-aperture/coherence penalty `S_eff = 1/sqrt(G_locked)` to the ideal missing-dimension field gain.
- This is not a 3D PIC result.

## Main numerical result

Selected table from the run:

```text
ND/a0=0.30:
K=-0.008  E2D=153.5 Ec, G_locked=4.368, sqrt=2.090 → E3D_cons=320.9 Ec, ideal=670.7 Ec
K=-0.005  E2D=156.9 Ec, G_locked=4.431, sqrt=2.105 → E3D_cons=330.3 Ec, ideal=695.4 Ec
K=-0.002  E2D=113.5 Ec, G_locked=3.101, sqrt=1.761 → E3D_cons=199.9 Ec, ideal=352.0 Ec
K=+0.000  E2D=71.42 Ec, G_locked=1.953, sqrt=1.397 → E3D_cons=99.8 Ec, ideal=139.5 Ec

ND/a0=0.50:
K=-0.008  E2D=123.4 Ec, G_locked=3.795, sqrt=1.948 → E3D_cons=240.3 Ec, ideal=468.2 Ec
K=-0.005  E2D=115.7 Ec, G_locked=3.545, sqrt=1.883 → E3D_cons=217.9 Ec, ideal=410.2 Ec
K=-0.002  E2D=80.36 Ec, G_locked=2.361, sqrt=1.537 → E3D_cons=123.5 Ec, ideal=189.7 Ec
K=+0.000  E2D=47.55 Ec, G_locked=1.309, sqrt=1.144 → E3D_cons=54.4 Ec, ideal=62.3 Ec

ND/a0=1.00:
K=-0.008  E2D=85.51 Ec, G_locked=3.339, sqrt=1.827 → E3D_cons=156.2 Ec, ideal=285.5 Ec
K=-0.005  E2D=82.93 Ec, G_locked=3.273, sqrt=1.809 → E3D_cons=150.0 Ec, ideal=271.4 Ec
K=-0.002  E2D=61.75 Ec, G_locked=2.372, sqrt=1.540 → E3D_cons=95.1 Ec, ideal=146.5 Ec
K=+0.000  E2D=30.69 Ec, G_locked=1.077, sqrt=1.038 → E3D_cons=31.9 Ec, ideal=33.1 Ec
```

Best conservative estimates:

- `ND/a0=0.30`: best `K=-0.005`, `E3D_cons≈330 Ec`; `K=-0.008` is nearly tied at `≈321 Ec`.
- `ND/a0=0.50`: best `K=-0.008`, `E3D_cons≈240 Ec`.
- `ND/a0=1.00`: best `K=-0.008`, `E3D_cons≈156 Ec`.

## Suggested wording for manuscript/notes

For current data, a safe statement is:

> The first twenty harmonic orders are phase locked at the harmonic focus. Orders above ~20 are not used for the 3D extrapolation because the present 250-cells-per-wavelength stored fields likely under-resolve their spatial oscillations in the post-processing. Using the phase-locked `n≤20` band, the 2D centerline harmonic focusing gain gives `G_locked≈3–4.5` in field amplitude for the best curved cases. Ideal symmetric 3D focusing would multiply the 2D focal field by `G_locked`; as a conservative estimate including 3D aberration/finite-aperture/coherence losses, we use `sqrt(G_locked)`. This gives estimated 3D fields of order `E_max≈3.2–3.3×10^2 Ec` for `a0=20, ND/a0=0.30, K≈-0.005…-0.008`, while the ideal upper estimate is `≈6.7–7.0×10^2 Ec`.

## Caveats

- This is an extrapolation, not a 3D PIC simulation.
- The conservative exponent `α=0.5` is adjustable in the notebook. For a more aggressive estimate use larger `α`; for no extra focusing use `α=0`.
- The estimate assumes the missing 3D transverse dimension has comparable curvature/coherence properties to the simulated 2D focusing dimension, but with a strong effective penalty.
- The model does not include true 3D foil deformation, astigmatism, finite-aperture clipping, polarization/vector effects, or 3D wavefront errors explicitly.
