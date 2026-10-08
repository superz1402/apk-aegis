# AGENTS.md — for coding agents working in this repo

You are working inside AEGIS, an agent-consumable knowledge system. Rules of the house:

1. **Load `SKILL.md` first** if your task is reviewing/building an APK. It is the procedure. This file is only repo etiquette.
2. **Preserve claim classes.** Edits to `playbook/**` and `research/**` must keep the `FACT / AGENT-SCAR / SINGLE-SOURCE / CANDIDATE / UNVERIFIED` labels intact and truthful. Do not upgrade a class in passing.
3. **Living standard, not frozen text.** Improvements are welcome; every substantive change updates `CHANGELOG.md` in the same commit.
4. **No marketing voice.** The docs describe procedures and evidence. Adjectives are not evidence ("robust", "cutting-edge" are findings-shaped bugs in a knowledge system).
5. **Checklist format is load-bearing.** Agents parse the playbooks. Keep checkbox/bullet structure stable; additions append, restructures justify themselves in the PR body.
6. **Security content changes are high-scrutiny.** Any change to `playbook/01-SECURITY.md` must cite the standard or scar that motivates it in the PR description. A wrong security line is worse than a missing one.
7. **When the playbooks conflict**, do not silently pick one: open the tension in the file (both statements, the conflict named), then apply SKILL.md's escalation rule (business call → ask the human owner).
