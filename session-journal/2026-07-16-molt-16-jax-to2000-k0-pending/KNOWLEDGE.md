---
name: 2026-07-16-molt-16-jax-to2000-k0-pending
description: >-
  Session record for completing the JAX-in-Cell final-field-only CPL convergence scan to 2000, reporting artifacts to Zhiping, updating knowledge, and continuing to monitor pending K=0 Curved_surface Slurm jobs.
type: session-journal
molt_count: 16
created_at: 2026-07-16T06:52:00-04:00
---

# 2026-07-16 molt 16 — JAX to 2000, K=0 pending

Timestamp: 2026-07-16 06:52 EDT.

TL;DR: Zhiping changed JAX-in-Cell to output only the final field via `snapshot_steps=[steps-1]`; I continued the CPL convergence scan through `1200,1500,1800,2000`, all completed without OOM, generated a combined report, sent artifacts, and updated `knowledge/jax-pic-survey`. The only live work left is a self-reminder to keep checking the long-pending K=0 reflection pm300 Slurm jobs `3512584–3512587`, which are still PENDING (Priority).

## What the segment was about

This segment began after a molt following the previous `.50`/`.90` JAX convergence scan. I first reoriented, checked internal email and Telegram, and slept. Zhiping then sent Telegram `main:6420513923:307` saying he modified JAX-in-Cell source around `lax.scan` so that only final field snapshots are output, with `params["solver_parameters"]["snapshot_steps"] = [steps-1]`, and asked to continue testing higher `CELLS_PER_LAMBDA` through 2000. In `main:6420513923:308`, he added that he had already tested the change in `/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/run_jaxincell_final_field.py`.

## Accomplishments

### JAX-in-Cell continuation to `CELLS_PER_LAMBDA=2000`

Loaded the required operational manuals before using bash/Princeton HPC. Checked adroit-vis with passwordless SSH and used short SSH timeouts. No Slurm was involved.

Inspected the runner and driver:
- Project root: `/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d`
- Runner: `run_jaxincell_final_field.py`
- Driver: `run_cpl_convergence_scan.py`
- Spectrum tool: `/scratch/network/zl8336/try_PIC_GPU_JAX/utility/spectrum_1D.py`

Confirmed current runner details:
- `solver_parameters` includes `snapshot_steps: [steps-1]`.
- Defaults include `FWHM_T0=1.5`, `LASER_A0=20.0`, `THETA_DEGREE=45.0`, `X_MIN_LAMBDA=-15`, `X_MAX_LAMBDA=15`, `N_PERIODS=abs(X_MIN_LAMBDA)*1.1`, reflection/transmission cutoffs `±3λ0`.

Launched the continuation on adroit-vis GPU1:

```text
root=/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/outputs/cpl_convergence_20260715_215429_finalonly_1200_2000
PID=1766390
SCAN_CPL_LIST=1200 1500 1800 2000
CUDA_VISIBLE_DEVICES=1
XLA_PYTHON_CLIENT_MEM_FRACTION=.95
```

The scan completed with `summary.json` status `done`, `failure=null`. Key metrics:

| CPL | side | total_energy_norm | HHG efficiency | Sx peak | x_at_Sx_peak (m) | runner elapsed |
|---:|---|---:|---:|---:|---:|---:|
| 1200 | reflection | 0.7383639579811371 | 0.3203013189272054 | 1.7002589962022039e25 | 6.775e-06 | 1061.766 s |
| 1200 | transmission | 0.07413819949463368 | 0.06158381904213842 | 7.531846912674663e24 | 6.652333333333331e-06 | 1061.570 s |
| 1500 | reflection | 0.7392117313558291 | 0.3388317398069617 | 2.038425150152874e25 | 6.780533333333333e-06 | 1660.492 s |
| 1500 | transmission | 0.08931107530760225 | 0.07529712326067588 | 1.1728427658182589e25 | 6.653066666666666e-06 | 1660.410 s |
| 1800 | reflection | 0.7510560115348838 | 0.33071896911164483 | 2.150800690080239e25 | 6.779333333333333e-06 | 2545.385 s |
| 1800 | transmission | 0.09548003419316266 | 0.07964414585729006 | 8.490064285403201e24 | 6.665555555555557e-06 | 2545.252 s |
| 2000 | reflection | 0.733004735065134 | 0.31555589874778506 | 2.206206271641699e25 | 6.7782e-06 | 3100.427 s |
| 2000 | transmission | 0.09530951989913795 | 0.07779164784421773 | 1.0733717176730697e25 | 6.663e-06 | 3100.227 s |

GPU observation: `cpl=1200/1500` checks used roughly 33 GiB on GPU1; `cpl=1800/2000` checks used roughly 66 GiB. `cpl=2000` completed and released GPU memory.

### Combined report and Telegram delivery

Generated a merged report combining old `100..1000` metrics with new final-only `1200..2000` metrics:

Remote report root:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/outputs/cpl_convergence_combined_20260716_0024_to2000_finalonly
```

Local copies:

```text
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260716/cpl_convergence_to2000/
```

Files:

```text
combined_convergence_report_to2000.html
combined_convergence_report_to2000.ipynb
combined_convergence_metrics_to2000.png
combined_metrics.tsv
high_cpl_metrics.tsv
adjacent_relative_changes.tsv
summary_to2000.json
```

Vision check of `combined_convergence_metrics_to2000.png` found it correctly rendered as a readable 2×2 figure. Minor caveats only: Sx panels use scientific notation, legends do not obscure data, vertical reference lines are visible but not captioned in-plot.

Final Telegram delivery:
- Text report: `main:6420513923:315`
- HTML: `main:6420513923:316`
- Notebook: `main:6420513923:317`
- PNG: `main:6420513923:318`
- combined metrics TSV: `main:6420513923:319`
- adjacent relative changes TSV: `main:6420513923:320`

### Durable knowledge

Updated `knowledge/jax-pic-survey/KNOWLEDGE.md` and wrote supporting note:

```text
knowledge/jax-pic-survey/jaxincell_cpl_to2000_finalonly_2026-07-16.md
```

`knowledge(action="info")` returned catalog size 6 and no problems.

## Decisions and reasoning

- Used `CUDA_VISIBLE_DEVICES=1` because GPU1 was effectively free; GPU0 had other use.
- Used `XLA_PYTHON_CLIENT_MEM_FRACTION=.95` to give the final-only scan enough headroom to test up to 2000 while keeping it on a single interactive A100 80GB GPU.
- Interpreted the result as a memory-mechanism fix: prior full-output `cpl=1200` failure was dominated by XLA input/output argument buffer pressure, not a fundamental impossibility at that grid size. The final-only `snapshot_steps=[steps-1]` path removes that blocker.
- Interpreted convergence cautiously: reflection HHG efficiency is reasonably stable at the few-percent level at high CPL, while local `Sx_peak`, especially transmission-side peak, remains more resolution/local-maximum sensitive.

## Artifacts and paths

JAX continuation root:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/outputs/cpl_convergence_20260715_215429_finalonly_1200_2000
```

Combined report root:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/outputs/cpl_convergence_combined_20260716_0024_to2000_finalonly
```

Local artifacts:

```text
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260716/cpl_convergence_to2000/
```

Knowledge:

```text
knowledge/jax-pic-survey/KNOWLEDGE.md
knowledge/jax-pic-survey/jaxincell_cpl_to2000_finalonly_2026-07-16.md
```

Temporary report-generation script:

```text
tmp/make_cpl_to2000_report.py
```

## Open tasks

### K=0 reflection pm300 order scan

Only active pending item at molt time.

Jobs:

```text
3512584-3512587
```

Logdir:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_order_focus_pm300_10T_K0_reflection_20260714_1944
```

Subset:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_k0_reflection_subset_20260714.tsv
```

Latest check at 2026-07-16 06:51 EDT:
- `squeue`/`sacct`: all four jobs still `PENDING (Priority)`, elapsed `00:00:00`, `ReqMem=900G`, `NodeList=None assigned`.
- Logdir still only has `manifest.tsv` (1489 bytes), no `.out/.err`.
- Prior checks from 2026-07-14 21:44 through 2026-07-16 02:50 were also all pending.

I scheduled the next internal self-reminder for about 2026-07-16 10:51 EDT (`Self-reminder: eleventh check K=0 reflection pm300 order scan jobs 3512584-3512587`). Next self should handle that email, run `squeue`/`sacct`, and:
- if still pending: dismiss/reschedule, no Telegram report unless asked;
- if running: inspect `.err` for OOM/tracebacks vs known warnings and report only meaningful progress/problem;
- if completed: validate per-band NetCDF outputs, extract/summarize per-band argmax for the four rows, report to Zhiping, and update Curved_surface knowledge.

## Collaborators

- Zhiping via Telegram chat `6420513923`; all human-facing replies went through Telegram.
- No avatars or daemons were created.

## Gotchas and lessons

- The previous full-output `cpl=1200` OOM was avoidable by reducing solver output to final field only. Future high-CPL JAX-in-Cell diagnostics should prefer final/sparse outputs or online objectives over returning full time histories.
- For high-resolution JAX diagnostics, integrated HHG metrics are more stable than local `Sx_peak`; present local peaks with uncertainty/caution.
- The K=0 Slurm jobs have been pending for a long time due to priority, not failure; repeated checks should not spam Telegram while still pending.
