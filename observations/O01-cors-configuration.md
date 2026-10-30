# O-01 — Permissive Cross-Origin Resource Sharing Configuration

**Classification:** Configuration observation — **not a confirmed exploitable vulnerability**  
**Date:** 30 October 2026  
**Target:** Local OWASP Juice Shop instance at `http://localhost:3000`  
**ZAP alert:** Cross-Domain Misconfiguration (Systemic), scanner risk Medium / confidence Medium

## Initial detection

ZAP passively flagged the HTTP response header:

```http
Access-Control-Allow-Origin: *
```

This permits cross-origin browser reads of non-credentialed responses. It is not necessarily insecure for intentionally public content.

## Manual header verification

The following requests were sent **only to the local Juice Shop host**; the example origin was supplied as an HTTP header and was not contacted.

```powershell
curl.exe -s -D - -o NUL -H "Origin: https://untrusted.example" "http://localhost:3000/"
curl.exe -s -D - -o NUL -H "Origin: https://untrusted.example" "http://localhost:3000/rest/products/search?q=apple"
```

### Results

| Endpoint | Status | ACAO header | ACAC header |
|---|---|---|---|
| `/` | 200 OK | `*` | Not observed |
| `/rest/products/search?q=apple` | 200 OK | `*` | Not observed |

Both endpoints appeared to serve public application content. No credentialed cross-origin data access or confidential data disclosure was demonstrated.

## Conclusion

The permissive CORS header is **confirmed as a configuration property**. An exploitable security weakness is **not demonstrated** by these tests. Do not count this as a confirmed medium-severity issue or equate ZAP's scanner risk with validated impact.

## Considerations

- Restrict allowed origins only where there is a clear reason to limit cross-origin access.
- Never use wildcard origins to permit sensitive credentialed cross-origin responses.
- Reassess if sensitive endpoints share the policy or if origin/credential handling changes.

## Additional note

Other ZAP alerts about security headers were observed on various resources. The manually inspected homepage and product-search responses included `X-Frame-Options: SAMEORIGIN` and `X-Content-Type-Options: nosniff`; therefore, missing-header alerts should be assessed **per affected resource**, not asserted as universally missing.
