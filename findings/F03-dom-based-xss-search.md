# F-03 — DOM-Based Cross-Site Scripting via Product Search

| Field | Value |
|---|---|
| Status | **Confirmed in local training lab** |
| Severity | **Provisional; final risk rating pending cross-user delivery and impact analysis** |
| Confidence | High — Juice Shop reported its DOM XSS challenge solved and an injected iframe was rendered |
| Date observed | 31 October 2026 |
| Target | `http://localhost:3000` (local OWASP Juice Shop) |
| Entry point | Product search box; client-side search result rendering |
| Classification | CWE-79 (Cross-Site Scripting); OWASP WSTG-CLNT-01 (DOM-based XSS) |
| Retest after remediation | Not performed |

## Summary

The client-side search functionality treated user-controlled input as active HTML rather than only as literal text. Submitting an iframe containing a harmless JavaScript alert expression resulted in a rendered frame and the application displayed the **"DOM XSS" challenge-completion banner**.

The initial screenshot supports a **working local proof of concept** in the intentionally vulnerable training application. A subsequent test opened the encoded search URL in a separate **Incognito** browser session and reproduced the injected iframe. This demonstrates link-triggered HTML injection without manual re-entry in that new session. After dismissing the welcome dialogue and reloading the crafted URL in Incognito, the tester confirmed that **no JavaScript `xss` alert appeared**. The injected iframe continued to render; script execution in the separate session and sensitive-data impact remain unverified.

## Reproduction in the authorized lab

1. Open the local Juice Shop through the OWASP ZAP-launched browser.
2. Search for a normal product term (e.g. `apple`) as the baseline.
3. Replace the search term with this safe proof-of-concept input:

   ```html
   <iframe src="javascript:alert(`xss`)"></iframe>
   ```

4. Submit the search.
5. Observe that an iframe appears in the search-result area and that Juice Shop displays **"You successfully solved a challenge: DOM XSS"**.
6. Copy the resulting local `http://localhost:3000/#/search?q=...` URL and open it in a separate Incognito window without manually entering the payload. Observe that the injected iframe is rendered again (confirmed on 31 October 2026).

**Expected secure behavior:** Search input is displayed as inert text or interpreted only as data, without execution or rendering of attacker-controlled active elements.

**Observed behavior:** Active iframe markup was rendered into the document and the application confirmed completion of its DOM XSS challenge.

## Impact and severity interpretation

DOM XSS can allow script execution under an application's origin when an attacker can cause another user to load a vulnerable client-side state. Actual consequences depend on attacker-controlled input flow, ability to trigger it for a victim, Content Security Policy, cookies and privileges.

**Established:** The local test payload is accepted and rendered in the tester's browser; the training challenge is solved; and the crafted URL reproduces the injected iframe in a separate Incognito session.

**Not established:** JavaScript execution in the Incognito window (tester explicitly observed **no alert**, including after dismissing the welcome dialog and refreshing), whether a different human user can be induced to follow the crafted link, or whether sensitive information/session data can be accessed. No exfiltration or harmful payloads were attempted.

Therefore a definitive **High** severity is not justified solely from this screenshot; rating awaits reachable victim scenario and context verification.

## Likely root cause

The observed behavior is consistent with unsafe transfer of search input into a browser DOM HTML sink without appropriate context-sensitive treatment. The exact source-to-sink chain and front-end code were **not yet inspected**.

## Remediation recommendations

1. Insert untrusted input using safe text sinks such as `textContent`, not active HTML parsing sinks such as `innerHTML`.
2. Do not bypass framework sanitization or trust untrusted HTML as safe.
3. When HTML must be accepted, apply an appropriate, maintained HTML sanitizer under an explicit allowlist.
4. Keep a restrictive Content Security Policy as **defense in depth**, not a substitute for fixing unsafe DOM operations.
5. Add front-end regression tests for search inputs containing active HTML and scriptable URL schemes.
6. Verify that a malicious URL or entered search string cannot cause script execution after patching.

## Evidence register

| Evidence | Description | Status |
|---|---|---|
| F03-01 | Screenshot showing injected search payload, rendered iframe, and green "DOM XSS" completion message | [Evidence screenshot](../evidence/F03-dom-xss-search-redacted.png) uploaded and verified on GitHub on 31 October 2026 |
| F03-02 | Normal search baseline | Manually exercised; screenshot not archived |
| F03-03 | Separate Incognito window loaded the crafted `/#/search?q=...` URL and rendered the injected iframe | [Incognito screenshot](../evidence/F03-incognito-url-reproduction.png) uploaded and verified on GitHub on 31 October 2026 |

## Limitations

- A JavaScript alert was **not observed** in Incognito even after dismissing the application's welcome modal and refreshing. The original session's challenge completion and active iframe rendering were observed.
- A second browser session reproduced **iframe rendering through the URL**. Independent script execution in that session and exposure to a different human user have not been verified.
- No source code inspection, mitigation implementation, or remediation retest was performed.
- Exact Juice Shop Docker image digest is still to be recorded.

## References

- [OWASP WSTG — Testing for DOM-Based Cross-Site Scripting (WSTG-CLNT-01)](https://owasp.github.io/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/11-Client-side_Testing/01-Testing_for_DOM-based_Cross_Site_Scripting)
- [OWASP DOM-Based XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html)
- [Juice Shop companion guide — XSS challenges](https://help.owasp-juice.shop/part2/xss.html)
- [CWE-79 — Cross-Site Scripting](https://cwe.mitre.org/data/definitions/79.html)
