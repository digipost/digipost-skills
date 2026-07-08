# Downloading content and deleting documents

Authoritative specs: [Get Document](https://digipost.github.io/digipost-technical-docs/api-spec/get-document.md) and
[Delete Document](https://digipost.github.io/digipost-technical-docs/api-spec/delete-document.md).

## Downloading the content

You download a document's bytes using the **`content-uri`** that the inbox listing returned for that document (or
attachment) — see `inbox-anatomy.md`. The content endpoint is a `GET` under the document's content path.

The behaviour developers most often get wrong: **the content request does not return the bytes directly.** Per the
docs, it responds with a **redirect (HTTP 307) to a one-time, time-restricted URL** on a separate Digipost host, and
the bytes are served from that redirect target. Key properties from the docs:

- The generated URL is **single-use** — it can be used **once**.
- It is **short-lived** — the docs state the token is valid for **30 seconds**.
- It points at a **different host** than the API (a `digipostdata.no` URL in the docs example), with a `token` query
  parameter.

Practical implications to convey to developers:

- **Follow the redirect immediately**; do not store the redirect URL to use later, and do not fetch it twice.
- The redirect target is reached over normal HTTPS; the one-time token is what authorises that specific fetch.
- If you need the content again, request the `content-uri` again to get a fresh one-time URL.

Do not document the exact token-generation algorithm from memory; if a developer needs those internals, point them at
the Get Document page.

## Validate the content type before storing

The bytes you fetch are **not guaranteed to be PDF** (or any particular format). Validating and handling the type is
the developer's responsibility: check the response `content-type` and the document's `file-type` metadata before
writing the file or handing it downstream. Do not assume a format.

## Deleting a document

Deletion uses the **`delete-uri`** from the inbox listing, via an `HTTP DELETE`, and a successful delete responds with
`200 OK` (per the Delete Document doc).

- Deletion is **optional** — you are not required to delete after downloading. Whether you delete depends on your
  retention needs.
- A sensible pattern is: download and confirm you have safely persisted the content, *then* delete, so a failed
  download never causes data loss.
- **Never auto-delete to resolve ambiguity.** If you can't confidently identify a sender (or otherwise classify a
  document), that is **not** a reason to destroy it. Flag it for human review and let the user decide. Deletion is
  irreversible — this is especially critical in production systems.

## Request signing for GET/DELETE

Both of these are requests **without a body**, which changes the signing slightly (e.g. the content-hash header is not
used). See the shared `../../references/signing-and-auth.md` for the bodiless-request rules, and defer to the
[security documentation](https://digipost.github.io/digipost-technical-docs/API/security.md) for the canonical details.