---
name: 2026-07-09-molt-9-jax-pic-focus-final
description: >-
  Completed the adroit-vis JAX PIC reproduction-route analysis and finalized the
  Curved_surface fine focus scan by updating focus.tsv from all 32 completed
  time-scan logs.
date: 2026-07-09
molt_count: 9
type: session-journal
---

**2026-07-09 19:27 EDT** — TL;DR: This segment completed two tasks for Zhiping: a JAX-in-Cell/PyPIC3D route for reproducing an EPOCH 1D laser-solid case, and the final Curved_surface fine-focus time-scan aggregation into `focus.tsv`. No active background job remains; wait for Zhiping's next instruction.

## What this segment was about

This segment began immediately after a cache-efficiency molt. The active tasks were:

1. Read Zhiping's adroit-vis JAX PIC trial reports and the target tiger-vis EPOCH 1D `input.deck`, then produce a concrete two-code reproduction route.
2. Continue monitoring the final heavy Curved_surface fine focus scan job `3447199` until completion.
3. After Zhiping confirmed all 32 fine time-scan tasks were complete, update `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/focus.tsv` by replacing each row's propagation time with the time of maximum `|centerline|_max` and adding `E_max(Ec)`.

## Accomplishments

### JAX PIC reproduction route

- Read adroit-vis reports:
  - `/scratch/network/zl8336/try_PIC_GPU_JAX/report.html`
  - `/scratch/network/zl8336/try_PIC_GPU_JAX/field_init_guide.md`
- Read adroit-vis test/source files for JAX-in-Cell and PyPIC3D field/particle initialization.
- Read the target tiger-vis EPOCH deck:
  - `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/1D/45/ND_a0=0.30/input.deck`
- Produced and sent the route to Zhiping:
  - Telegram report: `main:6420513923:173`
  - HTML attachment: `main:6420513923:174`
  - HTML artifact: `/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax-pic-repro-20260709/epoch1d_reproduction_route.html`
- Updated durable memory:
  - `knowledge/jax-pic-survey/KNOWLEDGE.md`
  - `knowledge/jax-pic-survey/epoch1d_reproduction_route_2026-07-09.md`

Key conclusions preserved there:
- JAX-in-Cell is the first-line fast 1D3V scan/active-learning candidate, but full-resolution EPOCH-like runs need sparse/final diagnostics rather than full time-history output.
- PyPIC3D is heavier but a strong cross-check because its `.npy` fields and `initial_vx/vy/vz` particle arrays are explicit.
- The target deck says `laser_a0=3` despite the parent path `a0=20`.
- EPOCH `drift_y=-m_s c tanθ` is a momentum-like Bourdier-frame expression; JAX/PyPIC velocity APIs should receive `v_y=-c sinθ≈-0.707c` for θ=45°, not `-c tanθ`.
- Existing EPOCH output files in the target directory likely do not match the current `input.deck` (`final_field.nc` has x=160000; old log final time ≈20T0 while current deck implies 25T0).
- Both JAX codes lack the deck's Nanbu collision physics, so first benchmark should be collisionless or compared to an EPOCH collision-off reference.

### Curved_surface fine focus scan monitoring

- Read and dismissed the self-reminders for final job `3447199`.
- Checked `3447199` twice:
  - At ~12:46 EDT it was still running at 5:42:34; reported in Telegram `main:6420513923:175`.
  - At ~14:47 EDT it was still running at 7:43:36, `.err` was 0 bytes, and `.out` had progressed to `t_idx=77`, `t=+167.0T0`; reported in Telegram `main:6420513923:176`.
- Scheduled follow-up self-reminders as needed, then dismissed the final stale reminder when Zhiping provided the newer instruction.
- Final validation after update showed `3447199` completed:
  - State: `COMPLETED`, elapsed `09:50:22`, MaxRSS `917610972K`, ReqMem `995G`.
  - Final outputs existed and were nonempty for `K=+0.000,ND_a0=0.50,L=0.00` reflection:
    - `reflection_beam_width_strength.h5` 15,470,800 bytes
    - `reflection_beam_width_strength.png` 135,217 bytes
    - `reflection_peak_vs_x.nc` 14,998 bytes

### focus.tsv aggregation

Zhiping's Telegram `main:6420513923:177` instructed: update `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/focus.tsv`; set `propagation_time (laser_period)` to the time of maximum `|centerline|_max` over each row's 101-point scan; add `E_max(Ec)`.

Completed:
- Acknowledged in Telegram `main:6420513923:178`.
- Parsed the 32 completed `.out` logs under:
  `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_fine_20260708_2014_fine_focus_tiered/`
- Selected one complete `=== Done ===` 101-point log per focus row, preferring successful rerun logs over failed originals.
- Backed up original file:
  `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/focus.tsv.bak_20260709_192443_before_Emax_update`
- Updated target file:
  `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/focus.tsv`
- Added column:
  `E_max(Ec)`
- Wrote provenance/argmax table:
  `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_fine_20260708_2014_fine_focus_tiered/focus_time_scan_argmax_summary_20260709_192443.tsv`
- Validated:
  - 32 rows
  - 0 missing `propagation_time (laser_period)`
  - 0 missing `E_max(Ec)`
  - 30/32 propagation times changed
  - E_max range `1.793–156.934 Ec`
- Reported completion in Telegram `main:6420513923:179`.

## Decisions and reasoning

- Used `.out` logs rather than re-reading HDF/NetCDF because Zhiping explicitly suggested the `.out` files and they contain exactly the 101 `t_idx`, `t`, `|centerline|_max`, and `valid_width_pts` records needed.
- Used the focus row order and the fine-scan `manifest.tsv` to map rows to log indices. This avoids brittle matching solely by case names.
- Preferred rerun logs for rows where original jobs OOMed/timed out. The selection rule required `=== Done ===` and at least 101 parsed records.
- Wrote a backup before modifying `focus.tsv` because this is an externally important project file under scratch.
- Did not submit or rerun any Slurm job during this segment. All HPC actions were read-only except the requested `focus.tsv` update and sidecar summary/backup writes.
- The initial dry-run parser failed because the regex did not allow spaces after `t=` in lines such as `t= -30.0T0`. Fixed regex to `t=\s*([+-]?...)T0`, dry-ran successfully, then wrote.

## Artifacts and paths

- JAX PIC route HTML:
  `/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax-pic-repro-20260709/epoch1d_reproduction_route.html`
- JAX PIC durable note:
  `knowledge/jax-pic-survey/epoch1d_reproduction_route_2026-07-09.md`
- Curved_surface fine scan memory:
  `knowledge/curved-surface-chf-project/fine_time_scan_tiered_resubmission_2026-07-08.md`
- Updated focus file:
  `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/focus.tsv`
- Backup focus file:
  `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/focus.tsv.bak_20260709_192443_before_Emax_update`
- Argmax provenance table:
  `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_fine_20260708_2014_fine_focus_tiered/focus_time_scan_argmax_summary_20260709_192443.tsv`
- Telegram anchors:
  - `main:6420513923:173-174` JAX PIC route report/artifact
  - `main:6420513923:175-176` job `3447199` progress updates
  - `main:6420513923:177` Zhiping focus.tsv update instruction
  - `main:6420513923:178` acknowledgement
  - `main:6420513923:179` completion report

## Open tasks

None actively pending.

Potential next steps only if Zhiping asks:
- Make plots/analysis from the updated `focus.tsv` and `focus_time_scan_argmax_summary_20260709_192443.tsv`.
- Start the JAX/PyPIC EPOCH reproduction implementation under `/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/`, beginning with a shared initializer and vacuum pulse propagation tests.
- If EPOCH reference reruns are requested, follow HPC rules: short SSH timeouts; password/Duo prompt means stop; before any `sbatch`/`salloc`, run `checkquota` and block if Tiger scratch free <2 TiB.

## Collaborators

- Zhiping via Telegram chat `6420513923`; he has been kept updated on Telegram and is not waiting on an unreported result.
- No peer/avatar involvement.

## Gotchas and lessons

- For fine time-scan `.out` parsing, allow whitespace after `t=` before the sign: e.g. `t= -30.0T0`, `t= +70.0T0`.
- `focus.tsv` may have no final newline; use CSV parsing rather than `wc -l` to count logical rows.
- The 32 original logs may include failed/incomplete rows, while rerun subdirectories contain final successful logs. Selection should require `=== Done ===` and 101 records, then prefer the newest successful rerun.
- When a remote file update is requested, always make a timestamped backup and report both backup and updated paths.
- For JAX/PIC work, be explicit about whether a variable is a velocity or a relativistic momentum; the Bourdier drift mapping can otherwise be wrong by a factor of `γ`.
