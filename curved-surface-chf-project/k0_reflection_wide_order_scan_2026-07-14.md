# K=0 reflection wide order-focus rescan — 2026-07-14

Source:
- Zhiping Telegram `main:6420513923:248` requested a wider time-range rescan for the four `K=+0.000` reflection cases because the previous `±100T0` order-resolved scan missed / clipped some order-specific foci. main acknowledged in `main:6420513923:249` and reported submission in `main:6420513923:250`.

## Motivation

Previous curved cases `K=-0.002,-0.005,-0.008` gave reasonable order-resolved foci. For flat `K=+0.000`, order foci are much more separated:

- `K=+0.000,ND_a0=0.15,L=0.00`: total peak at `x/λ0≈330`; band1 peak had not appeared inside the prior `total-peak ±100λ0` window and should be earlier than about `230λ0`.
- `K=+0.000,ND_a0=0.50,L=0.00`: total peak at `x/λ0≈220`; band1 peak is near/below `120λ0`, while high-order `[2.5,40.5]` (`band3`) peak lies beyond `320λ0`.

Mechanism: with curvature, different harmonic orders focus at relatively close positions because the imposed curved wavefront dominates. Without curvature (`K=0`), each order's propagation/focusing behavior separates more strongly, so a scan centered on total-field focus can miss low-order or high-order foci.

## Previous boundary check

Previous scan directory:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/*/order_focus_pm100_10T/
```

Summary table:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_order_focus_pm100_10T_20260712_2126_order_pm100_10T/order_focus_per_band_argmax_summary_latest.tsv
```

Boundary evidence from previous NetCDFs:

| case | previous x-range | boundary-clipped orders |
|---|---:|---|
| `K=+0.000,ND_a0=0.15,L=0.00` | `232–432 λ0` | `band1` max at index 0; `band2` at index 1 |
| `K=+0.000,ND_a0=0.30,L=0.00` | `182–382 λ0` | `band1` at index 1; `band2` at index 2 |
| `K=+0.000,ND_a0=0.50,L=0.00` | `116–316 λ0` | `band3` max at upper index 20 |
| `K=+0.000,ND_a0=1.00,L=0.00` | `65–265 λ0` | `band3` max at upper index 20 |

## New scan parameters

Subset TSV:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_k0_reflection_subset_20260714.tsv
```

Driver reused:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/submit_order_time_scan_from_focus.sh
```

New scan settings:

```text
HALF_WIDTH_T0 = 300
NUM_POINTS = 61
step = 10T0
SCAN_TAG = order_focus_pm300_10T_K0_reflection
side = reflection only
K = +0.000 only
```

Output directories are separate from previous products and do not overwrite old scans:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/K=+0.000,ND_a0=...,L=0.00/order_focus_pm300_10T_K0_reflection/
```

Logdir / manifest:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_order_focus_pm300_10T_K0_reflection_20260714_1944/manifest.tsv
```

## Resource rationale

Before submission, `checkquota` reported MIKHAILOVA scratch at `9.2/15 TiB`, so about `5.8 TiB` free, above the 2 TiB Tiger threshold.

Previous 21-point K=0 reflection jobs used 900G/12h and completed:

| old job | case | elapsed | MaxRSS |
|---|---|---:|---:|
| `3498159` | ND0.15 | `02:02:33` | `824347068K` |
| `3498161` | ND0.30 | `02:35:00` | `751221960K` |
| `3498163` | ND0.50 | `02:22:55` | `759360972K` |
| `3498165` | ND1.00 | `02:13:38` | `739439484K` |

The new 61-point scan has about 3× the time samples. Memory should be similar because array size is dominated by the clip, not number of time samples retained at once. Therefore `900G/12h` was retained rather than increasing resources blindly.

## Submitted jobs

Submitted at 2026-07-14 19:43 EDT. Immediate `squeue`: all four `PENDING (None)`.

| job | case | center | range | resources |
|---|---|---:|---:|---|
| `3512584` | `K=+0.000,ND_a0=0.15,L=0.00` | `217T0` | `[-83,517]T0` | `900G/12h` |
| `3512585` | `K=+0.000,ND_a0=0.30,L=0.00` | `224T0` | `[-76,524]T0` | `900G/12h` |
| `3512586` | `K=+0.000,ND_a0=0.50,L=0.00` | `-2T0` | `[-302,298]T0` | `900G/12h` |
| `3512587` | `K=+0.000,ND_a0=1.00,L=0.00` | `107T0` | `[-193,407]T0` | `900G/12h` |

A self-reminder was scheduled for ~2h later to check `squeue/sacct`, `.err` logs, and output NetCDFs. If all complete, extract per-band argmax summaries for these four rows and report to Zhiping.


## First scheduled check — 2026-07-14 21:44 EDT

Self-reminder `20260715T014421-01b5` fired and was handled. Tiger `squeue`/`sacct` showed all four jobs still pending due priority:

```text
3512584  ord_1_reflection  PENDING  00:00:00  ReqMem=900G  None assigned
3512585  ord_2_reflection  PENDING  00:00:00  ReqMem=900G  None assigned
3512586  ord_3_reflection  PENDING  00:00:00  ReqMem=900G  None assigned
3512587  ord_4_reflection  PENDING  00:00:00  ReqMem=900G  None assigned
```

Logdir contained only `manifest.tsv`; no `.out/.err` files and no nonempty `.err`. Reported to Zhiping in Telegram `main:6420513923:270`. A follow-up self-reminder was scheduled for 2 hours later.


## Second scheduled check — 2026-07-14 23:45 EDT

Self-reminder `20260715T034504-a21d` fired and was handled. Tiger `squeue`/`sacct` again showed all four jobs still pending due priority:

```text
3512584  ord_1_reflection  PENDING  00:00:00  ReqMem=900G  None assigned
3512585  ord_2_reflection  PENDING  00:00:00  ReqMem=900G  None assigned
3512586  ord_3_reflection  PENDING  00:00:00  ReqMem=900G  None assigned
3512587  ord_4_reflection  PENDING  00:00:00  ReqMem=900G  None assigned
```

Logdir still contained only `manifest.tsv`; no nonempty `.err`. No Telegram report was sent for this unchanged pending state; a third self-reminder was scheduled for 3 hours later.


## Third scheduled check — 2026-07-15 02:46 EDT

Self-reminder `20260715T064547-7935` fired and was handled. Tiger `squeue`/`sacct` again showed all four jobs still pending due priority:

```text
3512584  ord_1_reflection  PENDING  00:00:00  ReqMem=900G  None assigned
3512585  ord_2_reflection  PENDING  00:00:00  ReqMem=900G  None assigned
3512586  ord_3_reflection  PENDING  00:00:00  ReqMem=900G  None assigned
3512587  ord_4_reflection  PENDING  00:00:00  ReqMem=900G  None assigned
```

Logdir still contained only `manifest.tsv`; no nonempty `.err`. No Telegram report was sent for this unchanged pending state; a fourth self-reminder was scheduled for 4 hours later.


## Fourth scheduled check — 2026-07-15 06:46 EDT

Self-reminder `20260715T104620-5f29` fired and was handled. Tiger `squeue`/`sacct` again showed all four jobs still pending due priority:

```text
3512584  ord_1_reflection  PENDING  00:00:00  ReqMem=900G  None assigned
3512585  ord_2_reflection  PENDING  00:00:00  ReqMem=900G  None assigned
3512586  ord_3_reflection  PENDING  00:00:00  ReqMem=900G  None assigned
3512587  ord_4_reflection  PENDING  00:00:00  ReqMem=900G  None assigned
```

Logdir still contained only `manifest.tsv`; no nonempty `.err`. No Telegram report was sent for this unchanged pending state; a fifth self-reminder was scheduled for 4 hours later.

## 2026-07-19 resumed monitoring and row1 retry

After many days pending, the jobs were still queued through 2026-07-19 morning. At 15:21 EDT, `3512584` (`ord_1_reflection`, `ND/a0=0.15`) was running on `tiger-i05g12`, elapsed `01:53`, `ReqMem=900G`; `.err` was empty and `.out` showed progress around `t_idx=13`, `t=+47T0`, `|centerline|max≈32.9 Ec`. Jobs `3512585` was pending `(Resources)`, and `3512586–3512587` pending `(Priority)`. At 17:22 EDT, `3512584` was still running, elapsed `03:54`, with `.err` empty and progress around `t_idx=28`, `t=+197T0`, `|centerline|max≈34.4 Ec`.

At 19:23 EDT, `3512584` had failed:

```text
State: FAILED
Elapsed: 03:55:21
ExitCode: 134:0
MaxRSS: 801173292K
ReqMem: 900G
Start: 2026-07-19T13:28:21
End:   2026-07-19T17:23:42
```

The error log showed a JAX/XLA CPU all-reduce rendezvous termination timeout (`Expected 3 threads ... only 2 arrived`) followed by abort/core dump. This was **not** a Slurm OOM: MaxRSS was ~801 GB under a 900G request. The out log last showed progress near `t_idx=28`, `t=+197T0`; no final expected files (`reflection_peak_vs_x.nc`, env-total/band1-4 peak_vs_x NetCDFs, `reflection_beam_width_strength.h5/.png`) were present for row1 or the other still-pending rows.

There were no saved per-row Slurm scripts in the original logdir, only `manifest.tsv`. I generated a single-row retry Slurm script from the manifest and `propagate_2D_time_scan.py` env interface:

- local copy: `artifacts/k0_order_scan_retry_20260719/ord_1_reflection_retry1.slurm`
- remote copy: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_order_focus_pm300_10T_K0_reflection_retry1_20260719_1925/ord_1_reflection_retry1.slurm`
- working_dir: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/K=+0.000,ND_a0=0.15,L=0.00`
- output_dir: original row1 output dir `.../order_focus_pm300_10T_K0_reflection`
- scan: `side=reflection`, center `217T0`, half-width `300T0`, `61` points
- resources: `900G/12h`, `1 node`, `1 task`, `6 cpus`

Before submitting the retry I ran `checkquota`; MIKHAILOVA scratch was `9.5/15 TiB`, free ~`5.5 TiB`, above the 2 TiB guard. Retry job submitted as `3548700` and was immediately `PENDING (Priority)`. At that time `3512585` remained `PENDING (Resources)`, `3512586–3512587` remained `PENDING (Priority)`.

If `3548700` fails with the same JAX/XLA rendezvous abort, stop and report to Zhiping before further retries; likely next steps would involve changing JAX CPU device/threading settings or chunking the time scan rather than resubmitting unchanged.
