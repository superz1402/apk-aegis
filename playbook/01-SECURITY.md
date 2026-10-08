# 01 — Security playbook (the spine)

MASVS-mapped Android app security for agents. Sources: OWASP MASVS/MASTG (standard), official Android security checklist (standard), practitioner scars (labeled per class). Claim classes: `FACT` = standard/docs; `AGENT-SCAR` = attributed first-hand account; `CANDIDATE` = proposed, awaiting attack.

## The minimum viable bar (the five checks)

For a two-person app with no security team, these five checks give the most protection per unit of effort. **They are pass/fail, not advisory.** (`CANDIDATE` rule, originally posed as an open question to practitioners in r/security thread `cce955f1`; based on the scar pattern that real incidents overwhelmingly trace to these five.)

1. **Release build is not debuggable, does not back up blindly.** `android:debuggable="false"` (implied by release buildType — verify the artifact, not the intent), `android:allowBackup="false"` or a stated reason + backup rules XML. Why: `debuggable` turns every other mistake into remote code execution; `allowBackup` exfiltrates app data through `adb backup` and cloud restore onto any device the user signs into.
2. **No cleartext traffic.** `android:usesCleartextTraffic="false"` (default since targetSdk 28 — verify anyway) and a network security config that names every exemption. Why: any HTTP call is a MITM harvest on hostile networks, and apps accumulate legacy HTTP endpoints silently.
3. **No hardcoded secrets in the artifact.** Decompile and scan (jadx + gitleaks patterns over sources *and* `res/values`, *and* BuildConfig). API keys in `strings.xml` and BuildConfig fields are the two most common leak sites. Why: decompiling is one command; assume every byte in the APK is public.
4. **Every exported component justified.** Walk the manifest: each `exported="true"` activity/service/receiver/provider needs a documented caller and an input-validation story. Intent redelivery and launch-anywhere bugs live here. Why: exported components are the app's open ports.
5. **WebView hardened or absent.** JavaScript disabled unless a named screen needs it; `addJavascriptInterface` only with a reviewed bridge; no `setAllowFileAccessFromFileURLs` on JS-enabled WebViews; external URLs open in the browser, not in-app, unless the screen needs in-app. Why: WebViews are a browser embedded in your app's permission sandbox, usually forgotten.

**Gate rule:** if any of the five fail, this is the report. Cosmetic passes wait.

## MASVS v2 domain map

MASVS v2 defines two verification levels (L1 = baseline, L2 = defense-in-depth) plus a resilience profile (R). Domains and what an agent checks in each:

### MASVS-STORAGE — sensitive data at rest
- Sensitive data never in SharedPreferences plaintext, external storage, logs, or clipboard (`FACT`, standard). Use EncryptedSharedPreferences/EncryptedFile (androidx.security-crypto) or AndroidKeyStore-wrapped keys.
- Check: `grep` for `getSharedPreferences`, `MODE_WORLD_READABLE/WRITABLE` (world-readable is a findings-0 critical), `getExternalStorage`, `Log.` calls containing credentials/tokens — tokens in logs is the scar that keeps recurring (`AGENT-SCAR` class, multiple practitioner accounts in public postmortems).
- Screenshots: app screens with sensitive data enter the system screenshot; decide masking (`FLAG_SECURE`) consciously for data that outlives the screen.
- Session tokens: in-memory or encrypted storage, cleared on logout, and never written to crash reports.

### MASVS-CRYPTO — cryptography done right
- No custom crypto, no ECB mode, no static IVs, no MD5/SHA1 for security purposes (`FACT`, standard).
- Keys live in AndroidKeyStore (hardware-backed where the device offers it); key material never derivable from a hardcoded constant alone.
- Random from `SecureRandom`, seeded once, not per-call reseeded (per-call reseeding has produced repeated nonces in the wild — `FACT`, documented historical weakness class).
- Check the *purpose* of every crypto call: encryption without integrity (no AEAD) is decorative. Prefer AES-GCM or ChaCha20-Poly1305.

### MASVS-AUTH — authentication and authorization
- Auth tokens: scoped, expiring, refreshed; refresh token storage encrypted (KeyStore-backed).
- Biometric unlock (BiometricPrompt) only as a *convenience layer over* server-side auth, never as the only factor for anything valuable; crypto-bound (setUserAuthenticationRequired) where it matters.
- Server-driven authorization is the rule; "the app hides the admin button" is not access control. Check the app never *decides* entitlement locally without a server token backing it.
- Local-only auth flows (offline PIN): rate-limit and lockout enforced in the artifact, not in UI state that a reinstall resets.

### MASVS-NETWORK — transport security
- TLS everywhere; network security config pins the CA set; certificate pinning follows the **pinning discipline** below.
- Check OkHttp/retrofit config: no trust-all SSLSocketFactory, no hostname-verifier overrides (the `checkServerTrusted` empty-override pattern is a recurring scanner-miss that hands over the whole TLS guarantee — walk custom TrustManagers by eye, automation buries them).
- WebSocket and raw sockets get the same TLS + pinning review as HTTPS.

**Pinning discipline (the rotation-lockout scar).** Pinning protects against MITM and bricks clients when done carelessly. Rules: pin to the *intermediate/backup* chain too, ship a backup key well before you need it, set pin expiry so you fail open to system trust rather than hard-fail ancient installs, and never pin a leaf cert you cannot rotate server-side. Have shipped pinning ever bricked clients on rotation? Yes — this is the canonical pinning scar (`CANDIDATE` question posed to practitioners in thread `cce955f1`; historical public incidents exist, e.g. large apps hard-failing after CA rotation).

### MASVS-PLATFORM — platform interaction
- Permissions: least privilege per feature; no `READ_EXTERNAL_STORAGE` broad grants where media-scoped APIs exist; runtime permission requests explained at use-site.
- Deep links: every `android.intent.action.VIEW` scheme tested for parameter injection; app links verified (assetlinks.json) rather than autoVerify-claimed blindly.
- PendingIntents: mutable base (FLAG_MUTABLE with explicit component) or immutable (FLAG_IMMUTABLE) — the mutable-PendingIntent hijack is a classic privilege-escalation path (`FACT`, platform guidance).
- Broadcasts: no sensitive data in implicit broadcasts; receivers registered per-screen and unregistered.
- Clipboard: sensitive values not copied by default; clipboard is readable by every app pre-Android 13.

### MASVS-CODE — code quality and build settings
- R8/minify + resource shrinking on release (raises reverse-engineering cost, kills unused code); keep rules minimal and reviewed — blanket keeps (`-keep class ** { *; }`) disable the whole point (`FACT` + recurring scar).
- targetSdk current (Play requires recent targets; old targets bypass platform hardening like the cleartext default and pending-intent immutability).
- Dependencies inventoried: `gradle-versions-plugin` + OWASP dependency-check in CI; abandoned libraries with known CVEs are the slow leak nobody notices until the incident.
- CI secrets: signing keys and tokens live in encrypted CI secrets, never in the repo, never echoed in logs. `gitleaks`/`trufflehog` on every push.

### MASVS-RESILIENCE — reverse-engineering resistance (opt-in profile)
- R8 is the baseline; commercial obfuscation/packing only when the app's IP or users justify it.
- Root/emulator detection: decide consciously. For most small apps it is overhead without payoff and generates false-positive support pain; Play Integrity (below) covers the server-side need.
- Safety net for tampering: server-side integrity checks (Play Integrity verdicts on sensitive transactions) beat client-side self-checks, which are bypassable by definition.

### MASVS-PRIVACY — user data discipline
- Data minimization as architecture: every collected field needs a named use, or it is a liability with a schema.
- Consent flows real, not buried; analytics scrub PII; crash reporters (e.g. Crashlytics) configured to strip tokens/PII — default crash payloads include shared prefs snapshots and can leak sessions.
- Play Data Safety form matches the actual collection — mismatches are policy strikes.

## Play Integrity — the honest cost/benefit

Play Integrity API attests: genuine app binary, genuine Play install, genuine device. Real value where the threat is API abuse by modified clients; "apps using Play Integrity features see 80% lower unauthorized usage on average" (Android Developers Blog, Nov 2025 — vendor statistic, treat accordingly). Costs and limits: requires Play distribution and Google Play services; adds a server-side verdict check per sensitive call; limitations documented by practitioners — emulators and some rooted devices pass basic verdicts, and the verdict is only as good as what your backend does with it (`FACT` for mechanics, `CANDIDATE`-labeled for the small-app recommendation). **AEGIS recommendation for a small app: skip client-side anti-tamper entirely; if the backend has endpoints worth protecting, add Play Integrity verdict checks on those endpoints only.**

## Secrets checklist (where they actually leak)

1. BuildConfig fields with API keys (default `buildConfigField` pattern) — move to local.properties + CI injection, or better: proxy the secret behind your backend.
2. `strings.xml` and raw resources.
3. Git history (the artifact is clean but the repo leaks — `gitleaks` full-history in CI).
4. Crash logs and analytics payloads.
5. Gradle files checked into public repos (keystore passwords, Play service JSON).
6. The decompiled artifact: assume `jadx -d out app.apk` runs on your app the day you ship it. Run it on yourself before someone else does.

## Source notes

- OWASP MASVS v2 / MASTG: https://mas.owasp.org (standard; consulted 2026-10-08).
- Official Android security checklist: developer.android.com (standard; consulted 2026-10-08).
- Pinning risk framing: Palo Alto Networks unit42 pinning overview + practitioner incident reports (secondary sources, 2026).
- Play Integrity mechanics: developer.android.com + Android Developers Blog 2025-11-19; limitations: Guardsquare and Approov practitioner analyses (secondary, 2025-2026).
- Practitioner scars: accumulating in `research/MOLTBOOK_INSIGHTS.md` with attribution and thread IDs.
