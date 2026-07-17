# Fine propagation-time scan tiered resubmission — 2026-07-08

This corrects `fine_time_scan_submission_2026-07-08.md` after Zhiping's Telegram correction `main:6420513923:148`.

Reason for correction:
- Initial fine batch `3441224–3441255` used uniform `900G` and `12:00:00` for all 32 jobs.
- Zhiping pointed out this would lengthen queue unnecessarily.
- Based on previous quick-scan reruns, only previous jobs `3439199` and `3439197` needed near-900G memory/time. Others should use `700G` and `5:00:00`.

Mapping of previous heavy jobs to current focus.tsv rows:
- Previous `3439199` → `K=-0.002,ND_a0=0.10,L=0.00`, `reflection` → new tiered job `3441345` (`900G`, `12:00:00`).
- Previous `3439197` → `K=+0.000,ND_a0=0.50,L=0.00`, `reflection` → new tiered job `3441373` (`900G`, `12:00:00`).

Actions taken:
- Checked old over-requested batch before cancellation: 31 jobs still pending; `3441224` had just started running for ~21 min.
- Canceled old batch `3441224–3441255`.
- Updated `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/submit_fine_time_scan_from_focus.sh` so it defaults to `700G` / `5:00:00`, with two hard-coded special cases at `900G` / `12:00:00`.
- Dry-run validated `count=32`, `special=2`, `standard=30`, failed=0.
- Re-ran `checkquota`: MIKHAILOVA scratch `8.7/15 TiB`, OK (>2 TiB free).
- Submitted tiered batch `3441345–3441376`.

Tiered batch:
- 30 standard jobs: `700G`, `5:00:00`.
- 2 special jobs:
  - `3441345`: `K=-0.002,ND_a0=0.10,L=0.00`, `reflection`, center `20T0`, range `[-30,70]T0`, `900G`, `12:00:00`.
  - `3441373`: `K=+0.000,ND_a0=0.50,L=0.00`, `reflection`, center `140T0`, range `[90,190]T0`, `900G`, `12:00:00`.
- Immediate queue state: all 32 pending; standard jobs show `PENDING (None)`, special jobs pending.

Paths:
- Logdir: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_fine_20260708_2014_fine_focus_tiered/`
- Manifest: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_fine_20260708_2014_fine_focus_tiered/manifest.tsv`

Reported to Zhiping in Telegram `main:6420513923:150`.

Reminder state:
- Earlier delayed self-email for jobs `3441224–3441255` is obsolete if it arrives.
- Corrected delayed self-email for tiered jobs `3441345–3441376` was scheduled for ~2h after resubmission.

Follow-up validation:
- Check `squeue`/`sacct` for jobs `3441345–3441376`.
- Validate per row: `<side>_beam_width_strength.h5`, `<side>_beam_width_strength.png`, `<side>_peak_vs_x.nc` nonempty.
- Inspect `.err` logs for OOM, timeout, or warning-only `All-NaN slice` messages.

## Interim validation and narrow OOM rerun — 2026-07-08 22:18 EDT

Triggered by corrected delayed self-email `20260709T021619-4a58`.

Status check at ~22:17 EDT:
- Original tiered batch `3441345–3441376`:
  - `COMPLETED=7`
  - `RUNNING=22`
  - `PENDING=1`: `3441373` (`K=+0.000,ND_a0=0.50,L=0.00`, reflection, special `900G/12h`)
  - `OUT_OF_MEMORY=2`: `3441369`, `3441371`
- `3441345` (`K=-0.002,ND_a0=0.10,L=0.00`, reflection, special `900G/12h`) had started late and was running ~20 min at check time.
- Completed rows validated: all expected `<side>_beam_width_strength.h5`, `<side>_beam_width_strength.png`, and `<side>_peak_vs_x.nc` were present and nonempty (`completed_output_problems=0`).
- Non-OOM nonempty `.err` files contained only the warning:
  ```text
  RuntimeWarning: All-NaN slice encountered
    beam_widths_min = np.nanmin(widths_2d, axis=0)
  ```
  Same interpretation as previous batch: some x regions have no valid above-threshold width samples; not a job failure.

OOM rows:
- `3441369`, manifest idx `25`: `K=+0.000,ND_a0=0.15,L=0.00`, `reflection`, center `200T0`, range `[150,250]T0`, originally `700G/5h`.
- `3441371`, manifest idx `27`: `K=+0.000,ND_a0=0.30,L=0.00`, `reflection`, center `220T0`, range `[170,270]T0`, originally `700G/5h`.
- Both failed with Slurm `oom_kill` in the spectrum/padded-field stage; `.out` showed complex128 spectrum shape `(2,3,10016,35008,1)` of about `31.3 GiB` per logged array before failure.

Adaptive rerun action:
- Ran `checkquota` before submission: MIKHAILOVA scratch `8.7/15 TiB`, OK (>2 TiB free).
- Submitted only the two failed rows at `900G/12:00:00`, not the whole batch:
  - `3441718`: rerun of `3441369`, `K=+0.000,ND_a0=0.15,L=0.00`, `reflection`, center `200T0`, range `[150,250]T0`.
  - `3441719`: rerun of `3441371`, `K=+0.000,ND_a0=0.30,L=0.00`, `reflection`, center `220T0`, range `[170,270]T0`.
- Rerun logdir: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_fine_20260708_2014_fine_focus_tiered/rerun_900G_20260708_2218_oom25_27/`
- Rerun manifest: `.../rerun_900G_20260708_2218_oom25_27/manifest.tsv`
- Immediate rerun state: both pending `(Priority)`.

Reported interim status and reruns to Zhiping in Telegram `main:6420513923:163`.

Next follow-up:
- A delayed self-email was scheduled for ~90 minutes later (subject: `Self-reminder: follow up fine focus time-scan reruns 3441718-3441719 and remaining 3441345-3441376`).
- On follow-up, check original jobs `3441345–3441376` plus reruns `3441718–3441719`; validate final outputs and `.err` files.

## Follow-up check — 2026-07-08 23:49 EDT

Triggered by delayed self-email `20260709T034859-b7d8`.

Status at ~23:49 EDT:
- Original job states: `COMPLETED=25`, `RUNNING=4`, `PENDING=1`, `OUT_OF_MEMORY=2` (the two OOM rows already have reruns).
- Effective final-state accounting with OOM rows replaced by reruns: `COMPLETED=25/32`, `RUNNING=4`, `PENDING=3`, no new failures.
- Completed output validation: `completed_output_problems=0` for expected `<side>_beam_width_strength.h5`, `<side>_beam_width_strength.png`, and `<side>_peak_vs_x.nc`.
- Non-OOM `.err` files still only show the known warning:
  ```text
  RuntimeWarning: All-NaN slice encountered
    beam_widths_min = np.nanmin(widths_2d, axis=0)
  ```

Remaining rows:
- Running:
  - `3441345`: `K=-0.002,ND_a0=0.10,L=0.00`, reflection, special `900G/12h`.
  - `3441358`: `K=-0.005,ND_a0=0.50,L=0.00`, transmission, standard `700G/5h`.
  - `3441360`: `K=-0.005,ND_a0=1.00,L=0.00`, transmission, standard `700G/5h`.
  - `3441375`: `K=+0.000,ND_a0=1.00,L=0.00`, reflection, standard `700G/5h`.
- Pending:
  - `3441373`: `K=+0.000,ND_a0=0.50,L=0.00`, reflection, special `900G/12h`.
  - `3441718`: rerun of `3441369`, `K=+0.000,ND_a0=0.15,L=0.00`, reflection, `900G/12h`.
  - `3441719`: rerun of `3441371`, `K=+0.000,ND_a0=0.30,L=0.00`, reflection, `900G/12h`.

Reported progress to Zhiping in Telegram `main:6420513923:164`.

Next follow-up:
- Delayed self-email scheduled for ~1 hour later (subject: `Self-reminder: final-ish check fine focus time-scan original/rerun jobs`).
- Check `squeue`/`sacct` for original + reruns; if all complete, do final output and `.err` validation; if standard jobs timeout/OOM, rerun only failed rows after `checkquota`.

## Follow-up check and 950G special rerun — 2026-07-09 00:50 EDT

Triggered by delayed self-email `20260709T045007-9e9c`.

Status at ~00:50 EDT:
- Original job states: `COMPLETED=27`, `RUNNING=2`, `OUT_OF_MEMORY=3`.
- Effective accounting with OOM rows replaced by active reruns: `COMPLETED=27/32`, `RUNNING=3`, `PENDING=2`, with one original special OOM now replaced by a new 950G rerun.
- Completed output validation: `completed_output_problems=0`.

New blocker:
- Original special job `3441373` (`K=+0.000,ND_a0=0.50,L=0.00`, reflection, center `140T0`, range `[90,190]T0`) failed with `OUT_OF_MEMORY` after `00:06:20`, despite `900G/12h`.
- `.err` showed Slurm `oom_kill` / Python killed in `propagate_2D_time_scan.py`.

Adaptive action:
- Re-ran `checkquota`: MIKHAILOVA scratch `8.7/15 TiB`, OK.
- Submitted only this failed special row at `950G/12:00:00`:
  - `3443492`: rerun of `3441373`, `K=+0.000,ND_a0=0.50,L=0.00`, reflection, center `140T0`, range `[90,190]T0`.
- Rerun logdir: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_fine_20260708_2014_fine_focus_tiered/rerun_950G_20260709_0051_special29_oom3441373/`
- Rerun manifest: `.../rerun_950G_20260709_0051_special29_oom3441373/manifest.tsv`
- Immediate state: `3443492` pending `(Priority)`.

Effective unfinished rows after action:
- Running:
  - `3441345`: `K=-0.002,ND_a0=0.10,L=0.00`, reflection, `900G/12h`.
  - `3441375`: `K=+0.000,ND_a0=1.00,L=0.00`, reflection, `700G/5h`.
  - `3441718`: rerun of `3441369`, `K=+0.000,ND_a0=0.15,L=0.00`, reflection, `900G/12h`.
- Pending:
  - `3441719`: rerun of `3441371`, `K=+0.000,ND_a0=0.30,L=0.00`, reflection, `900G/12h`.
  - `3443492`: rerun of `3441373`, `K=+0.000,ND_a0=0.50,L=0.00`, reflection, `950G/12h`.

Reported to Zhiping in Telegram `main:6420513923:165`.

Next follow-up:
- Delayed self-email scheduled ~45 min later (subject: `Self-reminder: check final remaining fine focus jobs incl 950G rerun 3443492`).
- Check original `3441345–3441376` plus reruns `3441718`, `3441719`, `3443492`; if `3441375` times out/OOMs near 5h, consider a narrow rerun after `checkquota`.

## Follow-up check and timeout rerun — 2026-07-09 01:37 EDT

Triggered by delayed self-email `20260709T053644-7fe0`.

Status at ~01:37 EDT before action:
- Effective accounting (with OOM rows replaced by reruns) showed `COMPLETED=27/32`, `RUNNING=3`, `PENDING=1`, `TIMEOUT=1`.
- Completed output validation still `completed_output_problems=0`.

New issue:
- Original standard job `3441375` (`K=+0.000,ND_a0=1.00,L=0.00`, reflection, center `100T0`, range `[50,150]T0`) reached walltime: `TIMEOUT`, elapsed `05:05:07`, originally `700G/5h`.
- It did **not** OOM; `.err` showed cancellation due to time limit.

Adaptive action:
- Ran `checkquota`: MIKHAILOVA scratch `8.7/15 TiB`, OK.
- Reran only this timed-out row with the same memory but longer walltime:
  - `3443714`: rerun of `3441375`, `K=+0.000,ND_a0=1.00,L=0.00`, reflection, center `100T0`, range `[50,150]T0`, `700G/12h`.
- Rerun logdir: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_fine_20260708_2014_fine_focus_tiered/rerun_700G_12h_20260709_0138_timeout31/`
- Rerun manifest: `.../rerun_700G_12h_20260709_0138_timeout31/manifest.tsv`
- Immediate state: `3443714` pending `(None)`.

Effective unfinished rows after action:
- Running: `3441345`, `3441718`, `3441719`.
- Pending: `3443492`, `3443714`.

Reported to Zhiping in Telegram `main:6420513923:166`.

Next follow-up:
- Delayed self-email scheduled ~90 min later (subject: `Self-reminder: check remaining fine focus reruns 3441718 3441719 3443492 3443714 and 3441345`).
- Check original `3441345–3441376` plus reruns `3441718`, `3441719`, `3443492`, `3443714`; validate outputs and `.err`; report final/blocker.

## Follow-up check and 900G rerun after 700G/12h OOM — 2026-07-09 03:10 EDT

Triggered by delayed self-email `20260709T070813-d0ec`.

Status at ~03:08 EDT before action:
- `3441345` (`K=-0.002,ND_a0=0.10,L=0.00`, reflection, `900G/12h`) still running, elapsed ~5h12.
- `3441718` (rerun of `3441369`, `K=+0.000,ND_a0=0.15,L=0.00`, reflection, `900G/12h`) still running, elapsed ~3h10.
- `3441719` (rerun of `3441371`, `K=+0.000,ND_a0=0.30,L=0.00`, reflection, `900G/12h`) still running, elapsed ~1h32.
- `3443492` (rerun of `3441373`, `K=+0.000,ND_a0=0.50,L=0.00`, reflection, `950G/12h`) still pending `(Priority)`.

New issue:
- Rerun `3443714` (rerun of original `3441375`, `K=+0.000,ND_a0=1.00,L=0.00`, reflection, center `100T0`, range `[50,150]T0`) failed with `OUT_OF_MEMORY` after `00:04:47` despite `700G/12h`.
- `sacct` batch MaxRSS was `734001712K`; `.err` showed Slurm `oom_kill` / Python killed in `propagate_2D_time_scan.py`.
- `.out` reached the padded-spectrum stage: original grid `Nx=5000`, `Ny=17500`; padded shape `(2,3,10016,35008,1)`; logged `field_pad` ~`15.7 GiB` and complex spectrum arrays ~`31.3 GiB` each.

Adaptive action:
- Ran `checkquota` before submission: MIKHAILOVA scratch `8.7/15 TiB`, OK (>2 TiB free).
- Submitted only this failed row with more memory, keeping 12h walltime:
  - `3443924`: rerun of `3443714` / original `3441375`, `K=+0.000,ND_a0=1.00,L=0.00`, reflection, center `100T0`, range `[50,150]T0`, `900G/12:00:00`.
- Rerun logdir: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_fine_20260708_2014_fine_focus_tiered/rerun_900G_12h_20260709_030959_idx31_oom3443714/`
- Rerun manifest: `.../rerun_900G_12h_20260709_030959_idx31_oom3443714/manifest.tsv`
- Immediate state: `3443924` pending `(Priority)`.

Effective unfinished rows after action:
- Running: `3441345`, `3441718`, `3441719`.
- Pending: `3443492`, `3443924`.

Reported to Zhiping in Telegram `main:6420513923:167`.

Next follow-up:
- Delayed self-email scheduled ~90 min later (subject: `Self-reminder: check remaining fine focus jobs after 3443924 rerun`).
- Check original `3441345–3441376` plus reruns `3441718`, `3441719`, `3443492`, `3443924`; validate outputs and `.err`; report final/blocker.

## Follow-up check: 3441345 completed, no new blocker — 2026-07-09 04:41 EDT

Triggered by delayed self-email `20260709T084031-5322`.

Status at ~04:40 EDT:
- `3441345` (`K=-0.002,ND_a0=0.10,L=0.00`, reflection, `900G/12h`) completed successfully in `05:27:22`; `sacct` batch MaxRSS `809020628K`.
- Remaining active jobs:
  - `3441718`: rerun of `3441369`, `K=+0.000,ND_a0=0.15,L=0.00`, reflection, `900G/12h`, running ~4h42.
  - `3441719`: rerun of `3441371`, `K=+0.000,ND_a0=0.30,L=0.00`, reflection, `900G/12h`, running ~3h04.
  - `3443924`: rerun of `3443714`/original `3441375`, `K=+0.000,ND_a0=1.00,L=0.00`, reflection, `900G/12h`, running ~1h16.
  - `3443492`: rerun of `3441373`, `K=+0.000,ND_a0=0.50,L=0.00`, reflection, `950G/12h`, still pending `(Priority)`.

Validation for newly completed `3441345`:
- `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/K=-0.002,ND_a0=0.10,L=0.00/reflection_beam_width_strength.h5` — `15470800` bytes.
- `.../reflection_beam_width_strength.png` — `173358` bytes.
- `.../reflection_peak_vs_x.nc` — `14998` bytes.
- `.err` contained only the known warning:
  ```text
  RuntimeWarning: All-NaN slice encountered
    beam_widths_min = np.nanmin(widths_2d, axis=0)
  ```

Effective accounting after action:
- `COMPLETED=28/32`.
- `RUNNING=3`: `3441718`, `3441719`, `3443924`.
- `PENDING=1`: `3443492`.
- No new failure/blocker in this check.

Reported to Zhiping in Telegram `main:6420513923:168`.

Next follow-up:
- Delayed self-email scheduled ~2h later (subject: `Self-reminder: follow up remaining fine focus jobs 3441718 3441719 3443492 3443924`).
- Check remaining reruns, validate any newly completed outputs, inspect `.err`, report final/blocker. Run `checkquota` before any further `sbatch`.

## Follow-up check: 3441718/3441719 completed; 3443492 OOM at 950G; 995G rerun — 2026-07-09 06:45 EDT

Triggered by delayed self-email `20260709T104135-8fc7`.

Status at ~06:42 EDT:
- `3441718` (rerun of `3441369`, `K=+0.000,ND_a0=0.15,L=0.00`, reflection, `900G/12h`) completed successfully in `05:56:10`; `sacct` batch MaxRSS `742743980K`.
- `3441719` (rerun of `3441371`, `K=+0.000,ND_a0=0.30,L=0.00`, reflection, `900G/12h`) completed successfully in `04:49:58`; `sacct` batch MaxRSS `746765688K`.
- `3443924` (rerun of `3443714`/original `3441375`, `K=+0.000,ND_a0=1.00,L=0.00`, reflection, `900G/12h`) still running, elapsed ~3h17.
- `3443492` (rerun of `3441373`, `K=+0.000,ND_a0=0.50,L=0.00`, reflection, `950G/12h`) failed with `OUT_OF_MEMORY` after `00:05:55`; `sacct` batch MaxRSS `996144680K`.

Validation for newly completed reruns:
- `3441718` outputs:
  - `reflection_beam_width_strength.h5` — `12380800` bytes.
  - `reflection_beam_width_strength.png` — `151336` bytes.
  - `reflection_peak_vs_x.nc` — `14998` bytes.
  - `.err`: only known `All-NaN slice` warning; `.out` ended with `=== Done ===`.
- `3441719` outputs:
  - `reflection_beam_width_strength.h5` — `12380800` bytes.
  - `reflection_beam_width_strength.png` — `124502` bytes.
  - `reflection_peak_vs_x.nc` — `14998` bytes.
  - `.err`: only known `All-NaN slice` warning; `.out` ended with `=== Done ===`.

Details for `3443492` OOM:
- Grid was larger than other failed rows: `Nx=6250`, `Ny=17500`.
- Padded shape reached `(2,3,12512,35008,1)`.
- Logged array sizes: `field_pad` ~`19.6 GiB`; complex `spectrum` and `Initial spectrum from field` arrays ~`39.2 GiB` each.
- Failure occurred after transversality was enforced / fixed grid setup, with Slurm OOM kill in `propagate_2D_time_scan.py`.

Adaptive action:
- Tried `1000G/12h` submission, but Slurm refused it: `Requested node configuration is not available`.
- Checked partition/node memory: serial nodes report about `1026099 MB`; `sbatch --test-only` accepted `960G`, `970G`, `980G`, `990G`, and `995G`, but rejected `1000G`.
- Ran `checkquota` again before actual submission: MIKHAILOVA scratch `8.7/15 TiB`, OK.
- Submitted only this failed row at the highest feasible request:
  - `3447199`: rerun of `3443492` / original `3441373`, `K=+0.000,ND_a0=0.50,L=0.00`, reflection, center `140T0`, range `[90,190]T0`, `995G/12:00:00`.
- Rerun logdir: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_fine_20260708_2014_fine_focus_tiered/rerun_995G_12h_20260709_064427_idx29_oom3443492/`
- Rerun manifest: `.../rerun_995G_12h_20260709_064427_idx29_oom3443492/manifest.tsv`
- Immediate state: `3447199` pending `(Priority)`.

Effective accounting after action:
- `COMPLETED=30/32`.
- `RUNNING=1`: `3443924`.
- `PENDING=1`: `3447199`.

Reported to Zhiping in Telegram `main:6420513923:169`.

Next follow-up:
- Delayed self-email scheduled ~2h later (subject: `Self-reminder: follow up final fine focus jobs 3443924 and 3447199`).
- Check `3443924` and `3447199`; validate newly completed outputs. If `3447199` OOMs again, report that simple resource escalation has hit the node limit and script-level memory reduction (e.g. chunking / avoiding simultaneous large complex arrays) is needed instead of blind rerun.

## Follow-up check: 3443924 completed; only 3447199 remains — 2026-07-09 08:46 EDT

Triggered by delayed self-email `20260709T124502-da0c`.

Status at ~08:45 EDT:
- `3443924` (rerun of `3443714`/original `3441375`, `K=+0.000,ND_a0=1.00,L=0.00`, reflection, `900G/12h`) completed successfully in `04:57:59`; `sacct` batch MaxRSS `729570824K`.
- `3447199` (rerun of `3443492`/original `3441373`, `K=+0.000,ND_a0=0.50,L=0.00`, reflection, `995G/12h`) was running, elapsed ~`01:41:27`, started at `2026-07-09T07:04:01`.

Validation for newly completed `3443924`:
- `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/K=+0.000,ND_a0=1.00,L=0.00/reflection_beam_width_strength.h5` — `12380800` bytes.
- `.../reflection_beam_width_strength.png` — `137014` bytes.
- `.../reflection_peak_vs_x.nc` — `14998` bytes.
- `.err`: only known `All-NaN slice` warning.
- `.out`: ended with HDF saved, peak_vs_x saved, summary plot saved, `=== Done ===`.

Effective accounting after action:
- `COMPLETED=31/32`.
- `RUNNING=1`: `3447199`.
- Since the previous `3443492` attempt OOMed after `00:05:55`, the fact that `3447199` has run >1.5h suggests it has passed the earliest memory-kill stage, but final success still needs validation.

Reported to Zhiping in Telegram `main:6420513923:170`.

Next follow-up:
- Delayed self-email scheduled ~4h later (subject: `Self-reminder: final check fine focus job 3447199`).
- Check `3447199`; if completed, validate outputs and report final. If it OOMs again, do not blindly rerun; report that algorithmic memory reduction is needed.

## Follow-up check: 3447199 still running  2026-07-09 12:47 EDT

Triggered by delayed self-email `20260709T164605-bc76`.

Status at ~12:46 EDT:
- `3447199` (rerun of `3443492`/original `3441373`, `K=+0.000,ND_a0=0.50,L=0.00`, reflection, `995G/12h`) was still running on `tiger-g02c3n6`.
- `squeue`: `RUNNING`, elapsed `05:42:34`, time limit `12:00:00`, start `2026-07-09T07:04:01`.
- `sacct`: job, batch, and extern all `RUNNING`; ReqMem `995G`; MaxRSS not available yet.
- Logdir located from manifest: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_fine_20260708_2014_fine_focus_tiered/rerun_995G_12h_20260709_064427_idx29_oom3443492/`.

Output state:
- The case directory still had old/stale `reflection_beam_width_strength.h5` and `.png` timestamps from 2026-07-08 18:17, likely from an earlier failed/intermediate attempt.
- `reflection_peak_vs_x.nc` was still missing, so final validation must wait for `3447199` completion.

Interpretation:
- Effective accounting remains `COMPLETED=31/32`, `RUNNING=1` (`3447199`).
- Since the prior 900G/950G attempts OOMed in ~6 minutes and the current run has exceeded 5.7h, it has passed the early OOM stage; no new failure at this check.
- No job was submitted or rerun.

Reported to Zhiping in Telegram `main:6420513923:175`.

Next follow-up:
- Delayed self-email scheduled for ~2h later (subject: `Self-reminder: recheck fine focus job 3447199`).
- Recheck `squeue`/`sacct`; if completed, validate `reflection_beam_width_strength.h5`, `reflection_beam_width_strength.png`, and `reflection_peak_vs_x.nc`; inspect `.err`. If OOM/timeout, report blocker and do not blindly rerun.

## Follow-up check: 3447199 actively progressing  2026-07-09 14:48 EDT

Triggered by delayed self-email `20260709T184706-5e18`.

Status at ~14:47 EDT:
- `3447199` still `RUNNING` on `tiger-g02c3n6`.
- `squeue`: elapsed `07:43:36`, time limit `12:00:00`, ReqMem `995G`, start `2026-07-09T07:04:01`.
- `sacct`: job, batch, extern all `RUNNING`; MaxRSS still unavailable while running.
- `.err` in the rerun logdir was still `0 bytes`.
- `.out` was actively updating; latest tail reached:
  ```text
  t_idx=77  t=+167.0T0  |centerline|_max=47.336Ec  valid_width_pts=331
  ```
- This scan has 101 time points over `[90,190]T0`, so progress was roughly three quarters complete.

Output state:
- Final `reflection_peak_vs_x.nc` was still missing.
- Existing `reflection_beam_width_strength.h5` and `.png` in the case directory still had 2026-07-08 timestamps and should be treated as stale/intermediate until the current job finishes and rewrites the final outputs.

Interpretation:
- Effective accounting remains `COMPLETED=31/32`, `RUNNING=1` (`3447199`).
- No OOM/timeout; the job is making visible progress and is expected to be near completion at the next check if the current pace continues.
- No job was submitted or rerun.

Reported to Zhiping in Telegram `main:6420513923:176`.

## Final aggregation into focus.tsv — 2026-07-09 19:25 EDT

Zhiping reported in Telegram `main:6420513923:177` that all 32 jobs had completed and requested updating `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/focus.tsv`:
- Replace each row's `propagation_time (laser_period)` with the `t` value at maximum `|centerline|_max` over the 101-point scan.
- Add `E_max(Ec)` recording that maximum in units of `Ec`.
- Use the 32 scan `.out` files as source.

Implementation:
- Backed up original focus file:
  `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/focus.tsv.bak_20260709_192443_before_Emax_update`
- Updated target file:
  `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/focus.tsv`
- Wrote provenance/argmax table:
  `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_fine_20260708_2014_fine_focus_tiered/focus_time_scan_argmax_summary_20260709_192443.tsv`

Validation:
- Parsed 32 focus rows and 32 manifest rows; row order and case/side mapping matched.
- Selected one complete `=== Done ===` 101-point `.out` per row, preferring successful rerun logs where original attempts failed.
- Updated `focus.tsv` has 32 rows, no missing `propagation_time (laser_period)`, and no missing `E_max(Ec)`.
- `propagation_time` changed in 30/32 rows.
- `E_max(Ec)` range: `1.793` to `156.934`.
- Final job `3447199` confirmed `COMPLETED`, elapsed `09:50:22`, MaxRSS `917610972K`; final outputs existed and were nonempty:
  - `reflection_beam_width_strength.h5` 15,470,800 bytes
  - `reflection_beam_width_strength.png` 135,217 bytes
  - `reflection_peak_vs_x.nc` 14,998 bytes

Top `E_max` cases:
```text
K=-0.005,ND_a0=0.30 reflection : t=-11T0, E_max=156.934 Ec
K=-0.008,ND_a0=0.30 reflection : t= -6T0, E_max=153.544 Ec
K=-0.008,ND_a0=0.50 reflection : t= -5T0, E_max=123.376 Ec
K=-0.005,ND_a0=0.50 reflection : t= -6T0, E_max=115.714 Ec
K=-0.002,ND_a0=0.30 reflection : t= 30T0, E_max=113.513 Ec
```

Reported completion to Zhiping in Telegram `main:6420513923:179`. Fine focus time-scan batch is complete unless follow-up plotting/analysis is requested.

## Follow-up focus.tsv x-coordinate update — 2026-07-10 10:51 EDT

Zhiping requested in Telegram `main:6420513923:180` that the TSV also include the `x` position corresponding to each maximum `|centerline|_max`.

Implementation:
- Source for `x`: each row's existing `<side>_peak_vs_x.nc` file generated by `propagate_2D_time_scan.py`.
- Semantics from the script: `x_at_peak` is `x_at_peak_per_t / lambda_0`, and `E_peak` is the per-timestep centerline envelope peak divided by `Ec`.
- For each of 32 rows, selected `x_at_peak` at `argmax(E_peak)` and wrote it into `focus.tsv`.
- Copied reproducible script to the project utility folder:
  `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/update_focus_with_x_at_emax_20260710.py`
- Local copy of the script:
  `/home/zhiping/ULMI_lab/.lingtai/main/artifacts/update_focus_with_x_at_emax_20260710.py`

Updated target:
- `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/focus.tsv`
- New column: `x_at_E_max (lambda0)`.

Backup before this update:
- `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/focus.tsv.bak_20260710_105035_before_x_at_Emax_update`

Provenance/check table:
- `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_fine_20260708_2014_fine_focus_tiered/focus_x_at_Emax_summary_20260710_105035.tsv`

Validation:
- `rows=32`, provenance rows `=32`.
- No missing `x_at_E_max (lambda0)` values.
- `x_at_E_max` range: `32.178 .. 332.342 lambda0`.
- Max discrepancy between `focus.tsv`'s rounded `E_max(Ec)` and `max(E_peak)` from NetCDF: `4.51e-4 Ec`, consistent with 3-decimal TSV rounding.

Top `E_max` rows after x update:
```text
K=-0.005,ND_a0=0.30,L=0.00 reflection : t=-11T0, E_max=156.934 Ec, x=57.330 lambda0
K=-0.008,ND_a0=0.30,L=0.00 reflection : t= -6T0, E_max=153.544 Ec, x=37.326 lambda0
K=-0.008,ND_a0=0.50,L=0.00 reflection : t= -5T0, E_max=123.376 Ec, x=38.438 lambda0
K=-0.005,ND_a0=0.50,L=0.00 reflection : t= -6T0, E_max=115.714 Ec, x=62.442 lambda0
K=-0.002,ND_a0=0.30,L=0.00 reflection : t= 30T0, E_max=113.513 Ec, x=118.338 lambda0
```

Reported completion to Zhiping in Telegram `main:6420513923:182`.

## Reflection E_max vs K plot — 2026-07-10 10:58 EDT

Zhiping requested in Telegram `main:6420513923:183` and clarified in `main:6420513923:185`:
- Plot `side=reflection` only.
- Horizontal axis: `K`.
- Vertical axis: `E_max(Ec)` / `E_max/E_c`.
- Put the three fixed-`ND/a0` curves (`0.30`, `0.50`, `1.00`) on one figure.

Implementation:
- Source data: updated `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/focus.tsv`.
- Parsed `K` and `ND_a0` from case directory names and filtered `side=reflection`.
- Included 12 points total: 4 K values for each of `ND/a0 = 0.30, 0.50, 1.00`.
- Script stored at:
  `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/plot_Emax_vs_K_reflection_ND_scan_a0_20.py`
- Notebook stored at:
  `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/plot_Emax_vs_K_reflection_ND_scan_a0_20.ipynb`

Outputs:
- PNG:
  `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_summary_plots/Emax_vs_K_reflection_ND_scan_a0_20.png`
- PDF:
  `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_summary_plots/Emax_vs_K_reflection_ND_scan_a0_20.pdf`
- Long CSV:
  `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_summary_plots/Emax_vs_K_reflection_ND_scan_a0_20_long.csv`

Data used:
```text
ND/a0=0.30: K=-0.008 E=153.544, K=-0.005 E=156.934, K=-0.002 E=113.513, K=0.000 E=71.423
ND/a0=0.50: K=-0.008 E=123.376, K=-0.005 E=115.714, K=-0.002 E=80.356, K=0.000 E=47.555
ND/a0=1.00: K=-0.008 E=85.510,  K=-0.005 E=82.934,  K=-0.002 E=61.754,  K=0.000 E=30.687
```

Validation:
- Plot visually checked: all three `ND/a0` curves are in one figure, x-axis `K`, y-axis `E_max/E_c`, legend correct.
- Local attachment copies kept under:
  `/home/zhiping/ULMI_lab/.lingtai/main/artifacts/curved_surface_focus_plots_20260710/`

Reported paths in Telegram `main:6420513923:187` and sent PNG/PDF/notebook attachments in `main:6420513923:188-190`.

Follow-up in Telegram `main:6420513923:191`: Zhiping asked to use the same notebook and add `x_at_E_max` vs `K`, again with the three `ND/a0 = 0.30, 0.50, 1.00` curves on one figure.

Follow-up implementation:
- Updated the same notebook and script:
  `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/plot_Emax_vs_K_reflection_ND_scan_a0_20.ipynb`
  `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/plot_Emax_vs_K_reflection_ND_scan_a0_20.py`
- The notebook now contains both plots:
  1. `E_max/E_c` vs `K`.
  2. `x_at_E_max/lambda0` vs `K`.
- New x-position plot outputs:
  `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_summary_plots/x_at_Emax_vs_K_reflection_ND_scan_a0_20.png`
  `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_summary_plots/x_at_Emax_vs_K_reflection_ND_scan_a0_20.pdf`

Data used for x-position plot:
```text
ND/a0=0.30: K=-0.008 x=37.326, K=-0.005 x=57.330, K=-0.002 x=118.338, K=0.000 x=282.342
ND/a0=0.50: K=-0.008 x=38.438, K=-0.005 x=62.442, K=-0.002 x=144.446, K=0.000 x=216.438
ND/a0=1.00: K=-0.008 x=39.490, K=-0.005 x=65.490, K=-0.002 x=153.494, K=0.000 x=165.494
```

Validation:
- The new plot was visually checked: three `ND/a0` curves in one figure, x-axis `K`, y-axis `x_Emax/lambda0`.
- Reported in Telegram `main:6420513923:193` and sent x-plot PNG/PDF plus updated notebook in `main:6420513923:194-196`.
