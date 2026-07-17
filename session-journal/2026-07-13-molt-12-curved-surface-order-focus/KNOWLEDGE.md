---
name: 2026-07-13-molt-12-curved-surface-order-focus
description: >-
  Session journal for the Curved_surface order-resolved coarse focus scan, follow-up band-resolved Emax/xEmax-vs-K plotting, and associated cache-miss-budget molt handoff.
type: session-journal
created_at: 2026-07-13T10:09:00-04:00
---

# Session journal — Curved_surface order-resolved focus scan and plots

## TL;DR

This segment completed a full order-resolved coarse focus scan for Zhiping's `Curved_surface/a0=20/2D` cases, extracted per-band focal argmax diagnostics, and produced follow-up reflection plots of `E_max/Ec` and `x_Emax/λ0` versus curvature `K` for `ND/a0=0.30,0.50,1.00` and `band1/band2/band3`. All Slurm jobs completed successfully. All deliverables were reported and attached on Telegram. No active background jobs remain.

The session is ending because the since-last-molt cache-miss budget is nearly exhausted, not because of unresolved work.

## Human/channel context

- Human: Zhiping, Telegram chat `main:6420513923`.
- Key new instruction: Telegram `main:6420513923:217` — use updated tiger-vis `propagate_2D_time_scan.py` k-band filtering to scan `focus.tsv` rows over `propagation_time ±100T0`, step `10T0`, to determine order-specific focus locations. Zhiping noted `K=+0.000,ND_a0=0.50,L=0.00/reflection_clip.nc` had been re-cropped and should no longer need the old 995 GB exceptional memory.
- Follow-up instruction: Telegram `main:6420513923:224` — plot, analogous to the earlier total-field notebook, `E_max` and `x_Emax` versus `K` for the 1st, 2nd, and high-order bands under three `ND/a0` values.

## Work completed

### 1. Order-resolved coarse focus scan

Remote script inspected:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan.py
```

Band definitions already present:

- `total`: full field envelope
- `band1`: `|k| in [0.5,1.5] k0`
- `band2`: `|k| in [1.5,2.5] k0`
- `band3`: `|k| in [2.5,40.5] k0`

I patched the script with backward-compatible `output_dir` support to prevent overwriting previous fine-scan files. Backup:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan.py.bak_output_dir_20260712_212538
```

New driver:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/submit_order_time_scan_from_focus.sh
```

Status helper:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/check_order_scan.py
```

Summary extractor:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/summarize_order_focus_scan.py
```

Dry-run validated 32/32 rows and existing `*_clip.nc` inputs. Before submission, `checkquota` showed MIKHAILOVA scratch at 8.7/15 TiB, safely above the 2 TiB-free block threshold.

Submitted jobs:

```text
3498134–3498166  (Slurm skipped 3498143)
```

Logdir:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_order_focus_pm100_10T_20260712_2126_order_pm100_10T/
```

Resource policy:

- 27 standard jobs: `700G/5h`
- 5 big-clip jobs (`clip.nc >= 1.75 GiB`): `900G/12h`
- The re-cropped `K=+0.000,ND_a0=0.50,L=0.00` reflection row used `900G/12h` and completed; no `995G` special case was needed.

Final validation:

```text
COMPLETED = 32/32
output_ok = 32/32
FAILED/OOM/TIMEOUT = 0
```

Warning caveat: `.err` files only had `RuntimeWarning: All-NaN slice encountered` and one `PeakPropertyWarning`; no traceback/OOM/missing output.

### 2. Per-band focus summary table

Extracted per-band argmax diagnostics into:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_order_focus_pm100_10T_20260712_2126_order_pm100_10T/order_focus_per_band_argmax_summary_latest.tsv
```

Local copy / Telegram attachment source:

```text
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/curved_surface_order_scan_20260712/order_focus_per_band_argmax_summary_latest.tsv
```

Rows: `32 focus rows × 4 bands = 128`, all status `ok`.

Selection rule: sampled argmax of `E_peak` across the 21 propagation samples for each band; not a continuous focus optimization.

Strongest reflection rows from this table:

- total: `K=-0.005,ND_a0=0.30`, `E≈157.96 Ec`, `x≈57.33 λ0`, `width_y≈1.03 λ0`
- band1: `K=-0.008,ND_a0=1.00`, `E≈50.68 Ec`, `x≈39.81 λ0`, `width_y≈3.05 λ0`
- band2: `K=-0.008,ND_a0=0.30`, `E≈25.96 Ec`, `x≈37.82 λ0`, `width_y≈1.92 λ0`
- band3: `K=-0.005,ND_a0=0.30`, `E≈117.12 Ec`, `x≈57.33 λ0`, `width_y≈0.636 λ0`

### 3. Follow-up plots

Zhiping asked to plot order-resolved `E_max` and `x_Emax` vs `K` for three `ND/a0` values. I generated two 1×3 panel figures using:

```text
side = reflection
ND/a0 = 0.30, 0.50, 1.00
bands = band1, band2, band3
```

Remote figure outputs:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_summary_plots/order_reflection_Emax_vs_K_ND_scan_a0_20.png
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_summary_plots/order_reflection_Emax_vs_K_ND_scan_a0_20.pdf
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_summary_plots/order_reflection_xEmax_vs_K_ND_scan_a0_20.png
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_summary_plots/order_reflection_xEmax_vs_K_ND_scan_a0_20.pdf
```

Notebook/script:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/plot_order_focus_vs_K_reflection_ND_scan_a0_20.ipynb
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/plot_order_focus_vs_K_reflection_ND_scan_a0_20.py
```

Long CSV:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_summary_plots/order_reflection_band_focus_vs_K_ND_scan_a0_20_long.csv
```

Local copies under:

```text
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/curved_surface_order_scan_20260712/
```

Telegram final report and attachments:

- order scan final report: `main:6420513923:222`
- summary TSV attachment: `main:6420513923:223`
- plot request acknowledgement: `main:6420513923:225`
- plot report: `main:6420513923:226`
- plot attachments: `main:6420513923:227-232`

## Durable memory updates

Updated/created:

- `knowledge/curved-surface-chf-project/order_focus_scan_2026-07-13.md`
- `knowledge/curved-surface-chf-project/order_focus_plots_2026-07-13.md`
- `knowledge/curved-surface-chf-project/KNOWLEDGE.md` parent index / `last_updated`
- `system/pad.md` current work state

No new skill was needed. No character/lingtai identity update needed beyond existing HPC/resource-adaptive habits.

## Scientific interpretation and caveats

Mechanism-level reading: stronger negative curvature moves lower-order band1/band2 foci closer to the target, consistent with shorter focal length from curved surface focusing. The flat `K=0` rows have broader/weakly focused low-order peaks. High-order aggregate `band3` is closer to the total-field strong focus and is dominated by the higher-frequency content, so it should not be treated as a single harmonic.

Caveats:

- All per-band focus positions are sampled argmax values over 21 points (`center±100T0`, `10T0` step), not refined continuous optima.
- `E_peak` is field envelope peak per band, not efficiency or integrated energy.
- `band3` is a broad high-order aggregate `[2.5,40.5] k0`.
- Do not overwrite `focus.tsv` with band-resolved focus positions unless Zhiping explicitly asks.

## Open tasks / first five minutes after molt

No active Slurm jobs or reminders remain.

After molt:

1. Check Telegram/email for new Zhiping messages.
2. If no new message, sleep/idle.
3. If Zhiping asks to refine these plots, use the notebook/script paths above.
4. If Zhiping asks for more precise per-band foci, refine locally around the per-band sampled argmax values using narrower time windows; run `checkquota` before any new Slurm submissions.
5. If Zhiping asks to update `focus.tsv`, do not overwrite the total-field columns blindly; create separate band-resolved columns/table unless he explicitly specifies the format.
