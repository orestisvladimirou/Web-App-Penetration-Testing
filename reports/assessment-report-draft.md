# Web Application Penetration Test — Interim Report

**Assessment target:** OWASP Juice Shop, localhost-only Docker instance  
**Assessment type:** Authorized educational security assessment  
**Assessment dates:** 30–31 October 2026  
**Report status:** DRAFT — investigation and evidence review are ongoing

## Executive summary

An initial manual assessment of a deliberately vulnerable training web application was conducted using Docker, browser exploration, OWASP ZAP passive capture and focused HTTP validation.

**Two findings were confirmed:** SQL injection in the authentication flow allowed login as administrator without the password (High), and an authenticated user could read another user's basket by replacing a numeric object identifier (Medium). Both severities are qualitative.

One additional CORS configuration observation was recorded. Its security impact has **not** been demonstrated, so it is not counted as an exploitable finding.

## Results summary

| ID | Title | Assessment | Status |
|---|---|---|---|
| F-01 | SQL injection — administrator authentication bypass | High (qualitative) | Confirmed |
| F-02 | Broken access control — cross-account basket disclosure | Medium (qualitative) | Confirmed |
| O-01 | Permissive CORS response policy | Observation, no demonstrated exploitation | Recorded |

## Scope

Only `http://localhost:3000` and paths belonging to the local Juice Shop instance are included. Browser services, third-party domains and other systems are excluded. See [Rules of Engagement](../scope/rules-of-engagement.md).

## Methodology

Followed an OWASP WSTG-inspired process:
1. Define scope and safeguards.
2. Map relevant UI flows and capture HTTP history.
3. Triage passive ZAP alerts.
4. Manually reproduce candidate weaknesses.
5. Record observed impact, confidence, remediation and evidence limitations.

See [Testing Methodology](../methodology/testing-methodology.md).

## Technical findings

### F-01 — Administrator authentication bypass

A SQL injection payload supplied as the email parameter in `POST /rest/user/login` caused a response containing an authentication token, and the browser showed administrator identity with the completed “Login Admin” challenge. The administrator password was not known or supplied.

**Recommended fix:** parameterized database queries with correct, independent password verification; regression tests; least-privilege access. See [F-01](../findings/F01-sql-injection-authentication-bypass.md).

### F-02 — Cross-account basket disclosure (IDOR)

After obtaining baseline basket IDs 6 (Account A) and 7 (Account B), the tester modified Account B's authenticated basket request to target `GET /rest/basket/6`. The local API returned HTTP 200 with Account A's basket ID, owner ID and Apple Juice / Banana Juice items. This demonstrated unauthorized read-only access; no basket modification was attempted.

**Recommended fix:** check basket ownership against the authenticated principal on every object read and modification, and introduce cross-account negative tests. See [F-02](../findings/F02-idor-basket-access.md). The visible bearer token in the source screenshot must be redacted before publication.

### O-01 — Permissive CORS response policy

Both the public homepage and product-search API returned `Access-Control-Allow-Origin: *`; the assessed responses did not advertise credentialed cross-origin access. Exploitable impact not established. See [O-01](../observations/O01-cors-configuration.md).

## Limitations and next actions

- Add **redacted** screenshots and corresponding evidence register references.
- Upload a sanitized screenshot of the observed failed-login negative control (401 Unauthorized) for the SQL injection finding.
- Save sanitized evidence tying the IDOR Requester session to Account B without exposing authentication tokens.
- Record the exact target image digest and application version.
- Consider additional authorized access-control and injection scenarios, avoiding destructive operations.
- Apply a consistent severity framework and calculate CVSS vectors when appropriate.
- Retest remediation only if a patched target is available.
- Convert this draft to a final report after completing and reviewing the assessment.

## Disclaimer

This work concerns a purposely insecure application deployed locally for educational testing. It is not a penetration test of the public OWASP Juice Shop demo or any third-party system.
