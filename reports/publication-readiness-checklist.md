# Publication Readiness Review — Web Application Penetration Testing

**Review date:** 31 October 2026  
**Repository:** `orestisvladimirou/Web-App-Penetration-Testing` (`main`)  
**Repository visibility at review:** **Private** (confirmed via GitHub repository metadata)  
**Assessment scope:** Intentionally vulnerable local OWASP Juice Shop, not an external production system.

## Current coverage

| Finding | Demonstrated behavior | Evidence / remaining qualification |
|---|---|---|
| F-01 SQL injection | Local administrator-login bypass; negative login control returned 401 | Sanitized POST injection request, successful administrator challenge page and invalid-login HTTP 401 screenshot now uploaded; token-bearing JSON response screenshot withheld |
| F-02 IDOR | Account B retrieved Account A's basket after changing object ID | Redacted Requester screenshot uploaded. Independently saved evidence tying the original Requester token/session to Account B remains desirable; inspect screenshot redaction |
| F-03 DOM XSS | Initial Juice Shop DOM XSS completion and link-reproduced iframe rendering in Incognito | Two screenshots uploaded. **No JavaScript alert was observed in Incognito**; do not claim cross-session JavaScript execution |
| F-04 information disclosure | Cookie-free requests retrieved a confidential-marked lab document; index visible | Four screenshots uploaded. Directory index curl returned HTTP 200 and error 18 (incomplete transfer) |
| F-05 account enumeration | Existing recovery email returned question object; nonexistent email returned `{}` | Sanitized transcript plus two cropped original HTTP response screenshots uploaded; no unauthorized reset attempted |
| F-06 mass assignment | Public registration accepted `role:admin`; that user loaded Administration UI while normal customer was denied with an app-displayed 403 | Sanitized comparison and four cropped original screenshots (baseline, elevated registration, administration UI, ordinary-customer denial) uploaded; backend privileged state changes were not tested |
| O-01 CORS configuration | Wildcard `Access-Control-Allow-Origin` on public endpoints | Observation only; no confirmed exploit or credential exposure |

## Verified environment fingerprint

- Juice Shop application version: **20.3.0** from running API.
- Docker CLI: **29.9.2** (build `a82e528`).
- OWASP ZAP: **2.17.0**.
- Repository-qualified image digest as reported by Docker:
  `bkimminich/juice-shop@sha256:42843723a7bf4246aa5d63c324978b17ef296ac78434aeddd34c427bfb5853f4`
- The local image ID was recorded separately and happened to have the same SHA-256 string; do not assume this equality holds in other environments.
- The README contains a pinned image pull/run example. **A clean-machine reproduction has not yet been executed.**
- See [environment fingerprint](../scope/environment-fingerprint.md).

## Before switching the repository to Public

- [ ] Perform **manual pixel-level review** of all committed screenshots; ensure no bearer/JWT tokens, cookies, passwords, personal details or security answers are visible, including browser address bars and wrapped text.
- [x] Add safely redacted F-01 injection, challenge success and negative-control screenshots (JWT response remains intentionally withheld).
- [x] Add safely redacted F-06 registration and role-comparison screenshots.
- [x] Archive cropped original F-05 HTTP response screenshots alongside the sanitized comparison transcript.
- [ ] Review F-02 evidence for adequate Account B session attribution without exposing tokens.
- [ ] Ensure the executive summary consistently reports **six** technical findings and one separate CORS observation.
- [ ] Resolve qualitative severity labels, documenting assumptions and limits rather than retroactively inventing CVSS values.
- [ ] Review all report links, file names, scope statements, and technical claims for consistency.
- [ ] Preserve test date, recorded version and available digest; consider a clean-environment reproduction before claiming end-to-end reproducibility.
- [ ] Ensure the report never claims a fix was implemented or retested; **no remediation retest has been performed**.
- [ ] Make the repository public only after the user approves and final secrets review is complete.

## Publication decision

**Not yet publication-ready.** The repository is currently private, six technical findings and related evidence have been documented, and the lab fingerprint is now specific. Redacted screenshots for F-01, F-05 and F-06 are now committed. The main remaining priorities are full evidence/secrets review, consistency and severity review, and final report preparation. No need to generate more vulnerabilities simply to increase the finding count.

## Next practical sequence

1. Perform a final human review of all redacted screenshot evidence; keep the token-bearing F-01 response uncommitted.
2. Make the interim report a concise final report with consistent findings, limitations, and risk rationale.
3. Verify no sensitive material is committed and test the key links.
4. Review the final public-facing README and only then consider changing repository visibility.
