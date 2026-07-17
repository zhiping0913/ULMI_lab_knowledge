# Curved_surface a0=50/2D ND_a0=0.30 clip completion and focus-scan launch

Date: 2026-07-17 EDT

## Human request

Zhiping asked to submit every row of:

`/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=50/2D/propagate_and_clip.tsv`

using:

`/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_and_clip.slurm`

with `--mem=400G`, and then after clips completed, run the focal-position scan through:

`/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan.slurm`

calling:

`/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan.py`

with center `50T0`, half-width `100T0`, step `5T0`.

## Clip submission and validation

All Slurm submissions were preceded by `checkquota`. Tiger `/scratch/gpfs/MIKHAILOVA` remained about `5.50 TiB` free, above the 2 TiB safety threshold.

Clip logdir:

`/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_and_clip_logs/a0_50_2D_ND030_20260716_220905`

Manifest:

`/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_and_clip_logs/a0_50_2D_ND030_20260716_220905/manifest.tsv`

Retry manifest:

`/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_and_clip_logs/a0_50_2D_ND030_20260716_220905/retry_manifest.tsv`

Final validated clip outputs:

| row | K | final job | status | output |
|---:|---:|---:|---|---|
| 1 | -0.002 | 3525766 | completed | `K=-0.002,ND_a0=0.30,L=0.00/reflection_clip.nc` |
| 2 | -0.005 | 3525787 | completed retry | `K=-0.005,ND_a0=0.30,L=0.00/reflection_clip.nc` |
| 3 | -0.008 | 3525768 | completed | `K=-0.008,ND_a0=0.30,L=0.00/reflection_clip.nc` |
| 4 | -0.010 | 3526136 | completed retry | `K=-0.010,ND_a0=0.30,L=0.00/reflection_clip.nc` |
| 5 | -0.012 | 3525770 | completed | `K=-0.012,ND_a0=0.30,L=0.00/reflection_clip.nc` |
| 6 | +0.000 | 3526900 | completed corrected retry | `K=+0.000,ND_a0=0.30,L=0.00/reflection_clip.nc` |

Validation: all six `reflection_clip.nc` files are present, openable with xarray, dimensions `x=4000`, `y=6000`, variables `Ex,Ey,Bz`; row6 size about 535 MB.

## Failure/retry notes

- Row2 original `3525767` failed in ~5 s with JAX port-add failure and segfault. Retried as `3525787`; completed.
- Row4 original `3525769` failed after ~49 min with Gloo `process_allgather` timeout. Retried as `3526136`; completed.
- Row6 original `3525771` and retry `3526137` both failed after ~48 min with Gloo `process_allgather` timeout.
- Row6 single-task retry `3526891` (`--nodes=1 --ntasks=1 --mem=400G`) failed immediately because `EM_analyzer/Spectral_Maxwell/Normal_variable_method.py` expects `np.array(devices).reshape(2,3)` but one task exposes only 3 CPU devices.
- Row6 corrected retry `3526900` used `--nodes=1 --ntasks=2 --ntasks-per-node=2 --mem=400G`. This preserves 6 CPU devices / the `2×3` mesh while avoiding inter-node Gloo. It completed in ~4.6 min and produced valid row6 output.

## Focus scan launch

After all six clips validated, focus scans were submitted at 2026-07-17 01:17 EDT.

Focus scan logdir:

`/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_50_2D_focus_center50_hw100_step5_20260717_011743`

Manifest:

`/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_50_2D_focus_center50_hw100_step5_20260717_011743/manifest.tsv`

Parameters:

- `side=reflection`
- `propagation_time=50`
- `time_scan_half_width_T0=100`
- `time_scan_num_points=41` (step `5T0`)
- Output subdir under each K directory: `focus_scan_center50_hw100_step5`
- Slurm script has `--time=2:00:00`, `--mem=400G`, `--nodes=1`, `--ntasks=1`, `--cpus-per-task=6`.

Jobs:

| row | K | job | initial status at 01:17 EDT | output_dir |
|---:|---:|---:|---|---|
| 1 | -0.002 | 3527413 | RUNNING | `K=-0.002,ND_a0=0.30,L=0.00/focus_scan_center50_hw100_step5` |
| 2 | -0.005 | 3527414 | RUNNING | `K=-0.005,ND_a0=0.30,L=0.00/focus_scan_center50_hw100_step5` |
| 3 | -0.008 | 3527415 | RUNNING | `K=-0.008,ND_a0=0.30,L=0.00/focus_scan_center50_hw100_step5` |
| 4 | -0.010 | 3527416 | PENDING (Priority) | `K=-0.010,ND_a0=0.30,L=0.00/focus_scan_center50_hw100_step5` |
| 5 | -0.012 | 3527417 | PENDING (Priority) | `K=-0.012,ND_a0=0.30,L=0.00/focus_scan_center50_hw100_step5` |
| 6 | +0.000 | 3527418 | PENDING (Priority) | `K=+0.000,ND_a0=0.30,L=0.00/focus_scan_center50_hw100_step5` |

## Next action

A self-reminder is scheduled for ~2026-07-17 02:17 EDT with subject:

`Self-reminder: check a0=50 focus scan jobs 3527413-3527418`

At reminder:

1. SSH to `tiger-vis` with short timeout.
2. Run `squeue`/`sacct` for `3527413–3527418`.
3. If still pending/running, reschedule quietly unless Zhiping asks.
4. If failed, inspect `.err` and report.
5. If completed, validate expected outputs in each `focus_scan_center50_hw100_step5` output dir:
   - `reflection_beam_width_strength.h5`
   - `reflection_peak_vs_x.nc`
   - per-band `reflection_env_*_peak_vs_x.nc`
   - summary PNGs
6. Summarize focus positions / peak amplification per K, update this knowledge entry or supporting note, and report to Zhiping.

## Focus scan completion, retry, and first summary

Update: 2026-07-17 05:08 EDT

The original focus-scan batch did not all finish cleanly, but the missing rows were recovered with a limited retry.

### Original focus batch final status

| K | original job | final Slurm state | useful outputs? | note |
|---:|---:|---|---|---|
| -0.002 | 3527413 | FAILED 134 | no complete output | JAX/XLA CPU `all reduce Rendezvous` timeout after FFT/transversality setup; only 1--2 of 3 expected local participants arrived. |
| -0.005 | 3527414 | FAILED 134 | no complete output | Same all-reduce rendezvous abort. |
| -0.008 | 3527415 | FAILED 1 | yes | Completed all 41 time points and wrote HDF/NetCDF/summary PNG, then failed in final per-band plot due `colors_bands` length bug. |
| -0.010 | 3527416 | FAILED 1 | yes | Same final plotting bug after writing numeric outputs. |
| -0.012 | 3527417 | FAILED 134 | no complete output | Same all-reduce rendezvous abort. |
| +0.000 | 3527418 | FAILED 1 | yes | Same final plotting bug after writing numeric outputs. |

The late plotting bug was traced to `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan.py`: `K_BANDS` contains five entries (`total + band1..4`), but `colors_bands` had only four colors, causing `IndexError: list index out of range` at line 639. I backed up the script to:

`/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan.py.bak_20260717_0309_focus_colors`

and patched `colors_bands` to five colors. This was a plotting-only fix; the scientific arrays already written by K=-0.008/-0.010/+0.000 were usable.

### Retry recovery for missing rows

Before retry submission, `checkquota` showed Tiger MIKHAILOVA scratch used by all `9.5 TiB / 15 TiB`, i.e. about `5.5 TiB` free, above the 2 TiB safety threshold.

Retry logdir:

`/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_50_2D_focus_center50_hw100_step5_retry1_20260717_0309`

Retry jobs:

| K | retry job | state | elapsed | MaxRSS |
|---:|---:|---|---:|---:|
| -0.002 | 3528417 | COMPLETED | 01:07:26 | 214910748K |
| -0.005 | 3528418 | COMPLETED | 01:48:02 | 221802232K |
| -0.012 | 3528419 | COMPLETED | 01:05:25 | 226012976K |

So all six `focus_scan_center50_hw100_step5` output directories now contain the expected core products.

### Combined summary artifacts

Remote combined summary directory:

`/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/a0_50_focus_center50_hw100_step5_20260717`

Local artifact mirror:

`artifacts/a0_50_focus_scan_20260717/`

Files:

- `a0_50_focus_center50_hw100_step5_summary.tsv`
- `a0_50_focus_center50_hw100_step5_summary.png`
- `a0_50_focus_center50_hw100_step5_summary.pdf`
- `a0_50_focus_center50_hw100_step5_summary.ipynb`

The notebook reads the TSV and reproduces the combined plot. The per-K output directories still contain the full HDF/NetCDF and per-time PNG outputs.

### First-pass focus metrics

All metrics below are for reflection, center `50T0`, half-width `100T0`, 41 points (step `5T0`). `onaxis_peak_Ec` comes from `reflection_peak_vs_x.nc` / HDF centerline envelope diagnostic. `env_total_Epeak_max_Ec` is the 2D total-envelope peak from `reflection_env_total_peak_vs_x.nc`. `band4_Epeak_max_Ec` uses the script's current band4 definition (note: code currently sets band4 lower bound to `2.5 k0`, while its label says `[3.5,50.5]k0`; treat this as the script's high-order/third-plus band until that label/definition is reconciled).

| K | job used | onaxis_peak_Ec | time_T0_at_onaxis_peak | x_lambda_at_onaxis_peak | width_lambda_at_onaxis_peak | env_total_Epeak_max_Ec | env_total_x_lambda_at_max | band4_Epeak_max_Ec | min_width_lambda |
|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| -0.012 | 3528419 | 305.329 | -25.0 | 18.5475 | 0.3835 | 313.672 | 18.5425 | 233.348 | 0.2121 |
| -0.010 | 3527416 | 303.336 | -30.0 | 18.5375 | 0.2979 | 341.730 | 18.5325 | 267.920 | 0.2930 |
| -0.008 | 3527415 | 272.658 | -30.0 | 23.5525 | 0.4178 | 293.076 | 23.5475 | 218.923 | 0.3470 |
| -0.005 | 3528418 | 232.137 | -45.0 | 28.5525 | 0.4912 | 275.115 | 23.5325 | 212.003 | 0.3099 |
| -0.002 | 3528417 | 196.544 | -10.0 | 33.5525 | 0.7000 | 241.591 | 33.5375 | 184.263 | 0.4861 |
| +0.000 | 3527418 | 226.768 | -10.0 | 33.5525 | 0.7342 | 281.011 | 38.5475 | 228.192 | 0.2272 |

### Interpretation / caveats

- Within this post-propagation diagnostic, stronger negative curvature shifts the dominant focus earlier in x: roughly `33.5 λ0` for K=-0.002/+0.000, `28.6 λ0` for K=-0.005, `23.6 λ0` for K=-0.008, and `18.5 λ0` for K=-0.010/-0.012.
- Peak strength is not monotonic in every diagnostic. The centerline peak is highest for K=-0.012 (`~305 Ec`) but nearly tied with K=-0.010 (`~303 Ec`); the 2D total-envelope peak and script band4 peak are strongest for K=-0.010 (`~342 Ec` total envelope, `~268 Ec` band4). This suggests K=-0.010 is currently the cleanest high-strength focusing candidate, while K=-0.012 may give comparable on-axis peak with a slightly different transverse/2D envelope structure.
- This is not a new PIC simulation; it is vacuum/spectral propagation of clipped reflection fields. Interpret it as a focus-location/beam-strength diagnostic conditional on the clipped field quality and the script's envelope/band definitions.
- Three original early failures were JAX/XLA CPU collective aborts, not memory exhaustion (MaxRSS about 225 GB under a 400G request). The same rows succeeded on retry after the plotting-color patch; this points to a transient/stochastic JAX CPU collective issue rather than a deterministic physics/data failure, but it should be remembered if similar 6-device CPU sharded propagation jobs abort early.
