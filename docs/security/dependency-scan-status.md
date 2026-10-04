# Dependency Vulnerability Scan Status

## Current status

**Status: PENDING**

The Trivy dependency vulnerability scan is temporarily incomplete because
Maven Central returned HTTP `429 Too Many Requests` while Trivy attempted to
resolve Maven dependency metadata.

This is a rate-limit failure, not a clean vulnerability scan.

## Security policy

The pipeline distinguishes dependency-scan results as follows:

| Status | Meaning | Development CI | Release |
|---|---|---|---|
| `PASS` | Scan completed successfully and no blocking findings were detected | Allowed | Allowed |
| `FAIL` | Vulnerabilities, scanner errors, database failures, or unexpected errors were detected | Blocked | Blocked |
| `PENDING` | Scan could not complete because of a recognized temporary dependency-provider rate limit | Allowed with warning | Blocked |

A `PENDING` result must not be interpreted as a successful security scan.

While the status is `PENDING`, the pipeline must not:

- Publish an image to a release registry.
- Sign an image as an approved release.
- Create a production release.
- Promote an image through GitOps.
- Deploy the artifact to production.

Local development image builds and non-release validation may continue.

## Required remediation

Re-enable the dependency vulnerability gate after implementing and validating
one of these approaches:

1. Populate and cache Maven dependencies before the Trivy scan.
2. Cache the Trivy vulnerability database in CI.
3. Use a controlled Maven proxy or approved dependency mirror.
4. Scan resolved dependency manifests or lockfiles with a separately validated
   dependency scanner.
5. Use a controlled CI runner with reliable access to Maven Central.

The selected solution must preserve reproducibility and must not hide
dependency vulnerabilities or scanner failures.

## Failure behavior

The dependency scan must:

- Return `PASS` only after a complete successful scan.
- Return `FAIL` for vulnerabilities, scanner errors, database failures, or
  network errors other than the explicitly recognized Maven HTTP `429`.
- Return `PENDING` only for the recognized Maven rate-limit condition.
- Record the reason in the GitHub Actions job summary.
- Block image publication whenever the status is not `PASS`.

The pipeline must not use `continue-on-error: true` or `|| true` to convert
arbitrary scan failures into successful results.

## Ownership and review

**Created:** 2026-10-04  
**Review condition:** Revisit when the Maven rate-limit solution is implemented  
**Release requirement:** This document must be updated when the dependency scan
returns reliable `PASS` results in CI.
