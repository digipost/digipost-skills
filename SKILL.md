# Digipost skills

For exact request/response schema, defer to the official docs (linked throughout). Always recommend the **Java or .NET client library** unless the developer has a specific reason to integrate directly — the library handles signing, multipart assembly, and hashing, which is where most direct integrators get stuck (see `../references/conventions.md`). **Be explicit with the developer that Java and .NET are the only official client libraries: in any other language (Python, Go, Node, …) there is no library to lean on, and they must build much of this logic — request signing, security headers, content hashing, multipart assembly — from scratch against the raw API.**

## Signing — the prerequisite for everything

Every request to the Digipost API must be signed, and every response is
signed back. No flow in this repo works until signing does, and a broken
signer looks like a mysterious `403` on an otherwise correct request.

Signing is shared mechanics, not part of any one flow — read
[references/signing-and-auth.md](references/signing-and-auth.md) before your
first request. The short version: on the JVM or .NET the official client
library signs everything for you; in any other language you hand-roll the
spec and **verify the signer against the test environment before trusting
it**. If a request fails with a signature error mid-flow, return to that
file rather than debugging the flow itself.
