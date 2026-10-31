# Assessment Methodology

This is an educational web application penetration testing assessment aligned to the [OWASP Web Security Testing Guide (WSTG)](https://owasp.org/www-project-web-security-testing-guide/).

## Phase 1 — Scope and environment

- Document assessment authorization, host, port, limitations and exclusions.
- Run the training application locally in Docker, bound to `127.0.0.1`.
- Record tool versions: Windows 11, Docker Desktop, ZAP 2.17.0 (from the UI).
- Docker CLI 29.9.2 (build a82e528) and the container's local image ID were recorded from read-only PowerShell inspection on 31 October 2026. See [Environment Fingerprint](../scope/environment-fingerprint.md).
- The running application reported **Juice Shop 20.3.0**. `docker image inspect ...RepoDigests` reported `bkimminich/juice-shop@sha256:42843723a7bf4246aa5d63c324978b17ef296ac78434aeddd34c427bfb5853f4` on 31 October 2026. The registry-qualified digest was captured separately from the local image ID (the hash strings coincidentally match in the observed output). A clean-machine reproduction has not yet been tested.
- Configure a ZAP context restricted to the target and enable Protected Mode.

## Phase 2 — Reconnaissance and passive analysis

- Explore key workflows with the ZAP-launched browser.
- Map the home page, search, product, login, registration and basket flows.
- Inspect the Sites tree, HTTP History, Requests, Responses and Alerts.
- Review response security headers, cookies, request methods, parameters, and authentication data.
- Keep third-party browser services outside the target context.

## Phase 3 — Manual vulnerability validation

For each candidate issue:
1. Define the hypothesis, affected request and expected secure behavior.
2. Capture a **baseline or negative control** whenever possible.
3. Change one relevant input or access condition at a time.
4. Compare HTTP status **and** response body, session state, and user-visible outcomes.
5. Record a finding only if the behavior and security impact are demonstrated.
6. Stop after a sufficient proof of concept; avoid unnecessary data exposure.

Priority areas:
- Injection (especially SQL injection)
- Authentication and session handling
- Authorization and insecure direct object references (IDOR)
- Cross-origin policy and security header configuration
- Cross-site scripting where a safe demonstration is possible

## Phase 4 — Risk, reporting, and evidence

Each confirmed finding includes:
- Unique ID, title, affected endpoint and severity rationale
- Preconditions and exact reproduction steps
- Expected versus actual result
- Impact limited to what was observed
- Sanitized request/response evidence and screenshots
- CWE/WSTG references and remediation recommendations
- Re-test status: **not tested**, **passed**, or **failed**

Severity is a **qualitative estimate** until an applicable CVSS vector is calculated. ZAP's scanner risk and confidence are reported separately from final assessment judgments.

## Phase 5 — Remediation and retesting

- Propose fixes based on root cause rather than only blocking one payload.
- Retest in a patched version if a locally controlled fix is available.
- Never claim a fix has been implemented or verified solely from a recommendation.
- Produce a final report only after reviewing all findings and evidence.

## Current validated activity (30–31 October 2026)

- Passive reconnaissance captured HTTP requests from `localhost:3000`.
- CORS header check: `Access-Control-Allow-Origin: *` observed on public homepage and product search API; exploitability not established.
- Login test: an SQL injection payload in the login email field was followed by a successful administrator authentication response and Juice Shop's “Login Admin” completion banner. **A separate negative-control response was observed on 31 October 2026 (HTTP 401), but the sanitized screenshot has not yet been uploaded.**

- Access-control test: Account A basket ID 6 and Account B basket ID 7 were observed. A request originating from Account B's authenticated capture was changed to GET basket 6 and returned HTTP 200 with Account A's basket contents. See [F-02](../findings/F02-idor-basket-access.md). The session token must be redacted in publication evidence.

## References

- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [OWASP Authentication Bypass Testing — WSTG-ATHN-04](https://wstg.owasp.org/latest/4-Web_Application_Security_Testing/04-Authentication/04-Bypassing_Authentication_Schema/)
- [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)
- [ZAP guide to testing Juice Shop](https://www.zaproxy.org/docs/testapps/juiceshop/)
