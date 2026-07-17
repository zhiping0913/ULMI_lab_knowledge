# Autodiff vs memory/checkpointing for high-resolution JAX PIC

Date: 2026-07-14 EDT
Conversation: Telegram `main:6420513923:257-260`

## Context

Zhiping asked why increasing `cells_per_lambda` to ~1000 makes both JAX-in-Cell and PyPIC3D run out of GPU memory on two 80 GB A100s, and whether one can keep arrays in CPU memory while doing computation on GPU. I explained that the dominant issue is full time-history storage, not just `Nx`: for `Nx≈60000`, `Nt≈33330`, one `Nt × Nx × 3 × float64` field history is already ~48 GB, and storing E/B/particles plus XLA temporaries can exceed 160 GB. Two A100s do not automatically combine memory for a single PIC domain without explicit sharding/domain decomposition.

Zhiping then asked whether a chunked save/restart strategy prevents autodiff through the result.

## Key answer

Ordinary disk checkpoint/restart breaks the JAX autodiff graph by default. If a run is executed as:

```text
run segment A -> save arrays to disk -> read arrays -> run segment B
```

then arrays read from disk become new primal values; they usually no longer carry the gradient relationship to parameters used in segment A. This workflow is good for diagnostics, plotting, restart robustness, and parameter scans, but not for end-to-end reverse-mode autodiff unless custom differentiation is added.

This does **not** imply that full histories must be stored in GPU memory. For differentiable optimization, the correct workflow is a dedicated differentiable runner:

1. Return a scalar or low-dimensional objective `J` (e.g. harmonic-band energy, reflected-window fluence, peak field), not full `E(t,x)`, `B(t,x)`, `particles(t)` histories.
2. Advance with `jax.lax.scan` or nested scans.
3. Accumulate the objective and minimal diagnostics online inside the scan.
4. Use `jax.checkpoint` / `jax.remat` (or a custom checkpointing schedule) so reverse-mode recomputes forward chunks instead of storing all intermediate states.
5. Save only a few plotting snapshots outside the gradient-critical path.

The conceptual distinction:

```text
ordinary disk checkpoint/restart: saves memory, breaks autodiff graph by default
JAX checkpoint/remat/scan: preserves autodiff, saves memory by recomputing intermediate states during backward pass
custom_vjp + disk checkpoints: possible, but requires explicit backward/recompute rule and is much more engineering work
```

## Physics/numerics caveat

Autodiff through PIC can be noisy or piecewise non-smooth because particle shape functions, cell crossings, boundary conditions, filters, particle injection/removal, clipping, and discrete diagnostics can introduce discontinuities or non-smooth dependence on parameters. A first differentiable smoke test should use a small collisionless setup, fixed seeds/particles, smooth objective, and compare `dJ/dp` against finite differences before trusting gradients.

## Recommended next step if pursued

Implement a small JAX-in-Cell differentiable smoke runner that:

- uses the corrected `relativistic=True` pusher;
- runs a low-resolution collisionless slab or vacuum-reflection setup;
- defines one scalar objective `J` (e.g. reflected field energy in a fixed window);
- computes `jax.grad(J)` with respect to a simple parameter such as `a0`, density scale, or foil position;
- verifies the gradient sign/magnitude against finite differences.

Only after this passes should we attempt high-resolution rematerialized scans or differentiable optimization for EPOCH-like HHG objectives.
