# 07 — Testing playbook

Claim classes: `FACT` = official tooling guidance; `CANDIDATE` = AEGIS position.

## The priority order (for a small app with one agent)

1. **Unit tests on the money paths** — auth, payment, sync, crypto, parsers. 100% coverage of trivial code and 0% of the token-refresh race is the failure mode; test where state machines live (04-BACKEND-API acceptance list is literally a test plan).
2. **LeakCanary in debug** — automatic leak detection with reference chains; zero config, highest signal-per-effort tool in the Android test stack. `FACT`.
3. **Leak detection in UI tests / CI** — LeakCanary supports leak detection inside instrumented tests, so leaks are caught before merge, not before user review. `FACT` (square.github.io/leakcanary).
4. **StrictMode during development** — disk/network on main thread surfaced immediately; enabled in debug builds, loud on purpose.
5. **One instrumented run on a real low-end device** — emulators lie about memory pressure, thermal throttling, and process death frequency. `CANDIDATE` but the return rate is high.
6. **UI tests for the money paths only** (Maestro or Espresso/Compose test): login → primary task → settings. Broad flaky UI suites get ignored; three honest paths get maintained.
7. **Macrobenchmark + Baseline Profile** for cold start when the store listing promises speed; measure before and after, never trust vibes.

## The test-writing rules that keep suites honest

- Test state transitions, not implementation details: "kill server mid-submit → user sees defined state" survives refactors; "button calls presenter method X" does not.
- Deterministic time and randomness injected; `Date.now()` sprinkled in logic makes flake.
- Flaky tests are deleted or fixed same day; a suite with known-flaky members trains everyone to ignore red. (Imported law: stale red trains skipping — agent-ui-playbook `CANDIDATE` 8.)

## The pre-release gate

- [ ] Unit: green on CI, money paths covered.
- [ ] LeakCanary: no leaks on the three money paths (manual session) + CI UI-test leak check green.
- [ ] StrictMode: clean on debug run of money paths.
- [ ] Real device smoke: install-from-AAB build, login, primary task, background/foreground cycle, process death restore.
- [ ] Minified release build smoke (R8 has broken reflection paths that debug builds never exercised).
