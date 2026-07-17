---
name: 2026-07-09-molt-7-fine-scan-reruns-jax-source
description: >-
  Session record for source-verifying JAX-in-Cell transverse velocity support and adaptively monitoring/rerunning Curved_surface fine focus time-scan jobs.
type: session-journal
molt_count: 7
date: 2026-07-09
---

# 2026-07-09 molt-7 — fine-scan reruns and JAX source verification

Timestamp: 2026-07-09 01:39 EDT.  
TL;DR: Verified from source that JAX-in-Cell is 1D3V despite x-heavy examples, then monitored the Curved_surface fine focus scan through OOMs/timeouts, submitting only narrow reruns after `checkquota`. At molt time the active fine-scan work is not finished; a self-reminder is scheduled to re-check remaining jobs.

## What this segment was about

This segment began immediately after a deliberate cache-budget molt. Two threads were active:

1. **JAX PIC / thesis relevance.** Zhiping had asked whether JAX-based 1D PIC could help the earlier EPOCH+ML HHG thesis workflow. After the previous segment's survey/thesis answer, Zhiping challenged a specific technical point: JAX-in-Cell input seemed to expose only `vx`, while PyPIC3D exposed `vy`.
2. **Curved_surface fine propagation-time scan.** A 32-row fine focus time-scan batch for `a0=20/2D` was running on Tiger. The current tiered batch was `3441345–3441376`, created from `focus.tsv` centers with `center±50T0`, 101 points, mostly `700G/5h` plus two special `900G/12h` jobs.

## Accomplishments

### JAX-in-Cell source verification

- Read Zhiping's Telegram messages `main:6420513923:159-160`: he saw JAX-in-Cell input seemingly only supports `vx`; PyPIC3D can initialize `vy`; asked me to confirm.
- Acknowledged promptly in Telegram `main:6420513923:161`.
- Shallow-cloned sources under `tmp/jax_pic_verify/`:
  - `uwplasma/JAX-in-Cell`
  - `uwplasma/PyPIC3D`
- Verified from source that **JAX-in-Cell is not 1D1V**:
  - `_parameters/_species_definitions.py`: `SPECIES_AXES=("x","y","z")`; parameters include `vth_over_c_x/y/z`, `drift_speed_x/y/z`, `velocity_plus_minus_x/y/z`, `initial_velocities`.
  - `_parameters/_species_parameters.py`: explicit phase-space arrays must be `(number_pseudoparticles, 3)`.
  - `_state_initialization.py`: loops all axes and returns velocities shape `(N,3)`; `initial_velocities` overrides generated velocities.
  - `_particles.py`: Boris and relativistic pusher use `(N,3)` velocities and 3-vector E/B fields.
  - `_sources.py`: deposits `Jx,Jy,Jz`, using `vy=vs_n[i,1]`, `vz=vs_n[i,2]`.
  - Tests pass explicit velocities `[[1.0,1.1,1.2],[1.3,1.4,1.5]]` and assert they reach the simulation.
- Verified PyPIC3D's interface is more explicit for external `initial_vx/vy/vz` and `Tx/Ty/Tz`.
- Replied to Zhiping in Telegram `main:6420513923:162`: JAX-in-Cell remains a plausible lightweight 1D3V prototype, but docs/examples hide y/z; PyPIC3D is more transparent for transverse initialization.
- Updated `knowledge/jax-pic-survey/KNOWLEDGE.md` with a "Follow-up source verification: transverse velocity support" section.

### Fine focus time-scan monitoring and adaptive reruns

Initial active batch:
- Original over-requested jobs `3441224–3441255` were already canceled before this segment; their delayed reminder arrived and was read/cleared as obsolete.
- Correct active tiered batch: `3441345–3441376`.
- Original logdir: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_fine_20260708_2014_fine_focus_tiered/`
- Original manifest: `.../manifest.tsv`

Key follow-ups:

1. **22:17–22:18 EDT check.**
   - Original batch status around then: completed/running/pending plus two OOMs.
   - OOM rows:
     - `3441369`, idx 25: `K=+0.000,ND_a0=0.15,L=0.00`, reflection, center `200T0`, range `[150,250]T0`, `700G/5h`.
     - `3441371`, idx 27: `K=+0.000,ND_a0=0.30,L=0.00`, reflection, center `220T0`, range `[170,270]T0`, `700G/5h`.
   - Ran `checkquota`: MIKHAILOVA scratch `8.7/15 TiB`, OK.
   - Submitted only the two OOM rows as reruns at `900G/12h`:
     - `3441718` rerun of `3441369`.
     - `3441719` rerun of `3441371`.
   - Rerun logdir: `.../rerun_900G_20260708_2218_oom25_27/`
   - Reported in Telegram `main:6420513923:163`.

2. **23:49 EDT check.**
   - Effective state with OOM rows replaced by reruns: `COMPLETED=25/32`, `RUNNING=4`, `PENDING=3`; no new failures.
   - Completed outputs validated nonempty.
   - Non-OOM `.err` files only showed `RuntimeWarning: All-NaN slice encountered` at `beam_widths_min = np.nanmin(widths_2d, axis=0)`.
   - Reported in Telegram `main:6420513923:164`.

3. **00:50 EDT check.**
   - New blocker: original special job `3441373` (`K=+0.000,ND_a0=0.50,L=0.00`, reflection, center `140T0`, range `[90,190]T0`) OOMed even at `900G/12h` after `00:06:20`.
   - Ran `checkquota`: MIKHAILOVA `8.7/15 TiB`, OK.
   - Submitted only this failed row at `950G/12h`:
     - `3443492` rerun of `3441373`.
   - Rerun logdir: `.../rerun_950G_20260709_0051_special29_oom3441373/`
   - Reported in Telegram `main:6420513923:165`.

4. **01:37 EDT check.**
   - New issue: original standard job `3441375` (`K=+0.000,ND_a0=1.00,L=0.00`, reflection, center `100T0`, range `[50,150]T0`) hit walltime: `TIMEOUT`, elapsed `05:05:07`, originally `700G/5h`; no OOM.
   - Ran `checkquota`: MIKHAILOVA `8.7/15 TiB`, OK.
   - Reran only this row with the same memory but longer walltime:
     - `3443714` rerun of `3441375`, `700G/12h`.
   - Rerun logdir: `.../rerun_700G_12h_20260709_0138_timeout31/`
   - Reported in Telegram `main:6420513923:166`.

## Current state at molt

A delayed self-email has been scheduled for ~90 minutes after 01:38 EDT with subject:

`Self-reminder: check remaining fine focus reruns 3441718 3441719 3443492 3443714 and 3441345`

At the time of scheduling, effective unfinished rows were:

- Running:
  - `3441345`: original `K=-0.002,ND_a0=0.10,L=0.00`, reflection, `900G/12h`.
  - `3441718`: rerun of OOM `3441369`, `K=+0.000,ND_a0=0.15,L=0.00`, reflection, `900G/12h`.
  - `3441719`: rerun of OOM `3441371`, `K=+0.000,ND_a0=0.30,L=0.00`, reflection, `900G/12h`.
- Pending:
  - `3443492`: rerun of OOM `3441373`, `K=+0.000,ND_a0=0.50,L=0.00`, reflection, `950G/12h`.
  - `3443714`: rerun of timed-out `3441375`, `K=+0.000,ND_a0=1.00,L=0.00`, reflection, `700G/12h`.

Completed rows through the latest check had `completed_output_problems=0` for expected:
- `<side>_beam_width_strength.h5`
- `<side>_beam_width_strength.png`
- `<side>_peak_vs_x.nc`

Known warning:
- Many successful jobs write `RuntimeWarning: All-NaN slice encountered` at `beam_widths_min`; this is not a job failure but means some x regions lack valid above-threshold width samples.

## Decisions and reasoning

- I treated reruns as inside the existing authorized batch objective: Zhiping had explicitly corrected the resource strategy toward adaptive, failed-row-only reruns, and the task was to finish this fine scan.
- Before every rerun submission I ran `checkquota`; MIKHAILOVA scratch remained `8.7/15 TiB`, above the 2 TiB free threshold.
- For OOM rows I increased memory and time only on the failed row:
  - `700G/5h` OOM rows -> `900G/12h`.
  - Special `900G/12h` OOM row -> `950G/12h`.
- For the timeout row `3441375`, I did not increase memory because there was no OOM; I reran at `700G/12h`.
- I reported each meaningful state change or blocker to Zhiping over Telegram, not just private notes.

## Artifacts and paths

Knowledge files updated:
- `knowledge/jax-pic-survey/KNOWLEDGE.md`
- `knowledge/curved-surface-chf-project/fine_time_scan_tiered_resubmission_2026-07-08.md`
- `system/pad.md`

Important Telegram messages:
- `main:6420513923:161` acknowledgement of JAX source check.
- `main:6420513923:162` JAX-in-Cell 1D3V source-verification answer.
- `main:6420513923:163` report of two OOMs and reruns `3441718/3441719`.
- `main:6420513923:164` progress report: 25/32 done.
- `main:6420513923:165` report of `3441373` OOM at 900G and rerun `3443492` at 950G.
- `main:6420513923:166` report of `3441375` timeout and rerun `3443714` at 700G/12h.

Remote logdirs:
- Original: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_fine_20260708_2014_fine_focus_tiered/`
- OOM reruns 25/27: `.../rerun_900G_20260708_2218_oom25_27/`
- Special 29 rerun: `.../rerun_950G_20260709_0051_special29_oom3441373/`
- Timeout 31 rerun: `.../rerun_700G_12h_20260709_0138_timeout31/`

## Open tasks for next self

1. When the scheduled reminder arrives, check `squeue`/`sacct` for:
   - original `3441345–3441376`
   - reruns `3441718`, `3441719`, `3443492`, `3443714`
2. Build final accounting by replacing failed original rows with reruns.
3. Validate outputs for all effective completed rows:
   - `<side>_beam_width_strength.h5`
   - `<side>_beam_width_strength.png`
   - `<side>_peak_vs_x.nc`
4. Inspect `.err` files:
   - OOM/timeouts are blockers.
   - `All-NaN slice` warning is known warning-only unless accompanied by missing outputs or failed state.
5. Report final completion or the next blocker to Zhiping on Telegram.
6. Update `fine_time_scan_tiered_resubmission_2026-07-08.md` and `system/pad.md`.

## Gotchas and lessons

- The JAX-in-Cell example inputs emphasize x-directed cases; do not infer 1D1V from examples. Verify state shapes and source paths.
- For Slurm scans, use adaptive failed-row-only reruns and match the rerun resource to the failure mode: OOM -> more memory; timeout without OOM -> more walltime, same memory.
- `sacct` may not show MaxRSS for these jobs even when state is known; use `.err`, `.out`, and output validation as evidence.
- Avoid assuming OOM rows lack any partial outputs; validate final expected products only after a completed representative job/rerun.
