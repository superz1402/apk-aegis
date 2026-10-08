# CI audit pipeline

The pipeline runs on every PR and every release tag. It is deliberately boring: five jobs, all of which must be green before a human looks at a release.

```
build  ──► unit-tests ──► lint ──► dependency-audit ──► secret-scan ──► (optional) mobsf-audit
```

| Job | Runs | Catches | Setup |
|-----|------|---------|-------|
| build | `./gradlew assembleRelease` (or `bundleRelease`) | compile breakage; produces the artifact | — |
| unit-tests | `./gradlew testReleaseUnitTest` | logic, state machines | — |
| lint | `./gradlew lint` | manifest issues, a11y, API misuse | review `lint.xml` baseline quarterly |
| dependency-audit | OWASP dependency-check gradle plugin | known CVEs in dependencies | NVD API key as CI secret for rate limits |
| secret-scan | gitleaks over diff + full history | leaked keys/tokens | fail on any finding |
| mobsf-audit | MobSF static scan of the built APK/AAB | manifest/secrets/code/security quick pass | see workflow below |

## Wiring MobSF into CI (the workflow in `workflows/apk-audit.yml`)

MobSF publishes a Docker image; the workflow starts it, uploads the artifact from `build`, downloads the JSON report, and fails the job on threshold violations. Notes from practice:

- MobSF static analysis needs the APK; for AAB-only lanes run `bundletool build-apks` and audit the base APK, or also produce an APK lane for audit purposes.
- Treat MobSF findings as **layer 1**: triage every finding, but the SKILL's manual pass (TrustManager overrides, business-logic leaks, WebView judgment calls) is layer 2 and not optional.

## Reporting artifacts

The pipeline's outputs feed the AEGIS report (SKILL phase 7): CI job links, MobSF report JSON, dependency-audit report, gitleaks output. A report that cannot show its pipeline is a time-stamped belief.

## Copy-paste order

1. Copy `workflows/apk-audit.yml` into the target repo's `.github/workflows/`.
2. Add the dependency-check + gitleaks steps if not present (they are in the same file).
3. First run: expect findings. Triage into the SKILL report format; open issues per CRITICAL/HIGH.
4. Gate releases on green; audit runs that are advisory-only decay into decoration.
