# EPOCH CEP reflection phase fit at nearest real HH-field peak — 2026-07-17

## Trigger

Zhiping Telegram `main:6420513923:381` asked to modify tiger-vis `spectrum_1D.py` to find the `|E_forward_hh_real|` peak closest to the broadband HH envelope `peak_id`, then compute per-harmonic phases at that real-field peak, unwrap phases, and linearly fit strong harmonics for the same eight EPOCH CEP cases under:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/study_phy_cep/a0=20,ND_a0=0.3,45
```

Acknowledgement: `main:6420513923:382`. Report/artifacts: `main:6420513923:383-386`.

## Script change

Patched:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_1D.py
```

Backup:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_1D.py.bak_20260717_1300_real_peak_phase
```

New behavior:

1. After computing the broadband HH analytic signal and its envelope `peak_id`, compute `abs_hh_real = abs(E_forward_hh_real)`.
2. Find local maxima of `abs_hh_real` and choose the one with minimum index distance from the envelope `peak_id`; if tied, choose the larger `|E|`; if no local maxima are found, fall back to global `argmax(abs_hh_real)`.
3. During the existing per-harmonic loop, record both envelope-peak and real-field-peak values:
   - `envelope_at_peak_per_n`, `phase_at_peak_per_n` (old behavior)
   - `envelope_at_real_peak_per_n`, `phase_at_real_peak_per_n` (new)
4. Unwrap `phase_at_real_peak_per_n` over harmonic order.
5. Fit strong harmonics using default threshold:

```text
PHASE_FIT_MIN_REL = 1e-3
fit mask: n >= ceil(HARMONIC_MIN), n <= n_max, envelope_at_real_peak/(a0 Ec_M) >= threshold
fit model: phase_unwrapped(n) = slope*n + intercept
```

New per-case outputs:

```text
reflection_per_harmonic_at_real_peak.tsv
reflection_real_peak_phase_fit_summary.tsv
reflection_per_harmonic_at_real_peak.png
```

## Run

Async bash job `job-7a816527` completed successfully (exit 0), running all eight reflection cases.

Remote logdir:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/phase_real_peak_logs/epoch_cep_20260717_1302_real_peak_phase
```

Combined remote summary artifacts:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/epoch_cep_real_peak_phase_20260717/epoch_cep_reflection_real_peak_phase_fit_summary.tsv
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/epoch_cep_real_peak_phase_20260717/epoch_cep_reflection_real_peak_phase_fit_summary.png
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/epoch_cep_real_peak_phase_20260717/epoch_cep_reflection_real_peak_phase_fit_summary.pdf
```

Local mirror:

```text
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/epoch_cep_real_peak_phase_20260717/epoch_cep_reflection_real_peak_phase_fit_summary.tsv
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/epoch_cep_real_peak_phase_20260717/epoch_cep_reflection_real_peak_phase_fit_summary.png
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/epoch_cep_real_peak_phase_20260717/epoch_cep_reflection_real_peak_phase_fit_summary.pdf
```

## Numerical summary

All fits used `fit_count=99` strong harmonics.

| CEP/pi | real peak offset `T0_M` | `|E_real|/Ec_M` at real peak | intercept/pi | slope/pi/order | weighted residual RMS/pi |
|---:|---:|---:|---:|---:|---:|
| -0.75 | 0.00424264 | 31.1389 | -0.3213 | 0.00518 | 0.0821 |
| -0.50 | 0.00424264 | 33.8164 | -0.3422 | 0.00595 | 0.0909 |
| -0.25 | 0.00459619 | 33.6384 | -0.3547 | 0.00759 | 0.0482 |
| 0.00 | 0.00424264 | 32.3517 | -0.3726 | 0.00819 | 0.0468 |
| +0.25 | 0.00459619 | 31.5049 | -0.3847 | 0.00785 | 0.0498 |
| +0.50 | 0.00530330 | 21.7837 | -0.4599 | 0.01153 | 0.0448 |
| +0.75 | 0.00707107 | 17.9866 | -0.4543 | 0.01589 | 0.0880 |
| +1.00 | 0.00424264 | 25.5711 | -0.3248 | 0.00501 | 0.0914 |

The real-field peak is always close to the envelope peak: `12–20` grid cells later, `0.0042–0.0071 T0_M` (lab-frame `~0.003–0.005 T0` for θ=45°).

## Interpretation

Zhiping corrected the interpretation in Telegram `main:6420513923:387`; reply/acknowledgement in `main:6420513923:388`.

The better interpretation is:

- The real-field-peak run is a **sanity check for sampling-point translation**, not a replacement for envelope-peak sampling.
- Mathematically, if `E_n(x) ~ A_n exp[i(n k0 x + φ_n)]`, moving the sample point by `Δx` adds `Δφ_n = n k0 Δx`; therefore a linear phase-vs-order slope is exactly the signature of a shifted sampling point.
- The fitted slopes are indeed of the same size as the measured real-peak offset from the envelope peak. Example: CEP=0 has `Δx/λ_M = 0.00424`, while `slope/(2π)=0.00409`. Thus removing the real-peak slope is essentially equivalent to shifting the phase reference back toward the envelope/group-delay center.
- Therefore, **for diagnosing harmonic phase locking / common emission time, the broadband HH envelope peak is the more natural reference point**, precisely because the original envelope-peak phases had little slope.
- The remaining common `-π/2` at the envelope peak should not be treated as a failure of the reference point. It is the carrier/analytic phase of the attosecond HH wavepacket at its envelope center: a short pulse `A(t) cos(ωt+φ_CEP)` can have maximum envelope/energy while the real carrier is near zero if `φ_CEP≈-π/2`.
- The robust diagnostic is therefore: small slope and small residual scatter at the envelope peak. Real-peak sampling shows that moving to a carrier extremum introduces the expected linear slope.

This updates the earlier wording in this note: real-peak sampling plus linear fitting is **not** the better primary diagnostic; it is a useful check that the slope behaves like a sampling-time/space shift. The primary locking diagnostic should remain the envelope-peak/group-delay-center phase table, with the common intercept interpreted as the HH wavepacket carrier phase.

## Script revert

After the interpretation correction, Zhiping asked in Telegram `main:6420513923:389` to revert `spectrum_1D.py` to the backup that only samples phases at the envelope peak. Completed and reported in `main:6420513923:390`.

Current main script restored from:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_1D.py.bak_20260717_1300_real_peak_phase
```

to:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_1D.py
```

The reverted script passed `python3 -m py_compile`. The temporary real-peak diagnostic version was preserved as:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_1D.py.bak_20260717_1316_before_revert_real_peak_phase
```

Real-peak output files and combined plots were not deleted; they remain as sanity-check artifacts, but the production script is back to envelope-peak sampling only.

## Follow-up ideas

If Zhiping wants a stricter physical phase reference, prefer a separate diagnostic script/notebook rather than changing the production `spectrum_1D.py`: explicitly compare envelope-center phase, synthesized HH carrier phase, and source diagnostics (`J_y`, `dJ_y/dt`, bunch `a_y`) when particle data are available.
