# a0=50 ND/a0=0.30 reflection focus-at-y spectrum gain and estimated 3D Emax(K) — 2026-07-19

## Trigger

Zhiping asked on Telegram (`main:6420513923:460`) to repeat the a0=20 focus-at-y / spectrum_1D / 3D-estimate workflow for `a0=50`, `ND/a0=0.30`, `reflection` side. The fine on-axis scan `focus_scan_fine_onaxis_tm10_hw20_step1` family was already complete; he asked to:

1. write `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=50/2D/focus.tsv` analogous to `a0=20/2D/focus.tsv`;
2. run `propagate_2D_at_time.slurm` to generate each K case's `reflection_focus.nc`;
3. slice `y=0` using `slice_2D_at_y.py`;
4. run `spectrum_1D.py` to obtain per-order gain;
5. estimate the 3D focus field and plot estimated 3D `E_max(K)` analogous to `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/a0_50_reflection_fine_hw20_step1_20260718/a0_50_reflection_fine_hw20_step1_summary.ipynb`.

## Inputs

- Fine-scan summary:
  `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/a0_50_reflection_fine_hw20_step1_20260718/a0_50_reflection_fine_hw20_step1_summary.tsv`
- Fine-scan per-case HDF files:
  `.../focus_scan_fine_onaxis_tm*_hw20_step1/reflection_beam_width_strength.h5`
- a0=50 2D case root:
  `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=50/2D/`
- 1D reference:
  `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=50/1D/45/ND_a0=0.30/reflection.nc`

The 1D reference spectrum was rerun with `LASER_A0=50`, `THETA_DEGREE=45`, `SIDE=reflection`, `NC_NAME=reflection.nc`. This matters because `spectrum_1D.py` defaults `LASER_A0=20`, which would make `envelope_at_peak_over_a0EcM` inconsistent for a0=50 if reused blindly. 2D focus-at-y spectra were run with `LASER_A0=50`, `THETA_DEGREE=0`, `SIDE=reflection_focus_at_y`, `NC_NAME=reflection_focus_at_y.nc`.

## focus.tsv generation

Generated:
`/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=50/2D/focus.tsv`

Schema matches the a0=20 file:

```text
working_dir (/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=50/2D)	side	propagation_time (laser_period)	E_max(Ec)	x_at_E_max (lambda0)
```

Values were computed from each fine-scan `reflection_beam_width_strength.h5` by taking the index of `nanargmax(peak_amp_per_t_over_Ec)`, then reading `time_over_T0`, `peak_amp_per_t_over_Ec`, and `x_at_peak_per_t_over_lambda` at the same index. These values were checked against the fine summary `E_peak_max` and `x_at_E_peak_max`.

Rows:

```text
K=-0.012  T=-26T0  E2D=329.107 Ec  x=17.542 lambda0
K=-0.010  T=-29T0  E2D=307.297 Ec  x=19.542 lambda0
K=-0.008  T=-31T0  E2D=285.794 Ec  x=22.547 lambda0
K=-0.005  T=-47T0  E2D=236.269 Ec  x=26.547 lambda0
K=-0.002  T=-8T0   E2D=197.268 Ec  x=35.553 lambda0
K=+0.000  T=-8T0   E2D=227.555 Ec  x=35.553 lambda0
```

Helper script used:
- local copy: `artifacts/a0_50_focus_workflow_20260719/prepare_submit_a0_50_focus.py`
- remote copy: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/prepare_submit_a0_50_focus_20260719.py`

## Slurm focus-at-time generation

Pre-submission quota checks were performed before sbatch:

- First batch: `checkquota` showed MIKHAILOVA scratch `9.3/15 TiB`, free `~5.7 TiB` > 2 TiB threshold.
- K=0 resubmit: `checkquota` showed `9.4/15 TiB`, free `~5.6 TiB` > 2 TiB threshold.

Resource choice: `300G`, `1:00:00`, `1 node`, `1 task`, `6 cpus-per-task`, lower than the previous a0=20 default because the a0=50 focus fields are smaller and prior a0=50 time-scan MaxRSS was around 215–244 GB. This was enough: all jobs completed with MaxRSS about `244 GB`.

Log/manifest directory:
`/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_at_time_logs/a0_50_2D_reflection_focus_fields_20260719_115701/`

Jobs:

```text
K=-0.012  job 3547613  COMPLETED  00:02:36  MaxRSS~243,938,860K
K=-0.010  job 3547614  COMPLETED  00:02:39  MaxRSS~243,926,760K
K=-0.008  job 3547615  COMPLETED  00:02:39  MaxRSS~243,665,880K
K=-0.005  job 3547616  COMPLETED  00:02:37  MaxRSS~243,679,980K
K=-0.002  job 3547617  COMPLETED  00:02:25  MaxRSS~243,671,972K
K=+0.000  job 3547627  COMPLETED  00:02:34  MaxRSS~243,676,336K
```

The first K=0 submit attempt failed with `QOSMaxSubmitJobPerUserLimit`; after the first five jobs completed, it was submitted successfully as job `3547627`.

Validation: all six case directories contain nonempty `reflection_focus.nc` (~536–539 MB each) and `reflection_focus_Bz.png`.

## y=0 slicing and spectrum_1D.py

Remote script:
`/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/run_a0_50_focus_slice_spectrum_20260719.py`

Local copy:
`artifacts/a0_50_focus_workflow_20260719/run_a0_50_focus_slice_spectrum.py`

Log/metadata directory:
`/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_at_y_spectrum_logs/a0_50_focus_at_y_spectrum_20260719_121636/`

Metadata:
`/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_at_y_spectrum_logs/a0_50_focus_at_y_spectrum_20260719_121636/focus_at_y_spectrum_metadata.tsv`

All return codes were zero:

- 1D reference: `reflection_per_harmonic_at_peak.tsv`, `reflection_spectrum.nc` refreshed with `LASER_A0=50`, `THETA=45`.
- 2D focus cases: for all six K, `reflection_focus_at_y.nc`, `reflection_focus_at_y_per_harmonic_at_peak.tsv`, and `reflection_focus_at_y_spectrum.nc` were generated successfully.

## Harmonic gain / phase results

Analysis output directory:
`/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/a0_50_reflection_gain_3d_estimate_20260719/`

Local artifacts copied to:
`artifacts/a0_50_reflection_gain_3d_estimate_20260719/`

Main files:

- `a0_50_ND_a0_0p30_reflection_spectrum1D_gain_phase_n01_30_ref_1D.png/.pdf`
- `a0_50_ND_a0_0p30_reflection_spectrum1D_gain_phase_n01_30_ref_1D.tsv`
- `a0_50_ND_a0_0p30_reflection_spectrum1D_phase_gain_band_stats.tsv`
- `a0_50_reflection_estimated_3D_Emax_vs_K_locked_n01_20_alpha0p5.png/.pdf/.tsv`
- `plot_a0_50_reflection_gain_3d_estimate.ipynb`
- `plot_a0_50_reflection_gain_3d_estimate.py`

The harmonic gain definition is the same canonical `spectrum_1D.py` normalization as the a0=20 analysis:

```text
G_2D,E(n) = A_2D(n) / A_1D(n),
A_n = envelope_at_peak_over_a0EcM.
```

Band/gain summary:

```text
K       mean G(1-10)  mean G(11-20)  mean G(21-30)  G_locked(n=1-20, amp-weighted)
-0.012      3.60          2.51           0.93              3.19
-0.010      3.41          2.29           0.89              3.01
-0.008      3.13          1.96           0.90              2.75
-0.005      2.77          1.31           1.34              2.40
-0.002      2.32          0.96           0.99              2.01
+0.000      2.38          1.48           1.39              2.09
```

Phase locking over `n=5–15` using direct `phase_at_peak`:

```text
K       circular std / pi
-0.012      0.0348
-0.010      0.0391
-0.008      0.0462
-0.005      0.1176
-0.002      0.0972
+0.000      0.0391
1D ref      0.0718
```

Interpretation: for a0=50, the strongest negative curvatures (`K=-0.012,-0.010,-0.008`) have both large harmonic gain and good phase clustering over the main band. `K=-0.005/-0.002` are looser in this metric. This differs from a0=20, where the best conservative E3D estimates sat around `K=-0.005/-0.008`; for a0=50 the optimum shifts toward stronger negative curvature in the available scan.

## Estimated 3D Emax(K)

Locked band: `n=1–20`.

`G_locked` is the 2D amplitude-weighted mean of `G_2D,E(n)` over the locked band, with weights equal to the 2D focal per-order amplitude.

Conservative model:

```text
E3D_conservative = E2D * sqrt(G_locked)
```

Ideal upper model:

```text
E3D_ideal = E2D * G_locked
```

Results:

```text
K       E2D(Ec)  G_locked  E3D_conservative(Ec)  E3D_ideal_upper(Ec)
-0.012   329.1     3.19             588                  1050
-0.010   307.3     3.01             533                   926
-0.008   285.8     2.75             474                   786
-0.005   236.3     2.40             366                   567
-0.002   197.3     2.01             280                   396
+0.000   227.6     2.09             329                   476
```

Physical reading: a0=50 reflection focus is already strongest in 2D for the more negative curvatures, and the locked-band gain also grows toward more negative K. The conservative 3D estimate therefore peaks at `K=-0.012` (`~5.9e2 Ec`) and remains high at `K=-0.010` (`~5.3e2 Ec`). The ideal upper curve reaches `~0.9–1.1e3 Ec`, but this is explicitly an upper bound, not a 3D PIC result; it omits 3D aberration, finite aperture, and coherence/Strehl penalties.

## Telegram report

Completion report sent to Zhiping in Telegram `main:6420513923:463`, followed by attachments `main:6420513923:464–471`:

- estimated 3D Emax PNG/PDF;
- harmonic gain/phase PNG/PDF;
- estimated 3D Emax TSV;
- full per-harmonic gain/phase TSV;
- band stats TSV;
- editable notebook.

## Cross-a0 normalized-efficiency interpretation

After the report, Zhiping observed in Telegram `main:6420513923:472` that the a0=20 harmonic focusing gain `A2D/A1D` (order-10-ish at the strongest harmonics) is larger than the a0=50 gain (few-fold), so the estimated 3D a0=20 field can be comparable to the a0=50 result even though the absolute peak is lower. I agreed in `main:6420513923:473` and suggested the following quantitative factorization:

```text
a0=20, ND/a0=0.30, best K≈-0.005:
E2D≈156.9 Ec, G_locked≈4.43, sqrt(G)≈2.10
E3D_cons≈330 Ec, E3D/a0≈16.5

a0=50, ND/a0=0.30, best K≈-0.012:
E2D≈329.1 Ec, G_locked≈3.19, sqrt(G)≈1.79
E3D_cons≈588 Ec, E3D/a0≈11.8
```

Thus the a0=50 absolute conservative estimate is still higher by `~588/330≈1.8`, but the normalized field `E3D/a0` is higher for a0=20. In intensity-like normalized units `(E3D/a0)^2`, a0=20 is about `~272` while a0=50 is about `~138`, roughly a factor of two higher normalized coherent-focusing efficiency.

Physical interpretation: a0=50 provides a larger local source field, but stronger radiation pressure / surface deformation / transverse electron motion can degrade harmonic wavefront quality, divergence, and phase coherence, reducing `G_locked` or the Strehl-like focusing factor. a0=20 near `ND/a0≈0.30` appears closer to a clean coherent harmonic source plus focusable wavefront window. A useful future figure would compare `E2D/a0`, `sqrt(G_locked)`, and `E3D/a0` or `(E3D/a0)^2` versus `a0` and `K`.

## Follow-up / caveats

- This is an estimate, not a 3D PIC result. Keep the statement as `E3D ≈ E2D * sqrt(G_locked)` conservative extrapolation with the ideal `E2D * G_locked` only as an upper bound.
- The 2D-to-3D multiplier uses only the locked `n=1–20` band, analogous to the a0=20 result and avoiding the higher-order resolution caveat.
- The `K=+0.000` case has larger 2D `E2D` than `K=-0.002` in this fine scan, so its conservative E3D estimate is also larger than `K=-0.002`; do not overinterpret this as curvature focusing, because the flat case's mechanism differs.
- If Zhiping asks for refinement, likely next steps are: vary the conservative exponent/penalty, compare against any future 3D PIC, or analyze how the optimal K shifts with a0 and ND/a0.
