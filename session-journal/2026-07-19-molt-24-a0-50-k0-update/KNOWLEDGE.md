---
name: 2026-07-19-molt-24-a0-50-k0-update
description: >-
  Session record for completing the a0=50 reflection focus-at-y gain/estimated-3D workflow and aligned-Bz follow-up, monitoring the K=0 widened order scan through a row1 failure/retry, and handling a LingTai kernel update nudge.
type: session-journal
molt_count: 24
date: 2026-07-19
---

# 2026-07-19 molt 24 — a0=50 focus-at-y workflow, K0 retry, update nudge

Timestamp: 2026-07-19 20:03 EDT.

TL;DR: Completed Zhiping's a0=50 `ND/a0=0.30` reflection focus-at-y / `spectrum_1D.py` / estimated-3D `E_max(K)` workflow, delivered an aligned real-`Bz` follow-up plot, recorded and pushed durable knowledge, and continued K=0 widened order-scan monitoring. The K=0 row1 job `3512584` failed with a JAX/XLA rendezvous abort (not OOM); a row1 retry `3548700` was submitted after `checkquota` and is pending. A LingTai update nudge (`0.16.3 -> 0.17.1`) was surfaced to Zhiping; no update is authorized yet.

## What the segment was about

This segment began after the previous cache-budget molt that had just completed the a0=20 focus-at-y CHF/spectrum/3D-estimate work. The main new human ask came on Telegram `main:6420513923:460`: repeat the whole pipeline for `a0=50`, `ND/a0=0.30`, reflection side, using the already-completed `focus_scan_fine_onaxis_tm10_hw20_step1` runs. Zhiping asked to write `a0=50/2D/focus.tsv`, generate `reflection_focus.nc`, slice at `y=0`, run `spectrum_1D.py`, estimate 3D field strength, and plot estimated 3D `E_max(K)`.

Later in the same segment, Zhiping interpreted the cross-a0 result (`main:6420513923:472`) and asked for an a0=50 aligned real-`Bz` plot analogous to the a0=20 figure (`main:6420513923:474`). In parallel, I kept monitoring the old K=0 widened reflection order scan jobs `3512584–3512587`. A system nudge also reported that a newer LingTai kernel package is available.

## Accomplishments

### a0=50 focus-at-y + spectrum_1D + estimated 3D Emax(K)

Completed and reported on Telegram `main:6420513923:463–471`.

Durable note:
`knowledge/curved-surface-chf-project/a0_50_reflection_gain_3d_estimate_2026-07-19.md`

Remote output directory:
`/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/a0_50_reflection_gain_3d_estimate_20260719/`

Local artifacts:
`artifacts/a0_50_reflection_gain_3d_estimate_20260719/`

Key workflow facts:

- Generated `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=50/2D/focus.tsv` from each fine-scan `reflection_beam_width_strength.h5` by taking the time sample with max `peak_amp_per_t_over_Ec`.
- Ran mandatory `checkquota` before Slurm submission; MIKHAILOVA scratch free was well above the 2 TiB guard.
- Submitted six focus-at-time jobs with `300G/1h`, all completed successfully with MaxRSS about `244 GB`:
  - `3547613` K=-0.012
  - `3547614` K=-0.010
  - `3547615` K=-0.008
  - `3547616` K=-0.005
  - `3547617` K=-0.002
  - `3547627` K=+0.000 (submitted after an initial `QOSMaxSubmitJobPerUserLimit` once slots freed)
- Validated six nonempty `reflection_focus.nc` and `reflection_focus_Bz.png` outputs.
- Ran y=0 slicing and `spectrum_1D.py` for all six 2D cases. Also reran the a0=50 1D reference with `LASER_A0=50`, `THETA_DEGREE=45`, avoiding the default a0=20 normalization.
- Produced harmonic gain/phase diagnostics and estimated 3D `E_max(K)` plots with a reproducible notebook.

Main numerical result using locked band `n=1..20` and conservative `E3D = E2D * sqrt(G_locked)`:

```text
K       E2D(Ec)  G_locked  E3D_cons(Ec)  ideal upper(Ec)
-0.012   329.1     3.19        588          1050
-0.010   307.3     3.01        533           926
-0.008   285.8     2.75        474           786
-0.005   236.3     2.40        366           567
-0.002   197.3     2.01        280           396
+0.000   227.6     2.09        329           476
```

Physical reading: in this a0=50 scan the strongest negative curvature cases give both the highest 2D focus field and the largest locked-band gain, so the conservative 3D estimate peaks at `K=-0.012` (`~588 Ec`) and remains high at `K=-0.010` (`~533 Ec`). The ideal upper `E2D*G_locked` reaches `~0.9–1.1e3 Ec` but was explicitly framed as an upper bound, not a 3D PIC result.

### Cross-a0 normalized efficiency interpretation

Zhiping observed that a0=20 has a larger harmonic focusing gain than a0=50, so the estimated 3D fields can be close despite a0=50's larger source amplitude. I replied on Telegram `main:6420513923:473` and recorded the interpretation in the a0=50 knowledge note.

Factorization:

```text
a0=20, ND/a0=0.30, best K≈-0.005:
E2D≈156.9 Ec, G_locked≈4.43, sqrt(G)≈2.10
E3D_cons≈330 Ec, E3D/a0≈16.5

a0=50, ND/a0=0.30, best K≈-0.012:
E2D≈329.1 Ec, G_locked≈3.19, sqrt(G)≈1.79
E3D_cons≈588 Ec, E3D/a0≈11.8
```

Interpretation: a0=50 still has higher absolute conservative `E3D` by `~1.8x`, but the normalized field `E3D/a0` and intensity-like `(E3D/a0)^2` are higher for a0=20. Mechanistically, a0=50 has stronger local fields but likely more wavefront/coherence degradation from radiation pressure, surface deformation, and transverse electron dynamics; a0=20 near `ND/a0≈0.30` appears closer to a clean source plus focusable wavefront window.

### a0=50 aligned real-Bz follow-up

Zhiping asked in Telegram `main:6420513923:474` for the a0=50 version of the aligned real-`Bz` comparison. Completed and sent Telegram `main:6420513923:476–480`.

Remote output directory:
`/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/a0_50_focus_at_y_aligned_Bz_20260719/`

Local artifacts:
`artifacts/a0_50_focus_at_y_aligned_Bz_20260719/`

Files include PNG/PDF, summary TSV, notebook, and script. Plot includes 1D reference plus all six 2D K cases, aligned at the HH-envelope peak with a `±4` wavelength-unit window. 1D uses moving-frame units `λ_M=λ0/cos45°`, `B_{c,M}=B_c cos45°`; 2D uses `λ0,B_c`.

Alignment metadata:

```text
1D        x_peak=7.7129 λ_M   duration=0.0118 T0
K=-0.012  x_peak=17.5425 λ0  duration=0.0549 T0
K=-0.010  x_peak=19.5425 λ0  duration=0.0571 T0
K=-0.008  x_peak=22.5475 λ0  duration=0.0548 T0
K=-0.005  x_peak=26.5525 λ0  duration=0.0751 T0
K=-0.002  x_peak=35.5525 λ0  duration=0.0771 T0
K=+0.000  x_peak=35.5525 λ0  duration=0.0449 T0
```

### Knowledge and pad maintenance

Knowledge backup commits during this segment:

- `0e8ab68` — `Record Curved_surface CHF gain and 3D estimates`
- `0c3732e` — `Add cross-a0 CHF efficiency interpretation`
- `d271002` — `Record a0=50 aligned Bz follow-up`

The pad was updated with the a0=50 completion, aligned-Bz follow-up, latest knowledge commit pointer, current K=0 order-scan state, and the LingTai update authorization ask.

### K=0 widened reflection order-scan monitoring

Existing active jobs: `3512584–3512587` for K=0 reflection pm300 order scans.

At 15:21 EDT, job `3512584` was running on `tiger-i05g12` (elapsed `01:53`, `ReqMem=900G`), `.err` empty, `.out` progressing around `t_idx=13`, `t=+47T0`.

At 17:22 EDT, `3512584` was still running (elapsed `03:54`), `.err` empty, `.out` progressing around `t_idx=28`, `t=+197T0`, `|centerline|max≈34.4 Ec`.

At 19:23 EDT, `3512584` had failed:

```text
State: FAILED
Elapsed: 03:55:21
ExitCode: 134:0
MaxRSS: 801173292K
ReqMem: 900G
```

The failure was a JAX/XLA CPU all-reduce rendezvous abort (`Expected 3 threads ... only 2 arrived`), not Slurm OOM. No final expected output files existed. I generated a row1 retry Slurm script and submitted it after mandatory `checkquota`:

- Original logdir: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_order_focus_pm300_10T_K0_reflection_20260714_1944/`
- Retry logdir: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_order_focus_pm300_10T_K0_reflection_retry1_20260719_1925/`
- Retry script local copy: `artifacts/k0_order_scan_retry_20260719/ord_1_reflection_retry1.slurm`
- Retry script remote copy: `.../ord_1_reflection_retry1.slurm`
- `checkquota`: MIKHAILOVA scratch `9.5/15 TiB`, free `~5.5 TiB`, above guard
- Retry job: `3548700`, `PENDING (Priority)` immediately after submit
- Other jobs at that time: `3512585` pending `(Resources)`, `3512586–3512587` pending `(Priority)`

Durable note appended:
`knowledge/curved-surface-chf-project/k0_reflection_wide_order_scan_2026-07-14.md`

If retry `3548700` fails with the same rendezvous abort, stop and report to Zhiping before further retries; next likely approach is to change JAX CPU device/threading settings or chunk the time scan rather than repeatedly resubmitting unchanged.

### LingTai kernel update nudge

At 20:00 EDT a system nudge reported `lingtai` update availability: running/installed `0.16.3`, latest PyPI `0.17.1`. I read `runtime-update-checks`, checked minimal PyPI metadata, and sent Telegram `main:6420513923:481` to Zhiping explaining this is an availability nudge, not automatic. I asked for explicit authorization (e.g. “更新 LingTai”) before any update/install/refresh side effect. No update has been authorized yet.

## Decisions and reasoning

- For the a0=50 focus-at-time jobs, used `300G/1h` rather than `600G/900G` because prior a0=50 output sizes and measured MaxRSS suggested much smaller memory than a0=20; this succeeded with MaxRSS `~244 GB`.
- Reran 1D `spectrum_1D.py` with `LASER_A0=50` because using default `LASER_A0=20` would corrupt `envelope_at_peak_over_a0EcM` for a0=50 normalization.
- Used the same conservative missing-dimension model as a0=20 (`E3D=E2D*sqrt(G_locked)`) to keep the comparison consistent; explicitly labelled `E2D*G_locked` as an ideal upper bound.
- For K=0 row1 failure, treated exit 134 as a software/JAX rendezvous abort rather than memory exhaustion because MaxRSS was below the request and the error log identified XLA rendezvous timeout. One retry was reasonable because this class of JAX CPU rendezvous failure can be transient; repeated failures should be escalated and debugged rather than blindly retried.
- Did not update LingTai without explicit human authorization, per runtime-update-checks procedure.

## Artifacts and paths

- a0=50 focus/3D output: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/a0_50_reflection_gain_3d_estimate_20260719/`
- a0=50 focus/3D local artifacts: `artifacts/a0_50_reflection_gain_3d_estimate_20260719/`
- a0=50 aligned-Bz output: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/a0_50_focus_at_y_aligned_Bz_20260719/`
- a0=50 aligned-Bz local artifacts: `artifacts/a0_50_focus_at_y_aligned_Bz_20260719/`
- a0=50 generated focus table: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=50/2D/focus.tsv`
- a0=50 focus-at-time logdir: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_at_time_logs/a0_50_2D_reflection_focus_fields_20260719_115701/`
- a0=50 focus-at-y spectrum logdir: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_at_y_spectrum_logs/a0_50_focus_at_y_spectrum_20260719_121636/`
- K=0 original logdir: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_order_focus_pm300_10T_K0_reflection_20260714_1944/`
- K=0 retry logdir: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_order_focus_pm300_10T_K0_reflection_retry1_20260719_1925/`
- Knowledge notes: `knowledge/curved-surface-chf-project/a0_50_reflection_gain_3d_estimate_2026-07-19.md`, `knowledge/curved-surface-chf-project/k0_reflection_wide_order_scan_2026-07-14.md`
- Telegram anchors: `460`, `463–481` in chat `6420513923`.

## Open tasks

1. Monitor K=0 order-scan jobs at the scheduled self-reminder (~2026-07-19 21:26 EDT): check `squeue/sacct` for `3548700,3512585,3512586,3512587`. If retry still pending, reschedule. If running, tail `.err/.out`. If completed, validate outputs and decide whether to report. If retry fails with the same JAX/XLA rendezvous abort, stop and report to Zhiping before further retries.
2. Wait for Zhiping's response to the LingTai update prompt `main:6420513923:481`. Do not update without explicit authorization. If he says to update, use the normal LingTai/TUI runtime upgrade path and avoid disrupting active Slurm monitoring.
3. If future analysis is requested, a useful plot would factor `E2D/a0`, `sqrt(G_locked)`, and `E3D/a0` or `(E3D/a0)^2` versus `a0` and K to support the normalized-efficiency interpretation.

## Collaborators / channels

- Zhiping via Telegram chat `6420513923`. Always reply on Telegram for his messages there.
- No peers/avatars involved. Do not create persistent avatars unless Zhiping explicitly asks.

## Gotchas and lessons

- `spectrum_1D.py` defaults can silently use `LASER_A0=20`; for a0=50 workflows, explicitly set `LASER_A0=50` for both 1D and 2D post-processing.
- `spectrum_1D.py` stores HH peak/duration in stdout logs, not in `*_spectrum.nc`. Keep the logdir/metadata when later aligning raw fields.
- JAX/XLA CPU all-reduce rendezvous abort is distinct from Slurm OOM. Check `MaxRSS`, `ExitCode`, and `.err` before deciding whether to increase memory.
- For any Slurm retry/submission, always run `checkquota` and block if MIKHAILOVA scratch free falls below 2 TiB.
- Session cache-miss budget is nearly exhausted; this molt is being performed proactively to restore cache efficiency before the next self-reminder or human update reply.
