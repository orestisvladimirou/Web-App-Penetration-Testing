# Local Laboratory Environment Fingerprint

**Recorded:** 31 October 2026  
**Source:** User-run PowerShell commands against the local Docker Desktop instance, captured in a screenshot during the assessment.

| Attribute | Observed value |
|---|---|
| Host OS | Windows 11 |
| Target | OWASP Juice Shop, local-only assessment at `http://localhost:3000` |
| Docker container name | `juice-shop-lab` |
| Configured Docker image reference | `bkimminich/juice-shop` (no explicit tag in the configured reference) |
| Container's local image ID | `sha256:42843723a7bf4246aa5d63c324978b17ef296ac78434aeddd34c427bfb5853f4` |
| Docker CLI version | `29.9.2` |
| Docker CLI build | `a82e528` |
| ZAP version | `2.17.0` (observed in assessment screenshots) |
| Juice Shop application version | **Not yet verified** |
| Registry manifest digest (`RepoDigests`) | **Not yet verified** |

## Collection commands and observed output

```powershell
docker inspect juice-shop-lab --format "{{.Config.Image}}"
# bkimminich/juice-shop

docker inspect juice-shop-lab --format "{{.Image}}"
# sha256:42843723a7bf4246aa5d63c324978b17ef296ac78434aeddd34c427bfb5853f4

docker --version
# Docker version 29.9.2, build a82e528
```

### Interpretation

- `.Config.Image` is the image name/reference configured when the container was created. Without a specified tag, it normally resolves to `:latest` at image selection time; **this does not document a fixed Juice Shop release version**.
- `.Image` identifies the **local image object/configuration** used to create the container. This is **not interchangeable** with the registry's manifest digest and must not be presented as a pinned pullable `repository@sha256:...` value without verification.
- A reproducible lab setup should **pin a released image tag or a verified manifest digest**. Rerunning `docker pull bkimminich/juice-shop` without pinning could fetch a newer image and alter the behavior, so the quick-start README alone does not yet guarantee a byte-for-byte identical environment.

## Outstanding reproducibility checks

Run these read-only commands against the user's local machine:

```powershell
docker image inspect bkimminich/juice-shop --format "{{json .RepoDigests}}"
curl.exe -s "http://localhost:3000/rest/admin/application-version"
```

The first may expose a registry manifest digest if the image retains one; the second may report the running application version. **Do not fill in either field until the actual output has been verified.**

## Evidence handling

The original screenshot was reviewed in conversation, but has **not been uploaded** as an image to the GitHub repository. This document is a faithful transcription of the read-only environment observations. It contains no secrets or credentials.
