---
name: jax-pic-survey
description: First-pass ULMI Lab survey of JAX-based / differentiable particle-in-cell codes for fast 1D plasma scans, optimization, and classification.
---

# JAX-based PIC survey

Created after Zhiping's Telegram request `main:6420513923:153` while Curved_surface fine time-scan jobs were running. Initial answer delivered in Telegram `main:6420513923:155`.

## Question

Can ULMI Lab use JAX-based particle-in-cell (PIC) simulation for fast 1D parameter scans, optimization, and classification? Are there existing JAX/differentiable PIC codes worth trying before writing a custom prototype?

## Search workflow and evidence

Local KB / LightRAG:
- Loaded `knowledge-base-user` and `knowledge-base-lightrag`.
- Metadata search: `/home/zhiping/knowledge_base/utility/search.py --title JAX` returned no direct metadata hits; `--title particle-in-cell` returned many traditional PIC texts/papers.
- LightRAG dense/CLI search over `/home/zhiping/knowledge_base_lightrag` found APS-style leads:
  - APS DPP 2022 `Discovering Physics and Improving Simulations using Neural Networks and Differentiable Kinetic Simulations`: differentiable kinetic simulations / exact gradients for Vlasov-Fokker-Planck and improved PIC modeling.
  - APS DPP 2025 `PyPIC3D: A Just-in-time Compilable Particle-in-cell Code Written In Python`.
  - APS DPP 2024 `TORAX: A Fast and Differentiable Tokamak Transport Simulator in JAX` (not PIC, but evidence that JAX differentiable plasma simulation is a live direction).
  - APS DPP 2023 `Discovering novel nonlinear plasma physics using machine learning and differentiable programming`.
- Practical note: KB/LightRAG live on the local Debian machine, not on `tiger-vis`; use `bash -lc 'cd /home/zhiping/knowledge_base_lightrag && source rag-env.sh && ...'` for `source` and `$KBPY`.

External search / primary-source checks:
- `uwplasma/JAX-in-Cell` GitHub and docs: <https://github.com/uwplasma/JAX-in-Cell>, <https://jax-in-cell.readthedocs.io>
- JAX-in-Cell arXiv: `2512.12160`, *JAX-in-Cell: A Differentiable Particle-in-Cell Code for Plasma Physics Applications*, authors Longyu Ma, Rogerio Jorge, Hongke Lu, Aaron Tran, Christopher Woolford.
- `uwplasma/PyPIC3D` GitHub/docs and APS DPP 2025 abstract: <https://github.com/uwplasma/PyPIC3D>, <https://pypic3d.readthedocs.io>
- Smaller educational repo `SeanLim2101/PiC-Code-Jax`: <https://github.com/SeanLim2101/PiC-Code-Jax>
- APS DPP 2022 differentiable kinetic simulations abstract: <https://meetings-archive.aps.org/dpp/2022/np11/68/>

## Candidate codes

### 1. JAX-in-Cell — strongest first candidate

Evidence:
- GitHub description: `1D3V Particle in Cell Code in JAX`.
- Package/CLI: `jaxincell`, installable with `pip install jaxincell`; docs at `jax-in-cell.readthedocs.io`.
- GitHub metadata at survey time: MIT license, Python, updated 2026-07-08, ~30 stars / 10 forks.
- Examples include `Landau_damping.py`, `Langmuir_wave.py`, `Weibel_instability.py`, `two-stream_instability.py`, `auto-differentiability.py`, `inference_two_stream.py`, and `optimize_two_stream_saturation.py`.
- arXiv `2512.12160` describes a fully electromagnetic, multispecies, relativistic 1D3V PIC code implemented in JAX, with JIT / automatic vectorization, explicit Boris and implicit Crank-Nicolson/Picard-type solvers, standard benchmarks, and autodifferentiation demonstrations.

Assessment:
- Best match to ULMI's proposed first use: fast 1D scans / optimization / classification.
- 1D3V is the right dimensionality for a low-fidelity but fast scan engine; not too heavy like a 3D code.
- Built-in autodiff examples make it a good place to test whether differentiable PIC is numerically useful rather than only syntactically possible.

### 2. PyPIC3D — architecture reference / possible second-stage code

Evidence:
- GitHub description: `3D particle-in-cell code written in Python using Jax`.
- GitHub metadata at survey time: MIT license, Python, updated 2026-06-06, ~3 stars / 2 forks.
- Docs / README describe a config-driven CLI, Boris pusher, particle model, deposition modules (`J.py`, `rho.py`), current deposition methods `j_from_rhov` and Esirkepov, solvers, diagnostics, and openPMD output.
- APS DPP 2025 abstract claims Python/JAX, JIT-compilable, autodifferentiable, CPU/GPU support, C++-level performance, relativistic/electrostatic/electromagnetic solvers, and Esirkepov current deposition.
- Demos include convergence testing, Orszag-Tang vortex, magnetic reconnection 2D, two-stream, and Weibel.

Assessment:
- Valuable for implementation architecture: charge/current deposition, Esirkepov conservation, diagnostics/openPMD organization, 3D abstractions.
- Probably too heavy for the first fast-1D parameter scan prototype.

### 3. PiC-Code-Jax — educational/minimal reference

Evidence:
- GitHub `SeanLim2101/PiC-Code-Jax` README: `1D PiC Code for Simulating Plasmas with Google JAX`, an Imperial College London undergraduate research project supervised by Aidan Crilly.
- Uses JAX DeviceArrays, `vmap`, `jit`, and discusses single-precision/performance issues.
- GitHub metadata: no clear license at survey time, small repo.

Assessment:
- Useful to inspect as a minimal JAX-friendly PIC code structure.
- Not a mature research foundation for ULMI simulations.

### 4. Differentiable kinetic simulations — methodology lead

Evidence:
- APS DPP 2022 abstract says differentiable kinetic plasma solvers in JAX/PyTorch can use fast, exact/machine-precision gradients to discover physics in Vlasov-Fokker-Planck simulations and improve PIC modeling.

Assessment:
- Strong conceptual evidence for differentiable kinetic/PIC workflows.
- Not necessarily a ready-to-use code; treat as method inspiration.

## Mechanism and risks

JAX helps because:
- PIC loops have fixed array shapes in a controlled 1D/1D3V setting; JAX can compile time stepping with `jit` / `lax.scan` and parallelize over particles or parameters with `vmap`.
- JAX makes batch scans and ML/optimization loops easier: many small 1D runs can become a compiled GPU/TPU workload instead of Python loops.
- Autodiff can expose gradients for selected smooth objectives, parameter discovery, or optimization when the numerical path is differentiable enough.

JAX does **not** make PIC physics intrinsically more accurate:
- PIC particle noise remains; gradient descent on noisy observables can be unstable.
- Deposition/interpolation (CIC/TSC) is piecewise smooth; particle crossing cell boundaries creates derivative discontinuities/noisy gradients.
- `scatter_add`, sorting, boundary conditions, particle loss/injection, ionization, collisions, and adaptive algorithms are difficult for meaningful gradients.
- Strong laser-solid HHG physics may require kinetic effects, ionization, high harmonic diagnostics, absorbing/injecting boundaries, and numerical dispersion controls that a first JAX prototype may not yet reproduce.

## Recommendation for ULMI Lab

Treat JAX PIC first as a **fast low-fidelity 1D scan / data-generation engine**, not as a replacement for EPOCH.

Suggested staged plan:
1. Sandbox JAX-in-Cell.
   - Clone/install `uwplasma/JAX-in-Cell` in a scratch environment.
   - Run built-in Landau damping, Langmuir wave, two-stream, and autodiff/optimization examples.
   - Measure runtime, memory, JIT compile overhead, and GPU/CPU behavior.
2. Minimal EPOCH 1D benchmark.
   - Build a simple laser-overdense-slab / preplasma benchmark in both EPOCH and JAX-in-Cell if supported.
   - Compare energy conservation, reflection/absorption, electron phase space / spectrum, and rough reflection spectrum.
3. Use as scan/classifier generator.
   - Scan `a0`, `L`, `ND/a0`, thickness, pulse shape, and perhaps boundary/target parameters.
   - Use outputs as low-fidelity features for ranking/classification (e.g. promising HHG efficiency / bad wavefront / strong absorption).
4. Only then try gradient-based optimization.
   - First test gradients on smooth toy problems (Landau/two-stream objectives).
   - For laser-solid / HHG, expect black-box optimization or classifier-guided search to be more robust initially than direct gradients.

## Next action if Zhiping asks

Prepare a sandbox trial of JAX-in-Cell:
- Create an isolated environment under a scratch/project location (not inside Curved_surface production directories).
- Install/clone JAX-in-Cell.
- Run examples and collect a short reproducibility report: install commands, environment, first-run JIT time, second-run time, memory, output plots/data, and any limitations.
- Do not make claims about HHG applicability until at least one EPOCH-vs-JAX 1D benchmark has been defined and compared.

## Follow-up: Zhiping's 2023 ML+EPOCH HHG thesis

Trigger: Zhiping Telegram `main:6420513923:156` asked whether 1D JAX PIC would help his earlier thesis work at `/home/zhiping/knowledge_base/thesis/2023/2023--利用机器学习优化强激光等离子体高次谐波产生效率/thesis.md`. Answer delivered in Telegram `main:6420513923:158`.

Key facts read from the thesis:
- The thesis already uses **EPOCH(1D)** after a Bourdieu/moving-frame transform: oblique incidence is mapped to a normal-incidence 1D problem. The thesis explicitly notes this greatly reduces computation and aids fast analysis, but loses focal spot / transverse nonuniformity / divergence angle / transverse electron motion.
- Parameter space: five variables `a0`, `a0/N`, `theta`, `L`, `phi`. Discretized total was about `6,715,800` points.
- Actual sample generation combined greedy/adaptive random walk, random samples, and targeted scans, producing `94,517` PIC sample points.
- Main objective was HHG conversion efficiency `eta`, defined from reflected spectrum high-order energy (usually `n >= 2`) divided by incident spectrum energy. This was chosen because it only needs two EPOCH field outputs (incident fully in domain and reflected fully in vacuum). Diagnostics like gamma peak / pulse width need >100 outputs per laser period and slow EPOCH substantially via disk I/O.
- Workflow: EPOCH samples → correlation / t-SNE / DBSCAN on high-eta samples → neural network classifier for high-efficiency type → separate efficiency network for the key high-efficiency `type00` region → scan all discrete parameter points cheaply via networks → validate top candidates with PIC.
- Important numbers: high-efficiency sample maximum `eta ≈ 84.2%` at around `a0=15.849`, `a0/N=1.0`, `L=0.05 λ`, `phi=180°` (theta appears in conclusion as ~54°; OCR is imperfect). Final high-efficiency region: `a0≈15–20`, `a0/N≈0.9–1.1`, `theta≈54°–57°`, `L≈0–0.05λ`, `phi≈180°`, with `eta > 83%`.
- Thesis conclusion: classification + neural networks reduce the needed PIC simulation range; per-class training is better than one global efficiency network. The network's main use is to rapidly narrow the PIC search range, not to replace PIC truth.

Assessment for JAX PIC:
- Fit is unusually strong because the original workflow is already 1D PIC + ML. JAX-in-Cell/1D3V is likely useful as a fast low-fidelity sample generator / active-learning inner loop.
- Biggest value: replace the slow Python→EPOCH→file I/O→postprocess loop for exploratory scans with in-process JIT/vmap/lax.scan and in-memory diagnostics.
- JAX could cheaply add objectives that the thesis postponed or avoided due to output cost: `eta>=20/50/100`, cutoff, pulse width, waveform sharpness, gamma/electron-layer proxies.
- It should not directly replace EPOCH. Need benchmark against EPOCH for the thesis representative points and verify moving-frame/Bourdieu mapping, laser/target setup, density profile, boundary conditions, energy conservation, reflected spectrum, and noise.
- Proposed route: JAX-in-Cell sandbox → reproduce built-in examples → implement minimal thesis-style laser-solid benchmark → compare to EPOCH on high/low representative points → use JAX for coarse/adaptive scans and EPOCH for top-candidate validation.

## Follow-up source verification: transverse velocity support

Trigger: Zhiping Telegram `main:6420513923:159-160` noticed that JAX-in-Cell input examples seemed to support only `vx`, while PyPIC3D initialization visibly supports `vy`; he asked to verify. I cloned `uwplasma/JAX-in-Cell` and `uwplasma/PyPIC3D` under `tmp/jax_pic_verify/` and answered in Telegram `main:6420513923:162`.

Conclusion:
- **JAX-in-Cell is not 1D1V.** Source confirms a 1D3V particle state: positions/velocities are stored with shape `(N, 3)`, and the pusher/current deposition use all three velocity components.
- The confusion comes from examples/docs emphasizing x-directed two-stream/Langmuir-style cases: sample inputs often only set `vth_over_c_x`, `drift_speed_x`, and `velocity_plus_minus_x`, leaving y/z at default zero or hidden in terse `x,y,z` parameter notation.
- PyPIC3D's interface is more explicit for external initial conditions (`initial_vx`, `initial_vy`, `initial_vz` `.npy`) and for anisotropic temperatures (`Tx`, `Ty`, `Tz`), so it remains a useful second candidate / implementation reference when transverse distribution control matters.

Source evidence from the cloned JAX-in-Cell tree:
- `jaxincell/_parameters/_species_definitions.py`: `SPECIES_AXES = ("x", "y", "z")`; defaults include `vth_over_c_x/y/z`, `drift_speed_x/y/z`, `velocity_plus_minus_x/y/z`, and `initial_velocities`.
- `jaxincell/_parameters/_species_parameters.py`: `INITIAL_PHASE_SPACE_PARAMETERS = ("initial_positions", "initial_velocities")`; explicit initial phase-space arrays must have shape `(number_pseudoparticles, 3)`.
- `jaxincell/_state_initialization.py`: `initialize_species_phase_space()` loops over `SPECIES_AXES`, builds per-axis velocities from `vth_over_c_{axis}` + `drift_speed_{axis}` + optional plus/minus, and returns `jnp.stack(..., axis=1)` for `(N,3)` positions and velocities. If `species["initial_velocities"]` is provided, it overrides generated velocities.
- `jaxincell/_particles.py`: `boris_step()` and `boris_step_relativistic()` take `vs_n`, `E_fields_at_x`, and `B_fields_at_x` with shape `(N,3)` and use 3-vector Boris/cross-product updates.
- `jaxincell/_sources.py`: current deposition returns `(Jx,Jy,Jz)`; y/z current use `vy = vs_n[i,1]` and `vz = vs_n[i,2]`.
- `tests/test_runtime_input_parameters.py`: runtime overrides pass explicit velocities `[[1.0,1.1,1.2],[1.3,1.4,1.5]]` and assert they reach `initialized_sim.velocities`.

Implication for ULMI:
- Keep JAX-in-Cell as first lightweight 1D3V sandbox candidate, but the first benchmark should explicitly set nonzero `drift_speed_y` or `initial_velocities[:,1]` and verify that transverse momentum affects the fields/diagnostics as expected.
- Do not treat the documentation's `1D3V` label alone as sufficient; verify laser injection, boundary conditions, moving-frame/Bourdieu mapping, and HHG-relevant diagnostics against EPOCH before using it as a scan engine.

## Follow-up: Zhiping's adroit-vis GPU trial and next reproduction target

Trigger: Zhiping Telegram `main:6420513923:171` (acknowledged in `main:6420513923:172`).

New durable facts:
- Zhiping already tested both JAX-in-Cell and PyPIC3D on `adroit-vis` under:
  - `/scratch/network/zl8336/try_PIC_GPU_JAX`
- He reports that all examples for both codes have been run using GPU.
- Existing artifacts to inspect next:
  - `/scratch/network/zl8336/try_PIC_GPU_JAX/report.html`
  - `/scratch/network/zl8336/try_PIC_GPU_JAX/field_init_guide.md`
- Desired next benchmark target:
  - reproduce the EPOCH 1D case at `tiger-vis:/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/1D/45/ND_a0=0.30/input.deck` with both JAX-in-Cell and PyPIC3D.

Immediate next analysis plan (completed 2026-07-09):
1. Read the adroit-vis reports (`report.html`, `field_init_guide.md`) and extract each code's actual GPU status, example coverage, field/particle initialization API, and limitations.
2. Read the EPOCH `input.deck`; identify the physics to reproduce: normalization, density profile, target geometry, laser pulse, boundary conditions, moving-frame/Bourdieu assumptions if present, diagnostics, and expected outputs.
3. Produce a two-code reproduction route: what minimal physics each code must implement, how to map EPOCH fields/particles/density/laser to the code's input format, what diagnostics to compare first, and which discrepancies would make a code unsuitable.
4. Keep EPOCH as the reference truth. Treat JAX-in-Cell/PyPIC3D as candidate low-fidelity or prototype engines until benchmarked.

Completed deliverables:
- Telegram report: `main:6420513923:173`; HTML attachment: `main:6420513923:174`.
- Human-facing HTML artifact: `/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax-pic-repro-20260709/epoch1d_reproduction_route.html`.
- Durable supporting note: `epoch1d_reproduction_route_2026-07-09.md` in this knowledge entry.

Key reproduction-route conclusions:
- JAX-in-Cell is first-line for fast 1D scan/active-learning because it is 1D3V, passed A100 examples/tests, and supports explicit initial positions/velocities plus patched evolved-field injection. Its main engineering blocker for the full EPOCH-resolution case is output memory: the naive full time-history return would be enormous (`~1e5 steps × 1.12e5 cells × 3`), so sparse/final diagnostics are required.
- PyPIC3D is heavier but is the best cross-check for explicit transverse drift and field/particle array initialization because `initial_vx/vy/vz`, `initial_x/y/z`, and `[fieldN]` `.npy` interfaces are transparent.
- Current target `input.deck` has `laser_a0=3` despite the parent path `a0=20`; `N=350`, `D=0.002571λ0`, `ND/a0=0.30`, `θ=45°`, `n_eM≈8.62e29 m^-3`, `dx=0.2 nm`, `nx≈112010`, target ≈10.3 cells, macro electrons≈2057, ions≈343.
- Critical physics mapping: EPOCH `drift_y=-m_s c tanθ` is momentum-like Bourdier-frame notation. JAX-in-Cell/PyPIC3D velocity APIs should receive `v_y=-c sinθ≈-0.707c`, not `-c tanθ`.
- Current directory's old EPOCH outputs/log likely do not match the current deck (`final_field.nc` has x=160000; log final time ≈20T0 vs deck t_end=25T0). Quantitative comparison requires either locating the original deck or rerunning EPOCH from the current deck after authorization and `checkquota`.
- Both JAX codes currently lack the deck's Nanbu collision physics; first benchmark should be collisionless or compared to an EPOCH collision-off reference, not claimed as exact collision-on reproduction.
- Recommended next step if Zhiping authorizes implementation: create shared initializer under `/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/`, then run vacuum pulse propagation tests before adding the cold moving-frame slab.

## Follow-up: adroit-vis corrected a0=20 initialization sanity benchmark (2026-07-11)

Trigger: Zhiping Telegram `main:6420513923:198`; report sent in `main:6420513923:200`; HTML report attached in `main:6420513923:201`. Detailed note: `epoch1d_adroit_initialization_sanity_2026-07-11.md`.

Summary:
- Copied corrected a0=20 `input.deck` through `tiger-vis` because adroit-vis could not directly read the GPFS path.
- Created no-Slurm benchmark under `/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/`.
- Working Python on adroit is `/scratch/network/zl8336/.conda/envs/GPU_Python/bin/python`; default `python3` lacks JAX.
- Reduced-grid test (`25 cells/λ0`, `Nx=1000`) uses corrected deck fields (`a0=20`, `theta=45°`, `D/λ0=0.017142857`) and verifies code-specific Bz offsets.
- JAX-in-Cell: Bz at `x_E-0.5dx`; final left-going energy fraction after `5λ0` propagation `2.5e-6`.
- PyPIC3D: Bz shift `-(0.5+0.5*CFL)dx=-0.995dx`; final left-going energy fraction after `4.99λ0` propagation `8.8e-9`.
- Peak fields are ~`14.1 Ec/Bc`, consistent with moving-frame `a0 cos45°`, not lab-frame `a0=20`.
- Particle API arrays were written with `v_y=-c sin45°=-0.707c`, but no physical collisionless slab reproduction is claimed yet. EPOCH deck has Nanbu collisions, so collision-on agreement is out of scope without a collision operator or collision-off EPOCH reference.

Artifact paths:
```text
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/epoch1d_initialization_sanity.py
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/summary.json
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/epoch1d_initialization_sanity_report.html
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260711/epoch1d_initialization_sanity_report.html
```

## Follow-up: collisionless reflected-field smoke test (2026-07-11)

Trigger: Zhiping Telegram `main:6420513923:202`; report in `main:6420513923:204`; HTML/figures in `main:6420513923:205-207`. Detailed note: `epoch1d_reflection_smoke_2026-07-11.md`.

Summary:
- Created and ran `/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/epoch1d_reflection_smoke.py`.
- Reduced short-domain collisionless setup: `x/λ0=[-10,6]`, center `-5λ0`, `150 cells/λ0`, `Nx=2400`, `steps=2400`, `travel=8λ0`, foil thickness only `2.57` cells.
- PyPIC3D with explicit 3D volume weight (`λ0²` transverse area) produced clear reflection: final left-going energy in `x<-1λ0` = `0.603`, peak `Ey/Ec=16.35`, no NaNs.
- JAX-in-Cell with the same `λ0²` volume weight produced almost no reflection (`4.7e-6` left-going in `x<-1λ0`), showing the weight convention cannot be copied directly from PyPIC3D.
- JAX-in-Cell with line-density scaling (`weight_scale=1/λ0²=1.5625e12`) produced strong reflection: final left-going in `x<-1λ0` = `0.900`, peak `Ey/Ec=32.98`, no NaNs.
- Interpretation: PyPIC3D uses 3D volume macroparticle weights; JAX-in-Cell is 1D and needs a line-density/per-unit-transverse-area weight. This is a normalization issue, not evidence that JAX cannot reflect.
- Caveat: reduced smoke test only; no collision physics; JAX line-density normalization still needs a clean derivation before quantitative comparison.

Artifacts:
```text
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/reflection_smoke_report.html
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/reflection_smoke_summary.json
/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/reflection_jax_weight_scale_1.5625e12.json
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260711/reflection_smoke_report.html
```

## Follow-up: per-T0 full-x diagnostic profiles (2026-07-12)

Trigger: Zhiping Telegram `main:6420513923:208`; time-window correction `main:6420513923:210`; report `main:6420513923:212`; artifacts `main:6420513923:213-215`. Detailed note: `epoch1d_profiles_per_T0_2026-07-12.md`.

Summary:
- Created/reran `/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/epoch1d_profiles_per_T0.py` and notebook wrapper.
- Corrected setup after Zhiping's comment: `x/λ0=[-20,20]`, pulse center `-10λ0`, snapshots `t/T0=0..20`, `150 cells/λ0`, `Nx=6000`, `CFL=0.5`, `steps_per_T0=300`, `6000` total steps.
- Outputs: JAX-in-Cell 21 PNG + 21 NPZ, PyPIC3D 21 PNG + 21 NPZ; zip size 8.05 MB.
- Each figure contains full-x panels for `Ey/Ec`, `ne/Nc`, `ni/Nc`, electron `vx/vy/vz`, and ion `vx/vy/vz`.
- PyPIC3D diagnostic required handling the flat particle backend (`particles=[flat_all]`) by splitting via `flat.species_meta` counts.
- Artifacts are under `/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/profiles_per_T0/` and local copies under `/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260711/profiles_per_T0_20T/`.

Caveat: collisionless reduced diagnostic only; foil is 2.57 cells thick, so this is not a converged EPOCH comparison.

## Follow-up: JAX-in-Cell nonrelativistic-pusher bug in updated profiles (2026-07-14)

Trigger: Zhiping Telegram `main:6420513923:251` reported that after increasing `Ne_macro` and `cells_per_lambda` in `epoch1d_profiles_per_T0.py`, PyPIC3D looked closer to EPOCH but JAX-in-Cell showed electrons almost completely pushed out of the target after laser arrival. Zhiping clarified in `main:6420513923:253` that electrons and ions are initially colocated on target; blowout happens after the laser hits.

Detailed supporting note: `jaxincell_relativistic_pusher_diagnostic_2026-07-14.md`.

Key findings:
- Current JAX section omitted `relativistic=True`; JAX-in-Cell defaults to `relativistic=False`, so it used nonrelativistic Boris at `a0=20` while PyPIC3D was explicitly relativistic.
- This explains the impossible JAX velocity panels (`vx/c`, `vy/c` reaching tens) and invalidates the current nonrelativistic JAX profiles for strong laser-solid comparison.
- Density diagnostics showed total charge conserved and initial electron-ion overlap correct; electrons are displaced after laser arrival, not lost or initially misplaced.
- A JAX-only probe with `relativistic=True` kept `|v|/c≤1`, proving the pusher-setting bug, but still showed strong electron displacement in the reduced setup; remaining issues include coarse target resolution (`cells_per_lambda=400` gives only `~6.9` target cells vs EPOCH `~68`), boundary/physics mismatch, and JAX line-density normalization convergence.
- After Zhiping explicitly requested it in `main:6420513923:255`, main patched `/scratch/network/zl8336/try_PIC_GPU_JAX/epoch1d_benchmark/epoch1d_profiles_per_T0.py` with backup `epoch1d_profiles_per_T0.py.bak_relativistic_20260714_2012`; syntax check passed. No full rerun was performed at patch time to avoid overwriting existing outputs.
- Follow-up memory from Zhiping Telegram `main:6420513923:257-260`: high-resolution `cells_per_lambda≈1000` runs OOM mainly because current runners retain full time histories; ordinary disk save/restart breaks end-to-end JAX autodiff, while `lax.scan` + online objective accumulation + `jax.checkpoint`/`remat` can preserve gradients by recomputing intermediate chunks instead of storing the full history. Detailed note: `autodiff_checkpointing_memory_2026-07-14.md`.
- Follow-up implementation from Zhiping Telegram `main:6420513923:261`; report `main:6420513923:263`: created JAX-in-Cell-only project `/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/` with `run_jaxincell_final_field.py`, `write_centered_final_field.py`, and `README.md`. It omits PyPIC3D, preserves `relativistic=True`, saves raw final fields, shifts `Bz` from `x_Ey-dx/2` to `x_Ey` using `EM_analyzer.shift_field` with a periodic ghost point, and writes centered final `Ey/Bz` NetCDF. Smoke test `outputs/smoke_steps2_wrapper_ok/final_field_jaxincell_centered.nc` succeeded. Detailed note: `jaxincell_epoch1d_project_2026-07-14.md`.
- Follow-up final-field-only convergence from Zhiping Telegram `main:6420513923:307-308`; final report `main:6420513923:315-320`: Zhiping changed JAX-in-Cell `lax.scan` output to final-field only via `snapshot_steps=[steps-1]`, and main continued the convergence scan on adroit-vis to `CELLS_PER_LAMBDA=1200,1500,1800,2000`. All completed without OOM under `CUDA_VISIBLE_DEVICES=1`, `XLA_PYTHON_CLIENT_MEM_FRACTION=.95`; `cpl=2000` reflection HHG efficiency `0.31556`, reflection `Sx_peak=2.206e25`, transmission HHG `0.07779`, transmission `Sx_peak=1.073e25`. Interpretation: previous cpl=1200 OOM was mainly full-output XLA I/O buffer pressure; final-only output resolves the memory blocker through cpl=2000, while local Sx peak metrics remain more resolution-sensitive than integrated HHG. Detailed note: `jaxincell_cpl_to2000_finalonly_2026-07-16.md`. Zhiping then requested a revised one-curve plot in `main:6420513923:321`/`:323`; corrected artifacts are `combined_convergence_metrics_to2000_one_curve.png/.pdf` and `combined_convergence_report_to2000_one_curve.ipynb`, delivered in Telegram `main:6420513923:325-328`.
- Follow-up CEP scan from Zhiping Telegram `main:6420513923:329`/`:331`; final report `main:6420513923:335-340`: fixed `CELLS_PER_LAMBDA=1200` and scanned `φ_CEP = 0, ±π/4, ±π/2, ±3π/4, π` using final-field-only JAX-in-Cell on both adroit-vis A100 GPUs. All 16 reflection/transmission metric rows completed without OOM. HH band `[1.5,100.5] k0_M`; `Sx_peak` normalized by `laser_Sc_M`. Key extrema: reflection HHG max at `φ=-π/4` (`0.35193`), reflection `Sx_peak/ScM` max at `φ=π` (`2681.62`, close to `+π/4`), transmission HHG and `Sx_peak/ScM` both max at `φ=-π/4` (`0.09037`, `867.38`). Detailed note: `jaxincell_cep_scan_cpl1200_2026-07-16.md`.
- Follow-up EPOCH CEP scan from Zhiping Telegram `main:6420513923:341`; final report `main:6420513923:344-349`: processed EPOCH outputs under `/scratch/gpfs/MIKHAILOVA/zl8336/study_phy_cep/a0=20,ND_a0=0.3,45`, where each CEP directory already had `reflection.nc` and `transmission.nc`. Ran Curved_surface `utility/spectrum_1D.py` for all 8 phases × reflection/transmission with HH band `[1.5,100.5] k0_M` and `Sx_peak` normalized by `laser_Sc_M`. Key extrema: reflection HHG max at `φ=+3π/4` (`0.29731`), reflection `Sx_peak/ScM` max at `φ=-π/2` (`3966.50`), transmission HHG and `Sx_peak/ScM` both max at `φ=-π/4` (`0.08246`, `1229.12`). Detailed note: `epoch_cep_scan_cpl1200_2026-07-16.md`.

## Follow-up: CEP harmonic phase locking at reflection HH peak (2026-07-16)

Detailed note: `cep_harmonic_phase_locking_2026-07-16.md`.

Zhiping asked whether per-harmonic phases from the updated `spectrum_1D.py` are locked near `-π/2`, and whether JAX-in-Cell high-order detuning worsens this. I patched the EPOCH and JAX copies of `spectrum_1D.py` only to write numeric `<side>_per_harmonic_at_peak.tsv` files (backups kept), ran reflection CEP phase extraction, and delivered a self-contained report/notebook in `artifacts/cep_phase_locking_20260716/`.

Main conclusion at delivery time: EPOCH reflection harmonics show clear approximate locking near a common phase close to `-π/2` under the current analytic-signal diagnostic sampled at the broadband HH envelope peak (weighted mean phase roughly `-0.40π` to `-0.46π`, deviation `0.07π–0.12π`), while JAX CPL=1200 shows the same tendency but with larger scatter, especially negative CEPs, and shorter strong-order extent; this supports the observation that JAX high-order detuning/weak high-order coherence degrades the phase locking first.

## Follow-up: phase-reference caveat for the `-π/2` interpretation (2026-07-17)

Detailed note: `cep_phase_reference_rom_cse_2026-07-17.md`.

Zhiping challenged the physical meaning of `-π/2`: if the HH envelope / cycle-averaged flux is maximal while the real field is zero, that is not the same as a synchrotron-radiation picture in which the emitted radiation field from electron acceleration peaks at the burst. I checked ROM/CSE wiki pages, BGP 2006, Pirozhkov sliding-mirror 2006, Mikhailova CSE/synchrotron 2012, Cousens CSE trajectories 2020, LightRAG leads, and the current `spectrum_1D.py` / `EM_analyzer.spectrum` phase convention. Report sent in Telegram `main:6420513923:380`.

Refined conclusion: literature supports “phase locking” mainly as linear spectral phase / common emission time, not as a universal absolute lock at `-π/2`. BGP gives single-spike high-order phase approximately `φ_n≈-nΘ_1`; Pirozhkov defines perfect locking as spectral phase linear in frequency. CSE/synchrotron dynamics emission occurs near `|V_z|` max, `V_y≈0`, `a_y` max; acceleration-field intuition would put a physically referenced real radiation field peak near phase `0/π`, while current/velocity-like diagnostics can be quadrature near the `v_y` zero crossing. Current `spectrum_1D.py` measures `np.angle` of an analytic field constructed by `h=1+sign(k)` at the broadband HH envelope `peak_id`; therefore the observed common `-π/2` should be treated as a convention/diagnostic statement until group-delay and real-field/acceleration reference are fixed.

Future phase diagnostics should fit `φ_n = φ0 - n k0 x_emit`, report residual scatter after removing the linear group-delay term, sample phases at both the HH envelope peak and nearest real `E_HH` extremum, and compare HH timing with `J_y`, `dJ_y/dt`, or bunch `a_y` when particle diagnostics are available.

## Follow-up: implemented real-field-peak phase fit for EPOCH CEP scan (2026-07-17)

Detailed note: `epoch_cep_real_peak_phase_fit_2026-07-17.md`.

Zhiping requested in Telegram `main:6420513923:381` that tiger-vis `spectrum_1D.py` be extended to find the nearest `|E_forward_hh_real|` peak to the broadband HH envelope `peak_id`, read per-harmonic phases there, unwrap phases, and fit strong harmonics for the same eight EPOCH CEP cases under `/scratch/gpfs/MIKHAILOVA/zl8336/study_phy_cep/a0=20,ND_a0=0.3,45`. Patch backup: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_1D.py.bak_20260717_1300_real_peak_phase`. Outputs per case: `reflection_per_harmonic_at_real_peak.tsv`, `reflection_real_peak_phase_fit_summary.tsv`, `reflection_per_harmonic_at_real_peak.png`. Combined remote artifacts: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/epoch_cep_real_peak_phase_20260717/`; local mirror: `artifacts/epoch_cep_real_peak_phase_20260717/`. Report/artifacts sent in Telegram `main:6420513923:383-386`.

Key result: nearest real-field peak is only `12–20` cells after the envelope peak (`0.0042–0.0071 T0_M`, lab `~0.003–0.005 T0`), with `|E_HH_real|/Ec_M≈18–34`. Strong-harmonic linear fits used `fit_count=99` and have weighted wrapped residual RMS `0.045π–0.091π`. Zhiping then correctly pointed out in Telegram `main:6420513923:387` that the real-peak phase slope is just the sampling-point translation: moving the sample by `Δx` adds `n k0 Δx`, so removing the real-peak slope is essentially shifting the reference back to the envelope/group-delay center. Therefore this run should be interpreted as a sanity check confirming that envelope-peak sampling is the more natural harmonic-locking reference, not as a replacement diagnostic. The primary robust result remains that envelope-peak phases have small slope/residual; the common `-π/2` there is the HH wavepacket carrier/analytic phase at envelope center, not a reason to move the sample to a real-field extremum. After this correction, Zhiping asked to revert production `spectrum_1D.py`; completed in Telegram `main:6420513923:390`. Current production script is restored from `spectrum_1D.py.bak_20260717_1300_real_peak_phase` and only samples envelope-peak phase; the real-peak diagnostic version is preserved as `spectrum_1D.py.bak_20260717_1316_before_revert_real_peak_phase`.

## Follow-up: Mikhailova 2012 CSE model phase 0 vs PIC `-π/2` carrier phase (2026-07-17)

Detailed note: `mikhailova2012_cse_carrier_phase_2026-07-17.md`.

Zhiping asked in Telegram `main:6420513923:391` how to reconcile the PIC/EPOCH HH wavepacket carrier phase near `-π/2` at the envelope center with Mikhailova 2012 supplement Fig. S5, where the simplified CSE model field peaks at the pulse center (phase 0). Reply sent in `main:6420513923:393`. Key conclusion: CSE conditions fix emission time / envelope center / phase slope, not a universal carrier-phase intercept. Fig. S5's phase 0 follows from the simplified `E(t)=-Ne a_y(t')U(t')/(c^2R)` model with Gaussian/even `a_y` and positive/even `U`. In the same supplement, Fig. S2(d) PIC reflected field appears near a zero crossing at the intensity/envelope marker, and the text states at `t3` the transverse current `J_y=0` while `J_z` is maximal. Thus our PIC `-π/2` is plausibly the self-consistent reflected-field HH carrier CEP at envelope center, while the simplified analytic model captures envelope timing/intensity but not the spectral phase intercept.

## Follow-up: paused CEP/HH phase-locking research thread (2026-07-17)

Detailed synthesis note: `cep_hh_phase_locking_paused_research_2026-07-17.md`.

Trigger: Zhiping Telegram `main:6420513923:399` placed `/scratch/gpfs/MIKHAILOVA/zl8336/study_phy_cep/harmonic_superposition.ipynb` on `tiger-vis` for toy visualization of harmonic superposition under different per-order phases, then asked to temporarily set aside the detailed attosecond HH wavepacket carrier-phase research while preserving all current work.

Central preserved question: attosecond HH wavepackets appear phase-locked, but what absolute phase is locked? Why do broad PIC regimes — including the present ultra-thin `ND/a0=0.3`, `a0=20`, `θ=45°` case and Zhiping's earlier thick-target `/scratch/gpfs/MIKHAILOVA/zl8336/45thick` case (`a0=3`, `N=300`, `L=0.05`) — show sin-like attosecond pulses rather than the cos-like waveform suggested by the simplified CSE/Liénard-Wiechert toy model?

Compact resume point: robust evidence so far is small residual / near-linear spectral phase of strong harmonics at the HH envelope/group-delay center. The common EPOCH intercept near `-π/2` is a carrier CEP of the reconstructed HH wavepacket under the current analytic-field convention, not yet a universal mechanism constant. Real-field-peak sampling mainly adds the expected translation slope `Δφ_n=n k0 Δx`, so envelope-peak sampling remains the natural phase-locking reference. Future resolution likely requires electron trajectory/source diagnostics (`J_y`, `dJ_y/dt`, bunch acceleration, gamma spike, boundary response, propagation/filtering phase), not only far-field spectra.
