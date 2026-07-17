# CEP harmonic phase reference and ROM/CSE interpretation — 2026-07-17

## Trigger

Zhiping replied on Telegram `main:6420513923:378` to the previous CEP phase-locking report. He pointed out that interpreting the measured common phase near `-π/2` literally is troubling: at the analytic envelope / energy-flux peak a real field phase of `-π/2` means the real field crosses zero, whereas a synchrotron-radiation / CSE picture of electron acceleration would more naturally suggest the harmonic phases align near a real-field maximum (`cos 0 = 1`). He asked to check ROM/CSE knowledge and literature for whether harmonic locking is discussed as locking to a specific absolute phase.

Response delivered on Telegram `main:6420513923:380`.

## Sources checked

- Local wiki skill `wiki-laser-plasma-hhg`:
  - `concepts/rom-theory.md`
  - `concepts/cse-theory.md`
  - `concepts/unified-hhg-picture.md`
  - `comparisons/rom-vs-cse.md`
- Knowledge-base papers / thesis chunks:
  - Baeva, Gordienko, Pukhov, *Theory of high-order harmonic generation in relativistic laser interaction with overdense plasma*, Phys. Rev. E 74, 046404 (2006), DOI `10.1103/PhysRevE.74.046404`.
  - Pirozhkov et al., *Attosecond pulse generation in the relativistic regime of the laser-foil interaction: The sliding mirror model*, Phys. Plasmas 13, 013107 (2006), DOI `10.1063/1.2158145`.
  - Mikhailova et al., *Isolated Attosecond Pulses from Laser-Driven Synchrotron Radiation*, Phys. Rev. Lett. 109, 245005 (2012), DOI `10.1103/PhysRevLett.109.245005`.
  - Cousens et al., *Electron trajectories associated with laser-driven coherent synchrotron emission at the front surface of overdense plasmas*, Phys. Rev. E 101, 053210 (2020), DOI `10.1103/PhysRevE.101.053210`.
- LightRAG dense search:
  - Query: `relativistic plasma mirror harmonic phase locking spectral phase CSE ROM attosecond`.
  - Query: `coherent synchrotron emission harmonic spectral phase absolute phase zero pi over two`.
  - It found useful phase/spectral-property leads but no evidence that ROM/CSE literature asserts a universal absolute locking phase of `-π/2`.
- Script inspection:
  - Tiger `spectrum_1D.py`: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_1D.py`.
  - Adroit `spectrum_1D.py`: `/scratch/network/zl8336/try_PIC_GPU_JAX/utility/spectrum_1D.py`.
  - EM_analyzer helper: `/scratch/gpfs/MIKHAILOVA/zl8336/EM_analyzer/spectrum.py`.

## Main result

The previous statement “phases lock near `-π/2`” should be treated as a statement about **the current analytic-signal diagnostic at the chosen HH envelope peak**, not as a mechanism-level conclusion that ROM/CSE harmonics physically lock to `-π/2`.

The literature supports this more careful statement:

1. “Phase locking” in ROM/CSE papers primarily means **linear spectral phase / common emission time**.
   - Pirozhkov 2006 explicitly defines perfect phase locking as spectral phase linear in frequency; in time domain, different frequencies are emitted simultaneously.
   - BGP 2006 gives, for a single γ-spike, approximately `φ_n(t_g) ≈ - n Θ_1(t_g)` for high `n`; this is a group-delay/emission-time phase, not a universal absolute intercept.
   - Multiple γ-spikes can interfere and modulate spectra through relative spike phases.

2. A physical CSE/synchrotron interpretation does **not** naturally privilege `-π/2` as an absolute field phase.
   - Mikhailova 2012 describes emission at `|V_z|` maximum, `V_y≈0`, and `a_y` maximum. Its far-field expression is acceleration-like: `E(t) ∝ -a_y(t') 4γ^4/(...)`. If the emitted radiation field and emission time are used as the reference, one expects the real field extremum to align with the burst, i.e. a cosine-like phase `0` or `π` up to sign.
   - Cousens 2020 shows front-surface CSE occurs when `v_y≈0` and transverse acceleration is large; in their 1D sheet-current formulation the transverse field can be written in terms of `v_y/(1-s v_x/c)`. Thus a current/velocity-like source variable crosses zero at the same physical emission point where acceleration and high-frequency generation are strongest. This can create a quadrature relation depending on whether the diagnostic is field-like, current-like, acceleration-like, or Hilbert-envelope based.

3. The current `spectrum_1D.py` diagnostic samples a convention-dependent analytic phase.
   - The script filters each band `[n-0.5,n+0.5] k0`, calls `get_envelope_from_spectrum_with_coordinate`, and samples at the broadband HH envelope `peak_id`.
   - `EM_analyzer.spectrum.get_envelope_from_spectrum_with_coordinate` constructs an analytic spectrum with `h = 1 + sign(freq)` and returns `abs(field_analytic)` and `angle(field_analytic)`.
   - With the EM_analyzer Fourier convention, a real `cos(kx)` has analytic phase `0` at its maximum, while a real `sin(kx)` at its zero crossing has phase `-π/2`.
   - Therefore the measured `-π/2` literally says that, at the selected HH envelope peak, the narrowband real parts are approximately in quadrature / near zero crossing. It does not by itself identify the absolute physical radiation phase.

## Method reflection

- The important invariant for harmonic locking is the **phase slope and curvature** after fitting/removing the linear group-delay term, not the raw phase intercept at an arbitrarily chosen sample point.
- The right model-to-data comparison should anchor the phase reference physically:
  - either to the real HH field extremum,
  - or to the electron-bunch acceleration maximum,
  - or to a fitted emission time `x_emit` / group delay from `φ_n = φ0 - n k0 x_emit`.
- Once that reference is fixed, the remaining intercept `φ0` can be interpreted. Without fixing it, `φ0≈-π/2` is a diagnostic-convention statement.

## Surprise / nuance

CSE literature gives two nearby but not identical intuitions:

- Mikhailova 2012’s far-field, acceleration-based formula suggests the radiation field itself should peak with `a_y`, hence real-field phase near `0/π` if referenced to the burst.
- Cousens 2020’s sheet-current expression and trajectory analysis emphasize that emission occurs near `v_y≈0`, i.e. a transverse-current zero crossing. A bandpass/analytic-envelope diagnostic of a current-like variable can therefore show the envelope peak at a real zero crossing (`-π/2`) without contradicting the acceleration-based radiation picture.

This is probably why Zhiping’s physical objection is valuable: it exposes that our previous diagnostic mixed “envelope peak” and “real field phase” without explicitly choosing the reference.

## Unclear points / next diagnostics

The next post-processing should not patch physics; it should add diagnostics:

1. Fit strong-harmonic phases to
   `φ_n = φ0 - n k0 x_emit`
   and report:
   - fitted `x_emit` / group delay,
   - residual phase scatter after linear removal,
   - intercept `φ0` with a stated convention.
2. Sample per-harmonic phases at both:
   - the broadband HH envelope peak (`peak_id`, current behavior), and
   - the nearest real `E_HH` extremum or the maximum of signed/absolute real HH field near that envelope peak.
3. Report real/quadrature components at the envelope peak:
   - `Re(E_analytic) = E_HH_real`,
   - `Im(E_analytic) = Hilbert quadrature`,
   - `phase`, `cos(phase)`, `sin(phase)`.
4. If electron diagnostics are available, compare the HH burst timing with `J_y`, `dJ_y/dt`, and/or bunch `a_y` to determine whether the observed quadrature is current-like or radiation-field-like.

## Implication for ULMI/JAX-vs-EPOCH phase conclusion

Revise the old conclusion as follows:

- Old shorthand: “EPOCH phases lock near `-π/2`; JAX has similar tendency but larger scatter.”
- More precise: “Under the current analytic-signal diagnostic sampled at the broadband HH envelope peak, EPOCH strong harmonics share a common quadrature phase near `-π/2`, while JAX shows the same diagnostic tendency with larger scatter. This is evidence of coherent emission / low residual phase scatter, but the absolute intercept should not be interpreted until the group-delay and real-field/acceleration reference are fixed.”

This correction should be used in future reports or plots discussing the CEP phase-locking result.
