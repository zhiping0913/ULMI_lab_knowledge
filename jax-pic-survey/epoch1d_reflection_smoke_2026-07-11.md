# Collisionless reflected-field smoke test: JAX-in-Cell / PyPIC3D (2026-07-11)

Trigger: Zhiping Telegram `main:6420513923:202`: ignore collisions and use JAX-in-Cell/PyPIC3D to test and look at the reflected field. Acknowledged in `main:6420513923:203`; report sent in `main:6420513923:204`; HTML/figures sent in `main:6420513923:205-207`.

## Scope

No Slurm. Collisionless reduced short-domain smoke test only. This was meant to see whether reflected/left-going fields appear when the corrected a0=20 deck-derived pulse hits a thin foil in each code. It is not a converged EPOCH comparison.

Reduced setup:

```text
x/lambda0 range = [-10, 6]
initial pulse center = -5 lambda0
cells_per_lambda = 150
Nx = 2400
dx = 5.333e-9 m
CFL = 0.5
steps = 2400
travel = 8 lambda0
foil thickness D/lambda0 = 0.017142857 => 2.57 cells only
collisions omitted by request
```

Deck physics retained:

```text
a0 = 20
theta = 45 deg
moving-frame peak field = a0 cos(theta) ~= 14.14 Ec
N = 350 Nc, target_Ne_M = 8.622e29 m^-3
v_y API mapping = -c sin(theta) = -0.70710678 c
ion charge = 6, ion mass ~= 12 mp
```

## Paths

Remote:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/epoch1d_reflection_smoke.py
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/reflection_smoke_summary.json
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/reflection_jax_weight_scale_1.5625e12.json
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/reflection_smoke_report.html
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/reflection_figures/
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/reflection_arrays/
```

Local copies:

```text
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260711/reflection_smoke_report.html
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260711/reflection_smoke_summary.json
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260711/reflection_jax_weight_scale_1.5625e12.json
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260711/pypic3d_reflection_field.png
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260711/jaxincell_reflection_field_line_density.png
```

## Key results

Directional decomposition used:

```text
right-going component = (Ey + c Bz)/2
left-going/reflected component = (Ey - c Bz)/2
```

### PyPIC3D with explicit 3D volume/macroparticle weight

Used transverse box `y_wind=z_wind=lambda0`, so macro weight naturally includes a `lambda0^2` transverse area:

```text
macro_weight = target_Ne_M * D * lambda0^2 / Ne_macro = 6.3066e6
Ne_macro = 1200, Ni_macro = 200
Bz shift = -(0.5 + 0.5*CFL) dx = -0.75 dx
```

Result:

```text
final left-going energy in x < -1 lambda0 = 0.603
final total left-going energy fraction        = 0.604
final right-going energy in x > 0.5 lambda0 = 0.248
final peak Ey/Ec = 16.35
final peak Bz/Bc = 16.31
no NaNs
runtime ~14.1 s
```

Interpretation: PyPIC3D produced a clear reflected field in this reduced collisionless test.

### JAX-in-Cell with the same `lambda0^2` transverse-area weight

Using the same macro weight as PyPIC3D directly in JAX-in-Cell produced almost no reflection:

```text
final left-going energy in x < -1 lambda0 = 4.7e-6
final total left-going energy fraction        = 1.10e-4
final right-going energy in x > 0.5 lambda0 = 0.890
final peak Ey/Ec = 14.17
no NaNs
```

Interpretation: direct reuse of PyPIC3D's 3D-volume macroparticle weight underdrives the JAX-in-Cell 1D source. This is a normalization issue, not evidence that JAX-in-Cell cannot reflect.

### JAX-in-Cell with line-density weight scaling

A JAX-only weight sweep tested scale factors `1e4`, `1e8`, `1e12`, and exact `1/lambda0^2 = 1.5625e12`. The first two did not change reflection materially; the `~1e12` factors did.

Exact line-density conversion:

```text
weight_scale = 1/lambda0^2 = 1.5625e12
macro_weight = 9.8540e18
```

Result:

```text
final left-going energy in x < -1 lambda0 = 0.900
final total left-going energy fraction        = 0.906
final right-going energy in x > 0.5 lambda0 = 0.0128
final peak Ey/Ec = 32.98
final peak Bz/Bc = 29.21
no NaNs
runtime ~11.7 s
```

A nearby scale `1e12` also produced strong reflection (`left x<-1 lambda0 = 0.928`, peak Ey/Ec ~65). The exact `1/lambda0^2` case is the cleaner one to cite because it corresponds to converting a `lambda0^2` transverse-area volume weight into a 1D line-density weight.

## Mechanism / interpretation

The reflected field is driven by the transverse current of the overdense moving foil. PyPIC3D is a 3D code, so even with `Ny=Nz=1`, its macroparticle weight naturally refers to a finite transverse volume. JAX-in-Cell is 1D; its macroparticle weight must represent the source per unit transverse area / line density. Directly copying the PyPIC3D 3D-volume weight into JAX suppresses the source by the transverse area factor and gives almost no reflected field. Multiplying by `1/lambda0^2` restores a line-density-like source and produces strong reflection.

## Limitations

- This is a reduced smoke test; foil thickness is only 2.57 cells versus 68.6 cells in the full EPOCH deck.
- Periodic boundaries and final-time diagnostics only.
- No collisions by request.
- JAX line-density weighting needs a clean derivation from JAX-in-Cell's deposition/Gauss units before any quantitative comparison.
- JAX with line-density weight gives strong reflection but peak fields are larger than the simple incident `a0 cos(theta)` scale, so this is not yet a validated physical match.

## Next step

Derive and document the JAX-in-Cell 1D macroparticle-weight normalization, then run a small resolution/weight scan (`cells_per_lambda`, particle count, weight convention) and compare to a collision-off EPOCH reference if quantitative agreement is needed.
