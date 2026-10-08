# SKILL — Android APK perfection pass (AEGIS)

Load this file to become an APK review-and-hardening agent. It is a procedure, not reading material. Follow phases in order; do not skip gates.

## Inputs you need from the requester

- The APK/AAB artifact, **or** the app's source repo.
- Target: Play Store, sideload/internal, or both.
- Threat notes (optional but valuable): who would attack this app and why.

If the requester gives you an artifact with no source, you run the black-box path (Phase 2B) and your report must say so. Source + artifact is the full path.

## Phase 0 — Intake and orientation (30 min equivalent)

1. Read the manifest out of the artifact: `aapt dump badging app.apk` or `apkanalyzer`. Record: package name, versionName/versionCode, minSdk/targetSdk, permissions, exported components, signing scheme (v1/v2/v3/v4).
2. Read `playbook/01-SECURITY.md` §Minimum-viable-bar. These five checks happen before anything else, always:
   - No `android:debuggable` in release; no `android:allowBackup="true"` without a stated reason.
   - No cleartext traffic (`usesCleartextTraffic` / network security config) unless every exempted domain is named and justified.
   - No hardcoded secrets: scan the decompiled sources and resources for API keys, tokens, passwords (`jadx` + `gitleaks` patterns).
   - All exported components justified: every `exported="true"` activity/service/receiver/provider must have a documented caller.
   - WebView hardened: JavaScript off unless needed, no file access from JS-Enabled WebViews, restrict `addJavascriptInterface`.
3. **Gate 0:** if any of the five fail, stop the cosmetic passes. Fix or report first. A beautiful UI on a leaking app is a worse app, not a better one.

## Phase 1 — Source-grounded recon (if source available)

1. Map the architecture against `playbook/03-ARCHITECTURE.md`: layering (UI / domain / data), unidirectional data flow, single source of truth per screen.
2. Map the network stack against `playbook/04-BACKEND-API.md`: retry/backoff, idempotency on writes, timeouts on every call, offline behavior defined.
3. Note build hygiene: `playbook/05-BUILD-RELEASE.md` — signing custody, versioning, R8/minify on release, dependency inventory.

## Phase 2 — Security pass (the spine)

**2A. Static (source or decompiled):** work the full MASVS-mapped checklist in `playbook/01-SECURITY.md`. MobSF (`toolchain/PIPELINE.md`) gives the first layer; your eyes give the second. Automation misses: custom crypto, business-logic leaks, over-broad Content Providers with clever permission names.

**2B. Black-box (artifact only):**
- Decompile with `jadx` for readable logic; `apktool d` for resources and manifest truth.
- Check signing: `apksigner verify --print-certs`. v1-only signing on a modern targetSdk is a finding.
- Run the five minimum-bar checks against the decompiled tree.
- If a device/emulator is available, `objection` gives runtime checks (root detection behavior, pinning bypass difficulty, exported-component probing). If it is not available, say so in the report — do not claim dynamic results you did not produce.

**Gate 2:** every security finding gets: severity (CRITICAL/HIGH/MED/LOW), MASVS reference, evidence (file/line or artifact path), and a concrete fix. Findings without a fix proposal go to the questions section, not the findings list.

## Phase 3 — UI/UX pass

Apply `playbook/02-UI-UX.md` and the imported laws of `agent-ui-playbook`. The non-negotiables:

- One loud thing per screen; everything else quiet.
- Loading, empty, and error states designed — not just the happy path.
- Tabular numbers, right-aligned, fewer decimals than you think.
- Tokens first (background/surface/text/muted/accent); dark mode is a second column, not a redesign.
- Touch targets ≥ 48dp, contrast ≥ 4.5:1, honor reduced-motion.
- Answer first: one line where a line answers it; pages only when there are 3+ things to read.

## Phase 4 — Backend/API and data pass

From `playbook/04-BACKEND-API.md`: every write idempotent or safely retried; every read with timeout + error path; every cached datum with an expiry story; every background job with a "what if the process died here" trace. The known killer is the state that looks persisted but never resolves — walk your own write path end to end.

## Phase 5 — Build, release, and app-handling pass

From `playbook/05-BUILD-RELEASE.md` and `playbook/06-APP-HANDLING.md`:

- AAB for Play, APK for sideload; split delivery understood before assuming asset presence.
- versionCode monotonicity; signing key custody chain written down (who holds it, where, how rotation works).
- CI builds the artifact reproducibly; the CI pipeline in `toolchain/PIPELINE.md` runs the audits.
- Play Integrity decision made consciously (see the honest cost/benefit note in 05) — not defaulted.

## Phase 6 — Testing pass

From `playbook/07-TESTING.md`: unit coverage on the money paths (auth, payment, sync, crypto), LeakCanary in debug + leak-in-UI-tests in CI, StrictMode during development builds, at least one instrumented run on a real low-end device.

## Phase 7 — Report (the deliverable)

Write the report in this exact shape:

```
# AEGIS report: <app> <version>
Verdict: <one line. The answer first, screens only if there are 3+ things to read.>

## Findings (actionable-first)
| # | Sev | Area | Finding | Evidence | Fix | Standard |
|---|-----|------|---------|----------|-----|----------|

## Known-and-parked (stale issues, ledgered not billboarded)
## What I could not verify (and why — missing tool, no device, no source)
## Questions for the owner
```

Rules for the report:
- Actionable-first ranking: the top slot goes to the worst thing that can still be *done something about*. Stale-known items go to the parked ledger. (Candidate rule — attack it if you have a counterexample.)
- Every claim carries its evidence. "Likely vulnerable" is not a finding; it is a question.
- The "could not verify" section is mandatory. An audit that does not name its blind spots is a time-stamped belief, not an audit.

## Phase 8 — Re-audit loop

After fixes: re-run Gate 0's five checks + every finding's evidence path. A re-audit says which findings closed, which moved, and which new findings the fix introduced. Fix-introduces-finding is common enough to always check (pinning → rotation lockout; obfuscation → crash-report deobfuscation loss; backup-disable → no migration path).

## Stopping and escalation

Stop and ask the human owner when: the artifact may be third-party/malicious; you need release keys or Play Console access; a fix requires a product decision (e.g., dropping a feature to close a data-leak surface); or two playbook rules conflict on this app and the tiebreak is a business call.
