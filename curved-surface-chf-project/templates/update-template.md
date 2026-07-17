# Update template for Curved_surface / CHF project memory

When Zhiping sends a batch of information, parse it into this structure and update the project memory.

```text
Date / batch:
Source message(s):

Goal / question:

Simulation paths:
- path:
  dimension:
  geometry:
  key parameters: a0, w0, d, kappa, L, ND/a0, density, duration, polarization, etc.
  status: planned / running / done-unprocessed / processed / needs-rerun / superseded
  raw outputs:
  summary outputs:
  diagnostics:

Workflow / scripts:
- script path:
  purpose:
  command line:
  inputs:
  outputs:
  validation:
  failure modes / convention traps:

Results:
- numerical results:
- plots generated:
- comparison / controls:

Physics interpretation:
- which factor this supports: G_CSE/1D / G_spatial / C_wavefront
- mechanism:
- uncertainty / limitations:

Open questions:
- 

Next actions:
- 
```

Memory discipline:

- Record raw facts before interpretation.
- Preserve local paths and exact filenames.
- State whether results are measured, inferred, literature-supported, or speculative.
- Do not store secrets or credential material.
- If a workflow becomes reusable beyond this project, consider creating a skill under `/home/zhiping/skills` and symlinking it into `.library/custom/`.
