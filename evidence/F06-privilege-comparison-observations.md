# F-06 — Administrator-Role Registration and Customer-Access Comparison

**Evidence type:** Redacted, manually transcribed observations from screenshots of the local Juice Shop lab, taken 31 October 2026. This is not a byte-for-byte HTTP capture. No credentials, authentication headers, security answers, or tokens are reproduced.

## 1. Baseline registration (ordinary customer)

- Tool: ZAP 2.17.0, Requester / HTTP History.
- Route: `POST http://localhost:3000/api/Users/`
- Disposable account: `lab-registration-baseline@example.invalid`.
- HTTP response: `201 Created`.
- Returned account attribute: `"role":"customer"`.
- Source screenshot was reviewed in the conversation; source PNG is **not** committed here.

## 2. Modified public registration (elevated role)

- Route: `POST http://localhost:3000/api/Users/`.
- Disposable account: `lab-admin-test@example.invalid`.
- Client-supplied attribute: `"role":"admin"`.
- HTTP response: `201 Created`.
- Response body explicitly reported `"role":"admin"` for the created account.
- Registration was successful without prior administrator authorization.
- Original screenshot was reviewed in the conversation but contained a disposable password and security answer; **the unredacted original must never be uploaded**.

Sanitized illustrative request (omits fields and values; not a complete capture):

```http
POST /api/Users/ HTTP/1.1
Host: localhost:3000
Content-Type: application/json

{"email":"lab-admin-test@example.invalid","role":"admin","password":"[REDACTED]"}
```

Sanitized response excerpt, transcribed from screenshot:

```json
{"status":"success","data":{"email":"lab-admin-test@example.invalid","role":"admin"}}
```

Other response properties are omitted.

## 3. Administrator account route check

- Browser authenticated as the newly registered `lab-admin-test@example.invalid` account.
- URL: `http://localhost:3000/#/administration`.
- Observed: Administration page loaded, showing **Registered Users**, **Customer Feedback**, and **Settings** sections.
- No administrator-only state-changing operation was performed.
- Original screenshot reviewed in the conversation; not yet committed as a sanitized PNG.

## 4. Ordinary customer control

- Browser authenticated as `lab-user-b@example.invalid` (normal customer account).
- Exact URL: `http://localhost:3000/#/administration`.
- Observed browser message: **403 — You are not allowed to access this page!**
- The Account menu in the screenshot displayed the normal customer's email, confirming the tested account identity.
- The "403" here is the **application's displayed message**, not an independently verified HTTP response status code.
- Original screenshot was reviewed in the conversation; not yet committed as an image.

## Assessment

The public registration endpoint accepted a privileged `admin` role from an untrusted client. The resulting administrator account could load the Administration interface; an ordinary customer visiting the same route received a denial. Together, this is strong evidence of **mass assignment leading to privilege escalation** in the controlled lab.

**Boundaries:** The assessment did not inspect server authorization source code, independently validate role-based administrative API permissions, or make administrator-only state changes. A final publication pass should add original sanitized screenshots and verify the account identity and sensitive-data redactions.

Related technical report: [F-06](../findings/F06-mass-assignment-admin-registration.md).
