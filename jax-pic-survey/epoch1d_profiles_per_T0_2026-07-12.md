# Per-T0 full-x diagnostic profiles for JAX-in-Cell and PyPIC3D (2026-07-12)

Trigger: Zhiping Telegram `main:6420513923:208` asked both PyPIC3D and JAX-in-Cell to output full-x plots every T0 for `Ey`, `ne`, `ni`, and electron/ion `vx, vy, vz`. Zhiping corrected the time window in `main:6420513923:210`: from initial pulse to reflection should be more than 10 cycles, not only 8. I corrected the setup and reported results in `main:6420513923:212`; artifacts sent in `main:6420513923:213-215`.

## Scope

Collisionless reduced diagnostic run, no Slurm. This is still not a converged EPOCH reproduction. The goal was to produce full-x diagnostic plots/data at every integer `T0` for both JAX-in-Cell and PyPIC3D.

Final corrected setup:

```text
x/lambda0 = [-20, 20]
initial pulse center = -10 lambda0
snapshots = t/T0 = 0, 1, ..., 20
resolution = 150 cells/lambda0
Nx = 6000
CFL = 0.5
steps_per_T0 = 300
total steps = 6000
collisions = omitted
foil thickness D/lambda0 = 0.017142857 => 2.57 cells in this reduced grid
```

Physics/normalization:

```text
a0 = 20
theta = 45 deg
v_y/c = -sin(45 deg) = -0.70710678
JAX-in-Cell macro_weight_line = 9.854012203617694e18
PyPIC3D macro_weight_volume = 6306567.810315324
target_Ne_M = 8.622260678165482e29 m^-3
```

The JAX weight is the line-density convention found in the previous reflection smoke test; PyPIC3D uses explicit 3D volume weight with transverse area `lambda0^2`.

## Paths

Remote workdir:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark
```

Remote outputs:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/epoch1d_profiles_per_T0.py
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/epoch1d_profiles_per_T0.ipynb
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/profiles_per_T0/profiles_per_T0_summary.json
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/profiles_per_T0/profiles_per_T0_artifacts.zip
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/profiles_per_T0/figures/
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/profiles_per_T0/data/
```

Local copies sent via Telegram:

```text
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260711/profiles_per_T0_20T/profiles_per_T0_artifacts.zip
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260711/profiles_per_T0_20T/profiles_per_T0_summary.json
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260711/profiles_per_T0_20T/epoch1d_profiles_per_T0.ipynb
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260711/profiles_per_T0_20T/epoch1d_profiles_per_T0.py
```

## Output count

```text
JAX-in-Cell: 21 PNG figures + 21 NPZ files, elapsed ~20.7 s
PyPIC3D:     21 PNG figures + 21 NPZ files, elapsed ~32.3 s
ZIP size:    8.05 MB
```

Each PNG is a 3x3 panel over the full x axis:

1. `Ey/Ec`
2. `ne/Nc`, `ni/Nc`
3. metadata box
4. electron `vx/c`
5. electron `vy/c`
6. electron `vz/c`
7. ion `vx/c`
8. ion `vy/c`
9. ion `vz/c`

Each NPZ contains arrays: `x`, `Ey`, `ne`, `ni`, `e_vx`, `e_vy`, `e_vz`, `i_vx`, `i_vy`, `i_vz`, `t_over_T0`.

## Implementation notes

- JAX-in-Cell returns full histories of fields, positions, and velocities; species are split by macro-particle counts.
- PyPIC3D uses a flat particle backend in this run: `particles` is `[flat_all]`, not `[electrons, ions]`. The diagnostic script had to split the flat arrays using `flat.species_meta[0]['N_particles']` and `flat.species_meta[1]['N_particles']`. The first run failed for PyPIC because it assumed `particles[0]`, `particles[1]`; the patched rerun succeeded.
- The reproducible notebook wrapper is intentionally simple and runs `epoch1d_profiles_per_T0.py`; the real editable logic is in the script.

## Caveats / next step

- Foil is only 2.57 cells thick at the reduced resolution. Do not infer production physics or harmonic spectra from this run.
- The current outputs are diagnostic plots/data for checking field/particle evolution and code behavior.
- If Zhiping wants quantitative comparison, next step is a resolution/particle-count scan and/or a collision-off EPOCH reference.
