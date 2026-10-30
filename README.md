# Web Application Vulnerability Assessment & Penetration Testing

A structured, **authorized educational penetration testing assessment** of [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/) running in a local Docker container. The project demonstrates web security reconnaissance, HTTP analysis, manual exploitation validation, reporting, and remediation planning using the [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/) as a reference.

> **Status: In progress.** One high-severity vulnerability has been confirmed in the local training target. This is **not** an assessment of a production website or a third-party service.

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
| **O-01** | Permissive CORS header on public endpoints | **Configuration observation; exploitability not shown** | [Observation](observations/O01-cors-configuration.md) |

**F-01 evidence:** The login endpoint accepted an injection payload, issued an authentication response, and the browser displayed administrator identity with the Juice Shop “Login Admin” challenge completed. Sanitized evidence files have **not** yet been uploaded. The normal invalid-login control remains to be archived.

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
- [ ] Test access-control/IDOR scenarios using disposable lab accounts
- [ ] Document additional verified findings
- [ ] Review risks, recommendations and final report
- [ ] Publish the repository after evidence and secrets review

## Ethical boundary

All testing is performed against a **deliberately vulnerable application owned and operated locally**. Do not apply these tests to websites or systems without explicit authorization. No third-party targets or sensitive session credentials are included in this repository.

## Resources

- [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)
- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [ZAP Documentation](https://www.zaproxy.org/docs/)
- [OWASP SQL Injection Prevention](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
