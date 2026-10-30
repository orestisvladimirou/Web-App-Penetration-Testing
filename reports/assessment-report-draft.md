# Web Application Penetration Test — Interim Report

**Assessment target:** OWASP Juice Shop, localhost-only Docker instance  
**Assessment type:** Authorized educational security assessment  
**Assessment date:** 30 October 2026  
**Report status:** DRAFT — investigation and evidence review are ongoing

## Executive summary

An initial manual assessment of a deliberately vulnerable training web application was conducted using Docker, browser exploration, OWASP ZAP passive capture and focused HTTP validation.

**One high-severity issue was confirmed:** SQL injection in the authentication flow allowed login as the administrator without knowing the password.

One additional CORS configuration observation was recorded. Its security impact has **not** been demonstrated, so it is not counted as an exploitable finding.

## Results summary

| ID | Title | Assessment | Status |
|---|---|---|---|
| F-01 | SQL injection — administrator authentication bypass | High (qualitative) | Confirmed |
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

### O-01 — Permissive CORS response policy

Both the public homepage and product-search API returned `Access-Control-Allow-Origin: *`; the assessed responses did not advertise credentialed cross-origin access. Exploitable impact not established. See [O-01](../observations/O01-cors-configuration.md).

## Limitations and next actions

- Add **redacted** screenshots and corresponding evidence register references.
- Preserve an invalid-credential negative control for the SQL injection finding.
- Record the exact target image digest and application version.
- Assess access control, authorization, and further injection cases in the local lab.
- Apply a consistent severity framework and calculate CVSS vectors when appropriate.
- Retest remediation only if a patched target is available.
- Convert this draft to a final report after completing and reviewing the assessment.

## Disclaimer

This work concerns a purposely insecure application deployed locally for educational testing. It is not a penetration test of the public OWASP Juice Shop demo or any third-party system.
