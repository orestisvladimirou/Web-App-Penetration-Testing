# Web Application Penetration Test — Interim Report

**Assessment target:** OWASP Juice Shop, localhost-only Docker instance  
**Assessment type:** Authorized educational security assessment  
**Assessment dates:** 30–31 October 2026  
**Report status:** DRAFT — investigation and evidence review are ongoing

## Executive summary

An initial manual assessment of a deliberately vulnerable training web application was conducted using Docker, browser exploration, OWASP ZAP passive capture and focused HTTP validation.

**Three behaviors were confirmed in the training lab:** SQL injection bypassed administrator authentication (High), an authenticated user accessed another user's basket (Medium), and untrusted search input was rendered as active content in a DOM XSS proof of concept (final severity pending evaluation of cross-user triggering). Risk ratings are qualitative and provisional.

A further finding, F-04, records a browsable /ftp/ directory and an acquisitions document explicitly marked confidential. A credential-free PowerShell test has since confirmed HTTP 200 and confidential-marked content for `acquisitions.md`. The `/ftp/` index returned HTTP 200 with a curl error 18 incomplete-transfer warning; the browser screenshot separately shows the listing. One additional CORS configuration observation was recorded. Its security impact has **not** been demonstrated, so it is not counted as an exploitable finding.

## Results summary

| ID | Title | Assessment | Status |
|---|---|---|---|
| F-01 | SQL injection — administrator authentication bypass | High (qualitative) | Confirmed |
| F-02 | Broken access control — cross-account basket disclosure | Medium (qualitative) | Confirmed |
| F-03 | DOM XSS in product search | Severity under review; proof of concept confirmed | Confirmed local behavior |
| F-04 | Exposed file index and confidential-marked acquisitions document | Medium (provisional) | Unauthenticated document retrieval confirmed; Medium provisional |
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

### F-03 — DOM XSS via product search

The tester submitted a harmless iframe/JavaScript-alert payload into the product search box. A frame appeared in the search results, and Juice Shop displayed its successful **DOM XSS** challenge banner. A separate Incognito session subsequently loaded the crafted `/#/search?q=...` URL and rendered the injected iframe without manual input, strengthening the link-triggered HTML-injection evidence. The tester dismissed the welcome dialog, refreshed the Incognito page and observed **no `xss` JavaScript alert**; cross-user script execution and practical confidentiality impact remain unverified. Final severity remains under review. See [F-03](../findings/F03-dom-based-xss-search.md).

**Recommended fix:** use safe text DOM insertion rather than untrusted active HTML; avoid bypassing framework sanitization; add front-end regression tests.

### F-04 — Exposed FTP file index and confidential document

A terms-of-use link exposed the /ftp/ path. Its directory listing displayed filenames of internal-looking backups and a document titled acquisitions.md; the acquisitions document was displayed in the browser and explicitly marked confidential. The disclosure was observed in the controlled local lab. A **credential-free PowerShell request** returned HTTP 200 for `/ftp/acquisitions.md`, and a separate request printed the line `This document is confidential! Do not distribute!`. A cookie-free directory request returned HTTP 200 but also curl error 18 (incomplete transfer); directory-listing contents were established separately in the browser screenshots. No backup or password database contents were accessed. The training document uses placeholder content, so no real financial or corporate consequences are asserted. See [F-04](../findings/F04-exposed-directory-confidential-document.md).

**Recommended fix:** remove sensitive and backup artifacts from public paths, disable unneeded index listings, and enforce server-side authorization for restricted documents.

### O-01 — Permissive CORS response policy

Both the public homepage and product-search API returned `Access-Control-Allow-Origin: *`; the assessed responses did not advertise credentialed cross-origin access. Exploitable impact not established. See [O-01](../observations/O01-cors-configuration.md).

## Limitations and next actions

- F-03 screenshot evidence has been uploaded and linked in the technical finding; visually review all evidence for residual secrets before publishing.
- Upload a sanitized screenshot of the observed failed-login negative control (401 Unauthorized) for the SQL injection finding.
- Save sanitized evidence tying the IDOR Requester session to Account B without exposing authentication tokens.
- Documented F-03 negative follow-up: no JavaScript alert appeared in Incognito after dismissing the welcome dialog and refreshing; only iframe rendering via the URL was observed. Avoid claiming cross-session script execution without proof.
- Save sanitized /ftp/ directory and acquisitions screenshots plus the PowerShell screenshot documenting the cookie-free check; the directory index request included a curl error 18 despite HTTP 200.
- Record the exact target image digest and application version.
- Consider additional authorized access-control and injection scenarios, avoiding destructive operations.
- Apply a consistent severity framework and calculate CVSS vectors when appropriate.
- Retest remediation only if a patched target is available.
- Convert this draft to a final report after completing and reviewing the assessment.

## Disclaimer

This work concerns a purposely insecure application deployed locally for educational testing. It is not a penetration test of the public OWASP Juice Shop demo or any third-party system.
