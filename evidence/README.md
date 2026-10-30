# Evidence handling

This directory is reserved for **sanitized, human-reviewed** assessment evidence.

Suggested filenames (not yet uploaded):

- `F01-login-request-redacted.png`
- `F01-login-response-redacted.png`
- `F01-admin-access-redacted.png`
- `F01-invalid-login-control-redacted.png`

Before adding evidence:
1. Redact JWTs, bearer tokens, cookies, Authorization headers, real credentials, identifiers and unrelated personal information.
2. Crop screenshots to relevant HTTP fields or challenge results.
3. Preserve the request method, target endpoint, response status and key observable behavior.
4. Verify that every image belongs to the local authorized lab.
5. Do not upload raw ZAP sessions, unredacted HTTP archives or database dumps.

Screenshots already shared during development are **not automatically in this repository**. This README records the plan, not evidence of upload.
