---
name: digipost-auth-and-signing
description: >-
  Use when signing requests to the Digipost API or verifying its signed
  responses — the security headers, the canonical string, and the
  SHA256withRSA signature — and when diagnosing a 403 or signature error on an
  otherwise correct request. This is the shared prerequisite for every Digipost
  flow (send, inbox, Control). On the JVM or .NET the client library does all of
  this for you; in any other language you hand-roll it against the spec and
  verify it before trusting it. Defers the canonical format to the official
  Digipost security documentation.
---

# Request signing

Every request to the Digipost API is signed (SHA256withRSA over a canonical
string built from the method, path, a few headers, and the query), and every
response is signed back. The format is fully specified — follow it rather than
reconstructing it from memory:

> Canonical spec: https://digipost.github.io/digipost-technical-docs/api/security.md

This assumes you already have your signing certificate (a PKCS#12 `.p12` file
plus its password) and your Digipost sender id (the `X-Digipost-UserId` value).
Read more about what types of certificates are supported, and how to obtain one here: https://digipost.github.io/digipost-technical-docs/api/certificate.md.

## On the JVM or .NET

The official [Java](https://github.com/digipost/digipost-api-client-java) and
[.NET](https://github.com/digipost/digipost-api-client-dotnet) client libraries sign every request and verify every response
automatically. You write no signing code — you only supply a `Signer` built from
your `.p12`. There is nothing else to do for signing.

## Any other language (Python, Node, Go, Ruby, …)

There is no official client, so you implement signing yourself. Follow the
spec's signing (§3) and response-verification (§4) sections — they give the
canonical-string format, a Java sample, and the exact algorithm
(`SHA256WithRSAEncryption` = RSASSA PKCS#1 v1.5 with SHA-256). Then verify it as
below before trusting it.

## Verify before trusting a hand-rolled signer

- Send a sample signed request to the test environment first; on a signature error the API
  echoes the exact canonical string it expected, between `===START===` and
  `===SLUTT===`. Diff yours against it — the difference is the bug.

The spec's Troubleshooting section (§5) walks through the usual mistakes
(header-name casing, the missing empty parameter line, clock skew).
