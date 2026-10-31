# F-06 — Mass Assignment During Public Registration (Administrator Role)

| Field | Value |
|---|---|
| Status | **Confirmed client-controlled administrator role assignment; administrative authorization comparison pending** |
| Severity | **High, provisional (qualitative)** |
| Confidence | High for role assignment; moderate for downstream authorization impact |
| Date tested | 31 October 2026 |
| Target | OWASP Juice Shop, local Docker deployment at `http://localhost:3000` |
| Entry point | `POST /api/Users/` |
| Classification | CWE-915 — Improperly Controlled Modification of Dynamically-Determined Object Attributes (mass assignment) |
| Remediation retest | Not performed |

## Executive summary

The public user-registration endpoint accepted a caller-controlled `role` property and created an account with `"role":"admin"`. The HTTP response explicitly confirmed that role. A subsequent browser session logged in as the newly created test account and rendered the application's **Administration** interface, including registered-user listings and customer feedback management UI.

**Scope of proof:** The API-level creation of an administrator-role user is confirmed. The Administration page was visible after logging into the new account, but a same-path comparison against an ordinary customer session has **not yet been captured**. Do not use page access alone as conclusive evidence of effective privilege checks: an independently vulnerable route might expose the interface. No administrator-only changes were performed.

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
9. **Pending comparative control:** Log into an ordinary `role:customer` test account and try the exact same route. Compare authenticated identity, rendered page and any relevant data-loading API statuses without modifying user data.

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
- No account modifications, user-record viewing via management icons, feedback deletions or settings changes were attempted.

## Security impact

The registration API allows a public client to assign itself an administrator role instead of enforcing server-controlled role selection. The role assignment crosses a privileged trust boundary and may enable administrator-only features, contingent on the application's actual authorization checks.

**Confirmed:** Unauthorized `admin` role persisted in the created account response; administration interface rendered afterward.

**Not yet confirmed independently:** A same-route normal-customer denial, whether sensitive administrative APIs enforce the assigned role, and any actual privileged state-changing operation. No data was modified beyond creation of the disposable test account.

## Severity rationale

**High, provisional.** Untrusted public clients can self-assign an administrator role, with potential significant authorization impact. The rating should be revisited after the customer-session comparison and review of affected server-side permission checks. Do not claim full account control, data alteration or administrative takeover has been demonstrated.

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
| F06-01 | Baseline registration `POST /api/Users/` returned 201 and `role:customer` | Screenshot reviewed in conversation; not uploaded to repository |
| F06-02 | Modified registration with `role:admin` returned 201 and `role:admin` | Screenshot reviewed in conversation; original contained disposable password and security answer; **do not upload unredacted** |
| F06-03 | Administration interface rendered after logging into new test account | Screenshot reviewed in conversation; not yet uploaded; usernames/emails should be redacted before public publication |
| F06-04 | Normal customer session attempting the same administration route | **Pending** |

## References

- [MITRE CWE-915 — Mass Assignment](https://cwe.mitre.org/data/definitions/915.html)
- [OWASP API Security — Broken Object Property Level Authorization](https://owasp.org/API-Security/editions/2023/en/0xa3-broken-object-property-level-authorization/)
- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
