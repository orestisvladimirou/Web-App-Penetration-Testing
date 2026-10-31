# F-06 — Mass Assignment During Public Registration (Administrator Role)

| Field | Value |
|---|---|
| Status | **Confirmed administrator-role mass assignment and comparative UI access control bypass** |
| Severity | **High, provisional (qualitative)** |
| Confidence | High for role assignment and differential administration-route access; backend authorization effects not independently tested |
| Date tested | 31 October 2026 |
| Target | OWASP Juice Shop, local Docker deployment at `http://localhost:3000` |
| Entry point | `POST /api/Users/` |
| Classification | CWE-915 — Improperly Controlled Modification of Dynamically-Determined Object Attributes (mass assignment) |
| Remediation retest | Not performed |

## Executive summary

The public user-registration endpoint accepted a caller-controlled `role` property and created an account with `"role":"admin"`. The HTTP response explicitly confirmed that role. A subsequent browser session logged in as the newly created test account and rendered the application's **Administration** interface, including registered-user listings and customer feedback management UI.

**Scope of proof:** The API-level creation of an administrator-role user is confirmed. The Administration page was visible after logging into the newly created account, while a normal customer account was shown the browser message **“403 — You are not allowed to access this page!”** at the same route. This confirms a meaningful difference in UI authorization. The 403 was displayed by the app (not verified as an HTTP status), and privileged server-side state changes were not tested.

## Controlled test setup

Two disposable registration cases in the intentionally vulnerable local lab:

- Normal baseline: `lab-registration-baseline@example.invalid`
- Modified registration: `lab-admin-test@example.invalid`

Both were created from ZAP Requester using the captured `POST /api/Users/` registration flow, with the modified test using a previously unused email address.

## Reproduction procedure

1. Register the baseline account through Juice Shop's standard registration form.
2. In **ZAP → History**, identify the registration request (`POST /api/Users/`; example request ID 602).
3. Open the captured request in ZAP **Requester**.
4. Modify the JSON email to a new, unused disposable lab address.
5. Add `"role": "admin"` to the registration body, preserving the other required properties.
6. Submit only to the local Juice Shop endpoint.
7. Inspect the HTTP response body and registration status.
8. Sign into the newly created test account and navigate to `/#/administration`. Observe the interface without clicking destructive or modifying controls.
9. **Completed comparative control (31 October 2026):** Log into the normal customer account `lab-user-b@example.invalid` and visit the exact same `/#/administration` route. The browser displays **403 — You are not allowed to access this page!**, while the Account menu shows that the ordinary customer is logged in. No modifying actions were taken.

### Sanitized request excerpt

```http
POST /api/Users/ HTTP/1.1
Host: localhost:3000
Content-Type: application/json

{"email":"lab-admin-test@example.invalid","role":"admin","password":"[REDACTED]","passwordRepeat":"[REDACTED]","securityQuestion":{"id":2},"securityAnswer":"[REDACTED]"}
```

The object above is a **sanitized illustrative excerpt**, not a byte-for-byte capture. The actual request contained additional security-question metadata and was submitted once using the new email.

### Observed server response

- **HTTP 201 Created**
- Response JSON included `"status":"success"`, the new account email, and `"role":"admin"`.
- The normal baseline registration had also returned 201 but with `"role":"customer"`.
- The browser opened the Administration interface when logged in under the account created with the injected role. The screenshot shows a registered-user table, customer feedback entries, and UI management controls.
- Normal customer comparison: the account menu identified `lab-user-b@example.invalid`; visiting the same Administration route produced the UI message **403 — You are not allowed to access this page!**.
- No account modifications, user-record viewing via management icons, feedback deletions or settings changes were attempted.

## Security impact

The registration API allows a public client to assign itself an administrator role instead of enforcing server-controlled role selection. The role assignment crosses a privileged trust boundary and may enable administrator-only features, contingent on the application's actual authorization checks.

**Confirmed:** The untrusted request created an `admin`-role account that could render the Administration page, whereas the ordinary customer was denied at the same route by the application.

**Not independently confirmed:** The underlying API's HTTP denial status, enforcement of permissions on individual administrator API endpoints, or any privileged state-changing operation. No data was modified beyond creation of the disposable test account.

## Severity rationale

**High, provisional.** Untrusted public clients can self-assign an administrator role and thereby reach an administrative interface that rejects normal customers. Further risk analysis may assess the underlying API authorization and achievable state-changing impact; no such changes were attempted. Do not claim total application control or destructive capabilities.

## Root cause hypothesis

The registration handler appears to accept and persist caller-supplied authorization attributes rather than applying a server-side allowlist of legitimate public-registration fields and forcing the ordinary customer role. Source code was not examined.

## Remediation recommendations

1. **Explicitly allowlist** fields accepted by the public registration API.
2. Derive new user role **server-side**, with `customer` as the enforced registration role.
3. Restrict role changes to separate authorized administrator-controlled functions.
4. Apply server-side authorization checks to privileged resources, not only UI route protection.
5. Add regression tests for `role` and other restricted attributes in registration requests and verify resulting account permissions.
6. Retest registration and administrator-only routes after applying a fix.

## Evidence register

| Evidence | Observation | Status |
|---|---|---|
| F06-01 | Baseline registration `POST /api/Users/` returned 201 and `role:customer` | [Sanitized baseline screenshot](../evidence/F06-baseline-registration-redacted.png) — uploaded and verified |
| F06-02 | Modified registration with `role:admin` returned 201 and `role:admin` | [Sanitized source screenshot](../evidence/F06-registration-role-admin-redacted.png) — password, security answer and cookies masked; uploaded and verified |
| F06-03 | Administration interface rendered after logging into new test account | [Cropped screenshot](../evidence/F06-administration-screen-redacted.png) — showing page title and administrative section headings without user records; uploaded and verified |
| F06-04 | Normal customer `lab-user-b@example.invalid` visiting the same `/#/administration` route received visible 403 denial | [Cropped source screenshot](../evidence/F06-customer-route-control-redacted.png) — uploaded and verified |

See also the [sanitized comparative evidence record](../evidence/F06-privilege-comparison-observations.md), transcribed from observed screenshots.

## References

- [MITRE CWE-915 — Mass Assignment](https://cwe.mitre.org/data/definitions/915.html)
- [OWASP API Security — Broken Object Property Level Authorization](https://owasp.org/API-Security/editions/2023/en/0xa3-broken-object-property-level-authorization/)
- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
