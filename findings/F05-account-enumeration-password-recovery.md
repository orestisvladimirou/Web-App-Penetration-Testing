# F-05 — Account Enumeration and Security Question Disclosure Through Password Recovery

| Property | Result |
|---|---|
| Status | **Confirmed in local training lab** |
| Risk | **Low — provisional qualitative severity** |
| Date tested | 31 October 2026 |
| Target | `http://localhost:3000` (local OWASP Juice Shop) |
| Endpoint | `GET /rest/user/security-question?email=...` |
| OWASP category | Authentication and recovery |
| Relevant CWE | CWE-204 (Observable Response Discrepancy); CWE-203 (Observable Discrepancy) |
| Remediation retest | Not performed |

## Summary

The password recovery interface reveals registered account existence by returning account-specific security-question information. A lookup for an existing disposable test account returned a populated `question` object. A lookup for a nonexistent address returned an empty JSON object, while the browser displayed no security question and disabled later fields.

Both API calls returned **HTTP 200 OK**; it is the **difference in response bodies and UI state** that provides the enumeration signal.

## Test setup

The test used two email inputs in a local, intentionally vulnerable application:

- Registered lab address: `lab-user-a@example.invalid`
- Unregistered lab address: `nonexistent-lab-user@example.invalid`

No login, password reset, brute force, or password-answer guessing was needed.

## Reproduction steps

1. Open **Account → Login → Forgot your password?** in the local Juice Shop browser.
2. Enter `lab-user-a@example.invalid` in the email field and move focus elsewhere.
3. Observe that the registered account's security question, **Mother's maiden name?**, appears.
4. Replace the address with `nonexistent-lab-user@example.invalid`.
5. Observe that the security-question field is empty and the form cannot proceed.
6. Compare the underlying `GET /rest/user/security-question?email=...` requests in ZAP.
7. For a fresh, cookie-free registered-account response, run:

   ```powershell
   curl.exe -i "http://localhost:3000/rest/user/security-question?email=lab-user-a%40example.invalid"
   ```

8. Compare this with a request for the nonexistent address. Do **not** submit a password change.

### Expected secure behavior

Recovery responses should not reveal which account identifiers exist. Recovery workflows should avoid displaying account-specific recovery information to unverified requesters, and should rely on robust recovery factors rather than easily guessed personal-history answers.

### Actual results

| Input | HTTP status | Response excerpt | Browser behavior |
|---|---|---|---|
| Existing account | 200 OK | `{"question":{"id":2,"question":"Mother's maiden name?",...}}` | Displays question |
| Nonexistent account | 200 OK | `{}` | No question; reset fields disabled |

The response excerpt for the existing account is abbreviated for reporting; the actual JSON also contained timestamp fields.

## Demonstrated security impact

An unauthenticated requester can distinguish registered email addresses from unregistered ones and learn the associated recovery question for existing accounts. This can facilitate targeted credential or recovery attacks.

**Not demonstrated:** correct recovery-answer discovery, unauthorized password reset, account compromise, email delivery exploitation, or any real user impact.

**Provisional Low severity rationale:** limited account-existence disclosure plus exposure of a recovery question, with no demonstrated ability to bypass the recovery answer requirement. Reassess in the context of real user-account sensitivity and recovery implementation.

## Root cause hypothesis

The recovery endpoint returns structurally different data for existing versus nonexistent accounts and exposes account-specific question metadata before identity verification. The underlying implementation was not inspected.

## Recommended remediation

1. Use uniform, non-enumerating responses for password-recovery initiation, regardless of account existence.
2. Do not disclose account-specific recovery questions to an unverified requester.
3. Prefer single-use, short-lived recovery tokens delivered through a verified channel, with appropriate throttling.
4. Avoid knowledge-based security questions as a primary recovery factor; they may rely on guessable or discoverable information.
5. Add negative regression tests that compare content, status, and timing for both known and unknown account identifiers.
6. Apply monitoring/rate limiting to recovery initiation without exposing account-existence feedback.

## Evidence

- [F05 password recovery API comparison](../evidence/F05-password-recovery-api-comparison.md) — sanitized transcript of observed ZAP and PowerShell response evidence, with explicit source limitations.
- [Existing-account PowerShell response](../evidence/F05-existing-account-response-redacted.png) — original HTTP 200 and nonempty security question response screenshot.
- [Nonexistent-account ZAP response](../evidence/F05-nonexistent-account-response-redacted.png) — original HTTP 200 and `{}` response screenshot.
- These are cropped source screenshots of the observed responses; UI screenshots were also reviewed in conversation but are not part of this GitHub evidence set.

## References

- [OWASP Forgot Password Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html)
- [OWASP WSTG — Testing for Weak Password Change or Reset Functionalities](https://owasp.org/www-project-web-security-testing-guide/)
- [MITRE CWE-204](https://cwe.mitre.org/data/definitions/204.html)
