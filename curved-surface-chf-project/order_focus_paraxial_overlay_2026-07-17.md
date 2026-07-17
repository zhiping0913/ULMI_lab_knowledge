# a0=20 order-focus plot: paraxial Gaussian band1 overlay (2026-07-17)

Trigger: Zhiping Telegram `main:6420513923:402`.

Zhiping placed a Gaussian-beam reference notebook on `tiger-vis`:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/gaussian_beam_curved_wavefront.ipynb
```

It derives how an initial Gaussian beam width `w_i` and wavefront curvature radius `R_i` at `z=0` determine focal distance, Rayleigh range, waist, and divergence in the paraxial approximation.

Zhiping asked to add the paraxial estimate to the previous 1st-order focus-position-vs-curvature plot:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/plot_order_focus_vs_K_reflection_ND_scan_a0_20.ipynb
```

Only the 1st-order panel should receive the paraxial estimate; higher-order estimates are deferred because the effective harmonic source waist `w0(k)` is unknown and likely smaller for higher orders.

## Geometry / formula decision

Representative `input.deck` confirms the curved target is a circle with:

```text
Kappa = K                         # unit: 1/lambda0
R = 1/Kappa                       # unit: lambda0
target_f = sqrt((x-target_radius)^2 + y^2) - abs(target_radius)
```

Near the vertex, this is:

```text
x ≈ K y^2 / 2
```

So `K` is the surface curvature `1/R_surface` in units of `1/lambda0`.

For oblique incidence in the plane of curvature, the tangential mirror power increases as:

```text
K_eff = K / cos(theta)
```

not `K*cos(theta)`. Reflection doubles the optical wavefront curvature, so the Gaussian-beam initial wavefront radius to feed into `gaussian_beam_curved_wavefront.ipynb` is:

```text
R_i = 1 / (2 K_eff) = cos(theta) / (2 K)
```

For this requested overlay:

```text
theta = 45 deg
w_i = 12.5 lambda0
k/k0 = 1   # band1 / fundamental estimate
```

The paraxial comparison is therefore a 1st-order / fundamental estimate only.

## Remote modifications

Patched/regenerated:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/plot_order_focus_vs_K_reflection_ND_scan_a0_20.py
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/plot_order_focus_vs_K_reflection_ND_scan_a0_20.ipynb
```

The original `.py` did not exist at first; it was reconstructed from the notebook code cell, then patched. Backups created:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/plot_order_focus_vs_K_reflection_ND_scan_a0_20.ipynb.bak_20260717_184920_paraxial_band1
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/plot_order_focus_vs_K_reflection_ND_scan_a0_20.py.bak_20260717_184935_paraxial_band1
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/plot_order_focus_vs_K_reflection_ND_scan_a0_20.ipynb.bak_20260717_184935_paraxial_band1
```

Patch details:

- added `matplotlib.use('Agg')` for noninteractive reruns;
- added constants `THETA_DEG=45`, `LASER_W0_LAMBDA0=12.5`, `PARAXIAL_K_REL_BAND1=1`;
- added `paraxial_focus_from_surface_K()` implementing `K_eff=K/cosθ` and `R_i=1/(2K_eff)`;
- wrote paraxial estimate CSV;
- plotted the black dashed `x`-marker paraxial estimate only in the `band1` focus-position panel;
- moved the combined legend to the figure bottom for readability;
- left the Emax panel and higher-order panels without paraxial theory curves.

## Outputs

Remote outputs regenerated:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_summary_plots/order_reflection_xEmax_vs_K_ND_scan_a0_20.png
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_summary_plots/order_reflection_xEmax_vs_K_ND_scan_a0_20.pdf
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_summary_plots/order_reflection_band1_paraxial_focus_vs_K_a0_20.csv
```

Local mirror for Telegram delivery / future access:

```text
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/curved_surface_paraxial_focus_20260717/order_reflection_xEmax_vs_K_ND_scan_a0_20.png
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/curved_surface_paraxial_focus_20260717/order_reflection_xEmax_vs_K_ND_scan_a0_20.pdf
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/curved_surface_paraxial_focus_20260717/order_reflection_band1_paraxial_focus_vs_K_a0_20.csv
```

Paraxial CSV values:

```text
K       K_eff=K/cos45   R_i=lambda0       z0_focus(lambda0)   w0_focus(lambda0)
-0.008  -0.0113137      -44.1942          43.8388             1.1209
-0.005  -0.0070711      -70.7107          69.2732             1.7822
-0.002  -0.0028284      -176.7767         156.4823            4.2353
```

These focus-distance numbers are close to the band1 focus trend, especially for `K=-0.005` and `K=-0.008`; deviations are expected because the actual reflected fundamental source is not a perfect paraxial Gaussian and the plotted PIC focus comes from propagated fields with finite aperture and plasma-source effects.

## Validation

- Plot script rerun completed successfully and printed `rows 36`, `bands band1,band2,band3`, `ND 0.30,0.50,1.00`.
- Local visual check with the vision tool: the black dashed paraxial curve appears only in the 1st-order panel; bottom/global legend is readable; no major plotting issue.

## Reporting

Completion and formula explanation were reported to Zhiping on Telegram after generating artifacts.
