# F-01 — SQL Injection Leading to Administrator Authentication Bypass

| Field | Value |
|---|---|
| Status | **Confirmed in local training lab** |
| Assessed severity | **High (qualitative; no CVSS vector yet)** |
| Confidence | High — authentication and admin identity observed |
| Date observed | 30 October 2026 |
| Target | `http://localhost:3000` (local OWASP Juice Shop) |
| Endpoint | `POST /rest/user/login` |
| Category | CWE-89; OWASP WSTG-INPV-05; WSTG-ATHN-04 |
| Retest after remediation | Not performed |

## Summary

A crafted value in the authentication email field altered the application's intended login behavior, enabling login to the administrator account **without the administrator's password**.

This was observed in a purpose-built, intentionally vulnerable **local** training application. It does not imply vulnerability in another environment.

## Reproduction in the authorized lab

1. Launch the local Juice Shop container and the ZAP-proxied browser.
2. Open **Account → Login**.
3. Submit the following lab-only test values:

   **Email:** `' OR 1=1--`  
   **Password:** `WrongPassword123!`

4. Inspect the corresponding `POST /rest/user/login` request in ZAP.
5. Inspect the response body without exposing the returned token.
6. Check the authenticated identity in Juice Shop.

### Expected secure behavior

The application must reject credentials that do not correspond to an existing account and its correct password. SQL syntax should be treated as literal user input rather than query structure.

### Observed behavior

- The POST login request returned **HTTP 200 OK**.
- The response included an authentication token (value redacted and not stored in this repository).
- Juice Shop displayed the authenticated account `admin@juice-sh.op`.
- Juice Shop showed the challenge-completion banner **“Login Admin”**.

The combination of the response and visible account identity is evidence of successful administrator authentication bypass. HTTP 200 alone would not be sufficient proof.

## Likely root cause

The response is consistent with a server-side query that incorporates untrusted login input into SQL without safe parameterization. Source-level confirmation of the exact query construction is **not part of this assessment**.

## Security impact

An unauthenticated actor can impersonate the application administrator without knowing the password, crossing the application's authentication trust boundary. Potential administrator-only actions were **not** explored or used.

## Remediation recommendations

1. Use parameterized SQL statements (or ORM APIs with correctly bound parameters) for every database query.
2. Fetch the user by an exact account identifier; separately compare the submitted password using a secure password-hash verification function.
3. Prevent dynamic SQL string concatenation of credentials.
4. Apply least-privilege database permissions and add regression tests for malformed authentication inputs.
5. Re-run the same baseline and injection tests against a patched build.

## Evidence register

| Evidence | Source | Publication status |
|---|---|---|
| F01-01 | ZAP POST request with payload | Captured in conversation; sanitized image not uploaded |
| F01-02 | ZAP HTTP 200 JSON response containing auth token | Captured; **token must be redacted** before upload |
| F01-03 | Browser showing `admin@juice-sh.op` and “Login Admin” challenge | Captured in conversation; sanitized image not uploaded |
| F01-04 | Invalid-credential negative control | Pending archival / repeat test |

**Evidence hygiene:** No bearer tokens, JWTs, session cookies, sensitive headers or password values from live accounts should be published. Avoid uploading raw ZAP session databases or HTTP archive files.

## Assessment limitations

- Exact Juice Shop version/image digest has not yet been recorded.
- The standalone invalid-credential control response is not yet saved.
- No patched build was assessed; remediation is recommended, **not implemented or verified**.

## References

- [OWASP WSTG — Bypassing Authentication Schema](https://wstg.owasp.org/latest/4-Web_Application_Security_Testing/04-Authentication/04-Bypassing_Authentication_Schema/)
- [OWASP SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
- [MITRE CWE-89](https://cwe.mitre.org/data/definitions/89.html)
