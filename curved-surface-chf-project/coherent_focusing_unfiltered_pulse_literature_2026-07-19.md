# Coherent focusing and unfiltered attosecond pulse mechanism — literature note

Date: 2026-07-19
Telegram context: Zhiping asked in `main:6420513923:434` why the aligned raw `Bz` plot suggests harmonic focusing is coherent and can yield an attosecond-scale pulse without explicit spectral filtering, unlike 1D where filtering is usually needed to expose the attosecond waveform. Answer sent in `main:6420513923:436`.

## Mechanism interpretation

The raw reflected field may be represented schematically as

```text
E(t) = Σ_n A_n cos(n ω0 t + φ_n).
```

In 1D/planar diagnostics, the fundamental and low harmonics generally dominate the raw waveform, so the attosecond spike from phase-locked high harmonics is usually exposed by high-pass or band-pass filtering.

In curved-surface coherent harmonic focusing (CHF), propagation to the harmonic focus supplies a frequency-dependent spatial transfer function `G_n(x_focus)`. Because `λ_n = λ0/n`, a diffraction-limited harmonic focus has a smaller area for larger `n`; in the spherical/concave CHF theory, the Huygens/focal integral also gives extra frequency weighting. Thus the observation point at the harmonic focus is not a neutral sample of the reflected field: it preferentially weights phase-locked high harmonics and can suppress or defocus the fundamental/low orders relative to them. The focusing itself acts as an effective space-frequency filter.

Therefore a local raw `Bz` spike at the harmonic focus can be attosecond-scale without explicit post-processing filter. This should be phrased carefully: it does not mean the full reflected field everywhere is filter-free attosecond light; it means the harmonic focus is a spatially selected, frequency-weighted, phase-coherent caustic.

## Local literature evidence

Direct CHF / CRM references in the local knowledge base and HHG wiki:

1. Gordienko, Pukhov, Shorokhov & Baeva, **“Coherent Focusing of High Harmonics: A New Way Towards the Extreme Intensities,”** Phys. Rev. Lett. 94, 103903 (2005), DOI `10.1103/physrevlett.94.103903`.
   - Local path: `/home/zhiping/knowledge_base/paper/2005/2005--Coherent Focusing of High Harmonics_ A New Way Towards the Extreme Intensities/`.
   - Abstract: focal intensity boost relies on harmonic coherency and a slow power-law harmonic spectrum.
   - Key retained phrase from local full text: CHF works as a spectral filter and shortens the pulse duration down to the zeptosecond range.
   - This is the most direct support for “focusing itself provides spectral filtering/compression.”

2. Vincenti, **“Achieving Extreme Light Intensities using Optically Curved Relativistic Plasma Mirrors,”** Phys. Rev. Lett. 123, 105001 (2019), DOI `10.1103/physrevlett.123.105001`.
   - Local path: `/home/zhiping/knowledge_base/paper/2019/2019--Achieving Extreme Light Intensities using Optically Curved Relativistic Plasma Mirrors/`.
   - Mechanism: a curved relativistic mirror temporally compresses / Doppler upshifts the reflected field and focuses the Doppler-upshifted light to a smaller spot than possible at the incident wavelength.
   - The article still describes the attosecond pulse train “after filtering the incident laser frequency,” so it is best used as CRM mechanism support, not as a claim that no filtering is ever needed.

3. Quéré & Vincenti, **“Reflecting petawatt lasers off relativistic plasma mirrors: a realistic path to the Schwinger limit,”** High Power Laser Science and Engineering 9, e6 (2021), DOI `10.1017/hpl.2020.46`.
   - Local path: `/home/zhiping/knowledge_base/paper/2021/2021--Reflecting petawatt lasers off relativistic plasma mirrors a realistic path to the Schwinger limit/`.
   - Review statement: reflecting petawatt beams from relativistic mirrors compresses the pulse in time to the attosecond range and converts it to shorter wavelengths, which can be focused much more tightly than the initial laser light.

4. Dromey et al., **“Diffraction-limited performance and focusing of high harmonics from relativistic plasmas,”** Nature Physics 5, 146–152 (2009), DOI `10.1038/nphys1158`.
   - Local path: `/home/zhiping/knowledge_base/paper/2009/2009--Diffraction-limited performance and focusing of high harmonics from relativistic plasmas/`.
   - Supports the coherence/focusability premise: surface smoothing and plasma denting can produce high-quality X-ray/high-harmonic beams focused near the diffraction limit, opening a route to coherent harmonic focus.

5. Hörlein et al., **“Controlling the divergence of high harmonics from solid targets: a route toward coherent harmonic focusing,”** Eur. Phys. J. D 55, 173–181 (2009), DOI `10.1140/epjd/e2009-00084-x`.
   - Local path: `/home/zhiping/knowledge_base/paper/2009/2009--Controlling the divergence of high harmonics from solid targets a route toward coherent harmonic focusing/`.
   - Abstract/intro explicitly connect exceptional coherence, attosecond phase locking, wavefront control, and CHF using pre-shaped targets.

6. Background/contrast:
   - Plaja et al., **“Generation of attosecond pulse trains during the reflection of a very intense laser on a solid surface,”** JOSA B 15, 1904 (1998), DOI `10.1364/josab.15.001904`: phase-locked harmonics can form attosecond pulse trains; historically discusses filtering `N` plateau harmonics to obtain ultrashort pulses.
   - Thaury & Quéré, **“High-order harmonic and attosecond pulse generation on plasma mirrors: basic mechanisms,”** J. Phys. B 43, 213001 (2010), DOI `10.1088/0953-4075/43/21/213001`: tutorial on plasma-mirror temporal distortion, harmonic phase properties, and attosecond pulse trains.
   - Tsakiris et al., **“Route to intense single attosecond pulses,”** New J. Phys. 8, 19 (2006), DOI `10.1088/1367-2630/8/1/019`: planar-target route to single attosecond pulses in selected spectral ranges; useful contrast with spatial/CHF filtering.

## Suggested next diagnostic

To turn the qualitative explanation into evidence for the Curved_surface paper:

1. For 1D raw, 1D filtered, and 2D `focus_at_y` raw fields, compute local spectra in a small window around the HH-envelope peak / focus.
2. Plot frequency-resolved focusing gain, e.g. `G_n(K)=|B_n(x_focus)|/|B_n(reference)|`, relative to a pre-focus or flat/1D reference.
3. Plot phase residuals `φ_n - (n φ_1 + φ0)` or a linear-fit residual versus harmonic order. Near-flat residuals over the contributing band would demonstrate phase locking at the focus.
4. Reconstruct the time-domain field cumulatively with low-order cutoff varied. If the raw 2D focus field remains attosecond-scale because high-order spatial gain dominates, while 1D raw does not, that directly supports “spatial focusing acts as effective high-pass + coherent phase locking.”
