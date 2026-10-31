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

- [x] Visually inspect all **16 committed PNGs** at readable resolution; no exposed secret values or personal credentials were identified. Synthetic lab addresses and public challenge payloads remain intentionally visible. The successful-login token response screenshot was never committed.
- [x] Add safely redacted F-01 injection, challenge success and negative-control screenshots (JWT response remains intentionally withheld).
- [x] Add safely redacted F-06 registration and role-comparison screenshots.
- [x] Archive cropped original F-05 HTTP response screenshots alongside the sanitized comparison transcript.
- [ ] Review F-02 evidence for adequate Account B session attribution without exposing tokens.
- [x] Align the authoritative final report and main README to **six** technical findings, including F-03 **Unrated**, and one separate CORS observation.
- [x] Record qualitative priorities and explicit risk/impact limitations for all findings. No CVSS score or vector was invented.
- [x] Check all relative Markdown links against the repository tree; none were missing during the final check. Older working draft is labeled superseded.
- [ ] Preserve test date, recorded version and available digest; consider a clean-environment reproduction before claiming end-to-end reproducibility.
- [x] State explicitly that no remediation was implemented or retested.
- [ ] Make the repository public only after the user approves and final secrets review is complete.

## Publication decision

**Final report prepared; publication requires owner approval.** The repo is private, 16 committed screenshots have been visually audited for visible credential material, 17 existing text documents were scanned for token-like secrets, and a final report now lists six evidence-grounded findings and one non-exploitable observation. The main optional evidence improvement is independent Account B session attribution for F-02. The fresh-machine digest pull/run and any patched-remediation retests remain explicitly out of scope for this final report. No need to add findings simply to increase the count.

## Next practical sequence

1. Review the [final report](final-penetration-test-report.md) and public-facing README for presentation preferences.
2. Optional: archive independently attributable, token-free Account B basket baseline for F-02; current report already discloses this proof limitation.
3. Optional: test the pinned digest on a clean environment, without altering the assessment container.
4. Keep the repository private until its owner explicitly approves making it public.
