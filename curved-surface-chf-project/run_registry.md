# Curved_surface run / dataset registry

Use this file as the durable inventory of simulations, summary files, and diagnostic datasets for the Curved_surface / ultrathin-foil CHF project.

Update rule: every time Zhiping reports a simulation batch, add rows with paths, parameters, status, outputs, and caveats. Keep raw facts separate from interpretation.

## Table schema

| run_id | path | dimension | geometry / case | key parameters | status | summary / outputs | supports factor | caveats / notes |
|---|---|---|---|---|---|---|---|---|

## Inspection artifacts

Read-only a0=20/2D inspection captured locally at:

```text
artifacts/curved_surface_2d_a020_inspection_20260706_204001/
```

Important files:

- `inspection_report.md` — remote directory listing, `squeue`, and `input.deck` excerpts.
- `case_summary.md` / `case_summary.csv` — parsed compact case table.
- `start_2D.py`, `rotate_and_shift.py`, `propagate_and_clip.slurm` — copied key workflow files.
- `input_decks/` — copied `input.deck` files for 17 K,D,L case directories.

## Known 1D datasets as of 2026-07-06

| run_id | path | dimension | geometry / case | key parameters | status | summary / outputs | supports factor | caveats / notes |
|---|---|---|---|---|---|---|---|---|
| `1D-L-a0-all-thick-45` | `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=*/1D/45,thick/reflection.nc` | 1D summaries | L scan, thick/45 case across a0 | `a0 = 3,10,20,50,100,150,200,250,400`; scan key `L` | summaries exist and are plotted | `utility/reflection_1D_scan_plots/`; exact output names in `curved-surface-reflection-scan-plots` knowledge | `G_CSE/1D` / local normalized reflected gain | Need load detailed plotting knowledge for exact numeric values and files. |
| `1D-ND_a0-a0-all-45` | `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=*/1D/45/reflection.nc` | 1D summaries | all-a0 comparison of existing ND/a0 scans | `a0 = 3,10,20,50,100,150,200,250,400`; scan key `ND_a0 = ND/a0` | corrected all-a0 plots generated | `utility/reflection_ND_a0_scan_plots/reflection_efficiency_high_3p5_1000_vs_ND_a0_all_a0.png`; `reflection_Sx_peak_over_incident_vs_ND_a0_all_a0.png`; `reflection_1D_ND_a0_scan_summary.csv` | `G_CSE/1D` / local normalized reflected gain | `a0=200` has only four ND/a0 points (`0.05..0.2`); most others have 30 points to `1.5`; union-grid alignment with NaNs, no interpolation. |

## 2D a0=20 EPOCH cases — inspected 2026-07-06/07

Remote base:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D
```

Context from Zhiping: these are EPOCH 2D simulations of 45° incidence on different ultrathin-foil cases, producing reflection and transmission. They were generated from `/scratch/gpfs/MIKHAILOVA/zl8336/EPOCH_toolkit/start_2D.py`. `rotate_and_shift.py` has already been used to obtain reflection/transmission fields; `propagate_and_clip.slurm` has not yet been started and should wait for Zhiping's instructions.

Read-only inspection found 17 copied `input.deck` case directories plus non-case helper/legacy directories. `squeue -u zl8336` showed one running job:

```text
3203372 cpu test RUNNING 3-13:41:23 tiger-i02c14n1,tiger-i03c8n4,tiger-i03c11n1
```

This corresponds to the thick-reference-like case `K=-0.002,D=1.95,L=0.05` that Zhiping said is still running.

### Symbol notes from `input.deck` / `start_2D.py`

- `K` / `Kappa`: target curvature in units `1/λ0`; deck uses `target_curvature = Kappa/laser_lambda0`. For nonzero `Kappa`, `R=1/Kappa`, `target_radius=R*λ0`, and `target_f=sqrt((x-target_radius)^2+y^2)-abs(target_radius)`.
- `D`: target thickness in units of `λ0`; deck uses `target_thickness = D*λ0`.
- `L`: density-gradient scale length in units of `λ0`; deck uses `target_Ne_gradient=L*λ0`. For `L>0`, electron density has an exponential ramp in `target_f`; for `L=0`, it is a slab around `target_f=0`.
- `N`: electron density in critical-density units; here generally `N=350`, so `target_Ne=N*laser_Nc`.
- `laser_a0=20`, `λ0=0.8 μm`, `laser_W0=12.5 λ0`, `laser_FWHM=8 fs`.
- Current `start_2D.py` uses file-based initial fields (`fields.ex`, `fields.ey`, `fields.bz` from `../Initialize_Field/...`) and comments out the analytic `laser` block; the 45° incidence is therefore encoded in the prepared field/geometry workflow rather than as an active `theta` constant in the deck.
- Output is a final snapshot (`dt_snapshot=t_end`) of grid, `Ex`, `Ey`, `Bz`, and species number density.
- Caution: directory label `D=...` is not always identical to the `D` value inside `input.deck`. For physical thickness, use `input.deck`'s `D` and `target_thickness`, not only the directory name.

### Parsed 2D case table

| case | deck Kappa | deck D/λ0 | deck L/λ0 | target W/λ0 | x window | ppc | outputs/status |
|---|---:|---:|---:|---:|---|---:|---|
| `K=+0.000,D=0.01,L=0.00` | 0.0 | 0.008571428571428572 | 0 | 90 | about ±50λ0 in older deck | 100 | processed; reflection/transmission `.nc` exist |
| `K=+0.000,D=0.02,L=0.00` | 0.0 | 0.017142857142857144 | 0.0 | older deck field | about ±50λ0 in older deck | 100 | processed; reflection/transmission `.nc` exist |
| `K=+0.000,D=0.03,L=0.00` | 0.0 | 0.02857142857142857 | 0.0 | older deck field | about ±50λ0 in older deck | 100 | processed; reflection/transmission `.nc` exist |
| `K=+0.000,D=0.06,L=0.00` | +0.000 | 0.05714285714285714 | 0.0 | older deck field | about ±50λ0 in older deck | 100 | processed; reflection/transmission `.nc` exist |
| `K=-0.002,D=0.01,L=0.00` | -0.002 | 0.005714285714285714 | 0.0 | 90 | about ±50λ0 | 100 | processed; reflection/transmission `.nc` exist; some clip outputs already present |
| `K=-0.002,D=0.02,L=0.00` | -0.002 | 0.017142857142857144 | 0.0 | 90 | about ±50λ0 | 100 | processed; reflection/transmission `.nc` exist |
| `K=-0.002,D=0.03,L=0.00` | -0.002 | 0.02857142857142857 | 0.0 | 90 | about ±50λ0 | 100 | processed; reflection/transmission `.nc` exist |
| `K=-0.002,D=0.06,L=0.00` | -0.002 | 0.05714285714285714 | 0.0 | 90 | about ±50λ0 | 100 | processed; reflection/transmission `.nc` exist |
| `K=-0.002,D=1.95,L=0.05` | -0.002 | 1.95 | 0.05 | 70 | `x=-50..10λ0`, `y=-50..50λ0` | 30 | still running; only `fields0000.sdf`, `restart0000.sdf`; no reflection/transmission `.nc` yet |
| `K=-0.005,D=0.01,L=0.00` | -0.005 | 0.005714285714285714 | 0 | 90 | about ±50λ0 | 100 | processed; reflection/transmission `.nc` exist |
| `K=-0.005,D=0.02,L=0.00` | -0.005 | 0.017142857142857144 | 0 | 90 | about ±50λ0 | 100 | processed; reflection/transmission `.nc` exist; clip outputs already present |
| `K=-0.005,D=0.03,L=0.00` | -0.005 | 0.02857142857142857 | 0 | 90 | about ±50λ0 | 100 | processed; reflection/transmission `.nc` exist |
| `K=-0.005,D=0.06,L=0.00` | -0.005 | 0.05714285714285714 | 0 | 90 | about ±50λ0 | 100 | processed; reflection/transmission `.nc` exist |
| `K=-0.008,D=0.01,L=0.00` | -0.008 | 0.005714285714285714 | 0 | 90 | about ±50λ0 | 100 | processed; reflection/transmission `.nc` exist |
| `K=-0.008,D=0.02,L=0.00` | -0.008 | 0.017142857142857144 | 0 | 90 | about ±50λ0 | 100 | processed; reflection/transmission `.nc` exist |
| `K=-0.008,D=0.03,L=0.00` | -0.008 | 0.02857142857142857 | 0 | 90 | about ±50λ0 | 100 | processed; reflection/transmission `.nc` exist |
| `K=-0.008,D=0.06,L=0.00` | -0.008 | 0.05714285714285714 | 0 | 90 | about ±50λ0 | 100 | processed; reflection/transmission `.nc` exist |

## 2D a0=20 spectrum diagnostics — generated 2026-07-07

| run_id | path | dimension | geometry / case | key parameters | status | summary / outputs | supports factor | caveats / notes |
|---|---|---|---|---|---|---|---|---|
| `2D-a0-20-clipped-spectra` | `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/K=*,D=*,L=*/{reflection,transmission}_spectrum_*` | 2D post-processed spectra | finished thin K,D,L cases after rotate/shift and propagate/clip | `a0=20`; `K={0,-0.002,-0.005,-0.008}`; `D={0.01,0.02,0.03,0.06}` directory labels; `L=0`; sides reflection/transmission; 16 cases × 2 sides = 32 inputs | processed / validated | each side has `*_spectrum_kxky.png`, `*_spectrum_kr.nc/.png`, `*_spectrum_ktheta.nc/.png`; manifest `utility/spectrum_2d_logs/a0_20_2D_20260707_221434/manifest.tsv` shows `OK 32`; detailed record `spectrum_2d_batch_2026-07-07.md` | spectral/angular diagnostics for later `G_spatial` / `C_wavefront` analysis | Generated by direct tiger-vis bash, no Slurm; output is diagnostic only, not yet a physical focusing/wavefront conclusion. Use deck values for physical D if directory label differs. |

## Rows to add next

When Zhiping provides more simulation/workflow inventory, add rows for:

- exact 2D/3D propagation/clipping jobs after Zhiping specifies how to run `propagate_and_clip`;
- focal-plane diagnostics used to evaluate `G_spatial`;
- ideal coherent-focus comparison for `C_wavefront`;
- control cases: flat foil, no curvature, ideal wavefront, same-energy direct focus / reference intensity, different `ND/a0`, different `κ`, different `w0`.

## Status vocabulary

- `planned` — desired run not yet launched.
- `running` — submitted / active.
- `done-unprocessed` — raw output exists but diagnostics not yet extracted.
- `processed` — summary/figures generated.
- `needs-rerun` — result invalid or missing required output.
- `superseded` — old result kept for traceability but no longer used.
