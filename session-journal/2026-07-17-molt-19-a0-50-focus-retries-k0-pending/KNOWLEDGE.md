---
name: 2026-07-17-molt-19-a0-50-focus-retries-k0-pending
description: >-
  Completed the a0=50 focus-scan retries and summary deliverables, kept K=0 reflection pm300 jobs monitored/pending, and prepared a cache-budget molt.
date: 2026-07-17
molt_count: 19
type: session-journal
---

**2026-07-17 11:12 EDT** — TL;DR: Finished the a0=50/2D ND_a0=0.30 reflection focus-scan workflow after classifying original failures, patching a plotting bug, retrying missing rows, validating all six K values, and reporting artifacts to Zhiping. K=0 reflection pm300 order-scan jobs remain pending; the cache-miss budget is nearly exhausted, so this entry backs a deliberate molt.

## What this segment was about

This segment began after a cache-budget molt that launched the a0=50 focus scan. The active responsibilities were:

- Monitor `propagate_2D_time_scan` jobs for `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=50/2D` ND_a0=0.30 rows.
- Handle the older K=0 widened reflection order scan jobs `3512584–3512587` without Telegram spam while they remain pending.
- Preserve useful state in pad/knowledge and avoid blind retries.

## Accomplishments

- Recovered from molt, checked internal email and Telegram, and confirmed there were no new human instructions after the already-handled CEP/JAX requests.
- Handled the first a0=50 focus scan reminder:
  - Found original jobs `3527413/3527414/3527417` failed early with JAX/XLA CPU all-reduce Rendezvous aborts, exit 134, MaxRSS about 225–226 GB.
  - Reported those partial failures to Zhiping on Telegram (`main:6420513923:371`) and avoided blind immediate retries.
- Later found all original focus jobs had ended:
  - `3527415/3527416/3527418` ran all 41 time points and wrote core HDF/NetCDF/PNG outputs, but Slurm marked them failed due an end-of-script plotting bug: `K_BANDS` had `total + band1..4` (5 entries) while `colors_bands` had only 4 colors, causing `IndexError` at line 639.
  - Patched the plotting-only bug after backup: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan.py.bak_20260717_0309_focus_colors`.
  - Re-ran `checkquota` before retries; Tiger MIKHAILOVA scratch was about `9.5 TiB / 15 TiB` used, free ~5.5 TiB > 2 TiB threshold.
  - Submitted limited retries only for missing-output rows: `3528417` K=-0.002, `3528418` K=-0.005, `3528419` K=-0.012.
  - All three retries completed by 05:06 EDT.
- Validated all six focus output directories and generated combined summary artifacts:
  - Remote: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/a0_50_focus_center50_hw100_step5_20260717/`
  - Local: `artifacts/a0_50_focus_scan_20260717/`
  - Files: `a0_50_focus_center50_hw100_step5_summary.tsv`, `.png`, `.pdf`, `.ipynb`.
- Sent final Telegram report and artifacts to Zhiping (`main:6420513923:373–377`).
- Updated project knowledge:
  - Appended detailed completion/retry/summary section to `knowledge/curved-surface-chf-project/a0_50_clip_focus_scan_2026-07-17.md`.
  - Updated parent index `knowledge/curved-surface-chf-project/KNOWLEDGE.md` to say the focus scan is complete/summarized.
- Continued K=0 order-scan monitoring:
  - 03:06, 07:10, and 11:11 EDT checks all found `3512584–3512587` still `PENDING (Priority)`, elapsed 0, ReqMem 900G, no node assigned.
  - Rescheduled next K=0 reminder for ~15:11 EDT, subject `Self-reminder: eighteenth check K=0 reflection pm300 order scan jobs 3512584-3512587`.

## Decisions and reasoning

- Did not blind-retry after the first early failures. I inspected `.err` and saw JAX/XLA collective aborts rather than scratch or obvious memory exhaustion.
- Treated the `colors_bands` patch as a minimal plotting bug fix because it occurred after core numeric outputs were written and was structurally obvious: 5 bands but 4 colors.
- Retried only the rows without complete outputs, not all six, to avoid wasting Slurm resources and overwriting valid output.
- Did not report K=0 pending checks to Telegram because Zhiping had not asked for queue-noise and the reminder explicitly said to reschedule quietly if still pending.
- Chose to molt after this entry because session cache-miss usage is near the 1,000,000-token soft budget; sleeping to the next 4-hour reminder would likely exceed it. Pad and knowledge have been tended.

## Artifacts and paths

Focus-scan logs and outputs:

- Original focus logdir: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_50_2D_focus_center50_hw100_step5_20260717_011743`
- Retry logdir: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_50_2D_focus_center50_hw100_step5_retry1_20260717_0309`
- Retry manifest: `.../retry_manifest.tsv`
- Script backup: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan.py.bak_20260717_0309_focus_colors`
- Combined summary remote dir: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/a0_50_focus_center50_hw100_step5_20260717/`
- Combined summary local dir: `artifacts/a0_50_focus_scan_20260717/`
- Durable project note: `knowledge/curved-surface-chf-project/a0_50_clip_focus_scan_2026-07-17.md`

Key final metrics (reflection, center 50T0, half-width 100T0, step 5T0):

| K | job used | onaxis_peak_Ec | time_T0_at_onaxis_peak | x_lambda_at_onaxis_peak | env_total_Epeak_max_Ec | band4_Epeak_max_Ec |
|---:|---:|---:|---:|---:|---:|---:|
| -0.012 | 3528419 | 305.329 | -25.0 | 18.5475 | 313.672 | 233.348 |
| -0.010 | 3527416 | 303.336 | -30.0 | 18.5375 | 341.730 | 267.920 |
| -0.008 | 3527415 | 272.658 | -30.0 | 23.5525 | 293.076 | 218.923 |
| -0.005 | 3528418 | 232.137 | -45.0 | 28.5525 | 275.115 | 212.003 |
| -0.002 | 3528417 | 196.544 | -10.0 | 33.5525 | 241.591 | 184.263 |
| +0.000 | 3527418 | 226.768 | -10.0 | 33.5525 | 281.011 | 228.192 |

Telegram anchors:

- Partial failure report: `main:6420513923:371`
- Full retry/patch status: `main:6420513923:372`
- Final report + attachments: `main:6420513923:373–377`

## Open tasks

1. **K=0 reflection pm300 order scan monitoring**
   - Jobs: `3512584–3512587` on `tiger-vis`.
   - Latest check 2026-07-17 11:11 EDT: all still `PENDING (Priority)`, elapsed 0, ReqMem 900G, no node assigned.
   - Next self-reminder: ~2026-07-17 15:11 EDT, subject `Self-reminder: eighteenth check K=0 reflection pm300 order scan jobs 3512584-3512587`.
   - If still pending, reschedule quietly; if running/completed, inspect/validate/report per the pad.
2. **Potential project-code cleanup**
   - `propagate_2D_time_scan.py` band4 label/definition mismatch remains: code uses lower bound `2.5*k0` for band4 while label says `[3.5,50.5]k0`. I reported the caveat to Zhiping but did not patch it because it changes scientific interpretation, not just plotting robustness.

## Collaborators

- Zhiping Li is the human collaborator, reached via Telegram chat `6420513923`. Always reply on Telegram for this work.
- No peer agents were involved.

## Gotchas and lessons

- For Princeton HPC, keep using short SSH timeouts and BatchMode; if password/Duo appears, stop and report.
- Run `checkquota` before every future `sbatch`/`salloc`; this segment followed the rule before focus retry submissions.
- JAX CPU sharded Spectral_Maxwell jobs can fail transiently in XLA in-process all-reduce even with MaxRSS below request. Retry may succeed, but inspect `.err` first and avoid blind loops.
- Some Slurm `FAILED` states can occur after useful outputs were written; inspect output inventory and stdout tail before discarding a run.
- For generated figures to Zhiping, deliver rendered figure plus reproducible notebook and numeric TSV when possible.
