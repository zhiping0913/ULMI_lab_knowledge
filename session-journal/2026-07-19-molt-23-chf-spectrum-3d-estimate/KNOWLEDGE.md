---
name: 2026-07-19-molt-23-chf-spectrum-3d-estimate
description: Session journal for molt 23→24 covering Curved_surface coherent-harmonic-focusing interpretation, spectrum_1D.py harmonic gain/phase diagnostics, conservative estimated 3D Emax(K), and K=0 order-scan monitoring.
type: session-journal
session_journal: true
created: 2026-07-19T11:40:00-04:00
---

# Session journal — 2026-07-19 molt 23→24: CHF spectrum diagnostics and 3D estimate

## Why molt now

After a deliberate post-task context rebuild, the session cache-miss budget was nearly exhausted (`cache_miss_remaining_tokens` about 62k). The main human-facing task series was complete, reported, and deposited in knowledge/pad. Molt now to reset the cache-miss budget before the next scheduled self-reminder for the long-pending K=0 order-scan jobs.

## Post-molt recovery and scheduled K=0 scan checks

This segment began after a prior molt. Recovery found no new internal email and no new Telegram messages beyond the already-completed focus-at-y delivery.

The K=0 widened reflection order-scan jobs remained the only background monitoring task:

- Jobs: `3512584–3512587` on `tiger-vis`.
- Logdir: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_order_focus_pm300_10T_K0_reflection_20260714_1944`.
- Subset: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_k0_reflection_subset_20260714.tsv`.
- 2026-07-18 21:19 EDT check: all four still `PENDING (Priority)`, elapsed 0, `ReqMem=900G`, Start N/A.
- 2026-07-19 03:20 EDT check: all four still `PENDING`; job 3512584 had reason `Resources`, the other three `Priority`.
- 2026-07-19 09:20 EDT check: all four still `PENDING (Priority)`, elapsed 0, `ReqMem=900G`, Start N/A.
- Next self-reminder scheduled for about 2026-07-19 15:20 EDT.

No Telegram report was sent for the pending-only checks, per prior reminder rule. Pad was updated with the latest 09:20 status and next reminder.

## Zhiping question: why raw focus field can be attosecond-scale without explicit filtering

Zhiping asked on Telegram (`main:6420513923:434`) whether literature has discussed the surprising point that coherent harmonic focusing can produce an attosecond-scale pulse in the raw local field without explicitly filtering high harmonics, unlike the usual 1D presentation.

I replied in Telegram `main:6420513923:436` after checking the local HHG wiki and knowledge base. Main mechanism:

- In planar/1D HHG, the raw reflected waveform is dominated by the fundamental and low orders, so high-pass/band-pass filtering is usually needed to expose attosecond spikes from phase-locked high harmonics.
- At a curved-surface harmonic focus, propagation supplies a frequency-dependent spatial transfer function. Higher harmonics have shorter wavelengths and can focus tighter; in CHF theory, the Huygens/focal integral effectively weights harmonics, so the focus itself acts as a space-frequency filter.
- Therefore, a local raw `Bz` spike at the harmonic focus can be attosecond-scale without post-processing filter. Careful phrasing: this does not mean the full reflected field everywhere is filter-free attosecond light; the statement applies to the spatially selected harmonic focus/caustic.

Key sources found and cited:

- Gordienko et al. PRL 2005, **“Coherent Focusing of High Harmonics: A New Way Towards the Extreme Intensities,”** DOI `10.1103/physrevlett.94.103903`. Local text explicitly says CHF works as a spectral filter and can shorten pulses to zeptosecond range.
- Vincenti PRL 2019, **“Achieving Extreme Light Intensities using Optically Curved Relativistic Plasma Mirrors,”** DOI `10.1103/physrevlett.123.105001`.
- Quéré & Vincenti HPLSE 2021, DOI `10.1017/hpl.2020.46`.
- Dromey et al. Nature Physics 2009, DOI `10.1038/nphys1158`.
- Hörlein et al. EPJD 2009, DOI `10.1140/epjd/e2009-00084-x`.
- Background: Plaja 1998, Thaury & Quéré 2010, Tsakiris 2006.

Durable note written and indexed:

- `knowledge/curved-surface-chf-project/coherent_focusing_unfiltered_pulse_literature_2026-07-19.md`.

## First-pass local raw-Bz harmonic diagnostic

Zhiping then asked (`main:6420513923:437`) for a diagnostic plotting harmonic gain and phase residual over orders 1–30. I first built an independent local Fourier projection of raw `Bz(x)` around the focus window:

- Default parameters: `a0=20`, `ND/a0=0.30`, `side=reflection`, 1D reference plus 2D `K=+0.000,-0.002,-0.005,-0.008`, window `±4λ`, harmonics `n=1..30`.
- Coefficient definition: local Hann-window projection of `Bz/Bnorm` onto `exp(-i 2π n u)`, with `u=x/λ_norm-x_peak`.
- Artifacts delivered in Telegram `main:6420513923:439-443`.
- Remote/local artifact dirs:
  - `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/a0_20_focus_at_y_local_spectrum_20260719/`
  - `artifacts/a0_20_focus_at_y_local_spectrum_20260719/`
- Result: strong order-dependent spatial gain in 2D focus fields, but the simple wrapped `φ_n-nφ_1` residual looked jagged. I flagged that the result should not be used as proof of full-band phase locking.

This first-pass diagnostic was useful but was superseded by the canonical `spectrum_1D.py` output analysis below after Zhiping corrected the desired definition.

Durable note:

- `knowledge/curved-surface-chf-project/local_harmonic_spectrum_diagnostic_2026-07-19.md`.

## Corrected spectrum_1D.py per-harmonic gain/phase analysis

Zhiping corrected the definition (`main:6420513923:444`): use `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_1D.py`'s own `spectrum.nc` / per-order phase outputs to compute 1D→2D gain and phase.

I inspected `spectrum_1D.py` and outputs:

- Spectrum NetCDF contains `spectrum_intensity(f_over_f0_M)`.
- Per-harmonic TSVs exist:
  - 1D: `reflection_per_harmonic_at_peak.tsv`.
  - 2D focus-at-y: `reflection_focus_at_y_per_harmonic_at_peak.tsv`.
- The script computes the forward field `(Ey+cBz)/2`, normalizes spectrum intensity by `laser_spectrum_peak_M^2`, filters HH band, finds the HH-envelope peak, and writes per-order `envelope_at_peak_over_EcM`, `envelope_at_peak_over_a0EcM`, `phase_at_peak_rad`, `phase_at_peak_over_pi`, etc.

I wrote and ran a corrected script/notebook:

- Local/remote dirs:
  - `artifacts/a0_20_focus_at_y_spectrum1d_gain_3d_20260719/`
  - `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/a0_20_focus_at_y_spectrum1d_gain_3d_20260719/`
- Delivered Telegram `main:6420513923:446-452` with main plot, phase-at-peak auxiliary plot, TSVs, and notebook.

Main numerical results for `ND/a0=0.30`, `side=reflection`:

- `G_2D,E(n)=A_2D(n)/A_1D(n)` using `envelope_at_peak_over_a0EcM`.
- `K=-0.005`: sample gains `n1=2.055`, `n5=4.190`, `n10=6.242`, `n20=2.010`, `n30=2.287`; band means `1–10=4.41`, `11–20=5.24`, `21–30=2.35`; max near `n=15` of `11.13`.
- `K=-0.008`: sample gains `n1=1.832`, `n5=4.160`, `n10=6.318`, `n20=1.284`, `n30=1.049`; band means `1–10=4.40`, `11–20=4.86`, `21–30=2.55`; max near `n=15` of `10.07`.
- `K=-0.002` intermediate; `K=0` weakest.

Phase conclusion:

- The residual `wrap(φ_n-nφ_1)` over all `n=1..30` remains busy.
- Direct `phase_at_peak` is more meaningful for this question.
- For `n=5..15`, 2D focus phases cluster tightly:
  - 1D circular std `0.146π`.
  - K=+0.000 std `0.021π`.
  - K=-0.002 std `0.023π`.
  - K=-0.005 std `0.022π`.
  - K=-0.008 std `0.026π`.
- For `n≈20–30`, phases wrap/scatter, likely influenced by weaker amplitudes and/or finite resolution. Do not claim the entire `1–30` band is equally phase locked.

Durable note:

- `knowledge/curved-surface-chf-project/spectrum1d_gain_phase_3d_estimate_2026-07-19.md`.

## Estimated 3D Emax(K) plot

Zhiping agreed that `n=1–20` are locked and `>20` may be limited by `250 cpl` field-storage/postprocessing resolution. He asked (`main:6420513923:453`) for an estimated 3D focus `E_max(K)` plot based on the existing 2D notebook `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/plot_Emax_vs_K_reflection_ND_scan_a0_20.ipynb`, accepting a conservative realistic 2D→3D estimate.

I inspected the existing notebook:

- It reads `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/focus.tsv`.
- Case column: `working_dir (/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D)`.
- Parses K/ND_a0/L from case paths.
- Uses `E_max(Ec)`, `x_at_E_max (lambda0)`, and `propagation_time (laser_period)`.
- Plots 2D `Emax_vs_K_reflection_ND_scan_a0_20` and `x_at_Emax_vs_K_reflection_ND_scan_a0_20`.

I wrote and ran `plot_estimated_3d_Emax_vs_K_reflection_a0_20.py`:

- Remote/local dirs:
  - `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/a0_20_estimated_3d_Emax_vs_K_20260719/`
  - `artifacts/a0_20_estimated_3d_Emax_vs_K_20260719/`
- Delivered Telegram `main:6420513923:455-459` with PNG/PDF/TSV/notebook.

Model:

- Locked band: `n=1..20`.
- `G_locked` = amplitude-weighted mean of spectrum_1D.py per-order 2D/1D field gain over locked band, weights = 2D focal per-order amplitude.
- Ideal upper missing-dimension estimate: `E3D_ideal = E2D * G_locked`.
- Conservative estimate: `E3D_cons = E2D * sqrt(G_locked)`.
- The conservative estimate is a midpoint between no extra focusing and perfect separable/symmetric 3D focusing, equivalent to an effective Strehl/aberration/finite-aperture/coherence penalty `S_eff=1/sqrt(G_locked)` relative to ideal.
- Not a 3D PIC result.

Key results:

- `ND/a0=0.30`:
  - `K=-0.005`: `E2D=156.9 Ec`, `G_locked=4.431`, `sqrt=2.105`, `E3D_cons≈330.3 Ec`, `ideal≈695.4 Ec`.
  - `K=-0.008`: `E2D=153.5 Ec`, `G_locked=4.368`, `E3D_cons≈320.9 Ec`, `ideal≈670.7 Ec`.
  - `K=-0.002`: `E3D_cons≈199.9 Ec`.
  - `K=0`: `E3D_cons≈99.8 Ec`.
- `ND/a0=0.50`: best conservative K=-0.008, `E3D_cons≈240.3 Ec`.
- `ND/a0=1.00`: best conservative K=-0.008, `E3D_cons≈156.2 Ec`.

Durable note:

- `knowledge/curved-surface-chf-project/estimated_3d_Emax_vs_K_2026-07-19.md`.

## Pad and knowledge updates

Updated pad with a compact completed section for:

- CHF literature interpretation.
- First-pass local harmonic diagnostic.
- Corrected spectrum_1D.py gain/phase analysis.
- Estimated 3D Emax(K) plot.

Updated Curved_surface project parent knowledge index to point to:

- `coherent_focusing_unfiltered_pulse_literature_2026-07-19.md`
- `local_harmonic_spectrum_diagnostic_2026-07-19.md`
- `spectrum1d_gain_phase_3d_estimate_2026-07-19.md`
- `estimated_3d_Emax_vs_K_2026-07-19.md`

## Current state after molt

No human-facing reply is pending as of 2026-07-19 11:40 EDT. The latest handled Telegram message was `main:6420513923:453`, replied with artifacts through `459`.

Only ongoing known background task is the K=0 widened reflection order-scan monitoring. Next self-reminder should arrive around 15:20 EDT. At that time:

1. SSH `tiger-vis` with short timeout, no password/Duo retries.
2. Run concise `squeue/sacct` for jobs `3512584–3512587`.
3. If still pending, no Telegram report unless asked; schedule next self-reminder.
4. If running, inspect `.err` for OOM/tracebacks versus known warnings.
5. If completed, validate per-band NetCDF outputs, extract/summarize per-band argmax for the four rows, report to Zhiping, and update knowledge/pad.

Knowledge backup git push was not performed after the 2026-07-19 notes due to impending cache-miss-budget molt. If future-you has slack and the network’s normal practice still applies, consider committing/pushing the new knowledge notes and pad updates to the local knowledge backup repo.
