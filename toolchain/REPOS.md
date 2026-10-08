# Curated toolchain — the best repos per phase

Selection rule: a repo earns a row by being the tool practitioners actually reach for (confirmed by web research 2026-10-08, sources in `research/SOURCES.md`), not by stars. Star counts drift; roles don't. Each row: what it is for, which playbook phase it serves.

## Security (phase 2 — the spine)

| Repo | Role | Notes |
|------|------|-------|
| [OWASP/owasp-masvs](https://github.com/OWASP/owasp-masvs) | The standard itself | MASVS v2: L1/L2/R levels, 8 domains. The checklist language every finding cites. |
| [OWASP/owasp-mastg](https://github.com/OWASP/owasp-mastg) | The testing guide | MASTG: how to *test* every MASVS requirement, per-platform techniques. The manual behind every scanner. |
| [MobSF/Mobile-Security-Framework-MobSF](https://github.com/MobSF/Mobile-Security-Framework-MobSF) | Automated static + dynamic analysis | The first-pass scanner: decompiles, flags manifest issues, secrets, CVEs, produces report. Runs in Docker; CI-able. Layer 1 — your eyes are layer 2. |
| [skylot/jadx](https://github.com/skylot/jadx) | DEX decompiler | Readable Java-shaped sources from any APK. The single most-used RE tool; also the "run it on yourself" check. |
| [iBotPeaches/Apktool](https://github.com/iBotPeaches/Apktool) | Resource/manifest decode + rebuild | Manifest truth, smali, resource inspection; `b` rebuilds for authorized hardening tests. |
| [frida/frida](https://github.com/frida/frida) | Dynamic instrumentation | Runtime hooks: crypto calls, pinning behavior, component probing. Needs device/emulator. |
| [sensepost/objection](https://github.com/sensepost/objection) | Frida toolkit (no scripting required) | `objection explore` gives runtime review out of the box. |
| [owasp-dep-check/dependency-check-gradle](https://github.com/dependency-check/dependency-check-gradle) | SCA in the build | OWASP dependency-check Gradle plugin: known-CVE scanning of dependencies in CI. |
| [gitleaks/gitleaks](https://github.com/gitleaks/gitleaks) | Secret scanning | Repo + history scanning; also run patterns over decompiled artifacts. Alternative: trufflesecurity/trufflehog. |
| [semgrep/semgrep](https://github.com/semgrep/semgrep) | Rule-based source scanning | Custom rules for the recurring patterns (TrustManager overrides, logging of tokens). |

Reverse-engineering terrain map (when you need more): the awesome list `maddiestone/AndroidAppRE` and community tool roundups consistently rank jadx + apktool + Ghidra (for native libs) as the core trio (researched 2026-10-08).

## UI/UX (phase 3)

| Repo | Role | Notes |
|------|------|-------|
| [android/nowinandroid](https://github.com/android/nowinandroid) | The reference app | Official architecture + Compose + offline-first + testing, maintained by Google. When lost, copy NiA. |
| [android/compose-samples](https://github.com/android/compose-samples) | Compose patterns | Jetsnack/Jetchat etc.: tokens, theming, motion in practice. |
| [skydoves/compose-performance](https://github.com/skydoves/compose-performance) | Curated perf literature | Stability, recomposition, lazy-list optimization articles collected. |
| [google/accompanist](https://github.com/google/accompanist) | Compose utilities | Permissions, navigation-animation extras (check deprecation status per API — items graduate into the main libraries). |

## Architecture & data (phase 1)

| Repo | Role | Notes |
|------|------|-------|
| [android/architecture-samples](https://github.com/android/architecture-samples) | Layered-architecture reference | The classic todo app in layered form; pairs with the official guide. |
| [sqldelight/sqldelight](https://github.com/sqldelight/sqldelight) | Typed SQL DB | Alternative to Room; multiplatform. Room (androidx) remains the default. |
| [ktorio/ktor](https://github.com/ktorio/ktor) | Client HTTP (Kotlin-multiplatform) | Retrofit+OkHttp remains the default Android pairing; Ktor when multiplatform is in play. |
| [google/dagger](https://github.com/google/dagger) | DI (Hilt) | Hilt for Android; manual DI acceptable pre-3-screens (03-ARCHITECTURE). |

## Build, release, testing (phases 5–7)

| Repo | Role | Notes |
|------|------|-------|
| [google/bundletool](https://github.com/google/bundletool) | AAB → APKs locally | Reproduces Play split delivery; test from the AAB. |
| [patrickfav/uber-apk-signer](https://github.com/patrickfav/uber-apk-signer) | Signing utility | Quick re-sign during authorized rebuilds; CI uses apksigner directly. |
| [fastlane/fastlane](https://github.com/fastlane/fastlane) | Release automation | `supply` uploads AABs to Play tracks; screengrab for screenshots. |
| [square/leakcanary](https://github.com/square/leakcanary) | Memory-leak detection | Debug auto-detection + leak-in-UI-tests for CI. |
| [mobile-dev-inc/maestro](https://github.com/mobile-dev-inc/maestro) | UI testing (YAML flows) | Lowest-friction UI flows for the three money paths. |
| [ben-manes/gradle-versions-plugin](https://github.com/ben-manes/gradle-versions-plugin) | Dependency freshness | Update reports in CI; pairs with dependency-check for CVEs. |
| [android/nowinandroid](https://github.com/android/nowinandroid) | CI/CD reference too | Its GitHub Actions are maintained by Google — copy the shape. |

## Honest dispositions (things agents reach for that AEGIS does *not* recommend)

- **Facebook Flipper**: deprecated (sunset by Meta). Use Android Studio inspectors + Perfetto. 
- **Blanket ProGuard keep-rule collections**: found in many templates; they silently disable obfuscation. Write minimal keeps per reflection site.
- **Client-side anti-tamper SDKs for small apps**: cost/benefit documented in 01-SECURITY; Play Integrity server-side checks cover the real need at this scale.

## Adding rows

Propose with: the scar or task the tool serves, evidence it is maintained, and why the current row (if any) loses. Same attribution discipline as everywhere else in AEGIS.
