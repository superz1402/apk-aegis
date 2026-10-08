# 03 — Architecture playbook

Baseline: Google's official app architecture guide + the Now in Android (NiA) reference app (android/nowinandroid). The guide is the standard; NiA is the executable proof. Claim classes: `FACT` = official guidance; `CANDIDATE` = AEGIS position.

## The default architecture (start here, deviate with a reason)

- **Layers:** UI layer (Compose screens + ViewModel) → optional domain layer (use cases) → data layer (repositories over data sources). Dependencies point downward only. `FACT`.
- **Unidirectional data flow:** state flows down (StateFlow), events flow up. Two-way binding of state is where agent-built apps tangle first. `FACT`.
- **Single source of truth per datum:** one repository owns each business entity; UI caches are derived, not parallel. The divergence bugs that follow duplicate truth are the class of bug that survives every test suite because the tests also share the duplication. `FACT`.
- **Offline-first as default for read-heavy apps:** local DB (Room/SQLDelight) is the source of truth, network is the sync mechanism. The UI reads the DB and never waits on the network to render. `CANDIDATE` for small apps — server-truth-only is acceptable for tiny online-only tools, but the migration cost to offline-first later is high.

## Decision records (write them, not vibes)

For each of these, a one-paragraph decision record with the rejected options:

1. Navigation: type-safe Navigation Compose vs manual. (Default: Navigation Compose.)
2. DI: Hilt vs manual. (Default: Hilt when > ~3 screens; manual before that.)
3. Async: coroutines + Flow everywhere; no RxJava mixed in; no GlobalScope (`CANDIDATE`: GlobalScope usage is a finding by default — lifecycle-unowned work is how zombie states are born).
4. Serialization: kotlinx.serialization default; one codec, not three.
5. Image loading: Coil (Compose-native).

## The process-death test (the check agents skip)

Android kills and restores activities constantly. `FACT`: any ViewModel state not in SavedStateHandle is lost on process death. The pass: for each screen, answer "what does the user see after process death mid-form?" Unsaved input vanishing is a finding, not a quirk. Agents testing on emulators with "Do not keep activities" off never see this class of bug.

## Module hygiene

- Feature-per-module once the app passes ~5 features; before that, single module with package discipline. Premature modularization slows small teams more than it helps builds. `CANDIDATE` (NiA demonstrates the shape; the threshold is AEGIS judgment).
- No circular module deps; `:core` never imports features.
- API surface explicit: `internal` by default, public only cross-module.

## What "perfect" means here (the acceptance list)

1. Every screen's state machine has named states (loading/content/empty/error) — not booleans.
2. Every repository method answers: what happens offline? what happens on 401? what happens on 500?
3. No business logic in composables; composables render, ViewModels decide.
4. Process-death safe on every stateful screen.
5. The dependency graph is acyclic and understandable in one diagram.
