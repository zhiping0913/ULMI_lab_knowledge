---
name: 2026-07-14-molt-13-k0-jax-diagnostics
description: >-
  Session journal for K=0 reflection wide order-focus scan submission and JAX-in-Cell/PyPIC3D diagnostics, including the relativistic-pusher patch and sparse-memory plan.
type: session-journal
molt_count: 13
created_at: 2026-07-14T20:39:00-04:00
---

# 2026-07-14 molt-13 — K=0 order scan + JAX/PIC diagnostics

Timestamp: 2026-07-14 20:39 EDT.  
TL;DR: Submitted a widened `K=+0.000` reflection order-focus rescan on Tiger (`3512584–3512587`) and diagnosed an adroit-vis JAX-in-Cell failure as a missing `relativistic=True` pusher setting plus reduced-grid/full-history memory issues. Patched Zhiping's `epoch1d_profiles_per_T0.py` after approval.

## What this segment was about

Two active research threads from Zhiping arrived over Telegram:

1. Curved_surface order-resolved focusing: the previous `±100T0` scan was adequate for curved cases but clipped some order-specific foci for flat `K=+0.000` reflection cases. Zhiping asked to widen the time scan for four K=0 reflection rows while keeping `10T0` precision.
2. JAX-in-Cell / PyPIC3D adroit-vis benchmarking: Zhiping increased `Ne_macro` and `cells_per_lambda` in `epoch1d_profiles_per_T0.py`; PyPIC3D looked closer to EPOCH, but JAX-in-Cell showed electrons expelled from the target after laser arrival. He asked for explanation/solution, then approved patching the JAX section with `relativistic=True`, and asked about GPU memory limits for `cells_per_lambda≈1000`.

## Accomplishments

### 1. K=0 reflection wide order-focus scan submitted

Telegram messages:
- Request: `main:6420513923:248`
- Ack: `main:6420513923:249`
- Submission report: `main:6420513923:250`

Previous scan boundary check:

- Previous `order_focus_pm100_10T` scan summary showed flat cases have widely separated order foci.
- Boundary evidence:
  - `K=+0.000,ND_a0=0.15,L=0.00`: previous x range `232–432λ0`; band1 argmax at left boundary, band2 near left boundary.
  - `K=+0.000,ND_a0=0.30,L=0.00`: band1/band2 near lower side.
  - `K=+0.000,ND_a0=0.50,L=0.00`: band3 argmax at upper boundary.
  - `K=+0.000,ND_a0=1.00,L=0.00`: band3 argmax at upper boundary.

Created subset:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_k0_reflection_subset_20260714.tsv
```

Dry-run validated all four rows. Submitted after mandatory `checkquota`:

- `checkquota`: MIKHAILOVA scratch `9.2/15 TiB`, about `5.8 TiB` free > 2 TiB threshold.
- Previous 21-point K=0 reflection jobs took `2.0–2.6 h`, MaxRSS `739–824 GB`; reused `900G/12h` for 61-point jobs instead of blind over-allocation.

New scan settings:

```text
SCAN_TAG = order_focus_pm300_10T_K0_reflection
HALF_WIDTH_T0 = 300
NUM_POINTS = 61
step = 10T0
side = reflection only
K = +0.000 only
```

Jobs:

```text
3512584  K=+0.000,ND_a0=0.15,L=0.00  center=217T0  range=[-83,517]T0  900G/12h
3512585  K=+0.000,ND_a0=0.30,L=0.00  center=224T0  range=[-76,524]T0  900G/12h
3512586  K=+0.000,ND_a0=0.50,L=0.00  center=-2T0   range=[-302,298]T0 900G/12h
3512587  K=+0.000,ND_a0=1.00,L=0.00  center=107T0  range=[-193,407]T0 900G/12h
```

Log/manifest:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_order_focus_pm300_10T_K0_reflection_20260714_1944/manifest.tsv
```

Output directories:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/K=+0.000,ND_a0=...,L=0.00/order_focus_pm300_10T_K0_reflection/
```

Immediate `squeue`: all four pending `(None)`. A delayed self-email reminder was scheduled for ~2 hours after submission to check jobs, validate outputs, and summarize argmax if complete.

Durable note:

```text
knowledge/curved-surface-chf-project/k0_reflection_wide_order_scan_2026-07-14.md
```

### 2. JAX-in-Cell / PyPIC3D discrepancy diagnosed

Telegram messages:
- Zhiping discrepancy report: `main:6420513923:251`
- Ack: `main:6420513923:252`
- Zhiping clarification: `main:6420513923:253`
- Diagnostic report: `main:6420513923:254`
- Patch request: `main:6420513923:255`
- Patch report: `main:6420513923:256`
- GPU memory question: `main:6420513923:257`
- Memory answer: `main:6420513923:258`

Files inspected:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/epoch1d_profiles_per_T0.py
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/profiles_per_T0/
```

Current script/run facts:

```text
a0=20, theta=45°, N_over_nc=350, D_lambda=0.017142857
cells_per_lambda=400, Nx=24000, dx=2 nm
CFL=0.9, steps_per_T0=444, n_periods=30
Ne_macro=12000, Ni_macro=2000
target thickness ≈6.86 cells
```

Compared with EPOCH, this is still coarse: EPOCH dx is about `0.2 nm≈λ0/4000`, so the same target would be about `68` cells thick.

Primary bug found:

```python
# Current JAX section before patch had only
'solver_parameters': {'field_solver': 0, 'print_info': False}
```

JAX-in-Cell defaults to `relativistic=False`. Thus current JAX-in-Cell profiles used nonrelativistic Boris for `a0=20`, while PyPIC3D explicitly had `relativistic=True`. Vision inspection of `jaxincell_profiles_t021_21T0.png` showed impossible `vx/c` and `vy/c` values of order tens. Source inspection confirmed JAX's relativistic pusher would return true velocities, so those superluminal values were a real pusher-setting error.

Quantitative diagnostics from existing NPZs:

- Total electron and ion charge were conserved in both codes; particles were not lost.
- Electron/ion initial placement is fine; JAX electrons remain near target until laser arrival (`e_near≈1` through ~`12T0`).
- JAX electron displacement occurs after laser interaction (`e_near≈0.392` at `16T0`, `≈0` by `20T0`).
- PyPIC3D also shows target electron depletion in the reduced run, but less abruptly.

JAX-only relativistic probe:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/relativistic_probe_20260714/jax_relativistic_probe_summary.json
```

- Same current setup but with `relativistic=True` and only to `21T0`.
- Runtime ~210s.
- Speeds stayed `≤c`, proving the missing relativistic setting is a real bug.
- Electron displacement after laser arrival still occurred, so `relativistic=True` is necessary but not sufficient for EPOCH agreement; remaining issues include coarse target resolution, collisionless setup, boundary differences, and JAX line-density normalization convergence.

Patch applied after Zhiping approved:

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

Syntax check passed:

```text
/scratch/network/zl8336/.conda/envs/GPU_Python/bin/python -m py_compile epoch1d_profiles_per_T0.py
```

No full rerun was performed at patch time to avoid overwriting existing `profiles_per_T0` outputs. Note: that directory contains stale t31–t40 files from earlier output even though current summary says `n_periods=30`.

Durable note:

```text
knowledge/jax-pic-survey/jaxincell_relativistic_pusher_diagnostic_2026-07-14.md
```

### 3. GPU-memory / sparse runner plan explained

Zhiping asked why `cells_per_lambda≈1000` OOMs even with two 80GB A100s, and whether CPU memory can hold arrays while GPU computes.

Main explanation sent in Telegram `main:6420513923:258`:

For `cells_per_lambda=1000`:

```text
Nx = 60000
steps_per_T0 ≈ 1111
30T0 -> Nt ≈ 33330
```

One full field history costs:

```text
Nt × Nx × 3 × float64 ≈ 48 GB
```

Saving `E(t,x)`, `B(t,x)`, `positions(t,p)`, `velocities(t,p)`, plus XLA temporaries easily exceeds 160GB. Two A100s do not automatically combine memory for a single PIC domain; JAX would need explicit sharding/domain decomposition, which JAX-in-Cell does not implement. CPU-memory out-of-core stepping would be very slow; JAX active arrays live on device.

Recommended solution: sparse / streaming diagnostics, not full history. New runner design:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/epoch1d_profiles_sparse_jaxincell.py
```

Design idea:

1. Use current initialization plus `relativistic=True`.
2. Run chunks of `1T0` or `0.5T0`.
3. After each chunk, save only final profiles/diagnostics to host.
4. Use final `E/B/positions/velocities` as next chunk's initial state.
5. Output to separate `profiles_per_T0_relativistic_sparse/` directory.

Memory goes from `O(Nt×Nx)` to roughly `O(steps_per_snapshot×Nx)`, or ideally `O(Nx+Nparticles)` if JAX-in-Cell internals are later patched so `lax.scan` returns final carry only. For `cells_per_lambda=1000`, one `1T0` chunk reduces one field history from ~48GB to ~1.6GB.

## Decisions and reasoning

- For the K=0 scan, I chose `±300T0` rather than a smaller expansion because previous boundary clipping occurred on both sides and only four reflection jobs needed rerun. Step remained `10T0` as requested.
- I used `900G/12h` for the K=0 jobs because measured prior jobs already needed up to ~824GB but completed in <3h for 21 points; 61 points should mainly increase runtime, not memory. This followed the lab rule to use measured sacct evidence rather than blindly over-allocating.
- I did not patch JAX script until Zhiping explicitly approved patching. Before that, I used a separate minimal probe output directory.
- I did not rerun full profiles after patch because Zhiping asked only to patch; rerunning into the existing directory risks overwriting/mixing with stale outputs.
- I explained that current nonrelativistic JAX outputs are invalid for a0=20 and should not be used to judge JAX-in-Cell, but also that `relativistic=True` alone does not guarantee EPOCH agreement.

## Open tasks / next steps

1. **K=0 reflection Slurm scan**: wait for self-email reminder around 2026-07-14 21:44 EDT, then check jobs `3512584–3512587`.
   - Run `squeue`/`sacct`.
   - Inspect `.err` files in logdir for OOM/tracebacks vs known warnings.
   - If complete, validate `order_focus_pm300_10T_K0_reflection` NetCDF outputs and extract per-band argmax summary for the four rows.
   - Report to Zhiping via Telegram and update `k0_reflection_wide_order_scan_2026-07-14.md`.
2. **JAX sparse runner**: if Zhiping asks to proceed, create a separate `epoch1d_profiles_sparse_jaxincell.py` rather than overwrite current script. It should run chunked `1T0` / `0.5T0` with `relativistic=True`, save profiles/diagnostics to a new directory, and avoid full history.
3. **If full rerun is requested**: use new output tag/directory or clean stale outputs first. Do not mix current nonrelativistic profiles with patched relativistic outputs.
4. **If comparing to EPOCH**: compare against collisionless EPOCH reference or state the collision mismatch; current JAX/PyPIC reduced run remains much coarser than EPOCH.

## Artifacts and paths

Curved_surface:

```text
knowledge/curved-surface-chf-project/k0_reflection_wide_order_scan_2026-07-14.md
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_k0_reflection_subset_20260714.tsv
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_order_focus_pm300_10T_K0_reflection_20260714_1944/manifest.tsv
```

JAX/PIC:

```text
knowledge/jax-pic-survey/jaxincell_relativistic_pusher_diagnostic_2026-07-14.md
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/epoch1d_profiles_per_T0.py
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/epoch1d_profiles_per_T0.py.bak_relativistic_20260714_2012
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/relativistic_probe_20260714/jax_relativistic_probe_summary.json
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260714/jaxincell_profiles_t021_21T0.png
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260714/pypic3d_profiles_t021_21T0.png
```

## Gotchas and lessons

- Use scp'ed helper scripts for nontrivial remote Python. Nested `ssh "python <<'PY'"` quoting caused another string-literal failure; local temp script + `scp` was robust.
- JAX-in-Cell defaults matter: `relativistic=False` by default, even if the physics is relativistic. Always set `relativistic=True` explicitly for a0~20 laser-plasma work.
- Vision/plots can reveal impossible diagnostics (e.g. `vx/c` of tens), but always verify with code/source and numerical arrays.
- `profiles_per_T0` can contain stale figure/data files from earlier runs; trust `profiles_per_T0_summary.json` and timestamps, or clean directories before reruns.
- Two GPUs do not automatically add memory for one JAX domain. Multi-GPU requires explicit sharding/domain decomposition; otherwise use separate cases per GPU.
- For high-resolution JAX/PIC diagnostics, memory must be controlled by sparse/chunked output, not by trying to hold full histories.

## Collaborators / communication

- Zhiping is the human requester and received all updates via Telegram.
- No peer agents were involved.
- No new persistent avatars were created, per standing rule.

## First five minutes after molt

1. Check Telegram and internal email.
2. If the self-reminder for K=0 jobs has arrived, read it and check jobs `3512584–3512587` on tiger-vis.
3. If Zhiping asks to implement the sparse JAX runner, use the JAX diagnostic note and current patched script; do not overwrite existing outputs without a new tag or explicit instruction.
4. Remember current cache-miss budget was nearly exhausted; this molt was proactive to preserve efficiency before the next job-check / JAX implementation step.
