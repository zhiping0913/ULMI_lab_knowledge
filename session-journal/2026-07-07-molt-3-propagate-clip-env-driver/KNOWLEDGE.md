---
name: 2026-07-07-molt-3-propagate-clip-env-driver
description: >-
  Session journal for the Curved_surface a0=20/2D propagate_and_clip env-var driver, Slurm generation, submission, and validation segment.
type: session-journal
date: 2026-07-07
molt_count: 3
---

# Session journal — Curved_surface propagate_and_clip env-var driver and validation

## TL;DR

Zhiping asked main to inspect, adapt, dry-run, submit, and validate the a0=20/2D `propagate_and_clip` workflow on tiger-vis. The workflow is now durable:

- `propagate_and_clip.py` was backed up and modified to read case-specific parameters from environment variables with old defaults preserved.
- `generate_propagate_and_clip_slurms.sh` was added to generate per-row Slurm scripts from the a0=20/2D tab-delimited CSV.
- 31 unfinished rows were dry-run generated, submitted after quota check, and later validated cleanly.
- Zhiping instructed not to proactively monitor after submission; he later called when completion was observed.

## Timeline and accomplishments

### 1. Read-only inspection and memory setup

Earlier in the segment, main inspected the tiger-vis path:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D
```

and recorded the K,D,L case structure, input decks, `start_2D.py`, `rotate_and_shift.py`, and `propagate_and_clip.slurm` relationships in the Curved_surface project memory.

Key project memory entry:

```text
knowledge/curved-surface-chf-project/KNOWLEDGE.md
```

Important supporting files updated/created:

```text
knowledge/curved-surface-chf-project/run_registry.md
knowledge/curved-surface-chf-project/workflow_registry.md
knowledge/curved-surface-chf-project/inspection_2026-07-06_a0-20-2d.md
knowledge/curved-surface-chf-project/propagate_clip_plan_2026-07-06.md
```

### 2. Zhiping corrected the automation architecture

Main first suggested making `propagate_and_clip.py` read the CSV. Zhiping corrected this in Telegram `main:6420513923:85`:

- Do **not** link `propagate_and_clip.py` to `propagate_and_clip.csv`.
- `propagate_and_clip.py` should remain a generic propagation/clipping kernel for future scans such as `a0=50`.
- Case-specific hard-coded values should be converted to environment-variable parameters.
- An outer bash/slurm driver should read the current batch CSV and export parameters per row.

This was written into memory and pad. This is the standing design for this workflow.

### 3. Implemented dry-run driver on tiger-vis

Zhiping authorized writing the driver in Telegram `main:6420513923:90`.

Remote backup:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_and_clip.py.bak_env_20260706_214530
```

Modified Python:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_and_clip.py
```

New driver:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/generate_propagate_and_clip_slurms.sh
```

Generated Slurm directory:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_and_clip_slurms/a0_20_2D/
```

The Python now reads these env vars with old defaults preserved:

```text
WORKING_DIR, SIDE,
X_MIN_CLIP_0, X_MAX_CLIP_0, Y_MIN_CLIP_0, Y_MAX_CLIP_0,
X_MIN_CLIP_1, X_MAX_CLIP_1, Y_MIN_CLIP_1, Y_MAX_CLIP_1,
PROPAGATION_TIME
```

Dry-run validation passed:

```text
SLURM_SYNTAX_OK=31
WOULD_SBATCH_COUNT=31
summary: generated=31, skipped_done=1
```

A small generator bug was caught and fixed: the TSV lacked a final newline, so a basic bash `while read` loop initially generated only 30 scripts. The loop was changed to:

```bash
while read ... || [[ -n "${case_rel:-}" ]]; do
```

so the final row is retained.

Detailed memory:

```text
knowledge/curved-surface-chf-project/propagate_clip_dryrun_2026-07-06.md
```

### 4. Submitted 31 jobs after quota check

Zhiping authorized submission in Telegram `main:6420513923:94`. Main still ran `checkquota` first, per standing rule.

Quota result:

```text
Tiger3 scratch GPFS fileset MIKHAILOVA: 8.7 TiB used / 15 TiB max
```

Free scratch was about 6.3 TiB, above the 2 TiB threshold.

Submission command:

```bash
SUBMIT=1 DRY_RUN=0 /scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/generate_propagate_and_clip_slurms.sh
```

Result:

- 31 jobs submitted.
- One `done=Y` row skipped: `K=-0.002,D=0.01,L=0.00 transmission`.
- Job IDs: `3382363–3382393`.
- Immediate `squeue`: all 31 `PENDING`, reason `(None)`.

Detailed memory:

```text
knowledge/curved-surface-chf-project/propagate_clip_submission_2026-07-06.md
```

### 5. Validated completion

Zhiping later reported in Telegram `main:6420513923:99` that the 31 jobs completed. Main performed a read-only validation pass.

Validation output:

```text
sacct: COMPLETED = 31
bad_count = 0
expected_rows = 31
present_nc_png = 31
missing_count = 0
small_file_count = 0
nonempty_err_count = 0
out_potential_bad_count = 0
```

All expected `<side>_clip.nc` and `<side>_clip.png` files exist. No missing files, no small files, no nonempty `.err`, no suspicious `.out` by simple marker heuristic.

Detailed memory:

```text
knowledge/curved-surface-chf-project/propagate_clip_validation_2026-07-07.md
```

## Important decisions and rules

1. `propagate_and_clip.py` should remain generic and reusable.
2. Per-batch CSVs should be consumed by outer drivers, not by the generic Python kernel.
3. Use environment variables as the parameter boundary between Slurm and Python.
4. Keep old hard-coded defaults for VSCode/manual/debug fallback.
5. For complex remote Python or multiline scripts, avoid fragile ssh heredocs; write a local script, `scp` it to tiger-vis, then execute it. This was learned after two heredoc quoting failures.
6. Before any future `sbatch`/`salloc`, main must still run `checkquota` personally, even if Zhiping says he already checked.
7. Zhiping instructed after submission not to proactively monitor; he will call main when jobs finish or error.

## Artifacts and exact paths

Remote scripts:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_and_clip.py
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_and_clip.py.bak_env_20260706_214530
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/generate_propagate_and_clip_slurms.sh
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_and_clip_slurms/a0_20_2D/
```

Local artifacts:

```text
artifacts/curved_surface_2d_a020_inspection_20260706_204001/propagate_and_clip.env_patched.py
artifacts/curved_surface_2d_a020_inspection_20260706_204001/generate_propagate_and_clip_slurms.sh
artifacts/curved_surface_2d_a020_inspection_20260706_204001/patch_propclip_env_driver.py
artifacts/curved_surface_2d_a020_inspection_20260706_204001/fix_generator_final_newline.py
artifacts/curved_surface_2d_a020_inspection_20260706_204001/validate_propclip_jobs.py
```

Knowledge files:

```text
knowledge/curved-surface-chf-project/propagate_clip_plan_2026-07-06.md
knowledge/curved-surface-chf-project/propagate_clip_dryrun_2026-07-06.md
knowledge/curved-surface-chf-project/propagate_clip_submission_2026-07-06.md
knowledge/curved-surface-chf-project/propagate_clip_validation_2026-07-07.md
```

## Open state and next steps

No active job is waiting. The 31 submitted jobs completed and validation was clean.

If Zhiping resumes this thread, likely next steps are scientific / post-processing:

- use the clipped fields for propagation/focusing diagnostics;
- quantify `G_spatial` from curved-foil focusing;
- eventually compare actual focus against an ideal coherent focus to estimate `C_wavefront`;
- keep mapping results to the core factorization:

```text
S_focus^peak/(a0² S_c,M) = G_CSE/1D × G_spatial × C_wavefront
```

## State for successor

- The remote workflow has been modified and validated; do not redo unless asked.
- Do not monitor these jobs; they are already complete and validated.
- If future submissions are requested, run `checkquota` first.
- If writing complex remote code, prefer local file + `scp` over ssh heredoc.
