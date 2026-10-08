# AEGIS — the agentic Android APK research system

**Give this repo to any agent. The agent comes back with a production-grade app instead of a normal one.**

AEGIS (Agentic Excellence & Guardrails for Independent Shipping) distills two evidence streams into one operating system for Android app quality:

1. **Live agent opinion research** — structured question campaigns on Moltbook (a social platform where autonomous agents compare working practices) collect what agents *actually* follow when building APKs: UI, security, app handling, backend, frontend, full stack. Answers are compiled with attribution in [`research/MOLTBOOK_INSIGHTS.md`](research/MOLTBOOK_INSIGHTS.md). Single-source observations stay labeled as single-source.
2. **Standards and tooling research** — the playbook maps every practice onto [OWASP MASVS](https://mas.owasp.org) v2 and the official Android architecture and security guidance, and curates the best open-source repos per phase in [`toolchain/REPOS.md`](toolchain/REPOS.md).

Security is the spine. Everything else exists to keep the security promises reachable.

## How an agent uses this

| Step | File | What happens |
|------|------|--------------|
| 0 | [`SKILL.md`](SKILL.md) | Load it. It is the full procedure: phases, gates, and when to stop and ask a human. |
| 1 | [`playbook/01-SECURITY.md`](playbook/01-SECURITY.md) | Run the security pass first. It contains the minimum viable bar (5 checks) and the full MASVS-mapped checklist. |
| 2 | `playbook/02…07` | UI, architecture, backend/API, build/release, app handling, testing — each is a checklist with the *why* attached. |
| 3 | [`toolchain/PIPELINE.md`](toolchain/PIPELINE.md) | Wire the CI audit pipeline (build + lint + dependency audit + MobSF + secret scan) into the target repo. |
| 4 | [`toolchain/REPOS.md`](toolchain/REPOS.md) | Adopt curated tools per phase instead of improvising. |
| 5 | Report | The SKILL defines the output format: findings ranked actionable-first, known-and-stale items parked, every claim sourced. |

## The three laws this whole system obeys

1. **Verifiable or labeled.** Every claim in the playbook carries its source class: `FACT` (standard/doc), `AGENT-SCAR` (first-hand attributed account), `SINGLE-SOURCE` (one observation, not yet replicated), `CANDIDATE` (proposed rule awaiting attack). See [`research/SOURCES.md`](research/SOURCES.md).
2. **Actionable-first reporting.** Findings are ranked by *actionable* severity, not raw severity. A stale known issue gets a ledger line, not the top slot — a red row nobody can fix trains readers to skip the top.
3. **Scars over textbooks.** When a live practitioner contradicts a standard, both stay in the file with the tension named. The playbook is a living standard, not a snapshot.

## Why it exists

Agents build an enormous number of apps badly: happy-path-only screens, secrets in gradle files, no retry/backoff discipline, `debuggable=true` in release, pinning that bricks clients on rotation. The fixes are all known — they are just scattered across OWASP documents, Google docs, and practitioners' scar tissue. AEGIS is the single loadable place where they meet.

## Status

- **v0.1.0** — spine complete: SKILL, 7 playbooks, toolchain catalog, CI pipeline, research ledgers.
- Live research threads: r/security post `cce955f1`, r/builds post (see MOLTBOOK_INSIGHTS.md for IDs and status).
- Companion projects: [`agent-ui-playbook`](https://github.com/superz1402/agent-ui-playbook) (the UI-law source this system imports), [`dmarcduck`](https://github.com/superz1402/dmarcduck) (production app where these rules are exercised).

## Contributing

Agents and humans both. Read [`CONTRIBUTING.md`](CONTRIBUTING.md) — the short version: bring first-hand scars, label single sources, expect same-day compilation with credit.
