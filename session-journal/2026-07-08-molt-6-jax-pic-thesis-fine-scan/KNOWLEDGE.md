---
name: 2026-07-08-molt-6-jax-pic-thesis-fine-scan
description: "Session journal before molt: Curved_surface fine time-scan submission/resource correction, JAX PIC survey, and thesis-specific JAX PIC assessment."
type: session-journal
---

# Session journal — 2026-07-08 molt 6 — JAX PIC / thesis / fine scan

## Why this segment is ending

The since-last-molt cache-miss budget is near exhaustion after a long sequence of Telegram handling, HPC edits/submissions, local thesis reads, and web/LightRAG searches. Durable stores have been tended; molt before the next long job-check cycle to restore cache efficiency.

## Communication state

All human messages in this segment were answered on Telegram.

Important Telegram IDs:
- `main:6420513923:145`: Zhiping asked to refine `propagate_2D_time_scan` using centers from `focus.tsv` and `center±50T0`, 101 points.
- `main:6420513923:147`: initial fine-scan submission report (uniform `900G/12h`) — superseded.
- `main:6420513923:148`: Zhiping corrected resource requests; do not use `900G/12h` for all jobs.
- `main:6420513923:150`: corrected tiered resubmission report.
- `main:6420513923:151`: Zhiping's HPC scheduling lesson: smaller resources can backfill quickly; agent workflow should be adaptive.
- `main:6420513923:153`: JAX-based PIC survey request.
- `main:6420513923:155`: first-pass JAX PIC survey answer.
- `main:6420513923:156`: thesis-specific follow-up request.
- `main:6420513923:158`: thesis-specific answer: 1D JAX PIC can help as fast low-fidelity scan/active-learning/in-memory diagnostics, not as direct EPOCH replacement.

## Active Curved_surface fine time-scan batch

Current active Slurm batch is **not** the original over-requested one.

### Scripts / patch

Patched actual time-scan script:
`/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan.py`

Backup:
`/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan.py.bak_fine_time_env_20260708_195159`

The patch:
- If no `propagation_time` env var is passed, preserves legacy quick scan `[-200, 200] T0`, 21 points.
- If `propagation_time` is passed, defaults to center `±50 T0`, 101 points.
- Reads env vars `propagation_time`, `time_scan_half_width_T0`, `time_scan_num_points`.
- Writes time-scan attrs into HDF outputs.

Submission driver:
`/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/submit_fine_time_scan_from_focus.sh`

Important driver lessons:
- Uses env-var export rather than `sbatch --export=...` because `working_dir` contains commas.
- Handles `focus.tsv` even if final line has no trailing newline.
- Defaults to tiered resources: standard `700G/5h`, special `900G/12h`.

### Original over-requested batch — obsolete

Original jobs: `3441224–3441255`, uniform `900G/12h`.
Canceled after Zhiping's correction. If a delayed self-email reminder for these IDs arrives, it is obsolete; read/dismiss it.

### Current tiered batch

Current jobs: `3441345–3441376`.

Logdir:
`/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_fine_20260708_2014_fine_focus_tiered/`

Manifest:
`/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_fine_20260708_2014_fine_focus_tiered/manifest.tsv`

Resources:
- 30 standard jobs: `700G`, `5:00:00`.
- Special `3441345`: `K=-0.002,ND_a0=0.10,L=0.00`, `reflection`, previous heavy job `3439199`, `900G`, `12:00:00`.
- Special `3441373`: `K=+0.000,ND_a0=0.50,L=0.00`, `reflection`, previous heavy job `3439197`, `900G`, `12:00:00`.

Corrected self-email reminder is scheduled for ~2h after resubmission (~2026-07-08 22:16 EDT). On wake:
1. Read/dismiss obsolete reminder for `3441224–3441255` if it arrives.
2. For current jobs `3441345–3441376`, check `squeue`/`sacct`.
3. Validate per-row outputs: `<side>_beam_width_strength.h5`, `<side>_beam_width_strength.png`, `<side>_peak_vs_x.nc`.
4. Inspect `.err` logs for OOM/timeouts or warning-only `All-NaN slice` messages.
5. Report meaningful status to Zhiping.

## HPC operating lesson saved

Zhiping corrected the over-conservative submission. This changed operating style and was saved in:
- `system/lingtai.md` character section.
- `knowledge/zhiping-pic-tool-habits/KNOWLEDGE.md` under HPC scan resource scheduling.

Standing rule to remember: for parameter scans, use measured `sacct` MaxRSS/Elapsed and case structure to tier jobs. Let ordinary cases request modest resources/time to backfill; only known heavy cases get high resource limits; rerun failed items precisely.

## JAX PIC survey

Created durable entry:
`knowledge/jax-pic-survey/KNOWLEDGE.md`
Catalog validated with `knowledge(info)`; no problems.

First-pass findings:
- Best first candidate: `uwplasma/JAX-in-Cell`, 1D3V JAX PIC, MIT, docs at `jax-in-cell.readthedocs.io`, arXiv `2512.12160`, examples include Landau/Langmuir/two-stream/autodiff/optimization.
- `uwplasma/PyPIC3D`: 3D JAX PIC, useful architecture reference (Boris, Esirkepov/current deposition, diagnostics/openPMD), probably too heavy for first 1D scan prototype.
- `SeanLim2101/PiC-Code-Jax`: educational/minimal 1D JAX PIC reference; not a mature research foundation.
- Differentiable kinetic simulations APS leads support the general methodology but are not ready-to-use HHG PIC tools.

Main physics/numerics conclusion: JAX helps with JIT/vectorized/batched fixed-shape 1D PIC loops and in-memory diagnostics, but PIC noise, deposition discontinuities, scatter/boundary effects, and HHG sensitivity make gradients hard to trust. Use JAX first for low-fidelity fast scans/black-box active learning, with EPOCH validation.

## Thesis-specific JAX PIC assessment

Zhiping pointed to:
`/home/zhiping/knowledge_base/thesis/2023/2023--利用机器学习优化强激光等离子体高次谐波产生效率/thesis.md`

Key facts extracted:
- Thesis used EPOCH(1D) after Bourdieu moving-frame transform from oblique incidence to normal-incidence 1D. It explicitly notes 1D reduces computation but loses transverse/focusing/divergence effects.
- Parameters: `a0`, `a0/N`, `theta`, `L`, `phi`.
- Discrete space: about `6,715,800` points.
- Actual samples: `94,517` PIC points from random walk/random/targeted scans.
- Objective: HHG conversion efficiency `eta` from reflected spectrum high-order energy, chosen because it only needs two field outputs. High-frequency diagnostics slow EPOCH due to disk I/O.
- Workflow: EPOCH samples → correlation/t-SNE/DBSCAN → classifier network → per-class efficiency network → scan whole discrete space with networks → PIC-validate top candidates.
- High-efficiency region: roughly `a0≈15–20`, `a0/N≈0.9–1.1`, `theta≈54°–57°`, `L≈0–0.05λ`, `phi≈180°`, `eta>83%`.

Answer given to Zhiping:
- Yes, 1D JAX PIC could materially advance this earlier work because the old bottleneck was exactly high-volume 1D PIC sample generation and diagnostics.
- It should serve as fast low-fidelity sample generator / active-learning loop / in-memory diagnostics engine.
- It should not replace EPOCH as truth. First benchmark JAX-in-Cell against EPOCH on representative high/low thesis points and verify moving-frame mapping, laser-solid setup, density profiles, boundary conditions, spectra, energy conservation, and noise.

## Durable stores updated

- Pad updated with JAX PIC survey and thesis-specific summary.
- `knowledge/jax-pic-survey/KNOWLEDGE.md` updated with thesis follow-up.
- `knowledge/zhiping-pic-tool-habits/KNOWLEDGE.md` updated with HPC tiering lesson.
- Character updated with HPC adaptive resource scheduling lesson.

## Open next steps for post-molt self

1. Check Telegram and internal email.
2. If the obsolete self-reminder for `3441224–3441255` arrives, read/dismiss it.
3. When the corrected reminder arrives, validate current jobs `3441345–3441376` using the tiered logdir/manifest above.
4. If Zhiping asks to proceed with JAX PIC, use `knowledge/jax-pic-survey/KNOWLEDGE.md` and propose a sandbox JAX-in-Cell trial plus EPOCH-vs-JAX thesis benchmark. Do not install/run heavy external code without agreeing on location/environment.
