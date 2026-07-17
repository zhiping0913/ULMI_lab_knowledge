---
name: 2026-07-04-molt-0-ulmi-migration-setup
description: >-
  Session record for onboarding the Debian main agent, connecting Telegram, migrating ULMI Lab constitution and shared skills, and recording lab assets and role boundaries.
date: 2026-07-04
molt_count: 0
type: session-journal
---

**2026-07-04 12:45 EDT** — TL;DR: This segment turned `main` into the configured Debian scientific 主力 agent for ULMI Lab: Telegram works, shared skills/lab constitution/wiki links are installed, ULMI/HPC/knowledge-base context is durable, and no active task remains. Cache-miss budget is nearly exhausted, so molt now for token efficiency.

## What this segment was about

Zhiping opened the Debian LingTai session, configured Telegram, and migrated scientific/research context from the previous Ubuntu agent machine. The goal was not to create new persistent agents yet, but to make this `main` agent usable as the scientific main agent for physics and numerical-computation work.

## Accomplishments

- Sent the initial internal-email onboarding greeting and safety note, including `/suspend all`, `/kanban`/`/viz`, `/goal`, ctrl+o, Telegram/IM recommendation, avatar concept, and tutorial hook.
- Symlinked `/home/zhiping/skills` children into `.library/custom/`, including individual `physics_skills` children, so physics skills are in the catalog.
- Created and pinned `standing-rules.md` with shared skill authoring rules and later role boundaries.
- Configured Telegram MCP with Zhiping using secret-safe local script `scripts/setup_telegram_config.py`; verified account `main` / bot `@ULMI_lab_main_bot`; saved Telegram contact alias `human` for chat/user `6420513923`; replied to Zhiping's Telegram “Hi”.
- Tested SSH to old Ubuntu machine `zhiping-vivobook-asuslaptop-k6602zc-k6602zc`; it resolved to Tailscale `100.114.31.125` and accepted passwordless SSH as user `zhiping`.
- Read remote Ubuntu context files `SOUL.md`, `USER.md`, and old `ulmi-lab-constitution/SKILL.md`, plus local `princeton-hpc` skill. Sensitive credential material in `USER.md` was not quoted or stored.
- Created durable knowledge entry `knowledge/ulmi-lab-context/KNOWLEDGE.md` capturing Zhiping/ULMI research context, scientific rules, HPC rules, knowledge-base assets, research wiki, and role boundaries.
- Migrated `ulmi-lab-constitution` to `/home/zhiping/skills/ulmi-lab-constitution` and exposed it through network `.library_shared/ulmi-lab-constitution`; removed redundant local custom symlink to avoid duplicate names.
- Set avatar/network rules from the ULMI constitution, including no-new-agents, mechanism-before-conclusion, evidence discipline, no secret propagation, and HPC `checkquota` / password-Duo stop rules.
- Symlinked `/home/zhiping/skills/knowledge-base-lightrag` into `.library/custom/knowledge-base-lightrag` and refreshed; skill is in the catalog.
- Recorded ULMI knowledge-base assets as core lab assets like HPC:
  - `/home/zhiping/knowledge_base` via `knowledge-base-user` for metadata/path/direct reads.
  - `/home/zhiping/knowledge_base_tools/Academic_graph_miner` via `academic-graph-miner` for citation graph work.
  - `/home/zhiping/knowledge_base_lightrag` via `knowledge-base-lightrag` for semantic evidence retrieval; vectorization maintenance is owned by Claude in that directory, not by `main` unless assigned.
- Recorded role boundary: `main` focuses on physics/numerical computation; knowledge-base maintenance belongs to Librarian; Debian server/network/SSH operations belong to `openclaw`; report out-of-scope issues to Zhiping rather than spending main context fixing them unless explicitly asked.
- Symlinked `/home/zhiping/research_wiki/wiki-laser-plasma-hhg` into `.library/custom/wiki-laser-plasma-hhg`, refreshed, and reported completion on Telegram. This wiki should be loaded for ROM/CSE/CWE, plasma-mirror HHG, attosecond sources, preplasma, HHG scaling/efficiency, waveform engineering, and EPOCH/PIC HHG work.

## Decisions and reasoning

- Do not create persistent avatars now. Zhiping explicitly said `main` is the scientific 主力 agent for now, persistent agents will be requested later if needed, and the existing Ubuntu Librarian continues to serve as Librarian.
- Keep `ulmi-lab-constitution` shared through `.library_shared/` only for this agent; because this agent scans `.library_shared`, also keeping a `.library/custom/ulmi-lab-constitution` symlink caused duplicate skill-name clutter.
- Treat local lab knowledge assets as first-class evidence sources before relying on memory or web search: this supports the ULMI constitution's source-traceability rule.
- Treat server/network/knowledge-base maintenance as explicit boundaries; bounded checks are fine when requested, but real maintenance/debugging should be reported to Zhiping.
- Molt now because the since-last-molt cache-miss budget is almost exhausted after many refreshes/setup operations, while durable stores are already updated and no active task remains.

## Artifacts and paths

- Pad: `system/pad.md` (current state updated; no active task waiting).
- Character: `system/lingtai.md` (updated as scientific main agent with lab assets and boundaries).
- Standing rules: `standing-rules.md` (shared skill location, constitution/no-agents, role boundaries).
- Knowledge: `knowledge/ulmi-lab-context/KNOWLEDGE.md`.
- Session journal path: `knowledge/session-journal/2026-07-04-molt-0-ulmi-migration-setup/KNOWLEDGE.md`.
- Telegram key messages:
  - `main:6420513923:11` — migrate constitution, no new agents, continue Ubuntu Librarian.
  - `main:6420513923:17` — knowledge-base assets are like HPC resources.
  - `main:6420513923:19` — role boundaries: physics/numerics only; Librarian/openclaw own maintenance.
  - `main:6420513923:21` — symlink `wiki-laser-plasma-hhg`.
  - `main:6420513923:23` — completion report for wiki skill link.
- Skills/assets now relevant:
  - `ulmi-lab-constitution` via `/home/zhiping/skills/ulmi-lab-constitution` and `.library_shared/`.
  - `knowledge-base-user` for `/home/zhiping/knowledge_base`.
  - `knowledge-base-lightrag` for `/home/zhiping/knowledge_base_lightrag`.
  - `academic-graph-miner` for `/home/zhiping/knowledge_base_tools/Academic_graph_miner`.
  - `wiki-laser-plasma-hhg` for `/home/zhiping/research_wiki/wiki-laser-plasma-hhg`.
  - `princeton-hpc` for Tiger/Della/Adroit work.

## Open tasks

None currently active. If Zhiping asks later, optional next checks include Princeton HPC host access (`tiger-vis`, `adroit-vis`, `della`) with strict short timeouts and password/Duo stop discipline. Do not proactively create avatars or perform maintenance.

## Collaborators

- Zhiping Li: human/PI; reachable via Telegram contact alias `human` and internal email `human`.
- Existing Ubuntu Librarian: continues to own literature/knowledge-base maintenance; do not replace unless asked.
- Claude instance under `/home/zhiping/knowledge_base_lightrag`: owns vectorization/LightRAG maintenance unless Zhiping assigns otherwise.
- `openclaw`: owns Debian server/computer/network operations.

## Gotchas and lessons

- Never quote/store/transmit credential material from legacy files or config. Telegram bot token is stored only in `.secrets/telegram.json`.
- Use producer-channel replies: Telegram messages must be answered on Telegram, internal email on internal email.
- For scientific claims: mechanism before conclusion, evidence/source traceability, uncertainty discussion, and physical interpretation of simulations are mandatory.
- Before any Slurm `sbatch`/`salloc`, run `checkquota`; block if Tiger scratch has <2 TiB free or Della scratch has <1 TiB free.
- If SSH prompts for password/Duo on Princeton HPC, stop and report; do not retry.
- If a paper is missing, OCR is poor, metadata ingestion is broken, or server/network/SSH infrastructure has problems, report symptoms/evidence to Zhiping rather than self-assigning maintenance.
