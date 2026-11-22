---
name: ssrf-hunting
description: Find and exploit Server-Side Request Forgery across URL-fetching features, including cloud metadata bypasses and blind SSRF techniques. Use during web assessments and bug bounty hunting.
author: Niranjan
company: securityskills.sh
contact: https://www.linkedin.com/
---

# SSRF Hunting

Locate every place the server fetches a URL you control, then test it methodically.

## Where to Look

- Webhooks and callback URLs (payments, integrations, alerts)
- Importers: avatar-by-URL, import-from-URL, RSS feed readers
- Preview generators: link unfurlers, PDF/image renderers, screenshot services
- File parsers that resolve external entities or references (XInclude, XSLT, SVG)
- Proxy/redirect endpoints taking a `url=` parameter

## Test Matrix

### Basic
- `http://127.0.0.1:PORT` for internal services (find live ports via response differential)
- `http://localhost`, `http://[::1]`, `http://2130706433` (integer IP), `http://0x7f000001`
- Internal hostnames: `http://inventory`, `http://kubernetes.default.svc`

### Filter Bypass
- DNS names resolving to loopback: custom records, `localtest.me`,nip.io patterns
- Redirects: public URL 302→internal target (tests whether redirects are followed)
- DNS rebinding for single-resolution filters (rebind.network tools)
- Scheme abuse: `file:///etc/passwd`, `gopher://`, `dict://` where parsers allow

### Cloud Metadata (high impact)
- AWS IMDSv1: `http://169.254.169.254/latest/meta-data/iam/security-credentials/`
- AWS IMDSv2 requires token — try hop-limit tricks via 302 redirect
- GCP: `http://metadata.google.internal/computeMetadata/v1/` with `Metadata-Flavor: Google`
- Azure: `http://169.254.169.254/metadata/instance?api-version=2021-02-01` with `Metadata: true`

### Blind SSRF
- Out-of-band DNS/HTTP callbacks (interactsh-style) to confirm fetch without visible response
- Response-time differentials against open vs filtered ports
- Error-message leaks: connection refused vs timeout vs HTTP error codes

## Rules

- Reading metadata that returns credentials: capture minimal proof (role name, key prefix), report immediately, do not use credentials to pivot without authorization
- Never exfiltrate real user data through the SSRF

## Remediation Notes to Include

Allowlist of destinations, block link-local/loopback at network layer, disable redirect following, use IMDSv2, serve fetches from an isolated egress-controlled worker.
