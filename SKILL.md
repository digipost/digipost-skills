# Digipost skills

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