# Request signing

Every request to the Digipost API is signed (SHA256withRSA over a canonical
string built from the method, path, a few headers, and the query), and every
response is signed back. The format is fully specified — follow it rather than
reconstructing it from memory:

> Canonical spec: https://digipost.github.io/digipost-technical-docs/API/security.md

This assumes you already have your signing certificate (a PKCS#12 `.p12` file
plus its password) and your Digipost sender id (the `X-Digipost-UserId` value).

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

- If you have access to the Digipost MCP server, use the `validateRequest` tool, run a sample request
  through it and fix whatever it flags — no live send needed.
- Otherwise send a request to the test environment; on a signature error the API
  echoes the exact canonical string it expected, between `===START===` and
  `===SLUTT===`. Diff yours against it — the difference is the bug.

The spec's Troubleshooting section (§5) walks through the usual mistakes
(header-name casing, the missing empty parameter line, clock skew).
