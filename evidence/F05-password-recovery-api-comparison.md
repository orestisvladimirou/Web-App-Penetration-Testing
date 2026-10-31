# F-05 — Password Recovery API Response Comparison (Sanitized Evidence)

**Observed:** 31 October 2026  
**Environment:** OWASP Juice Shop running locally at `http://localhost:3000`  
**Tool sources:** ZAP 2.17.0 response inspection; PowerShell `curl.exe` without authentication  
**Evidence type:** **Transcribed excerpts from user-captured screenshots**, not raw HTTP archives or screenshots. Fields omitted for concision.

## Test A — Existing disposable lab account

Request reproduced in PowerShell (without Authorization header, cookie, or browser session):

```http
GET /rest/user/security-question?email=lab-user-a%40example.invalid HTTP/1.1
Host: localhost:3000
```

Observed response:

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"question":{"id":2,"question":"Mother's maiden name?"}}
```

**Note:** The JSON above is a **sanitized excerpt** retaining the observed `question` object's `id` and `question` fields; `createdAt` and `updatedAt` were present in the actual server response but are intentionally omitted here. It is not a byte-for-byte complete HTTP capture.

The browser's Forgot Password form also displayed **Mother's maiden name?** when this registered address was entered.

## Test B — Nonexistent lab account

Observed in ZAP History request ID **580**:

```http
GET /rest/user/security-question?email=nonexistent-lab-user%40example.invalid HTTP/1.1
Host: localhost:3000
```

Observed response:

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{}
```

The 2-byte empty JSON object was visible in ZAP's Response body. The browser form did not display a security question and disabled the next reset fields.

## Result

Although both requests produced **HTTP 200**, the response bodies reveal different account-existence states:

- Registered address → nonempty `question` object with a named recovery question.
- Unregistered address → empty `{}` object.

This confirms **account enumeration and disclosure of a registered account's recovery question**, but **not** successful password reset or account takeover.

## Evidence provenance and limitations

- The registered-account result was captured by the user in a PowerShell screenshot showing the actual 200 response and full JSON.
- The nonexistent-account result was captured in a ZAP screenshot showing response code 200 and body `{}`.
- Both were corroborated through separate Forgot Password UI screenshots.
- The current evidence document is a direct transcription of those observed results. **No screenshots or raw response files for F-05 have been uploaded to this repository.**
- No authentication credentials, tokens, cookies, password answers, or real personal data were recorded.
- A future reviewer can repeat both local `curl.exe` requests against a clean training instance using newly registered disposable accounts. Account identifiers may differ between deployments.
