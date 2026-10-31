# F-04 — Exposed File Directory and Confidential Document Disclosure

| Field | Value |
|---|---|
| Assessment status | **Confirmed unauthenticated access to confidential-marked document; directory listing displayed in browser** |
| Severity | **Medium (provisional, qualitative)** |
| Date observed | 31 October 2026 |
| Target | Local OWASP Juice Shop at `http://localhost:3000` |
| Affected paths | `/ftp/`, `/ftp/acquisitions.md` |
| Category | Information disclosure / exposed sensitive files |
| Reference | OWASP WSTG-CONF-04 — Review Old Backup and Unreferenced Files for Sensitive Information |
| Remediation retest | Not performed |

## Summary

While inspecting the application's **About Us → terms of use** link, the tester identified a public-facing document at `/ftp/legal.md`. Removing the filename from the URL opened a browsable index at `/ftp/`. The index included a document named `acquisitions.md`, which the browser displayed with the heading **Planned Acquisitions** and the explicit marking **“This document is confidential! Do not distribute!”**

The document contains placeholder text characteristic of the **intentionally vulnerable training application**; no real corporate transaction, stock-market consequence, or production data breach is claimed.

The browser screenshots establish visibility of the directory index and the document. On 31 October 2026, a separate PowerShell `curl.exe` test with **no authentication header or cookies** received HTTP 200 for the document and printed the confidential-marked text. Thus **unauthenticated document retrieval is confirmed**. A cookie-free request to the directory index also returned HTTP 200, but emitted `curl: (18) end of response with 59 bytes missing`, so that response transfer was incomplete; the earlier browser screenshot provides the directory-listing evidence.

## Reproduction within authorized local lab

1. Visit Juice Shop's **About Us** page and open its terms link, leading to `http://localhost:3000/ftp/legal.md`.
2. Request `http://localhost:3000/ftp/` and observe the directory listing.
3. Select `acquisitions.md` from the index, or navigate directly to `http://localhost:3000/ftp/acquisitions.md`.
4. Observe the **Planned Acquisitions** document and explicit confidential marking.
5. For independent authentication-control verification, request both URLs from a new PowerShell session using `curl.exe` **without any cookies or authorization header**:

   ```powershell
   curl.exe -sS -D - -o NUL "http://localhost:3000/ftp/"
   curl.exe -sS -D - -o NUL "http://localhost:3000/ftp/acquisitions.md"
   ```

   **Actual results (31 October 2026):**
   - `GET /ftp/` — HTTP 200; curl reported error 18 (59 bytes missing), so the body transfer was incomplete.
   - `GET /ftp/acquisitions.md` — HTTP 200 with no curl error shown.
   - A separate cookie-free document GET piped to `Select-String "confidential"` returned: `> This document is confidential! Do not distribute!`.
   - These commands sent no cookie or authorization header.

## Expected behavior

- Deliberately public terms can be served from an explicitly approved path.
- Directory indexing should not reveal unapproved internal files.
- Confidential or internal files should not be stored in a public static directory, or should require appropriate server-side authorization if legitimately served.

## Observed behavior

- Browser navigation to `/ftp/legal.md` displayed the legal information page.
- Browser navigation to `/ftp/` displayed multiple filenames, including `acquisitions.md`, `package.json.bak`, `package-lock.json.bak`, `coupons_2013.md.bak`, and `incident-support.kdbx`.
- Browser navigation to `/ftp/acquisitions.md` displayed a document explicitly marked confidential.

**Not established:** What any backup or `.kdbx` file contains; whether these other files are downloadable without authentication; or any real-world business impact. **Unauthenticated document retrieval is established**, independently of the browser session. No additional sensitive files were opened or downloaded in this test.

## Security impact and risk rationale

Directory listing simplifies discovery of files not intended for general users, and direct access to confidential-marked documents demonstrates a missing or ineffective access boundary for that content. In a production system, similar exposure could disclose business planning, credentials, source code, configuration or backups if such information is stored there.

**Medium, provisional:** A document designated confidential was retrieved through a credential-free HTTP request, and other potentially sensitive filenames were enumerated. The environment is synthetic, so the data has no verified real-world business sensitivity. Reassess if demonstrating a comparable issue in an authorized production-like setting.

## Likely cause

The application exposes files in a static `/ftp/` location and permits listing of its contents. Server-side controls are not preventing browser access to the displayed confidential-marked file. This was inferred from black-box behavior; the implementation was not inspected.

## Recommendations

1. Remove confidential files, backup artifacts, credential stores and other internal assets from web-accessible directories.
2. Disable directory indexing unless an explicit business requirement justifies it.
3. Explicitly allowlist intended public documents and require server-side authorization for restricted files.
4. Block public access to development and backup extensions and scan deployment artifacts for sensitive files.
5. Add tests verifying both unauthorized directory enumeration and direct file download are denied.
6. Retest the affected paths after remediation, including a cookie-free request.

## Evidence register

| Evidence | Observation | Status |
|---|---|---|
| F04-01 | Terms document displayed at `/ftp/legal.md` | Screenshot shared in conversation; not yet uploaded to GitHub |
| F04-02 | `/ftp/` directory listing and internal-looking filenames | Screenshot shared in conversation; not yet uploaded to GitHub |
| F04-03 | `/ftp/acquisitions.md` confidential-marked document displayed | Screenshot shared in conversation; not yet uploaded to GitHub |
| F04-04 | Credential-free PowerShell checks: document HTTP 200 plus confidential text; directory HTTP 200 with curl error 18 (incomplete response) | **Observed and reviewed in conversation on 31 October 2026; screenshot not yet uploaded to GitHub** |

## References

- [OWASP WSTG-CONF-04 — Review Old Backup and Unreferenced Files for Sensitive Information](https://wstg.owasp.org/v4.2/4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management_Testing/04-Review_Old_Backup_and_Unreferenced_Files_for_Sensitive_Information/)
- [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)
