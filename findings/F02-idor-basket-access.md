# F-02 — Broken Access Control: Cross-Account Basket Disclosure (IDOR)

| Field | Value |
|---|---|
| Status | **Confirmed in local training lab** |
| Assessed severity | **Medium (qualitative; no CVSS vector yet)** |
| Confidence | High, based on successive authenticated-session and response observations; token attribution should be documented in sanitized evidence |
| Date observed | 31 October 2026 |
| Target | `http://localhost:3000` (local OWASP Juice Shop) |
| Endpoint | `GET /rest/basket/{id}` |
| Classification | CWE-639 (Authorization Bypass Through User-Controlled Key); CWE-862 (Missing Authorization) |
| Relevant testing area | OWASP WSTG authorization testing, including IDOR |
| Retest after remediation | Not performed |

## Summary

A normal user account was able to submit a request for a different user's basket by changing the numeric basket ID in a previously captured request. The server returned a successful JSON response with the other user's basket and product information.

This is an **object-level authorization / insecure direct object reference (IDOR)** weakness in a deliberately vulnerable training application. The test demonstrated unauthorized **read access**, not modification or deletion.

## Preconditions and lab data

Two separate, disposable accounts were registered in the local lab:

- **Account A:** `lab-user-a@example.invalid` — basket ID **6** (response owner/UserId **25**).
- **Account B:** `lab-user-b@example.invalid` — basket ID **7** (different owner; observed in prior baseline).

Account A had Apple Juice and Banana Juice in its basket. Account B had a different basket. Both identifiers were observed through the local application and OWASP ZAP traffic capture.

## Reproduction in authorized local environment

1. Log in as Account A, add two products, and inspect `GET /rest/basket/6` to establish the reference basket.
2. Log out of Account A, then log in as Account B and inspect Account B's normal `GET /rest/basket/7` request.
3. In ZAP's **Requester** tool, open the request captured during Account B's authenticated session.
4. Change **only** the basket ID from `7` to `6`. Keep Account B's `Authorization: Bearer [REDACTED]` header and other relevant request properties.
5. Send the request to the local application and inspect the returned JSON.
6. Stop after confirming read access; do not attempt edits or deletion of another basket.

### Illustrative sanitized request

```http
GET /rest/basket/6 HTTP/1.1
Host: localhost:3000
Authorization: Bearer [REDACTED]
```

### Expected secure behavior

The API should check that the authenticated principal owns basket ID 6. If Account B requests Account A's basket, the application should deny access, ordinarily with **403 Forbidden** or **404 Not Found**. The endpoint should not return another account's basket details.

### Observed behavior

- ZAP Requester sent `GET http://localhost:3000/rest/basket/6`.
- The request included a Bearer authorization header from the Account B request being modified.
- The server returned **HTTP 200 OK** with `application/json`.
- The response contained basket `id: 6`, `UserId: 25`, and products matching Account A's basket, including **Apple Juice** and **Banana Juice**.

The successful response showing Account A's identifiable basket contents is stronger evidence than the status code alone. A redacted screenshot of the modified request has been uploaded; image pixels have not been independently re-inspected, so verify the redaction before making the repository public.

## Impact

An authenticated user can read shopping basket information belonging to another account by guessing or changing a predictable object identifier. The test establishes a confidentiality and tenant/user-isolation failure. It does **not** establish unauthorized editing of baskets, access to payment credentials, or wider administrative privileges.

**Severity rationale:** Medium provisionally because the demonstrated outcome is unauthorized read access to shopping basket contents, without evidence of more sensitive data or modification. A final severity should be revisited after determining the data sensitivity and reachable attack surface.

## Likely root cause

The API appears to look up a basket using a client-supplied numeric ID without adequately enforcing a server-side ownership check against the authenticated user. This behavior is observed from responses; source code was not inspected to confirm the exact implementation.

## Remediation recommendations

1. Enforce **server-side object-level authorization** for every basket retrieval: compare the basket owner with the authenticated user or use a query scoped to the authenticated principal.
2. Avoid relying on guess-resistant or sequential IDs as the primary authorization mechanism.
3. Return a denial (403 or non-disclosing 404) when the requested basket belongs to another user.
4. Apply the same ownership controls to basket modification and deletion endpoints, even though those paths were not tested here.
5. Add automated authorization tests for cross-account access, including positive owner access and negative non-owner access.
6. Retest with the same two-account workflow after implementing the fix.

## Evidence register

| Evidence | Observation | Publication status |
|---|---|---|
| F02-01 | Account A baseline: `GET /rest/basket/6`, products Apple Juice/Banana Juice | Captured in conversation; sanitized screenshot not uploaded |
| F02-02 | Account B baseline: `GET /rest/basket/7`, normal account basket | Captured in conversation; sanitized screenshot not uploaded |
| F02-03 | ZAP Requester: Bearer-authenticated `GET /rest/basket/6` returned 200 with Account A's products | [Redacted screenshot](../evidence/F02-cross-account-request-redacted.png) uploaded to GitHub on 31 October 2026; recheck token redaction before publication |
| F02-04 | Independently saved proof that the Requester token belongs to Account B | Still to be documented without disclosing the token |

## Assessment limitations

- The screenshot alone does not independently link the Requester token to Account B without the preceding user workflow; archive sanitized session attribution alongside the request for employer-facing reproducibility.
- No state-changing operations on Account A's basket were attempted.
- No patched build was tested; recommendations are **not verified fixes**.
- Exact Juice Shop image version and digest remain to be recorded.
- The F02-03 screenshot is committed in `evidence/`; baseline screenshots and independent Account B session attribution have not yet been uploaded.

## References

- [OWASP WSTG — Authorization Testing](https://owasp.org/www-project-web-security-testing-guide/)
- [OWASP Insecure Direct Object Reference Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html)
- [OWASP API1:2023 — Broken Object Level Authorization](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/)
- [MITRE CWE-639](https://cwe.mitre.org/data/definitions/639.html)
