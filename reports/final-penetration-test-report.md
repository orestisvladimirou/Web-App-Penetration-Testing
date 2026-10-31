# Web Application Vulnerability Assessment and Penetration Test

**Final technical report — Version 1.0**  
**Assessment period:** 30–31 October 2026  
**Report issued:** 31 October 2026  
**Target:** OWASP Juice Shop 20.3.0, self-hosted at `http://localhost:3000`  
**Assessment type:** Authorized, black-box educational security assessment  
**Report status:** **Final for the completed test scope**; no fixes or remediation retests were performed.  
**Distribution:** Portfolio project. The GitHub repository remains **private** pending publication approval.

---

## 1. Executive summary

An intentionally vulnerable instance of OWASP Juice Shop was deployed in a locally controlled Docker environment and assessed using OWASP ZAP 2.17.0, a proxied Chrome browser, and targeted PowerShell `curl.exe` requests. Testing followed a risk-focused workflow inspired by the OWASP Web Security Testing Guide (WSTG): establish scope, inspect functionality and HTTP traffic, reproduce suspected weaknesses, compare controls, and preserve redacted evidence.

**Six technical findings** were documented. The strongest demonstrations were (a) SQL injection bypassing administrator login and (b) public account registration accepting an `admin` role supplied by the requester. A separate customer account was denied access to the Administration page that the role-elevated account could view. An insecure direct object reference exposed another lab user's basket, a directory exposed a confidential-marked training document without authentication, and password recovery revealed whether an account existed. The application also completed its built-in DOM XSS challenge and rendered attacker-controlled iframe markup, but JavaScript execution was **not reproduced** in a clean Incognito session. A permissive CORS header was recorded separately as an observation **without demonstrated exploitable impact**.

These findings describe **a purpose-built vulnerable training target**, not an internet-facing organization or a real-world breach. Severity labels are **qualitative assessment priorities, not calculated CVSS scores**.

### Findings overview

| ID | Finding | Priority | Evidence-based status |
|---|---|---|---|
| [F-01](../findings/F01-sql-injection-authentication-bypass.md) | SQL injection — administrator login bypass | **High** | Confirmed in lab |
| [F-02](../findings/F02-idor-basket-access.md) | Basket insecure direct object reference (IDOR) | **Medium** | Confirmed read-only cross-account disclosure |
| [F-03](../findings/F03-dom-based-xss-search.md) | DOM XSS challenge / search HTML injection | **Unrated** | Challenge completion and iframe rendering confirmed; independent script execution not established |
| [F-04](../findings/F04-exposed-directory-confidential-document.md) | Public directory and confidential-marked document | **Medium** | Credential-free document read confirmed |
| [F-05](../findings/F05-account-enumeration-password-recovery.md) | Password recovery account enumeration | **Low** | Different API responses confirmed |
| [F-06](../findings/F06-mass-assignment-admin-registration.md) | Mass assignment — self-assigned administrator role | **High** | Role assignment and differentiated Administration UI access confirmed |
| [O-01](../observations/O01-cors-configuration.md) | Wildcard CORS response header | **Observation only** | Configuration confirmed; exploitability not established |

**Prioritization:** F-01 and F-06 demand attention first in a comparable real service because they cross authentication/authorization trust boundaries. F-02 and F-04 expose information; F-05 assists account targeting. F-03 needs separate evaluation before a firm impact-based rating. The test does **not** claim server takeover, administrative data modification, payment-data compromise, or script-based account theft.

## 2. Scope and engagement rules

The only authorized application origin was `http://localhost:3000`, including its application-served routes and APIs. The tester controlled the local Docker instance and created disposable accounts for account isolation tests. The scope explicitly excluded third-party websites, public Juice Shop demos, browser telemetry/autofill services, unrelated local services, host compromise, denial-of-service, destructive changes, and attacks on real users. ZAP was configured with a local application context and Protected Mode.

Testing avoided bulk enumeration, persistence, exploitation of external services, and administrator-only state-changing operations. Where access was demonstrated, the tester stopped at a minimal proof of impact. The project's [Rules of Engagement](../scope/rules-of-engagement.md) preserve the agreed boundaries.

## 3. Test environment and reproducibility

| Component | Recorded value |
|---|---|
| Host | Windows 11 |
| Target | OWASP Juice Shop **20.3.0**, locally served on port 3000 |
| Container | `juice-shop-lab` |
| Docker CLI | **29.9.2** (build `a82e528`) |
| ZAP | **2.17.0** |
| Browser | Chrome launched through ZAP for HTTP inspection |
| Configured Docker image | `bkimminich/juice-shop` |
| Repository-qualified image digest reported locally | `bkimminich/juice-shop@sha256:42843723a7bf4246aa5d63c324978b17ef296ac78434aeddd34c427bfb5853f4` |

The running application version came from `/rest/admin/application-version`; Docker image details came from `docker inspect` and `docker image inspect ...RepoDigests`. The local image ID happened to share the reported SHA-256 value but is **not conceptually interchangeable with** the repository digest. The image reference was **not pinned when originally deployed**; the README now includes a digest-pinned reproduction example. **A fresh-machine reproduction of that digest has not been performed.**

See the [Environment Fingerprint](../scope/environment-fingerprint.md).

## 4. Methodology

The assessment used an **OWASP WSTG-inspired manual workflow**, not a certification, exhaustive checklist, or full source-code security audit:

1. **Scope and safeguards:** constrain the local target, define exclusions, use disposable accounts.
2. **Reconnaissance:** browse login, registration, recovery, basket, search, informational documents, and administration views; inspect ZAP Sites, History, Requests, Responses and Alerts.
3. **Hypothesis and control:** select one test input or identifier change and compare it against a normal or negative-control result.
4. **Validation:** inspect response status, JSON/body content, authenticated identity and application-visible effects; supplement ZAP with `curl.exe` where caching or session state could confuse results.
5. **Documentation:** record reproducible steps, exact observations, uncertainty, plausible root cause, impact, remediation advice, and evidence file links. Classify scanner-only configuration observations separately.

The assessment was primarily black-box. No source audit or patched build was tested. Exact test details appear in the linked technical findings and [Methodology](../methodology/testing-methodology.md).

## 5. Severity rationale and confidence

Ratings below are **expert qualitative judgments for a hypothetical comparable deployed service**, based on the demonstrated boundary crossed and observed information exposure. They are **not CVSS v3/v4 scores**; no numerical CVSS score or vector is claimed. Risk in this intentionally vulnerable local lab should not be confused with a measured likelihood of attack against a production business.

| ID | Rationale | Key uncertainty |
|---|---|---|
| F-01 — High | Authentication bypass provided administrator identity without its password; valid incorrect-login control returned HTTP 401 | No post-login privileged operations tested |
| F-02 — Medium | Authenticated request for another user's basket returned products and owner metadata | Independent token-to-Account-B attribution is described by the workflow but not archived as a standalone source artifact |
| F-03 — Unrated | Application confirmed DOM XSS challenge and iframe markup injection; a crafted URL reproduced iframe rendering | No JavaScript alert in the clean session; practical execution and victim impact unverified |
| F-04 — Medium | Unauthenticated retrieval of a document expressly marked confidential, along with exposed directory filenames | Training document contains placeholder text; other files not accessed |
| F-05 — Low | Recovery API disclosed existence and named security question for a registered test address | No bypass of the recovery answer, password reset or account takeover |
| F-06 — High | Untrusted public registration input created an administrator-role account; the admin interface loaded while a regular customer's same route showed a 403 denial | No admin-only write action or direct backend privilege-check retest |

Priorities are provisional when transposed to real deployments; changes to data sensitivity, exploit preconditions, code fixes, or business exposure could change them. For formal CVSS reporting, select and document a **complete vector and computed score**, rather than labeling these judgments as CVSS metrics. See [FIRST CVSS v4.0](https://www.first.org/cvss/v4.0/specification-document).

## 6. Technical findings

### F-01 — SQL injection in administrator login (High)

**Endpoint:** `POST /rest/user/login`  
**Evidence:** [Injected request](../evidence/F01-login-request-redacted.png), [administrator challenge and identity](../evidence/F01-admin-access-redacted.png), [invalid-login HTTP 401 control](../evidence/F01-invalid-login-control-redacted.png).

An injection-style email input, paired with an incorrect password, caused the application's login flow to return an authenticated administrator session. The interface showed `admin@juice-sh.op` and completed its **Login Admin** challenge. A separate invalid-credential attempt was rejected (401). The token-bearing response screenshot was intentionally excluded from the repository.

**Impact demonstrated:** administrator authentication bypass. **Not tested:** privileged data modification.  
**Recommendation:** parameterize database lookups, perform independent password-hash verification, and add malformed-input regression tests. [Full finding](../findings/F01-sql-injection-authentication-bypass.md).

### F-02 — Basket IDOR / broken object authorization (Medium)

**Endpoint:** `GET /rest/basket/{id}`  
**Evidence:** [Redacted cross-account Requester exchange](../evidence/F02-cross-account-request-redacted.png).

Two disposable accounts had distinct baskets (Account A: ID 6; Account B: ID 7). During the Account B workflow, the captured basket request was modified to retrieve ID 6. The response returned HTTP 200 and Account A's basket metadata and Apple/Banana Juice entries. The screenshot redacts the Bearer token and therefore does not independently prove token ownership without the preceding session workflow.

**Impact demonstrated:** unauthorized cross-account **read**, not modification.  
**Recommendation:** enforce object ownership on the server for every read and write operation; add negative cross-account authorization tests. [Full finding](../findings/F02-idor-basket-access.md).

### F-03 — Search-field HTML injection / DOM XSS challenge (Unrated)

**Entry point:** product search and client-side rendering  
**Evidence:** [Original training challenge](../evidence/F03-dom-xss-search-redacted.png), [clean-session URL rendering](../evidence/F03-incognito-url-reproduction.png).

Submitting scriptable iframe markup caused a frame to render and prompted Juice Shop's **DOM XSS** challenge-completion banner. Opening the crafted search URL in a separate Incognito session rendered a frame again, but **no JavaScript alert appeared** despite dismissing the welcome overlay and refreshing the page.

**Impact demonstrated:** client-side rendering of active HTML markup and successful built-in challenge completion. **Not demonstrated:** reliable cross-session script execution, access to another person's information, or token extraction.  
**Recommendation:** treat search strings as inert text, avoid unsafe HTML sinks, retain framework sanitization, and apply a defense-in-depth Content Security Policy. **Do not assign a numerical/High severity from the current evidence alone.** [Full finding](../findings/F03-dom-based-xss-search.md).

### F-04 — Public `/ftp/` index and confidential-marked document (Medium)

**Endpoints:** `/ftp/`, `/ftp/acquisitions.md`  
**Evidence:** [Directory listing](../evidence/F04-ftp-directory-listing.png), [document](../evidence/F04-confidential-acquisitions.png), [cookie-free HTTP verification](../evidence/F04-unauthenticated-verification.png).

A public terms link led to the `/ftp/` file area. The directory exposed filenames of unreferenced files and backups. A direct, credential-free request returned HTTP 200 and the contents of `acquisitions.md`, explicitly designated confidential. An index-directory `curl` request also returned HTTP 200 but reported error 18 (incomplete transfer); the browser separately displayed the listing. The document uses fictional placeholder content.

**Impact demonstrated:** unprotected access to a confidential-marked lab document and filename disclosure. **Not tested:** contents of backups or password databases.  
**Recommendation:** remove sensitive artifacts from public paths, disable unnecessary listing, and enforce server-side access control. [Full finding](../findings/F04-exposed-directory-confidential-document.md).

### F-05 — Account enumeration through recovery API (Low)

**Endpoint:** `GET /rest/user/security-question?email=...`  
**Evidence:** [Existing email response](../evidence/F05-existing-account-response-redacted.png), [nonexistent email response](../evidence/F05-nonexistent-account-response-redacted.png), [sanitized comparison](../evidence/F05-password-recovery-api-comparison.md).

The registered disposable address returned HTTP 200 with its named recovery question; a nonexistent address returned HTTP 200 with `{}`. The browser behaved differently in each case. A fresh PowerShell request established that the question was retrievable without login credentials.

**Impact demonstrated:** account existence and recovery-question disclosure. **Not demonstrated:** recovery bypass or reset of an unauthorized account.  
**Recommendation:** use non-enumerating recovery initiation, avoid exposing questions before verification, and use single-use short-lived recovery tokens with rate controls. [Full finding](../findings/F05-account-enumeration-password-recovery.md).

### F-06 — Public registration accepts administrator role (High)

**Endpoint:** `POST /api/Users/`  
**Evidence:** [Baseline customer registration](../evidence/F06-baseline-registration-redacted.png), [elevated registration](../evidence/F06-registration-role-admin-redacted.png), [admin page](../evidence/F06-administration-screen-redacted.png), [customer denial](../evidence/F06-customer-route-control-redacted.png).

Normal account registration returned HTTP 201 with `role:customer`. A modified registration request for another disposable address included `role:admin`; the server returned HTTP 201 and `role:admin`. The new account subsequently loaded the Administration page. A separate ordinary customer viewing the same route received the application's displayed **403 access-denied** message. That displayed message was not independently checked as an HTTP status.

**Impact demonstrated:** self-assignment of a privileged role through untrusted registration properties, with differentiated access to an administrator UI. **Not demonstrated:** privileged backend data writes or destructive changes.  
**Recommendation:** server-side allowlist for public registration fields, force a nonprivileged default role, restrict role assignment and enforce authorization at backend endpoints. [Full finding](../findings/F06-mass-assignment-admin-registration.md).

## 7. Configuration observation — O-01

ZAP flagged `Access-Control-Allow-Origin: *` on public homepage and product-search responses. Targeted requests confirmed that header, but the responses were publicly accessible and credentialed cross-origin data exposure was **not demonstrated**. This remains a separate configuration observation, not a seventh validated exploit. [O-01](../observations/O01-cors-configuration.md).

## 8. Remediation roadmap for a comparable application

**Priority 1 — Authentication and privilege boundaries:** fix F-01 and F-06, centralize server-side authentication and role assignment, and regression-test public registration with unauthorized fields. Review every privileged endpoint rather than trusting frontend route gates.

**Priority 2 — Object and file access:** address F-02 by checking resource ownership for all object operations and F-04 by removing internal files from public serving locations and preventing unapproved index views.

**Priority 3 — Recovery and rendering:** address F-05 with opaque recovery responses and verified-channel tokens; assess F-03 script execution conditions, correct the unsafe client-side rendering path, and test a harmless regression case after a fix.

**Verification:** On a patched, **separate** local build, rerun baseline and negative controls. Expected outcomes include a rejected injection login, denial of cross-account basket access, absence of scriptable search markup, denial of unauthenticated confidential file access, non-enumerating recovery responses, and registration that always ignores/rejects a public `admin` role. **None of these fixes was implemented or retested during this assessment.**

## 9. Evidence review and limitations

- The repository contains **16 PNG evidence screenshots**, plus sanitized textual response/comparison records. All 16 committed PNGs were visually reviewed for exposed secrets. No visible token values, passwords, security answers, or private keys were identified; F-01's original token-bearing response was intentionally not committed.
- The screenshots contain **synthetic training data and disposable example-invalid addresses**. They are not claims about real customers.
- The F-02 screenshot omits its original Bearer-token value; identity linkage rests on the documented test sequence, not a standalone independent session-attribution screenshot.
- F-03's fresh-session test showed **no JavaScript alert**. Do not describe it as proven account compromise or reliably executable JavaScript in that session.
- F-06's customer denial is a **rendered 403 page message**, not a captured backend HTTP 403 response.
- F-04's directory `curl` body transfer was incomplete; the **document** read independently succeeded.
- This was a **time-limited, selected-flow assessment**. It is not comprehensive coverage of every API, platform, business flow, user role or web security issue.
- No code inspection, patch installation, formal CVSS vector calculation, infrastructure penetration test, destructive validation or remediation retest occurred.
- A risk rating attached to this intentionally insecure lab does not quantify live exploitation probability in real deployments.

## 10. Conclusion

The engagement achieved its educational objectives: establish and constrain a reproducible local target, perform manual HTTP/role-based vulnerability testing, differentiate exploitable weaknesses from configuration observations, preserve sanitized proof, and communicate remediation with explicit uncertainty. The most significant validated outcomes were an administrator login bypass, an administrator-role registration flaw, and cross-account basket disclosure.

This document closes the **testing and reporting scope** of the exercise. Publication, additional validation and any remediation testing are separate activities. The source repository should remain **private until its owner explicitly approves publication**.

## 11. References

- [OWASP Web Security Testing Guide, versioned v4.2](https://wstg.owasp.org/v4.2/)
- [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)
- [OWASP API Security Top 10 (2023)](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
- [OWASP SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
- [OWASP IDOR Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html)
- [OWASP Forgot Password Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html)
- [OWASP DOM-Based XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html)
- [FIRST CVSS v4.0 Specification](https://www.first.org/cvss/v4.0/specification-document)

---

**Report provenance:** All observations derive from user-executed tests and their captured screenshots, the connected GitHub technical finding files, and the locally verified Docker/version output. No independent live testing was performed by the report author.
