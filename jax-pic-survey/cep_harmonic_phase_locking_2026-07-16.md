# CEP harmonic phase locking at reflection HH envelope peak — EPOCH vs JAX-in-Cell

Date: 2026-07-16 EDT

Telegram context: Zhiping asked to use the updated `spectrum_1D.py` phase-extraction block to get each harmonic order's phase at the reflection HH envelope peak for the CEP scan, test whether phases are approximately locked near `-π/2`, and also compare with adroit-vis JAX-in-Cell results where high-order detuning looked serious.

## Source data

EPOCH CEP scan:
- Remote root: `/scratch/gpfs/MIKHAILOVA/zl8336/study_phy_cep/a0=20,ND_a0=0.3,45`
- Utility script patched (numeric output only; logic otherwise preserved): `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_1D.py`
- Backup: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_1D.py.bak_20260716_230738`
- Output root: `/scratch/gpfs/MIKHAILOVA/zl8336/study_phy_cep/a0=20,ND_a0=0.3,45/utility/cep_scan_plots/per_harmonic_phase_20260716_2307`

JAX-in-Cell CEP scan:
- Remote root: `/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/outputs/phi_cep_scan_cpl1200_20260716_1144`
- Utility script patched (numeric output only; logic otherwise preserved): `/scratch/network/zl8336/try_PIC_GPU_JAX/utility/spectrum_1D.py`
- Backup: `/scratch/network/zl8336/try_PIC_GPU_JAX/utility/spectrum_1D.py.bak_20260716_231016`
- Output root: `/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/outputs/phi_cep_scan_cpl1200_20260716_1144/per_harmonic_phase_20260716_2309`

Local deliverable root:
- `/home/zhiping/ULMI_lab/.lingtai/main/artifacts/cep_phase_locking_20260716`
- Self-contained HTML: `cep_harmonic_phase_locking_report.html`
- Notebook: `cep_harmonic_phase_locking_report.ipynb`
- Comparison figure: `epoch_jax_phase_locking_comparison.png/.pdf`

## Patch made to `spectrum_1D.py`

Both EPOCH and JAX copies already computed `envelope_at_peak_per_n` and `phase_at_peak_per_n` for harmonic bands `[n-0.5,n+0.5] k0_M`, sampled at the common HH envelope peak `peak_id`, then only plotted `*_per_harmonic_at_peak.png`. I added a numeric TSV output:

`<side>_per_harmonic_at_peak.tsv` columns:
- `harmonic_order`
- `envelope_at_peak_over_EcM`
- `envelope_at_peak_over_a0EcM`
- `phase_at_peak_rad`
- `phase_at_peak_over_pi`
- `phase_minus_neg_half_pi_rad`
- `phase_minus_neg_half_pi_over_pi`

The last two columns wrap the deviation from `-π/2`: `angle(exp(i*(phase + π/2)))`.

## EPOCH result

All 8 EPOCH reflection CEP values were processed: `φ/π = -0.75,-0.5,-0.25,0,0.25,0.5,0.75,1.0`.

At amplitude threshold `envelope_at_peak/(a0 Ec_M) >= 1e-3`:
- weighted circular mean phase ranges from about `-0.399π` to `-0.458π`.
- weighted mean absolute deviation from `-π/2` ranges about `0.071π–0.116π`.
- The closest-locking CEPs were `φ=+0.5π` and `+0.75π` with weighted fractions within `±0.1π` of about `0.77` and `0.69`, respectively.
- At stronger threshold `1e-2`, EPOCH strong-order extent often reaches `n≈50–75` depending on CEP, except `+0.75π` where it reaches `n≈27`.

Interpretation: EPOCH shows real phase locking near a common constant phase, but not an exact `-0.500π` in this convention; the amplitude-weighted center is more like `-0.40π` to `-0.46π`.

## JAX-in-Cell CPL=1200 result

Processed 7 JAX reflection CEP directories found under the scan root: `φ/π = -0.75,-0.5,-0.25,0.25,0.5,0.75,1.0`. The `φ=0` directory was not present in that output root.

At threshold `1e-3`:
- weighted circular mean phase ranges from about `-0.372π` to `-0.474π`.
- weighted mean absolute deviation from `-π/2` ranges about `0.085π–0.188π`, larger than EPOCH for several CEPs.
- Negative CEPs (`-0.75,-0.5,-0.25`) show noticeably weaker locking / larger scatter.
- At stronger threshold `1e-2`, strong-order extent is generally shorter than EPOCH: max strong `n≈28–55` for JAX vs EPOCH often `n≈50–75`.

Interpretation: JAX has the same qualitative phase-locking tendency, but high-order detuning/broadening and weaker high-order coherence make the phase more scattered, consistent with Zhiping's observation.

## Physical mechanism explanation

If reflected HHG is dominated by one attosecond burst / one relativistic-surface-current event, then the spectral phase for order `n` is approximately

`φ_n ≈ -ω_n t_emit + φ0`.

Sampling each narrow-band analytic signal at the common HH envelope peak removes most of the linear propagation/emission-time term, leaving a nearly common phase offset `φ0`. In the current `spectrum_1D.py` FFT/Hilbert convention and at the envelope peak, the band-passed real oscillation is near quadrature, so that common offset appears close to `-π/2`. If the analytic-signal sign convention changed, or if phases were sampled at a real-field peak instead of the envelope peak, the common absolute offset could shift by about `π/2` or change sign. The invariant physical statement is cross-harmonic phase locking, not the absolute `-π/2` value by itself.

JAX high-order detuning/broadening affects this because weak/high-order harmonics have smaller amplitudes and are more sensitive to numerical dispersion, resolution, and model differences; their phase becomes noisy or less coherent first. Therefore phase-locking statistics should always be interpreted with amplitude thresholds.

## Delivered to Zhiping

Telegram report sent as `main:6420513923:365`; attachments sent as:
- HTML report `main:6420513923:366`
- notebook `main:6420513923:367`
- comparison PNG `main:6420513923:368`
