# Adroit-vis EPOCH a0=20 initialization sanity benchmark (2026-07-11)

Trigger: Zhiping Telegram `main:6420513923:198`; report sent in `main:6420513923:200`; HTML attachment sent in `main:6420513923:201`.

## Scope

No Slurm. This was a reduced-grid sanity check on `adroit-vis`, not a production EPOCH-equivalent run. Purpose: verify that the corrected a0=20 EPOCH 1D deck can be converted into JAX-in-Cell and PyPIC3D initial fields with the correct code-specific Yee offsets, and that the resulting pulse propagates as a nearly pure +x wave.

Important limitation: the EPOCH reference deck has Nanbu collisions enabled; JAX-in-Cell/PyPIC3D tests here omit collisions. Do not claim collision-on EPOCH reproduction from this benchmark.

## Paths

Remote benchmark directory:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark
```

Key files:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/reference_input.deck
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/epoch1d_initialization_sanity.py
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/summary.json
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/epoch1d_initialization_sanity_report.html
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/figures/
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/arrays/
```

Local copies for Telegram/artifact access:

```text
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260711/epoch1d_initialization_sanity_report.html
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260711/epoch1d_benchmark_summary.json
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260711/deck_initial_field_reduced.png
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260711/jaxincell_deck_pulse_vacuum_like.png
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260711/pypic3d_deck_pulse_vacuum.png
```

## Access / environment notes

- `adroit-vis` has 2 × NVIDIA A100 80GB visible.
- `adroit-vis` could not directly `stat/read` `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/1D/45/ND_a0=0.30/input.deck` (`PermissionError`). Workaround: copied the corrected deck via `tiger-vis` to local artifact, then `scp` to adroit benchmark directory as `reference_input.deck`.
- Default `python3` on adroit was `/scratch/network/zl8336/.conda/envs/PyTorch_Python/bin/python3` and lacks `jax`/`matplotlib`. The working environment for these tests is:

```text
/scratch/network/zl8336/.conda/envs/GPU_Python/bin/python
```

This has JAX 0.10.0 and ran both JAX-in-Cell and PyPIC3D tests.

## Deck-derived physics

Corrected deck facts used:

```text
laser_a0 = 20
theta = 45 deg
lambda0 = 0.8 um
D/lambda0 = 0.017142857142857144
x range = [-20, 20] lambda0
full EPOCH resolution = 4000 cells/lambda0
Nanbu collisions enabled in EPOCH deck
```

Moving-frame/Bourdier field amplitude is `a0 cos(theta)`, so the initial Ey/Bz peak in normalized units should be about:

```text
a0 cos(45 deg) = 14.142
```

The reduced-grid test used `25 cells/lambda0` and `Nx=1000` to test staggering/propagation quickly; this is not convergence.

Velocity mapping preserved:

```text
EPOCH drift_y is momentum-like: -m_s c tan(theta)
JAX/PyPIC velocity API should receive v_y = -c sin(theta) = -0.70710678 c
```

## Offset rules verified

- JAX-in-Cell: `Ey` at cell center, `Bz` at vertex, so `x_B = x_E - 0.5 dx`.
- PyPIC3D: the offset notebook and source-level argument imply an effective Bz sampling shift

```text
shift_coef = -(0.5 + 0.5*CFL)
```

For the test CFL=0.99, this gives:

```text
shift_coef = -0.995 dx
```

## Results

Directional purity was measured with the decomposition for a +x wave (`Ey = c Bz`): the left-going fraction is the energy in `(Ey - cBz)/2` after interpolating B to E positions.

```text
JAX-in-Cell:
  initial left-going energy fraction = 3.998e-6
  final after 5.00 lambda0 travel     = 2.514e-6
  final Ey peak / Ec                  = 14.103
  final Bz peak / Bc                  = 14.062
  runtime                             = 9.27 s

PyPIC3D:
  initial left-going energy fraction = 8.677e-9
  final after 4.9896 lambda0 travel  = 8.815e-9
  final Ey peak / Ec                 = 14.126
  final Bz peak / Bc                 = 14.124
  runtime                            = 3.30 s
```

Interpretation: both code-specific offset rules give a nearly pure +x-propagating deck-derived pulse. PyPIC3D's `~ -1 dx` Bz offset is not an arbitrary empirical hack; it follows from spatial Yee staggering plus the half-step time-level mismatch in its loaded initial B field.

## Particle setup status

The benchmark wrote initial slab API arrays, but did not yet claim a physical collisionless slab evolution:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/arrays/initial_particles_slab_api_arrays.npz
```

Arrays use electron macro count 200 and ion macro count 33, with `v_y=-0.707c`. This only validates setup direction and file/API plumbing. A meaningful slab test still needs agreement on macroparticle weights, boundaries, and whether to compare to collision-off EPOCH.

## Next step

If Zhiping wants to continue, run a short collisionless slab test next, but keep claims limited:

1. Use the same corrected deck fields and `v_y=-c sin(theta)`.
2. Decide/validate macroparticle weights for both JAX-in-Cell and PyPIC3D.
3. Use no collisions and compare only to a collision-off EPOCH reference or label it as a model-difference test.
4. Full-resolution/spectrum comparisons need sparse/final diagnostics; naive full time-history at EPOCH resolution is too large.
