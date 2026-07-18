---
name: 2026-07-17-molt-21-slurm-focus-scans
description: >-
  Session journal for the segment after molt 21: CEP/HH phase thread archival, Curved_surface paraxial overlay, a0=20 focus-at-time Slurm batch with OOM retries, and a0=50 fine reflection focus scan submission.
type: session-journal
session_journal: true
date: 2026-07-17
---

# Session journal — molt 21 segment: Slurm focus scans and research-memory cleanup

## TL;DR
This segment handled several Zhiping Telegram requests, kept durable memory current, and ended with active Slurm monitoring. The pad is the operational source of truth for active jobs.

## Completed / recorded
- Post-molt recovery: checked internal email and Telegram; no unhandled messages at that time.
- CEP/HH phase-locking thread: Zhiping asked to pause detailed carrier-phase study but preserve it. Created `knowledge/jax-pic-survey/cep_hh_phase_locking_paused_research_2026-07-17.md`, indexed it in `knowledge/jax-pic-survey/KNOWLEDGE.md`, and pushed knowledge backup commit `ef0c9d5`. Core preserved question: why broad PIC regimes show sin-like attosecond HH pulses while simplified CSE/LW toy models look cos-like.
- Curved_surface a0=20 order-focus paraxial overlay: updated remote `plot_order_focus_vs_K_reflection_ND_scan_a0_20.ipynb` / `.py` to overlay a band1 paraxial Gaussian estimate only. Formula recorded: target `K` is surface curvature, tangential effective curvature `K_eff=K/cos45°`, reflected wavefront radius `R_i=1/(2K_eff)`, `w_i=12.5λ0`; higher-order curves deferred because effective harmonic waist is unknown. Sent PNG/PDF/CSV/notebook to Zhiping. Knowledge note `knowledge/curved-surface-chf-project/order_focus_paraxial_overlay_2026-07-17.md`; pushed commit `9d9c922`.
- K=0 widened reflection order scan quiet monitoring: jobs `3512584–3512587` remained `PENDING (Priority)` at 19:12 EDT; next reminder was rescheduled for ~23:12 EDT.
- Runtime update nudge: kernel latest PyPI `0.17.1` vs running/installed `0.16.3`; asked Zhiping on Telegram before any update side effect; no update performed.

## Active Slurm work at molt
### a0=20/2D focus-at-time fields
- Zhiping asked to run `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_at_time.slurm` for every row of `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/focus.tsv`, with `propagation_time_T0 = propagation_time (laser_period)`, `nc_name={side}_clip.nc`, `output_name={side}_focus`.
- Original submission after `checkquota` passed: jobs `3540190–3540221`, logdir `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_at_time_logs/a0_20_2D_focus_fields_20260717_213228`, manifest `.../manifest.tsv`, `600G`, `2h`, `6 CPU`.
- Status: 18 completed with outputs; 14 OOM at 600G.
- Retry after Zhiping allowed higher memory: jobs `3540403–3540416`, logdir `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_at_time_logs/a0_20_2D_focus_fields_retry_900G_20260717_221211`, retry manifest `.../retry_manifest.tsv`, `900G`.
- Latest check ~22:28 EDT: `3540404/405/406` completed with outputs; `3540407/408` running; `3540409–416` pending; row 1 retry `3540403` OOM again at 900G with `MaxRSS≈943715876K` and no outputs. Attempts to submit row1 at `1000G` and with Slurm `--mem=0` all-memory were rejected. If row1 is required, next step is algorithmic memory reduction (e.g. avoid full double-precision intermediates, chunk/stream, or a special diagnostic script), not more memory.

### a0=50/2D reflection fine focus scan
- Zhiping asked to refine the coarse a0=50 reflection focus scan: use each K case's coarse on-axis peak time from `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/a0_50_focus_center50_hw100_step5_20260717/a0_50_focus_center50_hw100_step5_summary.tsv`; scan `±20T0` around it at `1T0` spacing (41 points), reflection only.
- Submission after `checkquota` passed: jobs `3540394–3540399`, logdir `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_50_2D_reflection_fine_hw20_step1_20260717_221016`, manifest `.../manifest.tsv`, `400G`, `2h`.
- Centers: K=-0.012 `-25T0`, K=-0.010 `-30T0`, K=-0.008 `-30T0`, K=-0.005 `-45T0`, K=-0.002 `-10T0`, K=0 `-10T0`.
- Latest check ~22:28 EDT: five original jobs still running; K=-0.005 job `3540397` failed early with XLA all-reduce rendezvous abort (not memory; MaxRSS ~226G/400G), no outputs. Retried only this case as job `3540500`, pending, retry logdir `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_50_2D_reflection_fine_hw20_step1_retry1_Km005_20260717_223203`.

## Next actions for successor
1. Check inbox/Telegram after molt.
2. Await the scheduled combined reminder (~22:53 EDT) and K=0 reminder (~23:12 EDT). If a reminder arrives:
   - Use short noninteractive SSH to `tiger-vis`.
   - Run concise `squeue/sacct` checks; avoid giant outputs.
   - If only pending/running, reschedule quietly unless Zhiping needs a status update.
   - If terminal, validate outputs against manifests, inspect `.err` for OOM/tracebacks, report meaningful completion/failure to Zhiping, update pad and Curved_surface knowledge.
3. If cache-miss budget warning appears, finish the immediate check/report and then molt again; do not do expensive rebuild loops.

## Gotchas
- `propagate_2D_time_scan.py` may fail via transient JAX/XLA all-reduce rendezvous even when memory is sufficient; retrying the individual failed case is reasonable.
- `propagate_2D_at_time.py` row1 exceeded 900G and cluster rejected 1000G/all-memory; do not keep blindly resubmitting. Needs algorithmic memory reduction if required.
- Before any new Slurm `sbatch`/`salloc`, run `checkquota` and block if Tiger MIKHAILOVA scratch has <2 TiB free.
