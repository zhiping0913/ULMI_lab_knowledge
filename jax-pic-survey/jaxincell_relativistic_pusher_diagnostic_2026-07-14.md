# JAX-in-Cell vs PyPIC3D discrepancy: electron blowout after laser arrival — 2026-07-14

Source messages:
- Zhiping Telegram `main:6420513923:251`: while K=0 order-scan jobs run, inspect adroit-vis JAX-in-Cell / PyPIC3D profiles. Zhiping had increased `Ne_macro` and `cells_per_lambda` in `/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/epoch1d_profiles_per_T0.py`. PyPIC3D looked closer to EPOCH; JAX-in-Cell looked wrong, with almost all electrons pushed out of target after laser interaction, leaving ions at target.
- main ack `main:6420513923:252`.
- Zhiping clarification `main:6420513923:253`: electrons are not initially misplaced; before laser arrives, electrons and ions are both on target. Blowout occurs after the laser hits target.
- main diagnostic report `main:6420513923:254`.

## Files inspected

Remote adroit workdir:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/
```

Current script:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/epoch1d_profiles_per_T0.py
```

Current output directory:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/profiles_per_T0/
```

Representative figures copied locally for vision inspection:

```text
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260714/jaxincell_profiles_t021_21T0.png
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260714/pypic3d_profiles_t021_21T0.png
```

Focused JAX relativistic probe output:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/relativistic_probe_20260714/jax_relativistic_probe_summary.json
```

## Current script parameters

From `epoch1d_profiles_per_T0.py` and `profiles_per_T0_summary.json`:

```text
a0 = 20
theta = 45°
N_over_nc = 350
D_lambda = 0.017142857142857144
cells_per_lambda = 400
Nx = 24000
L_lambda = 60
CFL = 0.9
steps_per_T0 = 444
n_periods = 30
steps = 13320
Ne_macro = 12000
Ni_macro = 2000
macro_weight_line = 9.854012203617695e17
macro_weight_volume = 630656.7810315323
vy/c = -0.7071067811865475
target_cells = 6.857142857142857
```

Important resolution caveat: with `cells_per_lambda=400`, the target thickness is only `~6.9` grid cells. The EPOCH deck has `dx≈0.2 nm≈λ0/4000`, so the same target is `~68` cells thick in EPOCH. Current JAX/PyPIC diagnostics are still a reduced-grid test, about 10× coarser than EPOCH.

## Primary bug found: JAX run omitted `relativistic=True`

Current JAX-in-Cell parameters in the script contained only:

```python
'solver_parameters': {'field_solver': 0, 'print_info': False}
```

JAX-in-Cell defaults inspected from `_parameters/_solver_parameters.py` include:

```text
time_evolution_algorithm = 0
field_solver = 0
relativistic = False
```

Therefore the current JAX-in-Cell run used the **nonrelativistic Boris pusher** for an `a0=20` laser-solid interaction. PyPIC3D, by contrast, was explicitly configured with:

```python
'relativistic': True
```

Vision inspection of the JAX `t=21T0` figure showed electron velocity panels with `vx/c` and `vy/c` reaching tens, which is physically impossible for true velocities. Source inspection of JAX-in-Cell's relativistic pusher showed `out['velocities']` should be true velocities when the relativistic path is selected, not normalized momentum. Thus superluminal velocities in the current figure are a real pusher-setting problem.

## Density diagnostics from existing NPZs

A diagnostic script read the existing `profiles_per_T0/data/*.npz` files and computed total charge, target electron retention, and left/right electron fractions.

Key results:

- Total electron charge and ion charge were conserved in both codes (`e_tot/e0≈1`, `ionQ/e0≈1`), so particles were not numerically lost.
- JAX-in-Cell electrons remain near target before laser arrival: `e_near≈1` through `~12T0`.
- After laser interaction, JAX electrons are displaced strongly: by `16T0`, `e_near≈0.392`; by `20T0`, `e_near≈0` and electrons are split left/right.
- PyPIC3D also shows target electron depletion in this reduced run, but less abruptly: e.g. `e_near≈0.327` at `16T0`, `0.208` at `20T0`.

This supports Zhiping's clarification: **initial co-location is fine; the problem occurs after laser-target interaction.** It is charge-separation / blowout dynamics plus numerical settings, not an initialization offset.

## Relativistic JAX probe

I ran a focused JAX-only probe with the same current setup but:

```python
'solver_parameters': {
    'field_solver': 0,
    'time_evolution_algorithm': 0,
    'relativistic': True,
    'print_info': False,
}
```

Output:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/relativistic_probe_20260714/jax_relativistic_probe_summary.json
```

Results:

- Runtime to `21T0`: ~210 s.
- Electron speeds were correctly limited: `max_speed_e/c <= 1` throughout; at `21T0`, `max_speed_e/c≈0.999`.
- However, strong electron displacement still occurred after laser arrival:
  - electrons stayed near target through `~12T0`;
  - by `15–18T0`, `e_target≈0` and electrons were mostly pushed to one side;
  - by `21T0`, electrons were spread left/right with `e_target≈0.008`, `e_near≈0.292`.

Conclusion: `relativistic=True` is **necessary** and fixes the unphysical superluminal velocity bug, but it is **not sufficient** to guarantee EPOCH-like behavior in the current reduced setup.

## Physical / numerical interpretation

The current JAX result is not trustworthy mainly because of the missing relativistic pusher. At `a0=20`, nonrelativistic Boris can accelerate electrons to unphysical speeds and produce exaggerated blowout.

After that is fixed, remaining differences can still be real/numerical mixture:

1. **Reduced-grid target is coarse.** `D≈0.017λ0` is only `6.9` cells at `400 cells/λ0`, compared with `~68` cells in EPOCH. Skin-depth / charge-separation layer and surface current are under-resolved.
2. **Ultrathin foil physics is near blowout/light-sail regime.** For `ND/a0=0.30`, strong charge separation after laser arrival is not automatically impossible; the question is quantitative comparison to EPOCH.
3. **Boundary and physics mismatch remains.** Current tests are collisionless and use code-specific boundary/field evolution conventions; EPOCH reference may include collisions and finer boundary treatment.
4. **JAX line-density weight needs convergence/validation.** Prior smoke tests showed JAX-in-Cell needs 1D line-density normalization, unlike PyPIC3D's volume weight. The current line weight conserves total charge, but quantitative agreement still needs weight/resolution scans.
5. **Profile moment diagnostics need masking.** Velocity averages in low-density bins can look noisy. Current JAX superluminal velocities are mainly pusher-related, but even PyPIC3D velocity plots have low-density moment artifacts.

## Recommended fix / next steps

Immediate code fix before any comparison:

```python
'solver_parameters': {
    'field_solver': 0,
    'time_evolution_algorithm': 0,
    'relativistic': True,
    'print_info': False,
}
```

Then rerun **JAX only** to `20–21T0` first, with diagnostics:

- `max(|v|/c)` for electrons and ions;
- total electron charge and ion charge;
- electron fraction near target, left/right fractions;
- comparison of `ne/Nc`, `Ey/Ec`, and phase-space/moments with PyPIC3D.

After that, perform convergence/robustness tests before claiming EPOCH agreement:

1. `cells_per_lambda`: 400 → 800 → 1600, using sparse diagnostics only.
2. JAX line weight scale: `0.5×, 1×, 2×` around current `macro_weight_line`.
3. Boundary/solver settings: verify JAX field/particle boundary conventions match the intended EPOCH-like open/absorbing behavior as much as possible.
4. Compare only against a collisionless EPOCH reference or explicitly state collisions are absent.

Do **not** use the current nonrelativistic JAX profiles as evidence against JAX-in-Cell; they are invalid for `a0=20`.

## Patch applied after Zhiping approval

Zhiping Telegram `main:6420513923:255` explicitly requested patching `epoch1d_profiles_per_T0.py`'s JAX section to add `relativistic=True`. main applied the patch and reported in `main:6420513923:256`.

Patched file:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/epoch1d_profiles_per_T0.py
```

Backup:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/epoch1d_profiles_per_T0.py.bak_relativistic_20260714_2012
```

Patched JAX solver parameters:

```python
'solver_parameters': {
    'field_solver': 0,
    'time_evolution_algorithm': 0,
    'relativistic': True,
    'print_info': False,
}
```

Verification:

```text
/scratch/network/zl8336/.conda/envs/GPU_Python/bin/python -m py_compile epoch1d_profiles_per_T0.py
```

passed. No full profile rerun was performed at patch time, to avoid overwriting existing output. If rerunning, prefer a new output tag/directory such as `profiles_per_T0_relativistic/`, or clean stale old outputs first (the current `profiles_per_T0` directory contains stale files beyond the current `n_periods=30`, e.g. `t031–t040`).

## GPU-memory discussion: why `cells_per_lambda>=1000` OOMs

Zhiping Telegram `main:6420513923:257` asked why increasing `cells_per_lambda` to ~1000 causes both JAX-in-Cell and PyPIC3D to run out of GPU memory, despite having two 80GB A100s, and whether CPU memory can hold arrays while GPU computes. main replied in `main:6420513923:258`.

Main estimate for current full-history JAX-in-Cell approach:

```text
L = 60 λ0
cells_per_lambda = 1000 -> Nx = 60000
CFL = 0.9 -> steps_per_T0 ≈ 1111
30 T0 -> Nt ≈ 33330
```

A single complete field history costs

```text
Nt × Nx × 3 × float64
≈ 33330 × 60000 × 3 × 8 bytes
≈ 48 GB
```

Saving `E(t,x)`, `B(t,x)`, `positions(t,p)`, `velocities(t,p)`, plus XLA temporaries easily exceeds 160 GB. Two A100s do not automatically combine memory for one domain; JAX would need explicit sharding/domain decomposition, which this code does not implement. `pmap` is useful for running separate cases per GPU, not for automatically splitting one PIC domain.

CPU-memory plus GPU compute is also not the right primary solution: active JAX arrays live on device, and out-of-core CPU↔GPU transfers every step would be very slow. Environment variables can reduce preallocation/fragmentation but not solve arrays that genuinely exceed memory:

```bash
export XLA_PYTHON_CLIENT_PREALLOCATE=false
export XLA_PYTHON_CLIENT_MEM_FRACTION=0.85
```

Recommended solution: **sparse / streaming diagnostics**, not full history. Only per-`T0` profiles are needed, not every time step. Implement a new runner (do not overwrite the current script first), e.g.

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/epoch1d_profiles_sparse_jaxincell.py
```

Design:

1. Use the same initialization and `relativistic=True`.
2. Run in chunks of `1T0` or `0.5T0`.
3. After each chunk, extract only the final `E/B/positions/velocities`, save profile `.npz`/PNG and charge/speed diagnostics to host, then discard the chunk history.
4. Use final state as the next chunk's initial condition.
5. Write to a separate directory such as `profiles_per_T0_relativistic_sparse/`.

This changes memory scaling from `O(Nt × Nx)` to roughly `O(steps_per_snapshot × Nx)`, or ideally `O(Nx + Nparticles)` if JAX-in-Cell internals are later patched so `lax.scan` returns final carry only. For `cells_per_lambda=1000`, `1T0` chunks reduce one field history from ~48GB to ~1.6GB, making a single A100 feasible in principle. PyPIC3D should use the same principle; its manual loop is closer to streaming already, so if it still OOMs inspect JIT temporaries, dtype, and any hidden time-history diagnostics.
