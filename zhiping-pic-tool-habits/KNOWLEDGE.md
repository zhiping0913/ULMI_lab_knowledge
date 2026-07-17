---
name: zhiping-pic-tool-habits
description: Zhiping's PIC/EPOCH/EM_analyzer coding, input-checking, Slurm, I/O, and λ0/Ec normalization conventions observed on tiger-vis.
version: 1.0.0
updated_at: 2026-07-04
---

# Zhiping PIC / EPOCH / EM_analyzer tool habits

## Source and scope

Observed by read-only SSH to `tiger-vis` on 2026-07-04, at Zhiping's request. Local snapshot for evidence:

- `artifacts/tiger-vis-pic-tools-20260704/EM_analyzer/`
- `artifacts/tiger-vis-pic-tools-20260704/EPOCH_toolkit/`

Remote source paths:

- `/scratch/gpfs/MIKHAILOVA/zl8336/EM_analyzer`
- `/scratch/gpfs/MIKHAILOVA/zl8336/EPOCH_toolkit/start_1D.py`
- also inspected representative `start_2D.py`, `start_3D.py`, plotting and I/O helpers.

This is private project memory: load this entry before writing, reviewing, or interpreting Zhiping's PIC post-processing, EPOCH input-deck generation, or EM field analysis scripts.

## Safety / execution boundary

- Treat `EPOCH_toolkit/start_*.py` as executable job-submission scripts, not import-safe libraries. They contain top-level parameter sweeps that call `sbatch(...)` directly; `start_1D.py` even calls `exit(0)` after its first sweep.
- Do **not** run these scripts just to inspect or import their functions. Read them or copy/refactor into guarded code first.
- Before any actual `sbatch`/`salloc`, obey standing HPC rule: run `checkquota`; block Tiger if MIKHAILOVA scratch has <2 TiB free and Della if <1 TiB free.
- In this inspection I only read/copied text files and did not submit or modify anything on HPC.

## Core normalization conventions

Zhiping's PIC tooling normalizes nearly everything to the laser reference:

- Default laser wavelength is `λ0 = 0.8 μm` (`laser_lambda0 = 0.8*C.micron` or `laser_lambda = 0.8*C.micron`).
- `k0 = 2π/λ0`, `ω0 = 2πc/λ0`, `T0 = λ0/c`.
- Critical field scales:
  - `Bc = m_e ω0 / e` in tesla.
  - `Ec = Bc c = m_e ω0 c / e` in V/m.
  - `a0 = E_peak/Ec`; scripts often compute `laser_amp = laser_a0 * laser_Ec`.
- Critical density:
  - `Nc = ω0^2 m_e ε0 / e^2`.
  - Target density is usually `N * Nc`, where `N` is dimensionless density in critical densities.
- Lengths are expressed as multiples of `λ0`: `D` target thickness, `L` preplasma/gradient scale, `x_min_lambda`, `x_max_lambda`, `vacuum_length_*_lambda`, `target_thickness_lambda`, `w0_lambda`, etc.
- Spatial resolution is usually `cells_per_lambda_*` or in EM_analyzer `r_resolution = λ0/dx`; k-space resolution is `k_resolution = k0/dkx`.
- Times are physical seconds but parameterized by `T0` or laser FWHM: `laser_FWHM = 8 fs` (often), `laser_tau = FWHM/sqrt(2 ln 2)`, delays like `4*laser_tau` or `5*T0`.
- Plot labels and analysis outputs prefer dimensionless axes/fields:
  - field label often `a = E/Ec = B/Bc`;
  - axes `x/λ0`, `y/λ0`, `z/λ0`;
  - documentation writes `w0/λ0`, `λ0/dx`, `x_min/λ0`, etc.
- Directory names encode dimensionless scan parameters, e.g. `a0=3`, `1D/45`, `L=0.20`, `ND_a0 = N*D/a0`, `K=...`, `D=...`, `L=...`.

## Oblique-incidence / moving-frame convention

For oblique incidence with angle `theta_degree`:

- Convert once: `theta_rad = np.radians(theta_degree)`.
- Moving-frame/boost-like deck quantities in `start_1D.py`:
  - `laser_lambda0_M = laser_lambda0/cos(theta_rad)`;
  - `laser_k0_M = laser_k0*cos(theta_rad)`;
  - `laser_tau_M = laser_tau/cos(theta_rad)`;
  - `laser_E_amp_M = laser_a0*laser_Ec*cos(theta_rad)`;
  - `laser_B_amp_M = laser_a0*laser_Bc*cos(theta_rad)`;
  - `target_Ne_M = target_Ne/cos(theta_rad)`;
  - electron drift `drift_y = -me*c*tan(theta_rad)`;
  - ion drift `drift_y = -ion_mass*me*c*tan(theta_rad)`.
- For density profiles, 1D scripts use either an exponential ramp when `L>0` or a slab around zero when `L<=0`.

## EPOCH deck-generation style

Zhiping's `start_*.py` scripts generate EPOCH input decks with a small dynamic `block` class:

- `block(built='constant'|'control'|'boundaries'|'fields'|'species'|'output'...)` stores attributes dynamically.
- `block.print()` renders:
  - `begin : <built>`
  - one `key=value` line per attribute (except `built` and a special `supplemental` line)
  - `end : <built>`
- Deck constants deliberately mix Python-computed numeric values with EPOCH expression strings. Unit comments are embedded in the string values, e.g. `#unit: m`, `#unit: m^-3`, `#unit: lambda`, `#Normalized laser field strength`.
- Typical block order: `constant`, `control`, `boundaries`, `fields` or `laser`, `species` blocks for Electron/Ion/Silicon, collisions if needed, output blocks.
- Output is intentionally minimal for scans unless more diagnostics are needed: fields, grid, and number density; optional particle diagnostics/restarts are left as commented toggles.
- Ion/material defaults in observed scripts: carbon/silicon-like constants appear (`ion_atomic_weight=12`, `ion_charge=6`, `ion_number=14`, Si mass density), with some `Silicon` blocks commented or optional. Do not assume these are universal physical choices without checking the script/context.

## Slurm / EPOCH submission style

Each `sbatch(working_dir, block_list)` in the scripts:

1. chooses an EPOCH binary path, e.g. `/home/zl8336/Software/Epoch/epoch-4.19.5/epoch1d/bin/epoch1d` (or `epoch2d`, `epoch3d`);
2. writes `input.deck` from rendered blocks;
3. writes `deck.file` containing the working directory path;
4. writes a `slurm` script;
5. runs `sbatch <working_dir>/slurm` via `subprocess.Popen(..., shell=True)`.

Observed Slurm/module template:

- `#SBATCH --account=mikhailova`, `--job-name=test`, `--qos=standard`.
- 1D example: `nodes=1`, `ntasks=10`, `mem=20G`, `time=24:00:00`.
- 2D example: `nodes=3`, `ntasks=336`, `mem=900G`, `time=144:00:00`.
- 3D example: `nodes=1`, `ntasks=16`, `mem=128G`, `time=24:00:00`.
- modules: `module purge`, Intel runtime/TBB/oneAPI/MPI modulefiles, `export I_MPI_PMI_LIBRARY=/usr/lib64/libpmi2.so`.
- run line: `srun --mpi=pmi2 <epoch_binary> < <deck.file>`.

When I write new safer code, preserve the generated-deck style but separate generation from submission unless Zhiping explicitly wants immediate submission.

## HPC scan resource scheduling lesson (2026-07-08)

Zhiping corrected a Curved_surface fine time-scan submission where I initially requested `900G` and `12:00:00` for all 32 jobs. His lesson: with AI-agent help, do not submit rigid over-conservative batches like a manual workflow that sets `--time=24:00:00` everywhere. Use the previous run's `sacct` evidence (`MaxRSS`, `Elapsed`) and case structure to tier resources.

Practical rule for future parameter scans:
- Estimate resources from measured prior jobs, not from the worst case alone.
- Ordinary cases should request enough but modest memory/time so Slurm can backfill / “见缝插针” quickly.
- Only known heavy cases should receive high memory/long walltime.
- If a subset OOMs or times out, rerun only those failed items with higher resources.
- This is especially important on Tiger: complete 1TB nodes can be scarce, while smaller requests can start almost immediately.

Concrete example from Curved_surface fine scan:
- Initial overrequest: all jobs `900G/12h` queued slowly.
- Corrected tiering: 30 standard jobs `700G/5h`; only two heavy cases `900G/12h`:
  - `K=-0.002,ND_a0=0.10,L=0.00` reflection (previous heavy job `3439199`).
  - `K=+0.000,ND_a0=0.50,L=0.00` reflection (previous heavy job `3439197`).

## EM_analyzer workflow / coding style

`EM_analyzer` is a JAX-based toolkit for field generation, spectral Maxwell propagation, spectrum/envelope analysis, plotting, and I/O.

### Import/runtime habits

- Scripts commonly add absolute project paths:
  - `sys.path.append('/scratch/gpfs/MIKHAILOVA/zl8336')`
  - `sys.path.append('/scratch/gpfs/MIKHAILOVA/zl8336/EM_analyzer')`
- JAX configuration must happen before importing heavy solver modules. `CLAUDE.md` documents:
  - `jax.config.update('jax_num_cpu_devices', 6)`;
  - `jax.config.update('jax_enable_x64', True)`;
  - `jax.config.update('jax_platform_name', 'cpu')` when forcing CPU.
- `Normal_variable_method.py` auto-detects GPUs with `nvidia-smi`; uses GPU only if at least 2 GPUs and caller did not force CPU. Otherwise uses 6 CPU devices by default. It prints backend/device information.
- JAX sharding language uses `EM` axis for E vs `B*c` and `channel` axis for x/y/z components.

### Physics/numerical model

- Spectral Maxwell solver uses vacuum Maxwell equations in k-space, with `FB = cB`, dispersion `ω = c|k|`.
- Normal variables:
  - `FA+ = 1/2 (FE - k_hat × FB)` evolves as `exp(-iωt)`;
  - `FA- = 1/2 (FE + k_hat × FB)` evolves as `exp(+iωt)`.
- Solver enforces transversality in k-space by projecting fields onto the divergence-free subspace: subtract `k_hat (k_hat·F)` for nonzero k.
- Evolution can include a moving window; `k_hat·v/c` is precomputed/cached to avoid repeated device communication.
- FFT conventions are 0-centered (`fftshift`/`ifftshift` wrappers), and comments warn that naive `fftshift(fft(x))` gets phase wrong.

### Field-generation habits

- Gaussian/LG beam classes take physical wavelength but dimensionless beam/resolution parameters:
  - `wavelength` in meters;
  - `w0_lambda` in units of `λ0`;
  - `a0` as normalized peak field;
  - `r_resolution = λ0/dx`, `k_resolution = k0/dkx`.
- Beam objects compute and retain `period`, `k0`, `omega0`, `Ec`, `Bc`, `w0`, `z_R`, `amp=a0*Ec`.
- Generated fields are physical arrays (`E` in V/m, `B` in T), but diagnostics and metadata repeatedly report normalized quantities like `w0/λ0`, `λ0/dx`, coordinate limits over `λ0`.
- For 2D Gaussian beam mapping, documented convention maps physical X-Z plane to solver X-Y plane by transposing to preserve `∇·E=0`; after evolution, `result['Ey']` is the main oscillating field.

## Input checking / validation habits

EM_analyzer has much stronger internal validation than the EPOCH start scripts:

- Uses `assert` heavily for internal invariants:
  - at least one field component provided;
  - field component dimension must be 1/2/3D;
  - component shapes must match expected `(Nx,)`, `(Nx,Ny)`, `(Nx,Ny,Nz)`;
  - coordinate arrays must be 1D, nonempty, and match field dimensions;
  - output/padding/location lengths must match array ndim;
  - plot coordinate and field lengths must match;
  - solver `E0/B0` shapes must equal `(3,Nx,Ny,Nz)`.
- Field diagnostics print shapes/backend/device layout. This is part of the workflow, not noise.
- `check_divergence` computes `L |∇·F| / |F|`, prints pass/fail, and returns `True/False`; it does not raise by default. It masks low-field regions using a threshold to avoid numerical instability.
- `read_write.copy_to_dev_shm` does explicit file-access checks and raises/prints `FileNotFoundError`, `NotADirectoryError`, and `PermissionError`; then uses `/dev/shm` for fast local SDF/NetCDF reads and cleans up.
- NetCDF writing asserts dictionary schema (`name`, `coordinate`/`data`) and exact shape compatibility before writing.

The EPOCH `start_*.py` scripts themselves do **not** comprehensively validate physical inputs (`N`, `D`, `L`, `Kappa`, particle counts, quota, etc.). When writing new scripts, preserve Zhiping's compact deck-building style but add explicit guardrails for dangerous/expensive cases.

## I/O and metadata habits

- Preferred structured output is NetCDF via xarray/h5netcdf.
- Field dictionaries use:
  - `{'name': ..., 'data': ..., 'units': ..., 'long_name': ...}`.
- Coordinate dictionaries use:
  - `{'name': ..., 'coordinate': ..., 'units': ..., 'long_name': ...}`.
- NetCDF data variables are compressed (`zlib=True`, `complevel=5`).
- SDF access often goes through converted/readable NetCDF style now (`read_sdf` calls `read_nc` in current `read_write.py`), with old `sdf_helper` path retained in comments/older scripts.
- For large SDF reads, copy to `/dev/shm` first (`use_shm=True`) to avoid slow repeated scratch access.
- Plotting helpers expect fields already normalized by caller when desired, e.g. pass `Ey/Ec` and coordinates `/laser_lambda`; defaults label this dimensionless presentation.

## Style notes to preserve when helping Zhiping

- Keep physics units and normalizations visible in code comments and output metadata.
- Use dimensionless parameter names (`*_lambda`, `*_nc`, `a0`, `ND_a0`, `cells_per_lambda`) for scan-facing quantities; convert to SI inside constants/deck expressions.
- Prefer explicit formula strings in EPOCH `constant` blocks so the deck remains self-documenting.
- Preserve verbose shape/unit comments and printed diagnostics for scientific trust.
- For EM field tools, always think about transversality/divergence, centered FFT phase conventions, and whether fields are physical SI arrays or normalized analysis arrays.
- For plots/reports to Zhiping, present both the physical mechanism and the normalization basis (`λ0`, `Ec`, `Bc`, `Nc`, `T0`).
- Before executing copied scripts, inspect for top-level `sbatch` calls and add `if __name__ == '__main__'` or a dry-run/generate-only mode.
