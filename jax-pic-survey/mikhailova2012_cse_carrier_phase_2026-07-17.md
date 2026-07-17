# Mikhailova 2012 CSE model phase vs PIC HH carrier phase — 2026-07-17

## Trigger

Zhiping asked on Telegram `main:6420513923:391` how to explain the apparent discrepancy between:

- the attosecond HH wavepacket carrier phase at the envelope center extracted from our PIC/EPOCH diagnostic, near `-π/2`, and
- the Mikhailova 2012 CSE/synchrotron model (especially supplemental Fig. S5), where the modeled radiated electric field appears to peak at the pulse center, i.e. phase `0`.

Response sent on Telegram `main:6420513923:393`.

## Sources checked

Local knowledge-base entry:

```text
/home/zhiping/knowledge_base/paper/2012/2012--Isolated Attosecond Pulses from Laser-Driven Synchrotron Radiation/
```

Key files:

- `paper.md`
- `supplemental/supplemental--Supplemental_Material_revised.pdf.md`
- Fig. S2 image: `supplemental--Supplemental_Material_revised.pdf._page_2_Figure_0.jpeg`
- Fig. S5 image: `supplemental--Supplemental_Material_revised.pdf._page_7_Figure_8.jpeg`

Relevant supplemental statements:

- Fig. S2(d): reflected electric field (cyan) and intensity `|E(t)|^2` after spectral filtering `ω > 80ω0` (red).
- Fig. S3 text: at emission time `t3`, the attosecond pulse envelope is maximum; `J_z` is maximum in absolute value, and `J_y` is zero.
- Liénard-Wiechert derivation lines 77–139:
  - Radiation field for `n || Z`, `a || Y`:
    `E(t) = - e/(c^2 R) * a_y(t') / (1 - V_z(t')/c)^2`.
  - Coherent bunch model:
    `E(t) = - Ne/(c^2 R) * a_y(t') * U(t')`, with
    `U(t') = 4γ^4/(1 + αγ^2 t'^2)^2`.
  - Fig. S5 plots `t'(t)`, `a_y(t)`, `U(t)`, `E(t)`, and `V_z(t)`.

Vision inspection:

- Fig. S5 red `E(t)` is centered and peaked at the same time as `a_y` and `U`; this is a cosine-like/single-sign waveform at the model pulse center.
- Fig. S2(d), however, shows the PIC reflected electric field (cyan) near a zero crossing at the emission/envelope marker, while the red intensity/envelope marker peaks there.

## Main conclusion

The apparent discrepancy is likely not a contradiction between our PIC and CSE. It is a distinction between:

1. **The simplified CSE analytic model waveform** in Mikhailova Fig. S5, and
2. **The self-consistent PIC reflected-field carrier phase** / high-harmonic comb phase intercept.

The CSE condition fixes the emission time / envelope center / phase slope (group delay), but it does not universally fix the carrier-envelope phase / spectral phase intercept.

## Mechanism-level explanation

### 1. What the CSE model fixes

The Mikhailova CSE conditions identify the burst time:

- `|V_z|` maximal (or `V_z` closest to `c` toward the observer),
- transverse current/velocity `J_y` or `V_y` near zero,
- transverse acceleration `a_y` large,
- Doppler compression factor `U=(1-V_z/c)^-2` maximal.

These determine the time at which the attosecond envelope/intensity is maximum. In spectral language they constrain the linear phase slope/group delay.

### 2. Why Fig. S5 gives phase 0

Fig. S5 uses a simplified single-bunch/single-electron Liénard-Wiechert model:

```text
E(t) = -Ne/(c^2 R) * a_y(t') * U(t')
```

with `a_y(t')` approximated as a Gaussian centered at `t'=0`, and `U(t')` also positive/even and sharply centered. This construction builds in a single-sign, cosine-like pulse centered at zero. Thus the plotted phase 0 is largely a consequence of the simplified source model and chosen time/sign convention.

### 3. Why PIC can show `-π/2` at the envelope center

The actual PIC reflected field is produced by a self-consistent plasma boundary/current system, not just by the simplified `a_y U` waveform.

Important details:

- The same supplement says at the emission time `t3`, `J_y=0` while `J_z` is maximal. Thus transverse current-like variables naturally cross zero at the envelope center.
- Fig. S2(d) appears to show the PIC reflected electric field crossing near zero at the envelope/intensity marker, not peaking there.
- Spectral filtering and analytic-signal extraction separate envelope timing from carrier phase. A short wavepacket `A(t) cos(ωt+φ_CEP)` can have envelope/intensity maximum at `A(t)` peak while the real carrier is zero if `φ_CEP≈-π/2`.
- Plasma boundary response, finite bunch width, coherent summation over electrons, propagation phase, and reflected-field definitions can all change the spectral phase intercept without changing the group delay/envelope center.

Therefore, our EPOCH/PIC diagnostic finding a common phase near `-π/2` at the HH envelope center is best interpreted as: the attosecond HH wavepacket is sine-like at the envelope center. It does not invalidate the CSE emission condition if phase-vs-order slope and residual scatter show a coherent common emission time.

## Implication for ULMI phase diagnostic

Use this wording going forward:

- Envelope peak / group-delay center is the correct reference for phase-locking diagnosis.
- The robust evidence for locking is small phase slope and small residual scatter at the envelope peak.
- The common intercept near `-π/2` is the carrier-envelope phase of the HH wavepacket in the PIC/reflected-field diagnostic.
- Mikhailova Fig. S5’s phase 0 is a simplified analytic-model intercept; Mikhailova’s own PIC Fig. S2(d) may actually be consistent with an electric-field zero crossing at the intensity/envelope peak.

## Suggested strict test

To isolate the issue, run the exact same phase-extraction diagnostic on two synthetic/empirical signals:

1. Synthetic Mikhailova model signal `E_model(t) = -a_y(t') U(t')`, with `a_y` Gaussian and `U` as in the supplement. Expected: phase intercept near 0 at the model envelope center.
2. PIC reflected field signal filtered in the same harmonic band (Mikhailova data if available, or our EPOCH field). Expected: possible common intercept near `-π/2` if the PIC HH wavepacket is sine-like.

The difference would quantify “analytic model acceleration-field intercept” vs “self-consistent PIC reflected-field carrier phase.”
