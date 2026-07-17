---
name: 2026-07-12-molt-11-jax-pic-diagnostics
description: >
  Session segment covering corrected a0=20 JAX-in-Cell/PyPIC3D initialization, collisionless reflected-field smoke tests, and 0..20T0 full-x diagnostics for Zhiping.
type: session-journal
---

# Session journal: JAX/PyPIC corrected a0=20 diagnostics (2026-07-12, pre-molt 11)

## TL;DR

Completed Zhiping's adroit-vis JAX-in-Cell/PyPIC3D diagnostic sequence after the corrected a0=20 EPOCH input deck clarification. Delivered: reduced-grid initialization/offset sanity report, collisionless reflected-field smoke-test report, and full-x per-T0 diagnostic figures/data for `t/T0=0..20` for both codes. Durable knowledge and pad were updated. No background jobs remain.

## Human-channel timeline

- `main:6420513923:198`: Zhiping clarified the earlier `laser_a0=3` was an input-deck overwrite; corrected a0=20 decks are now source of truth. Task: use corrected deck plus `pypic3d_Bz_offset_sweep.ipynb` to test JAX-in-Cell/PyPIC3D setup on adroit-vis.
- `main:6420513923:199`: acknowledged plan.
- `main:6420513923:200-201`: reported initialization/offset sanity test and sent HTML report.
- `main:6420513923:202`: Zhiping asked to ignore collisions and test reflected field in JAX-in-Cell/PyPIC3D.
- `main:6420513923:203`: acknowledged collisionless reflected-field smoke-test plan.
- `main:6420513923:204-207`: reported reflected-field results and sent HTML + figures.
- `main:6420513923:208`: Zhiping requested full-x per-T0 plots for both codes: `Ey`, `ne`, `ni`, and electron/ion `vx,vy,vz`.
- `main:6420513923:209`: initially acknowledged with 0..8T0 short-domain plan.
- `main:6420513923:210`: Zhiping corrected that initial-to-reflection should be >10 cycles.
- `main:6420513923:211`: corrected plan to `x/λ0=[-20,20]`, center `-10λ0`, snapshots `0..20T0`.
- `main:6420513923:212-215`: reported completed 0..20T0 diagnostics and sent ZIP, notebook wrapper, and script.

## Accomplishments

### 1. Initialization / offset sanity benchmark

Remote workdir:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark
```

Key files:

```text
epoch1d_initialization_sanity.py
summary.json
epoch1d_initialization_sanity_report.html
figures/deck_initial_field_reduced.png
figures/jaxincell_deck_pulse_vacuum_like.png
figures/pypic3d_deck_pulse_vacuum.png
```

Local report copy:

```text
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260711/epoch1d_initialization_sanity_report.html
```

Findings:

- adroit-vis could not directly read the Tiger GPFS deck path (`PermissionError`); copied the corrected deck through `tiger-vis`.
- Corrected deck has `laser_a0=20`, `θ=45°`, `λ0=0.8 μm`, `D/λ0=0.017142857`, collisions on in EPOCH but omitted in tests.
- JAX-in-Cell offset: `Bz` at `x_E - 0.5 dx`.
- PyPIC3D effective offset: `Bz` shift `-(0.5+0.5*CFL)dx`, e.g. `-0.995dx` for CFL=0.99.
- Reduced-grid vacuum-like propagation preserved a nearly pure +x wave in both codes; peak fields ~`14.1 Ec/Bc`, consistent with moving-frame `a0 cos45°`.

### 2. Collisionless reflected-field smoke test

Key files:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/epoch1d_reflection_smoke.py
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/reflection_smoke_report.html
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/reflection_smoke_summary.json
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/reflection_jax_weight_scale_1.5625e12.json
```

Local copies:

```text
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260711/reflection_smoke_report.html
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260711/pypic3d_reflection_field.png
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260711/jaxincell_reflection_field_line_density.png
```

Findings:

- Reduced short-domain setup: `x/λ0=[-10,6]`, center `-5λ0`, `150 cells/λ0`, `Nx=2400`, `steps=2400`, travel `8λ0`, foil thickness 2.57 cells.
- PyPIC3D with explicit 3D volume macro weight (`target_Ne_M*D*λ0²/Ne = 6.3066e6`) produced clear reflection: final left-going energy in `x<-1λ0 = 0.603`, peak `Ey/Ec=16.35`.
- JAX-in-Cell with the same `λ0²` volume weight produced almost no reflection: `4.7e-6` left-going in `x<-1λ0`.
- JAX-in-Cell with line-density scaling `weight_scale=1/λ0²=1.5625e12` produced strong reflection: left-going in `x<-1λ0 = 0.900`, peak `Ey/Ec=32.98`.
- Interpretation: JAX-in-Cell needs a 1D line-density/per-transverse-area macro-particle weight; PyPIC3D's 3D volume weight cannot be copied directly.

### 3. Per-T0 full-x diagnostics

Remote output directory:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/profiles_per_T0/
```

Local copies sent to Zhiping:

```text
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260711/profiles_per_T0_20T/profiles_per_T0_artifacts.zip
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260711/profiles_per_T0_20T/profiles_per_T0_summary.json
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260711/profiles_per_T0_20T/epoch1d_profiles_per_T0.ipynb
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260711/profiles_per_T0_20T/epoch1d_profiles_per_T0.py
```

Final corrected setup:

```text
x/λ0 = [-20, 20]
initial pulse center = -10λ0
snapshots = t/T0 = 0..20
resolution = 150 cells/λ0, Nx = 6000
CFL = 0.5, steps_per_T0 = 300, total steps = 6000
collisionless
```

Outputs:

```text
JAX-in-Cell: 21 PNG + 21 NPZ, elapsed ~20.7 s
PyPIC3D:     21 PNG + 21 NPZ, elapsed ~32.3 s
ZIP:         8.05 MB
```

Each PNG contains full-x panels for `Ey/Ec`, `ne/Nc`, `ni/Nc`, electron `vx/c,vy/c,vz/c`, and ion `vx/c,vy/c,vz/c`. Each NPZ contains corresponding arrays.

## Decisions / reasoning

- Collision physics was omitted because Zhiping explicitly asked not to consider collisions.
- For JAX-in-Cell, used line-density macro-particle weighting after the reflected-field smoke test showed direct PyPIC3D volume weights underdrive the source.
- PyPIC3D used explicit 3D volume weight with transverse area `λ0²`.
- After Zhiping corrected the time window, changed from the short-domain 8T0 diagnostic to deck-like `[-20,20]λ0` and `0..20T0` snapshots.
- The reproducible notebook wrapper is simple; the real editable/reproducible logic is in the script `epoch1d_profiles_per_T0.py`.

## Gotchas / lessons

- adroit-vis cannot read the Tiger GPFS deck path directly; use tiger-vis as source and copy to adroit if needed.
- Default adroit `python3` lacks JAX. Use:

```text
/scratch/network/zl8336/.conda/envs/GPU_Python/bin/python
```

- PyPIC3D may use a flat particle backend: `particles=[flat_all]` rather than `[electrons, ions]`. Split electron/ion arrays with `flat.species_meta` counts.
- JAX/PyPIC output histories and particle arrays are enough for full-x diagnostic plots, but density/velocity profiles are binned diagnostics, not high-statistics converged distribution functions.
- The reduced grid has foil thickness only 2.57 cells, so it is diagnostic/interface testing only.

## Durable memory updated

- `knowledge/jax-pic-survey/epoch1d_adroit_initialization_sanity_2026-07-11.md`
- `knowledge/jax-pic-survey/epoch1d_reflection_smoke_2026-07-11.md`
- `knowledge/jax-pic-survey/epoch1d_profiles_per_T0_2026-07-12.md`
- `knowledge/jax-pic-survey/KNOWLEDGE.md` parent index appended.
- `system/pad.md` updated with completed benchmark/profile bullets.

## Open tasks / next steps

No active background job remains. If Zhiping continues this line:

1. Derive JAX-in-Cell's 1D macro-particle weight normalization cleanly from its deposition/Gauss-law units.
2. Run a small resolution/particle-count/weight scan.
3. If quantitative comparison is needed, generate or request a collision-off EPOCH reference.
4. Do not infer HHG/spectrum physics from these reduced diagnostic runs.

## Background work

None. Async jobs `job-42096d84` and `job-2110f299` are complete and handled. Obsolete self-reminder for `job-42096d84` was dismissed; a later self-reminder for `job-2110f299` may still arrive and should be dismissed as obsolete if it appears.
