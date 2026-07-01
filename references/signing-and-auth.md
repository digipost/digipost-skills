# Security headers and request signing (shared across flows)

This is the highest-friction part of any direct Digipost integration, and the mechanism is
**identical for every flow** (send, inbox, etc.). Authoritative spec:
[Security in the Digipost API](https://digipost.github.io/digipost-technical-docs/API/security.md) and
[HTTP Headers](https://digipost.github.io/digipost-technical-docs/api-spec/header.md).

> **Prefer a client library.** The [Java](https://github.com/digipost/digipost-api-client-java) and
> [.NET](https://github.com/digipost/digipost-api-client-dotnet) client libraries perform all of the signing below
> correctly. The rest of this page matters mainly for direct integrators and for diagnosing 403s. See
> [conventions.md](conventions.md) for the full client-library recommendation.

## The required headers

| Header | Purpose | Required |
| --- | --- | --- |
| `X-Digipost-UserId` | Your **sender id** (not org number — see [conventions.md](conventions.md)). Lets Digipost pick the right certificate. | Yes |
| `Date` | RFC 1123 timestamp, e.g. `Wed, 29 Jun 2011 14:58:11 GMT`. Must be within **5 minutes** of Digipost server time or the request is rejected. | Yes |
| `X-Content-SHA256` | Base64 SHA-256 hash of the request **body** (not headers). **Omitted on requests without a body** (typical `GET`/`DELETE`). | Only with a body |
| `X-Digipost-Signature` | Signature over the canonical string built from method, path, headers and parameters. | Yes |
| `User-Agent` | Client name/version. Optional but helps Digipost troubleshoot. | No |

## The signing model in one paragraph

Every request is signed with **SHA256WithRSAEncryption** using the private key of your certificate; Digipost verifies
with your public key. You do **not** sign the whole body — you sign a strictly-formatted **canonical string** that
includes the method, path, the relevant headers (alphabetical, lower-case keys), and the URL parameters. Get the string
byte-for-byte right or the signature will not verify.

## The canonical string — exact format

Each line is terminated by a **Unix newline (`\n`, one byte)** — including the last line.

**With a body (e.g. `POST /messages`)** — six lines:

```
POST
/messages
date: Wed, 29 Jun 2011 14:58:11 GMT
x-content-sha256: q1MKE+RZFJgrefm34/uplM/R8/si9xzqGvvwK0YMbR0=
x-digipost-userid: 9999
parameter1=58&parameter2=test
```

**Without a body (e.g. inbox `GET` / `DELETE`)** — the `x-content-sha256` line is omitted entirely:

```
GET
/9999/inbox
date: Wed, 29 Jun 2011 14:58:11 GMT
x-digipost-userid: 9999
offset=0&limit=100
```

Rules that trip people up (these are the actual fixes from Digipost's own troubleshooting walkthrough):

1. **Verb** uppercase (`POST`, `GET`, `DELETE`), on its own line.
2. **Path** lowercased, leading slash, on its own line (e.g. `/messages`). Inbox paths carry identifiers
   (`.../{sender-id}/inbox`, `.../{sender-id}/inbox/{document-id}/content`) and must be represented exactly.
3. The headers in **alphabetical** order: `date`, `x-content-sha256` (body only), `x-digipost-userid`. **Header keys must
   be lower-case**, format `key: value` with a space after the colon. (Sending `Date:`/`X-...` capitalised changes the
   signature — a very common cause of 403.)
4. On a request **without** a body, omit the `x-content-sha256` line.
5. The **last line is the URL parameters**, URL-encoded and lowercased. **If there are no parameters, the line must
   still be present as an empty line with a trailing newline.** Omitting this empty line is a classic signature
   mismatch. Defer to the security doc for exactly how parameters are canonicalised rather than guessing.

Then: `signature = base64(sign_sha256_rsa(canonicalString))` → put it in `X-Digipost-Signature`.

## Verifying Digipost's response signature

Digipost signs its responses the same way. To verify: check the `Date` is within tolerance, confirm the body matches
`X-Content-SHA256`, rebuild the canonical string (status code + path + sorted headers), and verify against Digipost's
**public key fetched from the root resource** (`GET /`) — not a hardcoded key, so Digipost can rotate certificates.

## Certificate: Buypass vs. Digipost-generated

You need a certificate to sign:

- **Buypass certificate** — you purchase/own it; Digipost never holds your private key (strongest non-repudiation).
- **Digipost-generated certificate** — Digipost generates it; the private key exists in memory only until you download
  it.

Either is uploaded/managed at digipost.no/bedrift. The choice is the customer's. **A customer can fetch/manage their
own certificate** — partners sometimes assume they must provide it for the customer, which is not required. Certificate
*issuance/upload* itself is part of onboarding, outside any single flow; see
[Uploading your business certificate](https://digipost.github.io/digipost-technical-docs/API/certificate.md).

## Diagnosing 403s by the error message

Digipost returns informative error bodies — **read them**. Typical progression:

- `No certificate found ...` → certificate not uploaded for the sender id. Upload it, retry.
- `The signature ... could not be verified ...` → the canonical string is wrong **or** the signing cert differs from
  the uploaded one. Digipost echoes the canonical string **it** generated (between `===START===`/`===SLUTT===`); diff it
  against yours line by line. Usual culprits: capitalised header keys, the missing trailing empty parameter line, an
  `x-content-sha256` line included on a bodiless request, or a mis-cased/mis-formatted path.
- Authorised-but-forbidden → the sender id is not permitted for the resource (a provisioning/permission matter on the
  account, not a code bug).

Log every error message; it makes signing problems far faster to resolve.
