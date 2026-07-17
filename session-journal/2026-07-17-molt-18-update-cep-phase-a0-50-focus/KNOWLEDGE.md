---
name: 2026-07-17-molt-18-update-cep-phase-a0-50-focus
description: Session segment covering LingTai 0.16.3 update, CEP harmonic phase-locking analysis, a0=50 clip retries, and a0=50 focus-scan launch.
type: session-journal
session_journal: true
created_at: 2026-07-17T01:20:00-04:00
---

# Session journal — molt 18 segment: update, CEP phase locking, a0=50 clip/focus

## Why this journal exists

This session segment grew long after the previous molt and hit both a provider input-window failure and a high since-last-molt API-call/cache-miss count. I tended pad and knowledge before molting so the next self can resume safely.

## Main tasks completed

### LingTai runtime update

- A kernel-version nudge reported installed/running LingTai `0.16.2` and latest PyPI `0.16.3`.
- I read the runtime-update guidance, notified Zhiping, and waited for explicit Telegram confirmation before side effects.
- After confirmation, I ran the normal `lingtai-tui doctor` path.
- Verified after refresh:
  - `lingtai-tui 0.10.7`
  - runtime venv `lingtai 0.16.3`
  - runtime Python `/home/zhiping/.lingtai-tui/runtime/venv/bin/python`
- Log: `artifacts/lingtai_update_20260716_2003/lingtai_tui_doctor.log`.
- Possible LingTai CLI bug surfaced to Zhiping: `lingtai-tui self-update --help` appeared to trigger an actual Homebrew update instead of only help. No issue filed yet; Zhiping was asked whether to prepare/file one.

### CEP harmonic phase locking analysis

Zhiping asked whether per-harmonic phases at the reflection HH envelope peak are roughly locked near `-π/2`, and to compare EPOCH with JAX-in-Cell where high-order detuning seemed serious.

Actions:
- Patched both EPOCH and JAX copies of `spectrum_1D.py` only to save numeric `reflection_per_harmonic_at_peak.tsv`; original plotting/logic kept and backups created.
  - EPOCH script backup: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_1D.py.bak_20260716_230738`
  - JAX script backup: `/scratch/network/zl8336/try_PIC_GPU_JAX/utility/spectrum_1D.py.bak_20260716_231016`
- Ran EPOCH reflection phase extraction for all 8 CEPs.
- Ran JAX-in-Cell CPL=1200 reflection phase extraction for 7 CEPs found in the output root; `φ=0` was absent there.
- Built local deliverables in `artifacts/cep_phase_locking_20260716/`:
  - `cep_harmonic_phase_locking_report.html`
  - `cep_harmonic_phase_locking_report.ipynb`
  - `epoch_jax_phase_locking_comparison.png/.pdf`
  - EPOCH/JAX summary TSVs and strong-order extent TSVs.
- Wrote durable note: `knowledge/jax-pic-survey/cep_harmonic_phase_locking_2026-07-16.md`.
- Sent Telegram report/artifacts: `main:6420513923:365-368`.

Main conclusion:
- EPOCH: significant reflection harmonics approximately lock near a common phase close to `-π/2`; weighted mean phase about `-0.40π` to `-0.46π`, deviation `0.07π–0.12π`.
- JAX CPL=1200: same qualitative tendency but larger scatter and shorter strong-order extent, consistent with high-order detuning / weaker high-order coherence.
- Physical interpretation: if HHG is dominated by one attosecond burst/reflection event, sampling narrow-band analytic signals at the common HH envelope peak removes the linear emission-time phase term, leaving a common phase offset. In the current Hilbert/FFT convention this offset appears close to `-π/2`; the physical invariant is cross-harmonic phase locking, not the absolute offset alone.

### a0=50/2D ND_a0=0.30 clip and focus scan

Zhiping asked to submit all rows of `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=50/2D/propagate_and_clip.tsv` with `--mem=400G`, then after completion run focal-position scans with `center=50T0`, `half_width=100T0`, step `5T0`.

Clip phase:
- Every Slurm submission was preceded by `checkquota`; Tiger `/scratch/gpfs/MIKHAILOVA` remained ~`5.50 TiB` free.
- Clip logdir: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_and_clip_logs/a0_50_2D_ND030_20260716_220905`
- Final completed/validated clip jobs:
  - row1 K=-0.002: `3525766`
  - row2 K=-0.005: retry `3525787` (original `3525767` failed with JAX port-add/segfault)
  - row3 K=-0.008: `3525768`
  - row4 K=-0.010: retry `3526136` (original `3525769` Gloo allgather timeout)
  - row5 K=-0.012: `3525770`
  - row6 K=+0.000: corrected retry `3526900`
- Row6 retry history:
  - `3525771` and `3526137` failed with multihost Gloo `process_allgather` timeout.
  - `3526891` with `--nodes=1 --ntasks=1` failed because EM_analyzer expects `2×3` devices and one task gives only 3 CPU devices.
  - `3526900` with `--nodes=1 --ntasks=2 --ntasks-per-node=2 --mem=400G` succeeded; it preserved 6 CPU devices on one node and avoided inter-node Gloo.
- All six `reflection_clip.nc` outputs validate with xarray, dims `x=4000,y=6000`, vars `Ex,Ey,Bz`.

Focus scan phase:
- Focus scan launched 2026-07-17 01:17 EDT after all clips validated and checkquota passed.
- Focus logdir: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_50_2D_focus_center50_hw100_step5_20260717_011743`
- Manifest: `.../manifest.tsv`
- Parameters: `side=reflection`, `propagation_time=50`, `time_scan_half_width_T0=100`, `time_scan_num_points=41` (step 5T0), output subdir `focus_scan_center50_hw100_step5`.
- Jobs:
  - `3527413` row1 K=-0.002 — running at initial check
  - `3527414` row2 K=-0.005 — running
  - `3527415` row3 K=-0.008 — running
  - `3527416` row4 K=-0.010 — pending priority
  - `3527417` row5 K=-0.012 — pending priority
  - `3527418` row6 K=+0.000 — pending priority
- Zhiping notified in Telegram `main:6420513923:370`.
- Durable note: `knowledge/curved-surface-chf-project/a0_50_clip_focus_scan_2026-07-17.md`.

## Other active work

### K=0 widened reflection order scan

- Jobs `3512584–3512587` remain pending priority as of 2026-07-16 23:04 EDT.
- A new reminder was scheduled for ~2026-07-17 03:06 EDT with subject `Self-reminder: fifteenth check K=0 reflection pm300 order scan jobs 3512584-3512587`.
- No Telegram report was sent because nothing changed.

## Current reminders after molt

1. `Self-reminder: check a0=50 focus scan jobs 3527413-3527418` due around 2026-07-17 02:17 EDT.
   - Check `squeue/sacct`; if pending/running reschedule quietly.
   - If failed, inspect `.err` and report.
   - If completed, validate focus output files in each `focus_scan_center50_hw100_step5` dir, summarize focus positions/peak amplification per K, update Curved_surface knowledge, and report to Zhiping.
2. `Self-reminder: fifteenth check K=0 reflection pm300 order scan jobs 3512584-3512587` due around 2026-07-17 03:06 EDT.
   - If still pending, reschedule quietly; if completed, validate outputs and summarize per-band argmax for the four K=0 rows.
3. Older reminders for a0=50 row6 / 3526891 / 3526900 may arrive; they are obsolete because row6 completed and focus scan jobs were launched. Dismiss if seen.

## Durable stores tended

- Pad updated with current a0=50 focus jobs, K=0 reminder, CEP phase-locking artifacts, and runtime update notes.
- Curved_surface knowledge updated with `a0_50_clip_focus_scan_2026-07-17.md` and parent pointer.
- JAX survey knowledge updated with `cep_harmonic_phase_locking_2026-07-16.md` and parent pointer.
- Session journal entry written here.
- No character update or new reusable skill was needed.
