---
name: 2026-07-15-molt-14-jax-runner-and-pending-k0
description: "Session journal: JAX-in-Cell runner refactor, Zhiping coding/data preferences, and pending K=0 reflection scan monitoring"
type: session-journal
---

# Session journal — molt 14 segment: JAX runner refactor and pending K=0 scan

## Timestamp / TL;DR

- Segment window: 2026-07-14 20:41 EDT through 2026-07-15 06:47 EDT.
- Main work: responded to Zhiping's JAX-in-Cell/PyPIC3D workflow instructions, created/refined a JAX-in-Cell-only project on adroit-vis, installed `h5py`, captured script-style preferences, and continued monitoring the K=+0.000 reflection pm300 order-scan Slurm jobs on tiger-vis.
- Reason for molt after this journal: cache-miss budget is nearing exhaustion (~132k remaining after the fourth scan check); the next scheduled self-reminder could exceed the soft budget if the context is not shed.

## Human-channel messages handled

Telegram conversation with Zhiping:

- `main:6420513923:259`: Zhiping asked whether chunked save/restart breaks autodiff. I answered in `:260`: ordinary disk checkpoint/restart breaks end-to-end JAX autodiff by default; differentiable memory saving should use `lax.scan` + online objective accumulation + `jax.checkpoint`/`remat`, or a custom VJP/checkpoint scheme.
- `main:6420513923:261`: Zhiping requested a JAX-in-Cell-only project extracted from `epoch1d_profiles_per_T0.py`, avoiding PyPIC3D, and writing final fields after `shift_field` centering with `EM_analyzer.read_write.write_fields_to_nc`. I acknowledged in `:262`, implemented, and reported in `:263`.
- `main:6420513923:264`: Zhiping authorized directly pip-installing missing libraries rather than maintaining a NetCDF fallback. I installed `h5py` in adroit-vis `GPU_Python` and reported success in `:266`.
- `main:6420513923:267`: Zhiping stated a script-interface preference: use `os.environ.get("NAME", "default")` parameters near the top of `.py` files so he can edit defaults in VSCode, while agents can override by `export`. I patched the runner and reported in `:269`.
- `main:6420513923:271`: Zhiping corrected that fundamental constants should come from `import scipy.constants as C` instead of handwritten literal values. I patched and reported in `:273`.
- `main:6420513923:274`: Zhiping requested merging `write_centered_final_field.py` into `run_jaxincell_final_field.py`, avoiding `.npz` field intermediates, and using NetCDF/HDF-style outputs via `EM_analyzer.read_write`. I patched and reported in `:276`.
- K=0 scan status was reported once in Telegram `main:6420513923:270` after the first check found all jobs still pending. Later unchanged pending checks were not reported to avoid spam.

## JAX-in-Cell-only project state

Remote project directory:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/
```

Final remote files:

```text
README.md
run_jaxincell_final_field.py
outputs/
```

`write_centered_final_field.py` was removed after its logic was merged into the main runner.

Final runner features:

- PyPIC3D is omitted.
- Parameters are editable/exportable via `os.environ.get(..., default)` near the top of `run_jaxincell_final_field.py`.
- Uses `scipy.constants as C` for standard constants (`C.speed_of_light`, `C.m_e`, `C.m_p`, `C.elementary_charge`, `C.epsilon_0`, `C.pi`, `C.micron`).
- Keeps `relativistic=True` in JAX-in-Cell solver parameters.
- Runs JAX-in-Cell, keeps `E_final/B_final` in memory, then directly calls:

```python
from EM_analyzer.pretreat_fields import shift_field
from EM_analyzer.read_write import write_fields_to_nc
```

- JAX staggering handled as:

```text
Ey grid: x_Ey = -L/2 + (i+0.5) dx
Bz grid: x_Bz = -L/2 + i dx = x_Ey - dx/2
```

- `Bz` is shifted to `x_Ey` with one periodic ghost point because `shift_field` uses constant extrapolation.
- No `.npz` field intermediates are produced.
- Output is:

```text
final_field_jaxincell_centered.nc
run_summary.json
```

Validation smoke test:

```bash
cd /scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d
export CELLS_PER_LAMBDA=20
export NE_MACRO=120
export STEPS=2
export OUTDIR=outputs/smoke_merged_nc
CUDA_VISIBLE_DEVICES=0 /scratch/network/zl8336/.conda/envs/GPU_Python/bin/python run_jaxincell_final_field.py
```

Result:

```text
final_field_jaxincell_centered.nc 41404 bytes
run_summary.json 3375 bytes
NetCDF sizes: {'x': 1200}
data_vars: ['Ey', 'Bz']
units: Ey V/m, Bz T
```

## Dependency update

On adroit-vis:

```text
/scratch/network/zl8336/.conda/envs/GPU_Python/bin/python
```

I installed:

```bash
python -m pip install h5py
```

Verified versions:

```text
h5py     3.16.0
h5netcdf 1.8.1
xarray   2026.4.0
```

This allows `EM_analyzer.write_fields_to_nc`'s hard-coded `engine='h5netcdf'` path to work normally.

## Durable preferences recorded

Updated `/home/zhiping/ULMI_lab/.lingtai/main/standing-rules.md` with:

1. Script parameter style: adjustable research scripts for Zhiping should default to `os.environ.get("NAME", "default")` near the top; Zhiping can edit defaults in VSCode and agents can override with `export`.
2. Scientific constants: use `scipy.constants as C` for standard constants instead of handwritten numeric literals unless a convention-specific override is intended.
3. Scientific data output format: high-dimensional field/simulation data should prefer NetCDF/HDF (`.nc`, `.h5`, `.hdf`) over `.npz`; field I/O should use `EM_analyzer.read_write` on tiger-vis/adroit-vis; avoid `.npz` intermediates when NetCDF/HDF is intended.

Also updated JAX PIC durable knowledge:

```text
knowledge/jax-pic-survey/autodiff_checkpointing_memory_2026-07-14.md
knowledge/jax-pic-survey/jaxincell_epoch1d_project_2026-07-14.md
knowledge/jax-pic-survey/KNOWLEDGE.md
```

## K=0 reflection pm300 order-scan monitoring

Active Tiger Slurm jobs:

```text
3512584  K=+0.000,ND_a0=0.15,L=0.00 reflection
3512585  K=+0.000,ND_a0=0.30,L=0.00 reflection
3512586  K=+0.000,ND_a0=0.50,L=0.00 reflection
3512587  K=+0.000,ND_a0=1.00,L=0.00 reflection
```

Logdir:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_order_focus_pm300_10T_K0_reflection_20260714_1944
```

Subset TSV:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_k0_reflection_subset_20260714.tsv
```

Output dirs:

```text
K=+0.000,ND_a0=0.15,L=0.00/order_focus_pm300_10T_K0_reflection/
K=+0.000,ND_a0=0.30,L=0.00/order_focus_pm300_10T_K0_reflection/
K=+0.000,ND_a0=0.50,L=0.00/order_focus_pm300_10T_K0_reflection/
K=+0.000,ND_a0=1.00,L=0.00/order_focus_pm300_10T_K0_reflection/
```

Parameters:

```text
HALF_WIDTH_T0=300
NUM_POINTS=61
step=10T0
reflection only
resource=900G/12h each
```

Status checks:

- 2026-07-14 21:44 EDT: all four jobs `PENDING (Priority)`, elapsed 00:00:00, no node assigned, logdir only `manifest.tsv`.
- 2026-07-14 23:45 EDT: unchanged pending.
- 2026-07-15 02:46 EDT: unchanged pending.
- 2026-07-15 06:46 EDT: unchanged pending.

No `.out/.err` files have appeared; no errors. Fifth self-reminder scheduled for 2026-07-15 ~10:47 EDT (4 hours after the fourth check). If still pending, do not Telegram-report unless Zhiping asks; just reschedule. If started/running, inspect `.err` for OOM/tracebacks vs known warnings and report if there is a problem or meaningful progress. If completed, validate per-band NetCDF outputs, extract/summarize per-band argmax for the four rows, report to Zhiping, and update:

```text
knowledge/curved-surface-chf-project/k0_reflection_wide_order_scan_2026-07-14.md
```

## Open tasks for successor

1. On the next self-reminder, check jobs `3512584–3512587` on tiger-vis.
2. If jobs complete, validate output NetCDFs and extract per-band argmax summary for the four K=0 reflection rows.
3. If Zhiping continues refining `jaxincell_epoch1d`, preserve his coding/data-format preferences: env defaults, `scipy.constants`, NetCDF/HDF via `EM_analyzer.read_write`, no unnecessary `.npz` field intermediates.
4. Watch cache-miss budget after the molt; this molt should reset it.

## Gotchas / lessons

- `EM_analyzer.pretreat_fields.shift_field` uses constant extrapolation internally, so staggered periodic grids need a ghost point for boundary-center interpolation.
- `EM_analyzer.pretreat_fields` sets JAX platform defaults for CPU, but importing it after the JAX-in-Cell `block_until_ready(sim.run())` worked in the merged smoke test.
- Zhiping strongly prefers scripts that are easy to open/edit in VSCode, not agent-only CLI ergonomics.
- For scientific scripts, do not handwrite fundamental constants when `scipy.constants` has them.
- For field data, `.npz` is not the lab-preferred format; NetCDF/HDF with coordinates/units/comments is preferred.
