# Curved_surface tradeoff objective — 2026-07-13

Source messages:
- Zhiping Telegram `main:6420513923:233`
- main reply `main:6420513923:234`

## Zhiping's stated objective

Zhiping emphasized that the project should not simply maximize the local focal field by making the curvature as large as possible. Larger curvature makes the focus closer to the target, shrinks the focal spot, and raises the local focal field, but a focus too close to the target is difficult to measure and use experimentally. Experiments often value a low-divergence X/XUV wave that can be diagnosed and transported. The useful design target is therefore a balance:

```text
strong enough E_peak at focus
+ focus not too close to target
+ transverse width / divergence acceptable
+ high-order energy and wavefront quality still good
```

This reframes the curvature scan as a tradeoff / Pareto-front problem, not a single-objective `max(E_peak)` problem.

## Mechanism interpretation

Curvature imposes an initial converging wavefront on the reflected/harmonic radiation. More negative curvature produces stronger geometric focusing, so the caustic moves closer to the foil, the focal spot becomes smaller, and the local `E_peak` increases. But near-field caustics close to the plasma surface are experimentally hard to access and may correspond to large angular spread rather than a low-divergence useful X-wave. A high `E_peak` at very small `x_focus` is therefore not automatically the best operating point.

The analysis should treat `E_peak`, `x_focus`, and `width_y`/divergence as simultaneous objectives/constraints. `E_peak` alone biases toward `K=-0.008` near-target focusing; adding an accessibility constraint shifts candidates toward intermediate curvature.

## Quick numerical read from current order-focus summary

Data source:

```text
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/curved_surface_order_scan_20260712/order_focus_per_band_argmax_summary_latest.tsv
```

Subset inspected: `side=reflection`, `ND/a0=0.30,0.50,1.00`, bands `band1`, `band2`, `band3`; each point is a 21-point sampled argmax from the coarse `center±100T0`, `10T0` scan, not a continuous optimum.

Representative threshold choices:

| Constraint | Band | Candidate | Coarse sampled result |
|---|---|---|---|
| `x_focus >= 50 λ0` | `band3` | `K=-0.005, ND/a0=0.30` | `E≈117.12 Ec`, `x≈57.33 λ0`, `width_y≈0.64 λ0` |
| `x_focus >= 50 λ0` | `band2` | `K=-0.005, ND/a0=0.30` | `E≈23.56 Ec`, `x≈57.80 λ0`, `width_y≈2.47 λ0` |
| `x_focus >= 50 λ0` | `band1` | `K=-0.005, ND/a0=1.00` | `E≈46.11 Ec`, `x≈65.79 λ0`, `width_y≈3.99 λ0` |
| `x_focus >= 100 λ0` | `band3` | `K=-0.002, ND/a0=0.30` | `E≈89.87 Ec`, `x≈118.33 λ0`, `width_y≈1.22 λ0` |
| `x_focus >= 100 λ0` | `band2` | `K=-0.002, ND/a0=0.30` | `E≈16.97 Ec`, `x≈128.76 λ0`, `width_y≈4.83 λ0` |
| `x_focus >= 100 λ0` | `band1` | `K=-0.002, ND/a0=1.00` | `E≈32.42 Ec`, `x≈153.79 λ0`, `width_y≈8.19 λ0` |

Interpretation:
- `K=-0.008` produces the strongest near-target focusing around `x≈38–40 λ0`; it is physically expected to look impressive in `E_peak` but may be too close experimentally.
- `K=-0.005` is the first strong-but-not-immediately-on-target compromise: high-order focus around `x≈57 λ0`, low-order around `x≈58–66 λ0`.
- `K=-0.002` is the more conservative, farther-focus option: high-order around `x≈118 λ0`, low-order around `x≈129–154 λ0`, at lower field.
- Flat `K=0` cases are generally farther but weaker and broader, so they are not competitive unless the experiment strongly prioritizes far-field accessibility over focal field.

## Recommended next diagnostic

Make the optimization explicit by plotting Pareto/tradeoff panels:

```text
x-axis: x_focus / λ0
 y-axis: E_peak / Ec
 marker color or size: width_y / λ0, or estimated divergence
 annotations: K and ND/a0
 panels: band1, band2, band3 (and optionally total)
```

Then choose candidate curvature by applying experimental accessibility constraints such as `x_min = 50, 80, 100 λ0` and reading the Pareto front, instead of ranking by `E_peak` alone. If later data provide far-field angular spectra or propagated beam divergence, replace `width_y` proxy with a direct divergence metric.