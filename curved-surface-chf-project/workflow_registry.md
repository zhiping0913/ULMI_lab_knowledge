# Curved_surface workflow registry

This file records scripts, responsibilities, inputs/outputs, and gotchas for the Curved_surface / ultrathin-foil CHF project.

## Known workflows as of 2026-07-06/07

### `plot_reflection_1D_scan.py`

Path:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot_reflection_1D_scan.py
```

Responsibility:

- Read existing NetCDF summary files (`reflection.nc`) using `EM_analyzer.read_write.read_nc`.
- Combine and plot existing reflection summary data.
- Default L scan over all available `a0=*/1D/45,thick/reflection.nc` cases.
- `--scan-key ND_a0` all-a0 comparison over existing `a0=*/1D/45/reflection.nc` summaries.

Not responsible for:

- Generating individual ND/a0 scan summaries.
- Processing raw simulation output into `reflection.nc`.

Important outputs:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/reflection_1D_scan_plots/
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/reflection_ND_a0_scan_plots/
```

Current gotchas:

- `a0=200` has sparse ND/a0 coverage.
- All-a0 plots use exact union-coordinate alignment and NaNs for missing points, not interpolation.

### `efficiency_1D_scan.py`

Path:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/efficiency_1D_scan.py
```

Responsibility per Zhiping's correction:

- Generate/process `ND_a0` scan summaries.
- Own the per-scan processing that creates summary files later combined by `plot_reflection_1D_scan.py`.

Need more details from Zhiping before documenting exact command-line usage and outputs.

### `start_2D.py` — EPOCH 2D generator

Path:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/EPOCH_toolkit/start_2D.py
```

Read-only copy captured locally:

```text
artifacts/curved_surface_2d_a020_inspection_20260706_204001/start_2D.py
```

Responsibilities / behavior observed:

- Defines `set_blocks(laser_a0=10,N=300,D=0.005,L=0,Kappa=0,theta_degree=0)` and generates EPOCH `input.deck` blocks.
- Uses `block` class pattern to print `constant`, `control`, `boundaries`, `fields`, species, and output blocks.
- Current top-level values at file end submit the thick-reference-like case: `N=350`, `a0=20`, `theta_degree=45`, `L=0.05`, `D=1.95`, `Kappa=-0.002`, working dir `Curved_surface/a0=20/2D/K=-0.002,D=1.95,L=0.05`.
- **Safety boundary:** this script calls `sbatch(...)` at top level. Do not run/import it just to inspect; it submits jobs. Read/copy/refactor only unless Zhiping explicitly authorizes submission and `checkquota` passes.

Important symbol meanings:

- `Kappa` (`K` in directory): target curvature in `1/λ0`; nonzero cases define `R=1/Kappa`, `target_radius=R*λ0`, `target_f=sqrt((x-target_radius)^2+y^2)-abs(target_radius)`.
- `D`: target thickness in `λ0`; `target_thickness=D*λ0`.
- `L`: density-gradient scale length in `λ0`; if `L>0`, electron density uses an exponential ramp in `target_f`; if `L=0`, slab density is used.
- `N`: electron density in critical densities; `target_Ne=N*laser_Nc`.
- `laser_W0=12.5λ0`, `laser_FWHM=8 fs`, `λ0=0.8 μm`.
- Current deck uses file-based initial fields (`fields.ex`, `fields.ey`, `fields.bz`) from `../Initialize_Field/...`; the analytic `laser` block is commented out.

### `rotate_and_shift.py` — completed reflection/transmission field extraction

Path:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/rotate_and_shift.py
```

Read-only copy captured locally:

```text
artifacts/curved_surface_2d_a020_inspection_20260706_204001/rotate_and_shift.py
```

Purpose:

- Reads `fields0001.sdf` from `working_dir`.
- Loads `Electric_Field_Ex`, `Electric_Field_Ey`, `Magnetic_Field_Bz` using `EM_analyzer.read_write.read_sdf`.
- Shifts `Ex` and `Ey` from Yee-staggered positions to common cell-centered coordinates via `shift_field`.
- Uses `Rotation` to rotate outgoing fields into reflection and transmission coordinate frames.
- Reflection rotation: `psi = 3π/4`.
- Transmission rotation: `psi = π/4`.
- New grid: 250 cells/λ0, `x` from `0` to `100λ0` (implemented as positive propagation coordinate), `y` from `-50λ0` to `+50λ0`.
- Writes NetCDF files:
  - `fields0001_250cpl_reflection.nc`
  - `fields0001_250cpl_transmission.nc`
- Also plots normalized `Ex/Ec`, `Ey/Ec`, and `Bz/Bc` PNGs for reflection and transmission.

Zhiping says this stage is already completed for the relevant finished cases. Inspection confirmed reflection/transmission `.nc` files for all thin completed K,D,L cases.

### `propagate_and_clip.slurm` — next stage, not yet started

Path:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_and_clip.slurm
```

Read-only copy captured locally:

```text
artifacts/curved_surface_2d_a020_inspection_20260706_204001/propagate_and_clip.slurm
```

Observed Slurm resources:

- `#SBATCH --nodes=2`
- `#SBATCH --ntasks-per-node=1`
- `#SBATCH --cpus-per-task=3`
- `#SBATCH --mem=600G`
- `#SBATCH --time=2:00:00`
- activates `/home/zl8336/.conda/envs/mpi_python`
- forces CPU JAX settings and `XLA_FLAGS="--xla_force_host_platform_device_count=3"`
- runs:

```text
srun /home/zl8336/.conda/envs/mpi_python/bin/python3.13 /scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_and_clip.py
```

Do not submit yet. Zhiping explained the intended `propagate_and_clip` workflow in Telegram `main:6420513923:81`, and the current CSV/script have now been inspected.

### `propagate_and_clip.py` and `propagate_and_clip.csv` — inspected after Zhiping's explanation

Remote script:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_and_clip.py
```

Remote TSV/CSV:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/propagate_and_clip.csv
```

Local copies:

```text
artifacts/curved_surface_2d_a020_inspection_20260706_204001/propagate_and_clip.py
artifacts/curved_surface_2d_a020_inspection_20260706_204001/propagate_and_clip.csv
artifacts/curved_surface_2d_a020_inspection_20260706_204001/propagate_and_clip_plan_summary.md
```

Purpose from Zhiping: first crop the true reflection/transmission pulse from the rotated field (`clip_0`), propagate it away from noisy target fields for a case-specific time, then crop again (`clip_1`). The table columns are tab-delimited and specify `working_dir`, `side`, `clip_0`, `clip_1`, `propagation_time`, and `done`.

Key observation: current `propagate_and_clip.py` does **not** read the CSV. It is hard-coded to one case/side:

```text
working_dir = .../K=-0.002,D=0.01,L=0.00
side = transmission
clip_0 = x 30..50 λ0, y -35..35 λ0
propagation_time = 140 T0
clip_1 = x 170..190 λ0, y -25..25 λ0
```

That row is the only CSV row currently marked `done=Y`. Running `propagate_and_clip.slurm` as-is would rerun that already-done transmission row, not process the 31 remaining `done=N` rows. Do not submit as-is unless Zhiping explicitly wants that rerun.

Correction from Zhiping (`main:6420513923:85`): do **not** make `propagate_and_clip.py` read or depend on this CSV. `propagate_and_clip.py` should remain a generic propagation/clipping kernel reusable for future `a0=50` and other scans. The compromise pattern for agent-friendly automation is:

- replace hard-coded case parameters in `propagate_and_clip.py` with `os.environ.get(...)` values, preserving hard-coded defaults for manual/debug use;
- write an outer bash/driver for this `a0=20/2D` batch that reads the tab-delimited CSV, skips `done=Y`, and for each `done=N` row generates/submits a Slurm job with the CSV parameters exported as environment variables;
- keep the CSV as a batch-driver input, not as a dependency inside the Python kernel;
- before any actual `sbatch`, run `checkquota` and block if Tiger scratch has <2 TiB free.

### Env-var Slurm driver implementation — dry run completed 2026-07-06

After Zhiping authorized dry-run implementation in Telegram `main:6420513923:90`, the following was done on tiger-vis:

- Backed up `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_and_clip.py` to `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_and_clip.py.bak_env_20260706_214530`.
- Modified `propagate_and_clip.py` so the previously hard-coded case parameters are now `os.environ.get(..., default)` values. Defaults preserve the old manual behavior; env vars override them for Slurm jobs.
- Added `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/generate_propagate_and_clip_slurms.sh`.
- Dry-run generated 31 Slurm files under `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_and_clip_slurms/a0_20_2D/`, skipping the one `done=Y` row.
- Validation: Python syntax OK; all 31 generated Slurm files pass `bash -n`; `SUBMIT=1 DRY_RUN=1` reports 31 would-sbatch commands; no real `sbatch` was executed.
- A generator bug from a missing final newline in the TSV was fixed by using `while read ... || [[ -n "${case_rel:-}" ]]`.

Detailed record: `propagate_clip_dryrun_2026-07-06.md`.

## EM_analyzer read utility

Path:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/EM_analyzer/read_write.py
```

Relevant functions:

```text
read_nc
read_sdf
write_fields_to_nc
```

Used by plotting/post-processing scripts for NetCDF/SDF I/O.

### `Spectrum_2D.py` — clipped-field spectrum extraction, direct bash batch completed 2026-07-07

Path:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/Spectrum_2D.py
```

Responsibility:

- Read a clipped NetCDF field `${side}_clip.nc` from `working_dir`.
- Transform `Bz` into Cartesian, radial, and angular spectra.
- Write spectrum plots and NetCDF summaries for each side/case.

Runtime interface:

```bash
env working_dir=/path/to/K=...,D=...,L=... side=reflection \
  /scratch/gpfs/MIKHAILOVA/zl8336/.conda/envs/GPU_python/bin/python \
  /scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/Spectrum_2D.py
```

Environment variable names are lowercase: `working_dir`, `side`.

Outputs per side:

```text
<side>_spectrum_kxky.png
<side>_spectrum_kr.nc
<side>_spectrum_kr.png
<side>_spectrum_ktheta.nc
<side>_spectrum_ktheta.png
```

Batch driver written/copied:

```text
local:  artifacts/curved_surface_2d_a020_inspection_20260706_204001/run_spectrum_2d_a0_20.sh
remote: /scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/run_spectrum_2d_a0_20.sh
```

Run/validation logs:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_2d_logs/a0_20_2D_20260707_221434/run.log
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_2d_logs/a0_20_2D_20260707_221434/manifest.tsv
```

Validation result:

```text
32 clip inputs processed: 16 reflection + 16 transmission
manifest status: OK 32
expected spectrum output files: 160
missing=0, empty=0
```

Gotchas:

- No `sbatch` was used; Zhiping explicitly said this compute is small enough for direct tiger-vis bash.
- SSH login emits a stale nonfatal `/home/zl8336/.conda/envs/GPU_Python` warning; actual successful Python was `/scratch/gpfs/MIKHAILOVA/zl8336/.conda/envs/GPU_python/bin/python`.
- `Spectrum_2D.py` transforms `Bz*c` and normalizes to laser constants; treat outputs as diagnostic spectra requiring later physical interpretation.

Supports factor:

- Provides near-field radial/angular spectral diagnostics needed for later `G_spatial` / `C_wavefront` analysis, but does not by itself establish either factor.

Detailed record: `spectrum_2d_batch_2026-07-07.md`.

### `plot_reflection_spectrum_compare_a0_20.py` — 1D-vs-2D reflection spectrum comparison

Remote path:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/plot_reflection_spectrum_compare_a0_20.py
```

Local source/artifacts:

```text
artifacts/curved_surface_2d_a020_inspection_20260706_204001/plot_reflection_spectrum_compare_a0_20_nd_scan.py
artifacts/curved_surface_2d_a020_inspection_20260706_204001/reflection_spectrum_1D_vs_2D_ND_scan_a0_20.ipynb
artifacts/curved_surface_2d_a020_inspection_20260706_204001/reflection_spectrum_1D_vs_2D_ND_scan_a0_20.png
```

Remote notebook/source:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/reflection_spectrum_1D_vs_2D_ND_scan_a0_20.ipynb
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/reflection_spectrum_1D_ND0p30_vs_2D_D0p02_K_scan.ipynb  # overwritten with same generalized logic for compatibility
```

Convention from Zhiping (`main:6420513923:117`): plotting scripts and reproducible notebooks for Curved_surface should live under `utility/plot/`; rendered outputs can live in task-specific output directories.

Purpose:

- Compare 1D `a0=20/1D/45/ND_a0=.../reflection_spectrum.nc` spectra with 2D `a0=20/2D/K=...,D=...,L=0.00/reflection_spectrum_kr.nc` spectra.
- Current generalized plot covers `ND/a0=0.10,0.30,0.50,1.00` with 2D `K=+0.000,-0.002,-0.005,-0.008`, giving 4 panels and 20 curves.
- Infer differing NetCDF variable names automatically.
- Use `EM_analyzer.plot.plot_1D.plot_multiple_1D_fields` for the actual plotting, per Zhiping's style instruction in Telegram `main:6420513923:120`.
- Compute the directory thickness from scan parameters instead of memorizing a correspondence table:

```python
D = ND_a0 * a0 / N
D_dir = "%3.2f" % D
```

For current defaults `a0=20,N=350`, this yields `D_dir=0.01,0.02,0.03,0.06` for `ND/a0=0.10,0.30,0.50,1.00`.

Plot style from Zhiping:

- x-axis label: `k/k0`;
- y-axis label: `I/I0`;
- legend: `1D` and curvature `K` only;
- panel titles carry `ND/a0` and formatted `D`.

Detected variables for the completed plots:

```text
1D: coord=f_over_f0_M, data=Ey_spectrum_intensity
2D: coord=k_rho_over_k0, data=Bz_spectrum_intensity_kr
```

Original single-`ND/a0=0.30` outputs remain:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_compare_plots/reflection_spectrum_1D_ND0p30_vs_2D_D0p02_K_scan.png
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_compare_plots/reflection_spectrum_1D_ND0p30_vs_2D_D0p02_K_scan.pdf
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_compare_plots/reflection_spectrum_1D_ND0p30_vs_2D_D0p02_K_scan_long.csv
```

Generalized multi-`ND/a0` outputs:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_compare_plots/reflection_spectrum_1D_vs_2D_ND_scan_a0_20.png
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_compare_plots/reflection_spectrum_1D_vs_2D_ND_scan_a0_20.pdf
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_compare_plots/reflection_spectrum_1D_vs_2D_ND_scan_a0_20_long.csv
```

Detailed record: `spectrum_compare_plot_2026-07-08.md`.

### `plot_reflection_spectrum_from_dir_list.py/.ipynb` — flexible spectrum comparison from arbitrary directory list

Remote source/notebook:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/plot_reflection_spectrum_from_dir_list.py
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/plot_reflection_spectrum_from_dir_list.ipynb
```

Purpose:

- Give Zhiping a more flexible notebook than the fixed a0=20/ND scan scripts.
- User edits `DIRECTORIES = [...]` with arbitrary Curved_surface result directories; examples may mix 1D/2D, different `a0`, `D`, `L`, `K`, `ND_a0`, and thick/scan categories.
- The notebook parses metadata from directory names and `input.deck` (`laser_a0`, `N`, `D`, `L`, `Kappa`) and constructs informative labels.
- It detects path/deck conflicts; example: the user-supplied `a0=20/1D/45/ND_a0=0.20` directory has `input.deck` `laser_a0=3`, so the label/warning prints `a0=3(path 20)` rather than silently trusting the path.
- It chooses spectrum input in order: `reflection_spectrum.nc`, `reflection_spectrum_kr.nc`, then `reflection.nc`.
- For precomputed spectrum files it auto-detects normalized coordinate/intensity variables (`f_over_f0_M`/`Ey_spectrum_intensity`, `k_rho_over_k0`/`Bz_spectrum_intensity_kr`, etc.).
- If only `reflection.nc` exists, it has an explicit quick-look 1D FFT fallback for `Ey/Bz`; the notebook warns this is not a replacement for the established spectrum pipeline for scientific claims.
- It uses `EM_analyzer.plot.plot_1D.plot_multiple_1D_fields` to plot all curves on one axis.

Default example list from Zhiping's instruction (`main:6420513923:129`) was run successfully with three curves and generated:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_compare_plots/reflection_spectrum_dirlist_example.png
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_compare_plots/reflection_spectrum_dirlist_example.pdf
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_compare_plots/reflection_spectrum_dirlist_example_long.csv
```

Delivered in Telegram: report `main:6420513923:131`, notebook `:132`, example PNG `:133`.

### `propagate_2D_time_scan.slurm/.py` — fast 2D clipped-field propagation focus scan

Remote Slurm/Python:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan.slurm
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan.py
```

Purpose:

- For a clipped 2D `reflection_clip.nc` or `transmission_clip.nc`, propagate fields over a time scan to estimate focal region.
- Reads `working_dir` and `side` from environment variables; default side is `reflection`.
- Reads `{side}_clip.nc`, constructs `Spectral_Maxwell_Solver_2D`, evolves `times=np.linspace(-200*T0,200*T0,21)`, saves per-time `Ey` PNG snapshots, an HDF summary, and a summary PNG.

Outputs per case/side:

```text
<side>_beam_width_strength.h5
<side>_beam_width_strength.png
<side>_Ey_t=...T0.png
```

HDF contains fixed absolute-x grid, transverse 1/e half-width arrays, max strength arrays, and per-timestep on-axis peak position/amplitude. Summary PNG plots transverse 1/e half-width and on-axis peak vs propagation position.

Submission note:

- Since case directory names contain commas, pass `working_dir` and `side` through the environment of the `sbatch` process rather than putting `working_dir=...` inside comma-separated `--export=...`.
- Before every `sbatch`, run `checkquota` and block if Tiger scratch free space <2 TiB.

Latest batch submission:

```text
knowledge/curved-surface-chf-project/propagate_2d_time_scan_submission_2026-07-08.md
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_20260708_152818/manifest.tsv
```

Submitted 32 jobs (`3438490–3438521`) for 16 renamed `a0=20/2D/K=...,ND_a0=...,L=...` directories × `reflection/transmission`.

### `estimate_1d_temporal_compression.py` — 1D Sx FWHM / temporal-compression estimate

Remote script:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/estimate_1d_temporal_compression.py
```

Remote output example:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/temporal_compression_estimates/a0_20_1D_45_ND0p30_reflection_Sx_FWHM_20260713.txt
```

Purpose:

- Read a 1D `reflection_Sx.nc` file containing moving-frame `x` coordinate and `Sx` in W/m².
- Use `EM_analyzer.pretreat_fields.get_peak_width(..., rel_height_each_axis=0.5)` to measure the main Sx peak FWHM in `x/λ0_M`.
- Convert spatial FWHM to temporal FWHM by `Δt=Δx/c`.
- Compare `Sx_peak × Δt_FWHM` with one moving-frame optical cycle incident fluence `a0² laser_Sc_M T0_M`, equivalently `[Sx_peak/(a0² laser_Sc_M)] × [Δx_FWHM/λ0_M]`.

Gotchas:

- For the first example `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/1D/45/ND_a0=0.30/reflection_Sx.nc`, `input.deck` says `laser_a0=20`.
- `utility/spectrum_1D.py` currently hard-codes `laser_a0=200`; do not use that hard-coded value for `a0` when analyzing an `a0=20` case. Reuse only its `laser_Sc_M = laser_Sc cos²θ` convention unless the script is parameterized later.
- The main-spike rectangular estimate for the first example was `≈0.102` of one-cycle incident fluence; direct integration inside the FWHM window was `≈0.0715`.

Detailed record: `temporal_compression_estimate_2026-07-13.md`.

## Workflows to document next

When Zhiping provides details, add sections for:

1. exact `propagate_and_clip.py` command/parameter workflow and clipping region rules;
2. near-field harmonic spectrum / phase / wavefront extraction;
3. propagation to focus and focal-plane diagnostics;
4. ideal coherent-focus construction for `C_wavefront`;
5. any 3D EPOCH input-deck generation and parameter naming conventions.

## Workflow memory rule

For each script or command sequence, record:

- Purpose.
- Input files / directories.
- Output files / directories.
- Key parameters and defaults.
- Exact command lines when known.
- Validation checks.
- Known failure modes / convention traps.
- Which physics factor it supports: `G_CSE/1D`, `G_spatial`, or `C_wavefront`.
