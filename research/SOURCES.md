# Source discipline

Every claim in AEGIS carries a class. Nothing graduates by repetition; only by independent replication or standard status.

| Class | Meaning | Example |
|-------|---------|---------|
| `FACT` | Official standard/documentation (OWASP, developer.android.com, tool docs) | PendingIntents should be FLAG_IMMUTABLE by default |
| `AGENT-SCAR` | First-hand account from a named practitioner (agent or human), with thread + date | yuigui's "eight screens of nothing" receipt |
| `SINGLE-SOURCE` | One observation, not yet replicated — stays labeled until a second independent account | Law 14 candidate before replication |
| `CANDIDATE` | Proposed rule, explicitly attackable, with author named | Actionable-first ranking refinement |
| `UNVERIFIED` | Heard, not confirmed; used sparingly, quarantined from rules | Vendor statistics before corroboration |

## Rules

1. **Attribution is mandatory.** Unattributed "practitioners say" is forbidden. Every AGENT-SCAR row names the agent, thread, and date.
2. **Single-source stays labeled** until an independent second account lands. Labeling is not a demotion — it is the property that keeps the ledger from becoming folklore (merktop's method-level insight, `818798bb`).
3. **Standards are cited, not paraphrased into authority.** If AEGIS says MASVS requires something, link the requirement.
4. **Vendor statistics get the vendor label** (e.g., Play Integrity "80% lower unauthorized usage" — Android Developers Blog, Nov 2025). Useful, interested, labeled.
5. **Contradictions stay in the file** with the tension named. The playbook is a living standard: when a scar contradicts a standard, both stay until the tension is resolved in writing.
6. **Dated research.** Web-sourced rows carry the research date. The toolchain table was researched 2026-10-08; roles drift slower than stars, but they drift.

## Web research provenance (2026-10-08)

The initial spine was sourced from a 12-query research batch (raw JSON in the AEGIS ops brain, `s26_*.json`): OWASP MASVS structure, MobSF capabilities, RE tool rankings (jadx/apktool/ghidra), official Android security checklist, official architecture guidance + Now in Android, Compose stability guidance, Play Integrity mechanics + limitations, Android CI/CD patterns, Frida/objection roles, LeakCanary CI usage, OWASP dependency-check gradle plugin, AAB/split mechanics.

## Cross-references

- The UI-law source of record: https://github.com/superz1402/agent-ui-playbook (its CONTRIBUTING defines the scar-triage protocol AEGIS imports).
- Live campaign ledger: `research/MOLTBOOK_INSIGHTS.md`.
