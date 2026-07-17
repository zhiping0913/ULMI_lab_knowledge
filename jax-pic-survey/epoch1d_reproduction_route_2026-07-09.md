# EPOCH 1D → JAX-in-Cell / PyPIC3D reproduction route (2026-07-09)

Source task: Zhiping Telegram `main:6420513923:171`; answer delivered in Telegram `main:6420513923:173-174` with standalone HTML artifact `/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax-pic-repro-20260709/epoch1d_reproduction_route.html`.

## Evidence read

- adroit-vis reports:
  - `/scratch/network/zl8336/try_PIC_GPU_JAX/report.html`
  - `/scratch/network/zl8336/try_PIC_GPU_JAX/field_init_guide.md`
- adroit-vis code/tests:
  - `/scratch/network/zl8336/try_PIC_GPU_JAX/test_jaxincell_wave.py`
  - `/scratch/network/zl8336/try_PIC_GPU_JAX/test_pypic3d_wave.py`
  - `/scratch/network/zl8336/try_PIC_GPU_JAX/test_pypic3d_gpu.py`
  - JAX-in-Cell source for species/field initialization and deposition.
  - PyPIC3D source for particle initialization, `.npy` field loading, boundaries/PML.
- tiger-vis target:
  - `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/1D/45/ND_a0=0.30/input.deck`
  - nearby existing outputs/logs for consistency checking.

## Software status

- JAX-in-Cell on adroit-vis: JAX 0.10.0 CUDA, 2×A100 80GB visible, 9/9 examples passed, 252/252 tests passed. Two-stream example ~694 it/s. It is 1D3V and supports `initial_positions`, `initial_velocities`, `drift_speed_y`, per-species `weight`, and patched `_initial_E_field` / `_initial_B_field` evolved-field injection.
- PyPIC3D on adroit-vis: 2×A100 visible but GPU must be forced before import; 2/2 demos passed; 86/87 tests passed with one PML first-JIT timeout. Two-stream example ~107 it/s. Heavier, but explicit `.npy` field and `initial_vx/vy/vz`, `initial_x/y/z` particle APIs make it a good cross-check.

## Current input.deck physics

Use the actual deck, not the parent path name:

- Path says `a0=20/.../ND_a0=0.30`, but current deck has `laser_a0=3`.
- `λ0=0.8 µm`, `T0=2.6685 fs`, `Ec=4.013e12 V/m`, `Bc=1.339e4 T`, `Nc=1.742e27 m^-3`.
- `theta=45°`; Bourdier/moving-frame quantities: `λM=λ/cosθ`, `kM=k cosθ`, `τM=τ/cosθ`, `EampM=a0 Ec cosθ≈8.51e12 V/m`, `BampM≈2.84e4 T`, `n_eM=350 Nc/cosθ≈8.62e29 m^-3`.
- Target: `N=350`, `D=0.002571λ0`, `L=0`, thickness ≈2.06 nm, `N*D=0.9`, `ND/a0=0.30`.
- Current deck domain: `x_min=-25λ0`, `x_max=3.002571λ0`, `cells_per_lambda=4000`, so `nx≈112010`, `dx=0.2 nm`, target ≈10.3 cells, macro electrons ≈2057, macro ions ≈343.
- Initial laser is analytical fields in the deck (`fields` block), not a laser boundary injection: Ey/Bz pulse centered at `x_min/2`, truncated for `x<-λ0`; `laser_pol=0`, so Ey/Bz only.
- Boundaries: `simple_outflow` both sides in EPOCH.
- Species: electron and charge-6 ion slab. EPOCH `drift_y=-m_s c tanθ` is momentum-like Bourdier-frame drift; when mapping to APIs that expect velocity, use `v_y=-c sinθ≈-0.707c`, not `-c tanθ`.
- Collisions: Nanbu collisions enabled every 10 steps. JAX-in-Cell and PyPIC3D do not currently supply an equivalent collision operator, so first benchmark should be collisionless.

## Reference consistency caveat

Existing outputs in the target directory appear inconsistent with the current `input.deck`:

- Current deck implies `nx≈112010` and `t_end=25T0≈66.7 fs`.
- Existing `final_field.nc` has dimension `x=160000`.
- Existing `slurm-3146506.out` final time is ~`0.5337e-13 s≈20T0` and 84210 iterations.

Therefore: quantitative comparison to existing `reflection_spectrum.nc` / `reflection.nc` requires either locating the original deck that produced those outputs or rerunning EPOCH from the current deck after authorization. Until then, the JAX route should be described as reproducing the current input physics, not necessarily matching the existing processed output files.

## Route

1. Freeze reference: copy current deck/reports, decide whether to compare current deck or old outputs.
2. Implement shared initializer under adroit-vis, e.g. `/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/`:
   - Convert centered code coordinate to EPOCH coordinate via `x_epoch = x_code + 0.5*(x_min+x_max)*λ0`.
   - Generate exact deck Ey/Bz pulse on each code's Yee grid.
   - Generate electron/ion slab arrays in `|x_epoch|<Dλ0/2` with moving-frame velocity `v_y=-c sinθ`.
   - Set explicit macro weights matching `n_eM` (JAX 1D planar weight; PyPIC3D with `Ny=Nz=1`, `y_wind=z_wind=λ0`).
3. Stage 1: vacuum pulse propagation, no particles, to validate field staggering/boundary/propagation direction.
   - JAX-in-Cell: E at cell centers, B at vertices; use patched `_initial_E/B_field`.
   - PyPIC3D: save `Ey_init.npy` / `Bz_init.npy`, test Bz declared `(i+0.5)dx` and effective `(i-0.5)dx`; field guide found `(i-0.5)dx` minimizes backward component for pure field propagation, but particle-coupled runs require care.
4. Stage 2: cold neutral slab, collisionless.
   - Use absorbing/PML/outflow-like boundaries where possible.
   - Do not compare collision-on EPOCH as exact truth unless a collision-off EPOCH reference is available.
5. Stage 3: diagnostics and comparison.
   - Generate `final_field.nc`, `reflection.nc`, `transmission.nc`, spectra, and energy metrics in EM_analyzer-compatible normalization.
   - Compare waveform (`Ey/Ec`, `Bz/Bc`), reflected/transmitted energy, spectrum slope/cutoff, integrated HHG efficiency thresholds, energy conservation, and particle loss.
6. Stage 4: convergence.
   - Start reduced/resolution tests, then approach `cells_per_lambda=4000`; JAX-in-Cell full-resolution cannot return full time history (`~1e5 steps × 1.12e5 cells × 3`) without final-only / sparse diagnostics patch.

## Recommendation

If Zhiping asks to proceed, first implement the shared initializer + vacuum pulse tests on adroit-vis. Do not submit Slurm for EPOCH rerun unless explicitly authorized and after `checkquota`.

## 2026-07-11 correction and new test authorization

Telegram `main:6420513923:198`: Zhiping clarified that the previous `laser_a0=3` in `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/1D/45/ND_a0=0.30/input.deck` was an accidental overwrite by an `a0=3` scan input. He has now corrected all `input.deck` files under `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/1D/45`; the existing simulation outputs are still valid `a0=20` outputs and were not affected. The corrected `ND_a0=0.30/input.deck` may now be used as the initial-condition reference for adroit-vis JAX-in-Cell / PyPIC3D testing.

Additional reference from Zhiping: `/scratch/network/zl8336/try_PIC_GPU_JAX/pypic3d_Bz_offset_sweep.ipynb`, where he tested adding initial fields (`E`, `B` on Yee-grid offsets) in both JAX-in-Cell and PyPIC3D; the two codes use different offsets. Next active task is to inspect this notebook and the corrected deck, then run a minimal sanity benchmark in `/scratch/network/zl8336/try_PIC_GPU_JAX` on adroit-vis (no Slurm): shared initial fields/particles, vacuum propagation, then short collisionless slab test if the field initialization is sound.
