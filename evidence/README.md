# Evidence handling

This directory is reserved for **sanitized, human-reviewed** assessment evidence.

## Uploaded evidence

- [F01-login-request-redacted.png](F01-login-request-redacted.png) — original ZAP SQL injection request with password and cookie field hidden.
- [F01-admin-access-redacted.png](F01-admin-access-redacted.png) — administrator identity and Juice Shop challenge success screenshot.
- [F01-invalid-login-control-redacted.png](F01-invalid-login-control-redacted.png) — HTTP 401 invalid-login negative control.
- [F05-existing-account-response-redacted.png](F05-existing-account-response-redacted.png) — registered address returns a security question object (PowerShell source screenshot).
- [F05-nonexistent-account-response-redacted.png](F05-nonexistent-account-response-redacted.png) — nonexistent address returns empty JSON object (ZAP source screenshot).
- [F06-baseline-registration-redacted.png](F06-baseline-registration-redacted.png) — registration returns `role:customer`, with sensitive input masked.
- [F06-registration-role-admin-redacted.png](F06-registration-role-admin-redacted.png) — client-supplied `role:admin` returns `role:admin`, with password, answer and cookie masked.
- [F06-administration-screen-redacted.png](F06-administration-screen-redacted.png) — Administration page and section headings; users and feedback bodies excluded.
- [F06-customer-route-control-redacted.png](F06-customer-route-control-redacted.png) — ordinary customer receives visible 403 denial.

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



- `F01-login-response-redacted.png`
- `F02-account-a-basket-redacted.png`
- `F02-account-b-basket-redacted.png`

Before adding evidence:
1. Redact JWTs, bearer tokens, cookies, Authorization headers, real credentials, identifiers and unrelated personal information.
2. Crop screenshots to relevant HTTP fields or challenge results.
3. Preserve the request method, target endpoint, response status and key observable behavior.
4. Verify that every image belongs to the local authorized lab.
5. Do not upload raw ZAP sessions, unredacted HTTP archives or database dumps.

The linked PNGs above are actual sanitized/cropped source screenshots committed to GitHub. Other images not listed as uploaded remain outside the repository; do not infer that all screenshots shared during testing are published.
