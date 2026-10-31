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

The screenshot supports a **working local proof of concept** in the intentionally vulnerable training application. The alert dialog itself was not captured, and delivery to a different browser/user was **not** tested.

## Reproduction in the authorized lab

1. Open the local Juice Shop through the OWASP ZAP-launched browser.
2. Search for a normal product term (e.g. `apple`) as the baseline.
3. Replace the search term with this safe proof-of-concept input:

   ```html
   <iframe src="javascript:alert(`xss`)"></iframe>
   ```

4. Submit the search.
5. Observe that an iframe appears in the search-result area and that Juice Shop displays **"You successfully solved a challenge: DOM XSS"**.

**Expected secure behavior:** Search input is displayed as inert text or interpreted only as data, without execution or rendering of attacker-controlled active elements.

**Observed behavior:** Active iframe markup was rendered into the document and the application confirmed completion of its DOM XSS challenge.

## Impact and severity interpretation

DOM XSS can allow script execution under an application's origin when an attacker can cause another user to load a vulnerable client-side state. Actual consequences depend on attacker-controlled input flow, ability to trigger it for a victim, Content Security Policy, cookies and privileges.

**Established:** The local test payload is accepted and rendered in the tester's browser, and the training challenge is solved.

**Not established:** Whether a specially crafted URL can trigger the same behavior in a **separate clean browser session**, whether another user can be induced to trigger it, or whether sensitive information/session data can be accessed. No exfiltration or harmful payloads were attempted.

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
| F03-01 | Screenshot showing injected search payload, rendered iframe, and green "DOM XSS" completion message | Captured in conversation on 31 October 2026; **not yet uploaded to GitHub** |
| F03-02 | Normal search baseline | Manually exercised; screenshot not archived |
| F03-03 | Second clean-browser / URL-delivery test | **Not tested** |

## Limitations

- Current screenshots do not show a separate JavaScript alert modal; challenge completion and active iframe rendering were observed.
- No browser-to-browser or cross-user trigger has been reproduced yet.
- No source code inspection, mitigation implementation, or remediation retest was performed.
- Exact Juice Shop Docker image digest is still to be recorded.

## References

- [OWASP WSTG — Testing for DOM-Based Cross-Site Scripting (WSTG-CLNT-01)](https://owasp.github.io/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/11-Client-side_Testing/01-Testing_for_DOM-based_Cross_Site_Scripting)
- [OWASP DOM-Based XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html)
- [Juice Shop companion guide — XSS challenges](https://help.owasp-juice.shop/part2/xss.html)
- [CWE-79 — Cross-Site Scripting](https://cwe.mitre.org/data/definitions/79.html)
