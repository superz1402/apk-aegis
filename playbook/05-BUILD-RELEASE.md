# 05 — Build & release playbook

Claim classes: `FACT` = official Android/Gradle/Play guidance; `CANDIDATE` = AEGIS position.

## Artifacts: AAB vs APK, and split delivery

- **Play distribution = AAB.** Play generates split APKs per device via bundletool; you never ship one monolithic APK to Play again. `FACT`.
- **Sideload/internal = APK** (universal or per-ABI). Debugging "works on my phone, crashes on theirs" often traces to split delivery: a feature's asset/dex landed in a dynamic feature or per-density split the tester's build never exercised. Check what the *installed set* contains, not what the project contains.
- `bundletool build-apks` + `bundletool install-apks` reproduces Play's split behavior locally. If you ship AAB, test from the AAB, not from `assembleDebug`. `CANDIDATE` discipline with high scar-return.
- 64-bit: any native code ships arm64 (Play requirement; 32-only native libs are a store rejection).

## Versioning

- versionCode strictly monotonic, forever; versionName for humans. A/B the same versionCode never uploads twice. `FACT`.
- Bake versionName + build commit into the app's About screen and into crash reports; "which build is this?" is the first question of every incident.

## Signing custody (write the chain down)

- Play App Signing: Play holds the release key; you hold the upload key. The custody question moves to the upload key: where does it live, who can mint another, what happens on laptop loss. Write the answer in the repo's private docs. `FACT` mechanics.
- Keystore passwords: CI secrets only; never in build files; never echoed. Rotation story written *before* the first leak, not after.
- CI builds sign with a keystore injected from secrets at build time and **deleted in the same job**; artifacts are the only thing that leaves the runner.

## R8 / minify / shrink

- `minifyEnabled true` + `shrinkResources true` for release; keep-rules minimal and reviewed. Blanket keeps are a finding (they disable obfuscation wholesale).
- Shipping obfuscation without crash deobfuscation = crashes arrive as gibberish. Keep and archive mapping.txt per release — Play Console stores it if you let it. `FACT`.
- Smoke-test the minified release build; R8 removal has broken reflection-dependent code paths that debug builds exercised happily. `SCAR-CLASS` (universally documented).

## CI/CD (GitHub Actions as the common case)

- CI builds the exact artifact type you distribute (AAB for Play lanes) — see `toolchain/PIPELINE.md` for the audit pipeline and `toolchain/workflows/apk-audit.yml`.
- Release lanes: tag → build signed AAB → upload to Play (internal track) via Gradle Play Publisher or fastlane supply → promote manually. Automate the build; gate the promotion. `CANDIDATE`.
- Every PR runs: build + unit tests + lint + dependency audit + secret scan. Audits that run only at release time arrive too late to be cheap.

## Play policies (the rejections that cost a cycle)

- Data Safety form matches actual collection; privacy policy URL live; target SDK current (`FACT` — Play enforces API-level deadlines annually).
- Permissions: background location, SMS/call-log categories need declaration review — avoid unless the product *is* that.
- First upload: expect the internal-testing track to be the real onboarding; production direct-ship for a new personal account is not the default.

## Play Integrity — decision note

See 01-SECURITY. One paragraph in the release doc: "we use / don't use Play Integrity because X, verdicts checked on endpoints Y." A conscious 'no' is fine; an unconsidered default is not. `CANDIDATE`.
