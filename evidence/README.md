# Evidence handling

This directory is reserved for **sanitized, human-reviewed** assessment evidence.

## Uploaded evidence

- [F02-cross-account-request-redacted.png](F02-cross-account-request-redacted.png) — cross-account basket request and response; redacted image uploaded 31 October 2026. **Redaction should be reviewed again before publication.**
- [F03-dom-xss-search-redacted.png](F03-dom-xss-search-redacted.png) — original search input, injected iframe and completion banner; file verified on GitHub 31 October 2026.
- [F03-incognito-url-reproduction.png](F03-incognito-url-reproduction.png) — crafted search URL and iframe rendering in a separate Incognito session; file verified on GitHub 31 October 2026.

- [F04-ftp-directory-listing.png](F04-ftp-directory-listing.png) — directory index and filenames; upload verified 31 October 2026.
- [F04-confidential-acquisitions.png](F04-confidential-acquisitions.png) — confidential-marked document; upload verified 31 October 2026.
- [F04-unauthenticated-verification.png](F04-unauthenticated-verification.png) — credential-free HTTP checks including an incomplete-transfer warning for the directory; upload verified 31 October 2026.
- [F04-legal-document.png](F04-legal-document.png) — initial terms-of-use route; upload verified 31 October 2026.

- [F05-password-recovery-api-comparison.md](F05-password-recovery-api-comparison.md) — sanitized, explicitly transcribed comparison of registered versus nonexistent account responses; **not an original network capture**; verified on GitHub 31 October 2026.

- [F06-privilege-comparison-observations.md](F06-privilege-comparison-observations.md) — sanitized transcription of baseline registration, admin-role registration, admin UI access, and ordinary-customer 403 page; **not original screenshots**; committed 31 October 2026.

## Planned evidence (not yet uploaded):

- `F06-registration-role-admin-redacted.png` — edited registration request and server-side `role:admin` response, with password and security answer hidden.
- `F06-administration-screen-redacted.png` — administration page screenshot; redact any user identifiers or other sensitive details.
- `F06-customer-route-control-redacted.png` — ordinary customer access comparison to the same administration route.


- `F01-login-request-redacted.png`
- `F01-login-response-redacted.png`
- `F01-admin-access-redacted.png`
- `F01-invalid-login-control-redacted.png`
- `F02-account-a-basket-redacted.png`
- `F02-account-b-basket-redacted.png`

Before adding evidence:
1. Redact JWTs, bearer tokens, cookies, Authorization headers, real credentials, identifiers and unrelated personal information.
2. Crop screenshots to relevant HTTP fields or challenge results.
3. Preserve the request method, target endpoint, response status and key observable behavior.
4. Verify that every image belongs to the local authorized lab.
5. Do not upload raw ZAP sessions, unredacted HTTP archives or database dumps.

Screenshots already shared during development are **not automatically in this repository**. This README records the plan, not evidence of upload.
