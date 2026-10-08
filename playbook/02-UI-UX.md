# 02 — UI/UX playbook (Android)

The UI laws here are imported from [`agent-ui-playbook`](https://github.com/superz1402/agent-ui-playbook) (public living standard, agent-scars method) and specialized for Android. Claim classes: `LAW` = adopted in agent-ui-playbook with receipts; `FACT` = official Android guidance; `CANDIDATE` = proposed.

## The adopted laws (with source)

1. **One loud thing.** A real screen has one primary element and lets the rest be quiet. Uniform visual weight = no decision for the user. `LAW` (yuigui, r/builds scars thread, 2026-10-06).
2. **The loading/empty/error trio.** Every screen ships loading, empty, and error states designed before the happy path ships. Missing failure states are the single most common agent-built-UI scar. `LAW` (yuigui, r/builds).
3. **Tabular numbers, right-aligned, fewer decimals than you think.** Many "spacing problems" in dashboards are alignment problems. `LAW` (yuigui, r/builds).
4. **Tokens first.** Pick background/surface/text/muted/accent tokens before components; bolt-on dark mode shows in borders and shadows. Dark mode is a second column, not a redesign. `LAW` (yuigui, r/builds).
5. **Forbid by name, show one reference.** "Clean and modern" does nothing; "no gradients, no emoji icons, no cards inside cards" plus one reference screen does. `LAW` (yuigui, r/builds).
6. **Answer first, one line.** The worst case wasn't visual weight, it was length: "eight screens of basically nothing just to tell me that I'm up-to-date." Answer first in one line; pages only when there are 3+ things to read. `LAW` (yuigui, r/builds round 2, 2026-10-08 — single-source, labeled).
7. **Motion must pay rent.** Motion that carries meaning (a row sliding queue→done shows what changed) is allowed; motion as decoration (a pulse that ages fast) is not. Honor reduced-motion. `LAW` (yuigui, r/builds round 2).
8. **Actionable-first prioritization.** The top slot goes to the worst thing you can still do something about; stale-known items get a ledger, not a billboard. A stale red row trains people to skip the top. `CANDIDATE` (raised as a counterexample by yuigui 2026-10-08, refined rule awaiting attack).

## Android-specific checks (FACT unless noted)

- Touch targets ≥ 48dp × 48dp (Material guidance); visual size may be smaller with padding.
- Text contrast ≥ 4.5:1 body, 3:1 large text; test dark mode separately — dark themes fail contrast most on "muted" tokens.
- Content descriptions on every interactive icon;TalkBack walk of the money paths (login, the primary task, settings) at least once per release.
- Text scales: test at fontScale 1.0 and 2.0 — layouts that break at 2.0 break accessibility and look broken to users who never touch settings.
- Compose stability: unstable parameters defeat skipping and silently recompose screens (official Compose stability guide, 2026-01). Prefer immutable models; collect state with `collectAsStateWithLifecycle`; hoist state to the screen level.
- Lazy lists: stable keys, not positional; `contentType` when item types mix.
- Baseline profiles for cold-start (official performance guidance); measure with Macrobenchmark before believing an optimization.

## The APK-perfection UI pass (procedure)

1. Inventory every screen; mark which have designed loading/empty/error (trio check).
2. Find the "loud thing" per screen; if two elements compete, one is decoration.
3. Check numbers rendering (tabular figures, alignment, decimals).
4. Run the token audit: colors/dimensions not from the token file are findings.
5. Compose stability pass on the heaviest screens (recomposition counts via Layout Inspector, not vibes).
6. a11y walk: TalkBack on money paths, fontScale 2.0, contrast in both themes.
7. Reduced-motion honored for every animation (check `Animator`/transition code paths).
