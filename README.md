# Web Application Vulnerability Assessment & Penetration Testing

A structured, **authorized educational penetration testing assessment** of [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/) running in a local Docker container. The project demonstrates web security reconnaissance, HTTP analysis, manual exploitation validation, reporting, and remediation planning using the [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/) as a reference.

> **Status: In progress.** Three vulnerability behaviors plus an exposed confidential-marked document and file listing have been demonstrated in the local training target. Severity is provisional; F-04's unauthenticated document retrieval has now been verified. This is **not** an assessment of a production website or a third-party service.

## Project at a glance

| Item | Details |
|---|---|
| Target | OWASP Juice Shop — `http://localhost:3000` |
| Environment | Windows 11, Docker Desktop, local container |
| Testing tool | OWASP ZAP 2.17.0, browser, PowerShell/`curl.exe` |
| Testing approach | Scoped passive reconnaissance and manually validated vulnerability testing |
| Scope | Local application only; external browser services explicitly excluded |
| Report status | Interim assessment draft; evidence review ongoing |

## Verified results

| ID | Finding | Outcome | Documentation |
|---|---|---|---|
| **F-01** | SQL injection leading to administrator authentication bypass | **Confirmed — High (qualitative)** | [Technical finding](findings/F01-sql-injection-authentication-bypass.md) |
| **F-02** | Cross-account basket disclosure through insecure direct object reference (IDOR) | **Confirmed — Medium (qualitative)** | [Technical finding](findings/F02-idor-basket-access.md) |
| **F-03** | DOM XSS through product search | **Confirmed local proof of concept — severity under review** | [Technical finding](findings/F03-dom-based-xss-search.md) |
| **F-04** | Exposed /ftp/ directory and confidential-marked acquisitions document | **Unauthenticated document access confirmed — Medium provisional** | [Technical finding](findings/F04-exposed-directory-confidential-document.md) |
| **O-01** | Permissive CORS header on public endpoints | **Configuration observation; exploitability not shown** | [Observation](observations/O01-cors-configuration.md) |

**F-01 evidence:** The login endpoint accepted an injection payload, issued an authentication response, and the browser displayed administrator identity with the Juice Shop “Login Admin” challenge completed. Sanitized evidence files have **not** yet been uploaded. A normal invalid-login control was observed returning HTTP 401 on 31 October 2026; its redacted screenshot remains to be uploaded.

## Methodology and documentation

- [Rules of engagement and scope](scope/rules-of-engagement.md)
- [Testing methodology and validation standards](methodology/testing-methodology.md)
- [Interim penetration test report](reports/assessment-report-draft.md)
- [Evidence handling guidance](evidence/README.md)

Findings distinguish **scanner alerts**, **configuration observations**, and **manually confirmed exploitable weaknesses**. Risk ratings are not equated with a tool's alert severity without validation.

## Reproduce the authorized lab

Install Docker Desktop, run the container bound to the local loopback interface, and open the URL shown below.

```powershell
docker pull bkimminich/juice-shop
docker run -d --name juice-shop-lab -p 127.0.0.1:3000:3000 bkimminich/juice-shop
docker ps --filter "name=juice-shop-lab"
```

Navigate to <http://localhost:3000>. If the container already exists but is stopped, use `docker start juice-shop-lab`.

For repeatable research, record the exact image version or digest before testing; the initial assessment used a pulled image whose digest was not yet saved.

**ZAP scope:** Include only `^http://localhost:3000(?:/.*)?$` in the dedicated context, put only that context in scope, and select **Protected Mode** before any active scanning. The assessment so far used manual browsing, passive capture, and a targeted local login test.

## Planned work

- [x] Establish a local Juice Shop environment
- [x] Capture and triage initial application traffic
- [x] Manually confirm an authentication bypass via SQL injection
- [x] Validate and downgrade the initial CORS scanner finding to an observation
- [ ] Save and redact screenshots, including a negative control
- [ ] Record version information and image digest
- [x] Demonstrate cross-account basket disclosure (IDOR) using two disposable lab accounts
- [x] Document the confirmed IDOR finding
- [ ] Capture redacted proof of Account B session attribution for the IDOR request
- [x] Demonstrate DOM XSS using a harmless search-field payload
- [x] Document F-03 and note missing cross-user trigger validation
- [x] Verify that a clean Incognito browser renders F-03's injected iframe when opening the crafted local URL
- [ ] Independently validate script execution in a separate session before assigning F-03 final severity
- [x] Discover exposed directory listing and confidential-marked document at /ftp/
- [x] Document F-04 with limits of evidence
- [x] Verify credential-free HTTP 200 and confidential text retrieval for F-04; `/ftp/` returned HTTP 200 with an incomplete-transfer warning
- [x] Upload and link four F-04 screenshots and PowerShell verification evidence
- [ ] Review risks, recommendations and final report
- [ ] Publish the repository after evidence and secrets review

## Ethical boundary

All testing is performed against a **deliberately vulnerable application owned and operated locally**. Do not apply these tests to websites or systems without explicit authorization. No third-party targets or sensitive session credentials are included in this repository.

## Resources

- [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)
- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [ZAP Documentation](https://www.zaproxy.org/docs/)
- [OWASP SQL Injection Prevention](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
