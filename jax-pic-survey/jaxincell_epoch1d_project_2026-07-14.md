# JAX-in-Cell-only epoch1d project extraction

Date: 2026-07-14 EDT
Telegram: Zhiping request `main:6420513923:261`; report `main:6420513923:263`.

## Request

Zhiping decided to temporarily avoid PyPIC3D because the current PyPIC3D workflow supplies inputs through `.npz` files, making it inconvenient to differentiate with respect to those variables. JAX-in-Cell can receive arrays directly. He asked to extract the JAX-in-Cell portion of:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/epoch1d_profiles_per_T0.py
```

into a new project under:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX
```

For output, he requested using:

```text
/scratch/network/zl8336/EM_analyzer/pretreat_fields.py::shift_field
/scratch/network/zl8336/EM_analyzer/read_write.py::write_fields_to_nc
```

to shift `Ey` and `Bz` onto cell centers and save the final field as NetCDF. Reference: tiger-vis

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/final_field_1D.py lines 36-44
```

with the important sign difference:

```text
EPOCH:       x_Bz = x_Ey + dx/2
JAX-in-Cell: x_Bz = x_Ey - dx/2
```

## Implemented project

Remote project directory:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/
```

Files:

```text
README.md
run_jaxincell_final_field.py
write_centered_final_field.py
```

`run_jaxincell_final_field.py`:

- Contains only the JAX-in-Cell runner; PyPIC3D is intentionally omitted.
- Uses the same constants/initialization as the benchmark script.
- After Zhiping's preference note in Telegram `main:6420513923:267`, the user-facing parameters are defined near the top as `os.environ.get("NAME", "default")` rather than argparse-only flags. Zhiping can edit defaults directly in VSCode; agents can override with `export NAME=value`. This preference was also recorded in `standing-rules.md`.
- After Zhiping's correction in Telegram `main:6420513923:271`, the script now imports `scipy.constants as C` and uses `C.speed_of_light`, `C.m_e`, `C.m_p`, `C.elementary_charge`, `C.epsilon_0`, `C.pi`, and `C.micron` instead of handwritten fundamental constant literals. This scientific-script style preference was also recorded in `standing-rules.md`.
- Preserves the corrected JAX solver settings:

```python
'solver_parameters': {
    'field_solver': 0,
    'time_evolution_algorithm': 0,
    'relativistic': True,
    'print_info': False,
}
```

- Saves raw final field arrays to:

```text
final_field_raw_jaxincell.npz
```

`write_centered_final_field.py`:

- Runs as a separate Python process because `EM_analyzer.pretreat_fields` sets JAX platform to CPU on import; separation avoids changing the GPU backend used by the simulation.
- Uses `shift_field` for the centering workflow.
- Treats JAX-in-Cell staggering as:

```text
Ey grid: x_Ey = -L/2 + (i+0.5) dx
Bz grid: x_Bz = -L/2 + i dx = x_Ey - dx/2
```

- Shifts `Bz` from `x_Bz` to `x_Ey`. Because `shift_field` uses constant extrapolation, appends one periodic ghost point `(x_Bz[-1]+dx, Bz[0])` so the last cell center is interpolated from the periodic neighbor instead of zero.
- Writes:

```text
final_field_centered_jaxincell.npz
final_field_jaxincell_centered.nc
postprocess_summary.json
```

## Validation

Syntax check passed with:

```bash
/scratch/network/zl8336/.conda/envs/GPU_Python/bin/python -m py_compile \
  run_jaxincell_final_field.py write_centered_final_field.py
```

A 2-step smoke test was run:

```bash
cd /scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d
CUDA_VISIBLE_DEVICES=0 /scratch/network/zl8336/.conda/envs/GPU_Python/bin/python run_jaxincell_final_field.py \
  --cells-per-lambda 20 \
  --Ne-macro 120 \
  --steps 2 \
  --outdir outputs/smoke_steps2_wrapper_ok
```

Result:

```text
outputs/smoke_steps2_wrapper_ok/final_field_jaxincell_centered.nc
sizes: {'x': 1200}
data_vars: ['Ey', 'Bz']
```

The wrapper returned success; the NetCDF could be opened with `xarray.open_dataset(..., engine='netcdf4')`.

## NetCDF writer dependency

Initial smoke testing showed that adroit-vis `/scratch/network/zl8336/.conda/envs/GPU_Python/bin/python` had `xarray`, `h5netcdf`, and `netCDF4`, but lacked `h5py`. Since `EM_analyzer.write_fields_to_nc` hard-codes `engine='h5netcdf'`, it initially failed with missing `h5py` even though `netCDF4` was available.

Zhiping then authorized directly installing the needed libraries in Telegram `main:6420513923:264`. I ran:

```bash
/scratch/network/zl8336/.conda/envs/GPU_Python/bin/python -m pip install h5py
```

and verified:

```text
h5py     3.16.0
h5netcdf 1.8.1
xarray   2026.4.0
```

Re-running `write_centered_final_field.py` on the smoke raw output succeeded through the normal `EM_analyzer.write_fields_to_nc` / `h5netcdf` path, and `xarray.open_dataset(..., engine='h5netcdf')` read back the resulting file with `sizes={'x': 1200}` and `data_vars=['Ey','Bz']`. The postprocessor still retains a `netcdf4` fallback for robustness, but it should not trigger now that `h5py` is installed.

## Next if Zhiping continues

- Run the full default command if desired:

```bash
cd /scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d
/scratch/network/zl8336/.conda/envs/GPU_Python/bin/python run_jaxincell_final_field.py
```

- For differentiable optimization, this runner is only a field-output baseline. A future differentiable runner should return a scalar objective and use `jax.lax.scan` + `jax.checkpoint`/`remat` rather than returning full histories.


## Addendum: merged NetCDF-only runner after Zhiping data-format preference

Telegram: Zhiping `main:6420513923:274`; acknowledgement `:275`; completion report to be sent after this patch.

Zhiping requested merging `write_centered_final_field.py` into `run_jaxincell_final_field.py` and avoiding `.npz` field intermediates. His data-saving preference is NetCDF/HDF-style output (`.nc`, `.h5`, `.hdf`) because it carries data, coordinates, units, and comments together. High-dimensional field I/O should use `EM_analyzer.read_write` on both tiger-vis and adroit-vis. This preference was recorded in `standing-rules.md`.

Implemented remote state:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/
  README.md
  run_jaxincell_final_field.py
  outputs/
```

`write_centered_final_field.py` was removed from the remote project. The main runner now:

1. Runs JAX-in-Cell.
2. Keeps `E_final/B_final` in memory.
3. Imports `EM_analyzer.pretreat_fields.shift_field` and `EM_analyzer.read_write.write_fields_to_nc`.
4. Shifts `Ey` and `Bz` to the common cell-center grid (Bz uses one periodic ghost point for the final cell center).
5. Writes only:

```text
final_field_jaxincell_centered.nc
run_summary.json
```

No `final_field_raw_jaxincell.npz` or `final_field_centered_jaxincell.npz` is produced.

Validation command:

```bash
cd /scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d
export CELLS_PER_LAMBDA=20
export NE_MACRO=120
export STEPS=2
export OUTDIR=outputs/smoke_merged_nc
CUDA_VISIBLE_DEVICES=0 /scratch/network/zl8336/.conda/envs/GPU_Python/bin/python run_jaxincell_final_field.py
```

Validation result:

```text
outputs/smoke_merged_nc/final_field_jaxincell_centered.nc 41404 bytes
outputs/smoke_merged_nc/run_summary.json 3375 bytes
data_format: final field written directly to NetCDF via EM_analyzer.read_write.write_fields_to_nc; no npz field intermediate
steps: 2
Nx: 1200
NetCDF sizes: {'x': 1200}
data_vars: ['Ey', 'Bz']
units: Ey V/m, Bz T
```

## Superseded addendum: temporary x-windowed one-way final-field outputs

Superseded by the reflection/transmission split addendum below after Zhiping's correction in Telegram `main:6420513923:280`. The current runner no longer contains `(Ey±cBz)/2` output calculations; this section is retained only as provenance of the short-lived intermediate patch.

Date: 2026-07-15 EDT. Telegram: Zhiping `main:6420513923:277`; acknowledgement `:278`.

Zhiping requested that, besides saving `final_field_jaxincell_centered.nc`, the runner also save the one-way field combinations:

```text
(Ey - c Bz)/2 on the x < 0 subset
(Ey + c Bz)/2 on the x > 0 subset
```

Physical convention recorded in the script/README: for this 1D `Ey/Bz` geometry, a right-going `+x` wave has `Ey = +c Bz`; a left-going `-x` wave has `Ey = -c Bz`. Thus `(Ey-cBz)/2` isolates the left-going/reflected component and `(Ey+cBz)/2` isolates the right-going/transmitted component, with Zhiping's requested spatial windows.

Remote changes:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/run_jaxincell_final_field.py
/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/README.md
```

Backups:

```text
run_jaxincell_final_field.py.bak_directional_20260715_110155
README.md.bak_directional_20260715_110259
```

New env-default output names in `run_jaxincell_final_field.py`:

```python
LEFTGOING_NC_NAME = os.environ.get("LEFTGOING_NC_NAME", "final_field_leftgoing_xlt0")
RIGHTGOING_NC_NAME = os.environ.get("RIGHTGOING_NC_NAME", "final_field_rightgoing_xgt0")
```

The runner now writes, directly through `EM_analyzer.read_write.write_fields_to_nc` with no `.npz` intermediary:

```text
final_field_jaxincell_centered.nc      # Ey, Bz on full centered x grid
final_field_leftgoing_xlt0.nc          # variable Ey_minus_cBz_over_2, units V/m, x < 0
final_field_rightgoing_xgt0.nc         # variable Ey_plus_cBz_over_2, units V/m, x > 0
run_summary.json                       # includes directional_netcdf paths
```

Validation smoke test:

```bash
cd /scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d
export CELLS_PER_LAMBDA=20
export NE_MACRO=120
export STEPS=2
export OUTDIR=outputs/smoke_directional
CUDA_VISIBLE_DEVICES=0 /scratch/network/zl8336/.conda/envs/GPU_Python/bin/python run_jaxincell_final_field.py
```

Readback result:

```text
final_field_jaxincell_centered.nc  41404 bytes, sizes {'x': 1200}, vars ['Ey','Bz']
final_field_leftgoing_xlt0.nc      17623 bytes, sizes {'x': 600}, var ['Ey_minus_cBz_over_2'], x range -2.398e-05 to -2.0e-08 m
final_field_rightgoing_xgt0.nc     17952 bytes, sizes {'x': 600}, var ['Ey_plus_cBz_over_2'], x range  2.0e-08 to  2.398e-05 m
run_summary.json                   3613 bytes, includes directional_netcdf paths
```

## Addendum: corrected reflection/transmission NetCDF split

Date: 2026-07-15 EDT. Telegram: Zhiping correction `main:6420513923:280`; acknowledgement `:281`.

Zhiping clarified that `run_jaxincell_final_field.py` should **not** compute or store `(Ey-cBz)/2` and `(Ey+cBz)/2`. Instead it should generate `reflection.nc` and `transmission.nc` in the style of tiger-vis:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/final_field_1D.py
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_1D.py
```

`run_jaxincell_final_field.py` should only finish centering `Ey/Bz`, select the reflection/transmission spatial windows, flip the reflection coordinates/fields, and write NetCDFs. The `(Ey+cBz)/2` forward-wave projection belongs in the downstream spectrum-analysis step (`spectrum_1D.py`), which is intended to be common to EPOCH and JAX-in-Cell outputs.

Remote state after correction:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/run_jaxincell_final_field.py
/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/README.md
```

Backups:

```text
# temporary directional-output state, before replacing with reflection/transmission
run_jaxincell_final_field.py.bak_reftrans_20260715_112358
README.md.bak_reftrans_20260715_112358

# clean pre-directional base used by the correction patch
run_jaxincell_final_field.py.bak_directional_20260715_110155
README.md.bak_directional_20260715_110259
```

New env-default output/window controls:

```python
REFLECTION_NC_NAME = os.environ.get("REFLECTION_NC_NAME", "reflection")
TRANSMISSION_NC_NAME = os.environ.get("TRANSMISSION_NC_NAME", "transmission")
REFLECTION_X_MAX_LAMBDA = float(os.environ.get("REFLECTION_X_MAX_LAMBDA", "-1.0"))
TRANSMISSION_X_MIN_LAMBDA = float(os.environ.get("TRANSMISSION_X_MIN_LAMBDA", "1.0"))
```

Split convention implemented from `final_field_1D.py`:

```python
reflection_mask = x_center <= REFLECTION_X_MAX_LAMBDA * lambda0
transmission_mask = x_center >= TRANSMISSION_X_MIN_LAMBDA * lambda0

x_refl  = -x_center[reflection_mask][::-1]
Ey_refl = -Ey_center[reflection_mask][::-1]
Bz_refl =  Bz_center[reflection_mask][::-1]
```

Current runner writes directly through `EM_analyzer.read_write.write_fields_to_nc`, with no `.npz` intermediary and no one-way `(Ey±cBz)/2` files:

```text
final_field_jaxincell_centered.nc  # full centered Ey/Bz
reflection.nc                      # Ey/Bz in flipped reflection coordinate
transmission.nc                    # Ey/Bz in +x transmission coordinate
run_summary.json                   # includes split_netcdf paths and split_outputs metadata
```

Validation smoke test:

```bash
cd /scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d
export CELLS_PER_LAMBDA=20
export NE_MACRO=120
export STEPS=2
export OUTDIR=outputs/smoke_reftrans
CUDA_VISIBLE_DEVICES=0 /scratch/network/zl8336/.conda/envs/GPU_Python/bin/python run_jaxincell_final_field.py
```

Readback result:

```text
final_field_jaxincell_centered.nc  41404 bytes, sizes {'x': 1200}, vars ['Ey','Bz']
reflection.nc                      26445 bytes, sizes {'x': 580}, vars ['Ey','Bz'], x range 8.2e-07 to 2.398e-05 m
transmission.nc                    26672 bytes, sizes {'x': 580}, vars ['Ey','Bz'], x range 8.2e-07 to 2.398e-05 m
run_summary.json                   4006 bytes, includes split_netcdf and split_outputs
```

Verification grep confirmed no `Ey_minus`, `Ey_plus`, `leftgoing`, `rightgoing`, or `cBz_over` references remain in the current runner/README.

## Addendum: adroit spectrum_1D utility for JAX-in-Cell outputs

Date: 2026-07-15 EDT. Telegram: Zhiping request `main:6420513923:283`; dependency authorization `:286`.

Zhiping asked to copy/adapt tiger-vis:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_1D.py
```

onto adroit-vis under:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/utility/spectrum_1D.py
```

with defaults aligned to the current `jaxincell_epoch1d` work. Implemented adroit changes:

- `sys.path` now points to `/scratch/network/zl8336` and its `EM_analyzer` paths.
- Default working directory:

```python
WORKING_DIR=/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/outputs/jaxincell_final_field
SIDE=reflection
FIELD_NAME=E_forward
LASER_A0=20.0
THETA_DEGREE=45.0
LASER_LAMBDA_UM=0.8
LASER_FWHM_FS=8.0
```

- Parameters use `os.environ.get(...)` style so Zhiping can edit defaults in VSCode and agents can override with `export`.
- The script remains the common downstream analyzer: it reads `reflection.nc` or `transmission.nc`, computes `E_forward=(Ey+cBz)/2`, writes `<side>_spectrum.nc`, `<side>_Sx.nc`, and plots `<side>_E_forward.png`, `<side>_spectrum.png`, `<side>_Sx.png`, `<side>_Ey.png`.

Initial smoke test on `outputs/smoke_reftrans` failed because adroit `GPU_Python` lacked `interpax`, imported by `EM_analyzer.coordinate_transformation` during `EM_analyzer.spectrum` import. After Zhiping authorized installation, ran:

```bash
/scratch/network/zl8336/.conda/envs/GPU_Python/bin/python -m pip install interpax
```

Installed `interpax 0.3.14` plus dependencies `equinox 0.13.8`, `jaxtyping 0.3.11`, `lineax 0.1.0`, `wadler-lindig 0.1.7`.

Smoke validation on `outputs/smoke_reftrans` ran `SIDE=reflection` and `SIDE=transmission`; both completed. Readback:

```text
reflection_spectrum.nc     size 25322, sizes {'f_over_f0_M': 1168}, vars ['E_forward_spectrum_intensity']
transmission_spectrum.nc   size 24207, sizes {'f_over_f0_M': 1168}, vars ['E_forward_spectrum_intensity']
reflection_Sx.nc           size 17509, sizes {'x': 580}, vars ['Sx']
transmission_Sx.nc         size 17468, sizes {'x': 580}, vars ['Sx']
```

The same smoke run also generated the expected PNG plots for both reflection and transmission. The previous combined validation command had a final here-doc quoting typo in the readback snippet, but the spectrum script itself had already completed and created the files; a corrected readback command verified the outputs.

## Addendum: FWHM_T0 and CEP controls in runner

Date: 2026-07-15 EDT. Telegram: Zhiping `main:6420513923:289`; acknowledgement `:290`.

Zhiping requested that `/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/run_jaxincell_final_field.py` read the laser FWHM from an environment variable in units of `T0`, and add `phi_cep` (default 0) inside the carrier cosines in both `Ey_func` and `Bz_func`.

Implemented in the runner and README:

```python
FWHM_T0 = float(os.environ.get("FWHM_T0", "1.8737029"))  # laser FWHM duration in units of T0
PHI_CEP = float(os.environ.get("PHI_CEP", "0.0"))      # carrier-envelope phase in radians
```

Important: the current script's pre-patch literal was `FWHM = 5e-15`, not 8 fs. The default `FWHM_T0=1.8737029` preserves that 5 fs value for `lambda0=0.8 um` (`T0=lambda0/c≈2.6685 fs`). This corrects my initial acknowledgement assumption that the old default was 8 fs.

The physics block now uses:

```python
T0 = lambda0 / c
FWHM = FWHM_T0 * T0
phi_cep = PHI_CEP
```

and both carrier functions use:

```python
np.cos(k0_M * (x - CENTER_LAMBDA * lambda0) + phi_cep) * envelope(x)
```

`run_summary.json` now records `FWHM_T0`, `FWHM_s`, and `phi_cep_rad` under `physics`.

Backups were written with suffix:

```text
.bak_fwhm_phi_20260715_154703
```

Validation smoke test:

```bash
cd /scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d
export CELLS_PER_LAMBDA=20
export NE_MACRO=120
export STEPS=2
export FWHM_T0=1.8737029
export PHI_CEP=0.25
export OUTDIR=outputs/smoke_fwhm_phi
CUDA_VISIBLE_DEVICES=0 /scratch/network/zl8336/.conda/envs/GPU_Python/bin/python run_jaxincell_final_field.py
```

Readback:

```text
physics.FWHM_T0     = 1.8737029
physics.FWHM_s      = 5.000000100069229e-15
physics.phi_cep_rad = 0.25
final_field_jaxincell_centered.nc sizes {'x': 1200}, vars ['Ey','Bz']
reflection.nc                     sizes {'x': 580},  vars ['Ey','Bz']
transmission.nc                   sizes {'x': 580},  vars ['Ey','Bz']
```

## Addendum: initial-condition diagnostic plot

Date: 2026-07-15 EDT. Telegram: Zhiping `main:6420513923:292`; acknowledgement `:293`.

Zhiping asked to plot initial field, density, and electron/ion velocities in the style of:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/epoch1d_profiles_per_T0.py
```

Implemented directly in `run_jaxincell_final_field.py` so the plot uses the same initialization as the actual run. New env-default controls:

```python
PLOT_INITIAL = os.environ.get("PLOT_INITIAL", "1").strip().lower() not in ("0", "false", "no", "off")
INITIAL_PLOT_NAME = os.environ.get("INITIAL_PLOT_NAME", "initial_profiles.png")
```

When enabled, the runner writes `initial_profiles.png` and records its path as `initial_plot` in `run_summary.json`. The figure is a 3×3 layout like the benchmark:

- field panel: `Ey/Ec` and `cBz/Ec`;
- density panel: `ne/Nc` and `ni/Nc` from full-domain histograms;
- metadata panel with `D`, `N`, `theta`, `FWHM_T0`, `phi_cep`;
- electron `vx/c`, `vy/c`, `vz/c` panels;
- ion `vx/c`, `vy/c`, `vz/c` panels.

Backups were written with suffix:

```text
.bak_initial_plot_20260715_160529
```

Validation:

```bash
cd /scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d
export STEPS=0
export OUTDIR=outputs/initial_plot_default
CUDA_VISIBLE_DEVICES=0 /scratch/network/zl8336/.conda/envs/GPU_Python/bin/python run_jaxincell_final_field.py
```

This generated a default-resolution initial diagnostic without time stepping:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/outputs/initial_plot_default/initial_profiles.png 122K
run_summary.json includes initial_plot=outputs/initial_plot_default/initial_profiles.png
```

Local copy sent/available at:

```text
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260715/initial_profiles_default.png
```

Vision check confirmed the PNG is a readable 3×3 diagnostic with field, density, metadata, electron velocity, and ion velocity panels; no obvious rendering corruption.

## Addendum: domain and laser parameterization cleanup

Date: 2026-07-15 EDT. Telegram: Zhiping `main:6420513923:295`; acknowledgement `:296`.

Zhiping requested:

- `CENTER_LAMBDA` should not be an independent env parameter; use `4*FWHM_T0` for the pulse-center distance.
- Do not use `L_LAMBDA`; specify `x_min_lambda` and `x_max_lambda` separately.
- `N_PERIODS` should be derived from `x_min_lambda` (one light-transit scale), not separately specified.
- `a0` and `theta` should be read from environment/default strings.

Implemented in `run_jaxincell_final_field.py` and README:

```python
X_MIN_LAMBDA = float(os.environ.get("X_MIN_LAMBDA", "-15"))
X_MAX_LAMBDA = float(os.environ.get("X_MAX_LAMBDA", "15"))
A0 = float(os.environ.get("A0", "20.0"))
THETA_DEGREE = float(os.environ.get("THETA_DEGREE", "45.0"))
N_PERIODS = abs(X_MIN_LAMBDA)
```

The defaults preserve the current edited box length (`L_LAMBDA=30`) as `[-15,15]`. The runner now derives:

```python
center_lambda = -4.0 * FWHM_T0     # left of target
L_lambda = X_MAX_LAMBDA - X_MIN_LAMBDA
Nx = round(L_lambda * CELLS_PER_LAMBDA)
Lbox = L_lambda * lambda0
x_Ey = X_MIN_LAMBDA*lambda0 + (i+0.5)*dx
x_Bz = X_MIN_LAMBDA*lambda0 + i*dx
```

The sign choice for `center_lambda` was made because the incident pulse is on the left (`x<0`); `4*FWHM_T0` is treated as a distance from the target. `STEPS` still overrides the derived duration for smoke/debug runs.

Backups were written with suffix:

```text
.bak_domain_params_20260715_163829
```

Validation smoke test:

```bash
export CELLS_PER_LAMBDA=20
export X_MIN_LAMBDA=-8
export X_MAX_LAMBDA=8
export FWHM_T0=1.5
export PHI_CEP=0.2
export A0=7
export THETA_DEGREE=30
export NE_MACRO=120
export STEPS=2
export OUTDIR=outputs/smoke_domain_params
CUDA_VISIBLE_DEVICES=0 /scratch/network/zl8336/.conda/envs/GPU_Python/bin/python run_jaxincell_final_field.py
```

Readback:

```text
grid: x_min=-8, x_max=8, L=16, center=-6, cells_per_lambda=20, Nx=320, N_PERIODS=8, steps=2
physics: a0=7, theta_deg=30, FWHM_T0=1.5, phi_cep_rad=0.2
outputs: initial_profiles.png, final_field_jaxincell_centered.nc, reflection.nc, transmission.nc
final_field x range: -6.38e-06 to 6.38e-06 m, sizes {'x': 320}
```

## Addendum: CELLS_PER_LAMBDA convergence scan and XLA memory-cap test

Date: 2026-07-15 EDT. Telegram request: `main:6420513923:298`; OOM question: `:301`; reports: `:302`, `:303`, `:304`, `:305`.

Zhiping requested a convergence scan: run `/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/run_jaxincell_final_field.py`, then use `/scratch/network/zl8336/try_PIC_GPU_JAX/utility/spectrum_1D.py` for reflection/transmission spectra, sharing `LASER_A0`, `FWHM_T0`, and `THETA_DEGREE`; scan `CELLS_PER_LAMBDA` from 100 upward until OOM; analyze reflection/transmission `Sx` peaks and HHG efficiency printed by `spectrum_1D.py` line 99.

Driver created and kept at:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/run_cpl_convergence_scan.py
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260715/run_cpl_convergence_scan.py
```

Original scan root:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/outputs/cpl_convergence_20260715_1927
```

Original environment inherited `XLA_PYTHON_CLIENT_MEM_FRACTION=.50`. It completed `CELLS_PER_LAMBDA=100,150,200,300,400,600`, then failed at `800` with:

```text
input/output arguments = 42,951,705,784 bytes ≈ 40.00 GiB
base limit             = 42,559,602,688 bytes ≈ 39.64 GiB
```

`nvidia-smi` showed adroit-vis has A100 80GB GPUs, so the ~39.64 GiB base limit was the `.50` JAX memory-fraction cap, not physical GPU size.

Continuation root with higher process memory fraction:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/outputs/cpl_convergence_20260715_1944_mem90
CUDA_VISIBLE_DEVICES=1
XLA_PYTHON_CLIENT_MEM_FRACTION=.90
```

This passed `CELLS_PER_LAMBDA=800` and `1000`, then failed at `1200`:

```text
input/output arguments = 85,517,492,392 bytes ≈ 79.64 GiB
base limit             = 76,607,282,809 bytes ≈ 71.35 GiB
rematerialization could not reduce below ~79.64 GiB
later allocation failed: 17.70 GiB buffer
```

Combined report artifacts:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/outputs/cpl_convergence_combined_20260715_2026/
  combined_convergence_report.html
  combined_convergence_metrics.png
  combined_metrics.tsv
  raw_metrics_with_failures.tsv
  last_step_relative_changes.tsv
  combined_convergence_report.ipynb
  summary_combined.json
```

Local copies:

```text
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260715/cpl_convergence_combined/
```

Key combined successful points:

```text
.50 run:  cpl=100,150,200,300,400,600
.90 run:  cpl=800,1000
fail:     cpl=1200 OOM
```

800→1000 relative changes:

```text
reflection HHG_eff: 0.3136616 -> 0.3014017   (-3.91%)
reflection Sx_peak: 1.42877e25 -> 1.44950e25 (+1.45%)
transmission HHG_eff: 0.0924890 -> 0.0894219 (-3.32%)
transmission Sx_peak: 8.99833e24 -> 6.41294e24 (-28.7%)
```

Interpretation reported to Zhiping: reflection-side HHG efficiency and `Sx` peak look relatively stable by `cpl=800–1000` (few-percent scale), transmission HHG efficiency also changes only a few percent, but transmission `Sx_peak` is still sensitive to resolution/local peak structure. The `cpl=1200` failure is not merely the old `.50` cap; the compiled JAX/XLA input/output buffer itself is already near/above 80GB scale. To go beyond this on A100 80GB, the code must reduce the compiled memory footprint (e.g. chunk timesteps/restart, ensure final-state-only scan outputs, lower domain/time, or use lower precision where physically acceptable), or use larger memory hardware.
