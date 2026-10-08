# 06 — App handling (APK/AAB internals)

What an agent must know about the artifact itself — the file is a crime scene, walk it. Claim classes: `FACT` = platform mechanics; `CANDIDATE` = AEGIS procedure.

## Anatomy of an APK (and what each part means for review)

- `AndroidManifest.xml` (binary XML in the APK): components, permissions, exported surface, backup/debuggable flags, deep links. Truth lives here, not in the source repo's manifest (merges and build variants can differ).
- `classes.dex` (+ `classes2.dex`…): the bytecode. `jadx` decompiles to readable Java/Kotlin-shaped sources; `apktool d` gives smali + decoded resources and can rebuild (`apktool b`) after edits.
- `res/`, `assets/`, `resources.arsc`: strings (grep for secrets and forgotten debug copy), raw assets, configs.
- `lib/`: native code per ABI — check arm64 presence; native libs reviewed with Ghidra when the threat model justifies it.
- `META-INF/`: signatures. `apksigner verify --print-certs app.apk` — the signing-scheme truth.
- Split APKs (from AAB): base + config splits (ABI/density/language) + dynamic feature splits. Review the *installed union*, or you will miss the feature that ships in a split.

## Signing schemes v1–v4 (know what you are looking at)

- v1 (JAR signing): legacy, no whole-file integrity; alone on modern targetSdk it is a finding. `FACT`.
- v2/v3: whole-file APK signing (v3 adds key rotation proof). v3+ is the modern baseline; v4 supports incremental installs (requires v2/v3). `FACT`.
- Verify: `apksigner verify --verbose` shows exactly which schemes are present.

## The decode-and-inspect procedure (black-box pass)

```
jadx -d out/ app.apk          # readable sources
apktool d app.apk -o smali/   # manifest truth + resources + smali
apksigner verify --print-certs app.apk
aapt dump badging app.apk     # metadata, permissions, components
apkanalyzer manifest print ... # if SDK available
```
Then run the five minimum-bar checks (01-SECURITY) against `out/` and `smali/`. Grep order: secrets patterns, `exported="true"`, `usesCleartextTraffic`, `allowBackup`, `debuggable`, `addJavascriptInterface`, `MODE_WORLD_`, `TrustManager`, `HostnameVerifier`, `X509TrustManager` empty overrides.

## Modifying and rebuilding (only on artifacts you own or are authorized to test)

`apktool b` re-packs; you must re-sign (`uber-apk-signer` or `apksigner`) for the rebuilt APK to install. Rebuilding is how you test manifest hardening claims (e.g., flipping `allowBackup` and proving the backup behavior change). Keep modifications inside an authorized scope; this playbook covers defensive review and hardening of your own or consented apps.

## Dynamic analysis (when a device/emulator is available)

- `objection explore` (on Frida) for runtime inspection: hook crypto calls, list exported components, check pinning behavior, dump the class tree.
- Emulator + `adb backup` (when allowBackup=true) proves data-exfiltration surface concretely.
- Report honesty rule: dynamic claims only with the run that produced them. No device = the report says static-only. (The verification philosophy: a result you cannot reproduce through an independent channel is a claim, not a finding — reuse the double-entry rule from AEGIS ops: the artifact hash identifies the exact bytes audited; publish it.)

## Publish the hash

Every AEGIS report names the artifact it audited: filename + SHA-256. Findings attach to bytes, not to vibes. If the artifact changes, the audit restarts. `CANDIDATE` rule (imported from AEGIS's own verification protocol: checksums are how independent pipelines agree).
