---
name: fine-scan-near-final-jax-pic-handoff
description: >-
  Session record for Curved_surface fine time-scan follow-ups from 3443714 OOM through the final single running 3447199 job, plus Zhiping's new JAX PIC reproduction handoff.
type: session-journal
molt_count: 8
created_at: 2026-07-09T15:41:00Z
---

# Fine scan near-final state + JAX PIC handoff

Timestamp: 2026-07-09 11:41 EDT.  
TL;DR: I handled several delayed self-reminder follow-ups for the Curved_surface fine focus time-scan. The batch is now effectively `31/32` complete; only `3447199` (`K=+0.000,ND_a0=0.50` reflection, `995G/12h`) remains running, with a self-reminder scheduled for final check. Zhiping also introduced a new JAX PIC task: inspect his adroit-vis GPU trials of JAX-in-Cell and PyPIC3D and plan reproduction of a Tiger EPOCH 1D case.

## What this segment was about

This segment began after molt 8. The active job was monitoring and adapting the Curved_surface `a0=20/2D` fine propagation-time scan (101 time points around `focus.tsv` centers), while preserving Zhiping's requested adaptive HPC strategy: use measured `sacct` failures/resource needs, rerun only failed rows, and avoid rigid over-requesting.

The segment also received a new research direction from Zhiping on Telegram: he has already tested JAX-in-Cell and PyPIC3D on `adroit-vis` GPUs and wants both codes to reproduce a specific EPOCH 1D `input.deck`.

## Accomplishments

### Curved_surface fine scan monitoring and reruns

Handled all scheduled self-reminders through 2026-07-09 08:46 EDT:

1. `3443714` OOM and rerun:
   - `3443714` was the `700G/12h` rerun of original `3441375` (`K=+0.000,ND_a0=1.00,L=0.00`, reflection, center `100T0`, range `[50,150]T0`).
   - It OOMed after `00:04:47`; `sacct` batch MaxRSS `734001712K`.
   - After `checkquota` (MIKHAILOVA `8.7/15 TiB`, OK), I reran only that row as `3443924` at `900G/12h`.
   - Telegram report: `main:6420513923:167`.

2. `3441345` completion:
   - `3441345` (`K=-0.002,ND_a0=0.10,L=0.00`, reflection, `900G/12h`) completed in `05:27:22`, MaxRSS `809020628K`.
   - Outputs validated nonempty:
     - `reflection_beam_width_strength.h5` — `15470800` bytes.
     - `reflection_beam_width_strength.png` — `173358` bytes.
     - `reflection_peak_vs_x.nc` — `14998` bytes.
   - `.err` only had the known `All-NaN slice` warning.
   - Telegram report: `main:6420513923:168`.

3. `3441718` and `3441719` completions:
   - `3441718` (`K=+0.000,ND_a0=0.15,L=0.00`, reflection, `900G/12h`) completed in `05:56:10`, MaxRSS `742743980K`; outputs nonempty (`h5=12380800`, `png=151336`, `peak_vs_x.nc=14998`).
   - `3441719` (`K=+0.000,ND_a0=0.30,L=0.00`, reflection, `900G/12h`) completed in `04:49:58`, MaxRSS `746765688K`; outputs nonempty (`h5=12380800`, `png=124502`, `peak_vs_x.nc=14998`).
   - Both `.err` files only had the known `All-NaN slice` warning.

4. `3443492` OOM at 950G and `3447199` rerun:
   - `3443492` was the rerun of original `3441373` (`K=+0.000,ND_a0=0.50,L=0.00`, reflection, center `140T0`, range `[90,190]T0`) at `950G/12h`.
   - It OOMed after `00:05:55`; `sacct` batch MaxRSS `996144680K`.
   - Log showed this is the genuinely largest case: grid `Nx=6250`, `Ny=17500`, padded shape `(2,3,12512,35008,1)`, `field_pad` ~`19.6 GiB`, complex spectrum arrays ~`39.2 GiB` each.
   - Direct `1000G/12h` submission failed with `Requested node configuration is not available`.
   - `sbatch --test-only` accepted `960G`, `970G`, `980G`, `990G`, `995G`, and rejected `1000G`.
   - After another `checkquota` (MIKHAILOVA `8.7/15 TiB`, OK), I submitted only this row as `3447199` at `995G/12h`.
   - Telegram report: `main:6420513923:169`.

5. `3443924` completion:
   - `3443924` (`K=+0.000,ND_a0=1.00,L=0.00`, reflection, `900G/12h`) completed in `04:57:59`, MaxRSS `729570824K`.
   - Outputs validated nonempty:
     - `reflection_beam_width_strength.h5` — `12380800` bytes.
     - `reflection_beam_width_strength.png` — `137014` bytes.
     - `reflection_peak_vs_x.nc` — `14998` bytes.
   - `.err` only had the known `All-NaN slice` warning; `.out` ended with `=== Done ===`.
   - Telegram report: `main:6420513923:170`.

Current fine-scan state at molt:
- Effective `COMPLETED=31/32`.
- Only running job: `3447199`, `K=+0.000,ND_a0=0.50,L=0.00`, reflection, `995G/12h`, started 2026-07-09 07:04 EDT; it had run ~1h41 at the 08:45 EDT check.
- A delayed self-email reminder is scheduled for ~4h after 08:46 EDT with subject `Self-reminder: final check fine focus job 3447199`.

### New JAX PIC handoff

Zhiping sent Telegram `main:6420513923:171`:
- He already tried JAX-in-Cell and PyPIC3D on `adroit-vis` under `/scratch/network/zl8336/try_PIC_GPU_JAX`.
- He has run all examples for both using GPU.
- Reports/artifacts:
  - `/scratch/network/zl8336/try_PIC_GPU_JAX/report.html`
  - `/scratch/network/zl8336/try_PIC_GPU_JAX/field_init_guide.md`
- Next desired goal: use both codes to reproduce the Tiger EPOCH 1D case:
  - `tiger-vis:/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/1D/45/ND_a0=0.30/input.deck`

I acknowledged promptly in Telegram `main:6420513923:172` and promised to read the two adroit-vis reports plus the EPOCH `input.deck`, then produce a concrete reproduction route.

## Decisions and reasoning

- I continued the adaptive Slurm strategy: rerun only failed rows, not the whole batch.
- OOM implies memory escalation; timeout without OOM implies walltime escalation. When 3443492 OOMed at 950G and 1000G was unavailable, I used `sbatch --test-only` to find the highest feasible request (`995G`) instead of guessing.
- If `3447199` OOMs again, simple memory escalation has effectively reached the Tiger single-node limit; the next step should be algorithmic memory reduction (chunking or avoiding simultaneous large complex arrays) or a different computational approach, not another blind rerun.
- For JAX PIC, keep EPOCH as reference truth. JAX-in-Cell/PyPIC3D are candidates until they reproduce a controlled benchmark.

## Artifacts and paths

Curved_surface memory:
- `knowledge/curved-surface-chf-project/fine_time_scan_tiered_resubmission_2026-07-08.md` — updated through this segment.
- `knowledge/curved-surface-chf-project/fine_time_scan_submission_2026-07-08.md` — original fine-scan submission context.

Fine-scan main logdir:
- `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_fine_20260708_2014_fine_focus_tiered/`

Important rerun logdirs:
- `rerun_900G_12h_20260709_030959_idx31_oom3443714/` — `3443924` completed.
- `rerun_995G_12h_20260709_064427_idx29_oom3443492/` — `3447199` running.

JAX PIC memory:
- `knowledge/jax-pic-survey/KNOWLEDGE.md` — updated with Zhiping's adroit-vis trial and reproduction target.

New JAX PIC artifact paths to inspect next:
- `adroit-vis:/scratch/network/zl8336/try_PIC_GPU_JAX/report.html`
- `adroit-vis:/scratch/network/zl8336/try_PIC_GPU_JAX/field_init_guide.md`
- `tiger-vis:/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/1D/45/ND_a0=0.30/input.deck`

## Open tasks

1. When the delayed self-email fires, check `3447199`:
   - Use `ssh -o BatchMode=yes -o ConnectTimeout=10 tiger-vis`.
   - Check `squeue`/`sacct` for `3447199`.
   - If completed: validate `reflection_beam_width_strength.h5`, `.png`, and `reflection_peak_vs_x.nc` in `K=+0.000,ND_a0=0.50,L=0.00`; inspect `.err`; report final completion to Zhiping; update memory/pad.
   - If OOM again: do **not** blindly rerun. Report the node-limit/memory-reduction conclusion to Zhiping and update memory.
   - If still running: report only if useful; otherwise schedule another later self-reminder.

2. Start the JAX PIC reproduction-route task after molt:
   - Read the adroit-vis `report.html` and `field_init_guide.md`.
   - Read the EPOCH `input.deck` on tiger-vis.
   - Produce a mechanism-first plan: physics elements to reproduce, normalization/units, density/laser/boundary/diagnostic mapping, code-specific initialization steps for JAX-in-Cell and PyPIC3D, and benchmark pass/fail criteria.
   - Use `princeton-hpc`, `jax-pic-survey`, `research-software`, and likely `zhiping-pic-tool-habits` as relevant context.

## Collaborators and channels

- Zhiping is on Telegram chat `6420513923`; reply on Telegram.
- Internal self-reminders are via LingTai email.

## Gotchas and lessons

- Cache-miss budget is near exhaustion; this molt is proactive to restore cache efficiency before doing the JAX PIC inspection.
- For Princeton HPC: if SSH prompts for password/Duo, stop and report; do not retry.
- Run `checkquota` before any future `sbatch`/`salloc`.
- 3447199 is already at the practical node-memory limit; another OOM is an algorithmic-memory problem, not a scheduling problem.
