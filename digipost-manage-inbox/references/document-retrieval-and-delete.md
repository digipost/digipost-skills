# Downloading content and deleting documents

Authoritative specs: [Get Document](https://digipost.github.io/digipost-technical-docs/api-spec/get-document.md) and
[Delete Document](https://digipost.github.io/digipost-technical-docs/api-spec/delete-document.md).

## Downloading the content

Fetch a document's bytes by `GET`ing the **`content-uri`** from the inbox listing (attachments have their own
`content-uri`). The behaviour developers most often get wrong: **the response is not the bytes.** Per the docs it is an
**HTTP 307 redirect** to a one-time, time-restricted URL on a separate content host — the URL can be used **once** and
its token is valid for **30 seconds**. So follow the redirect immediately, don't store or reuse the redirect URL, and
re-request the `content-uri` whenever you need the content again. For the token internals, defer to the Get Document
page.

The bytes are not guaranteed to be PDF (or any particular format): check the document's `content-type` metadata (and
the `Content-Type` of the downloaded response) and handle the format accordingly before storing the file or passing it
downstream.

Requesting the content is also what sets the document's `first-accessed` element — listing the inbox does not (see
`inbox-anatomy.md`).

## Deleting a document

`DELETE` the **`delete-uri`** from the inbox listing; a successful delete responds with `200 OK`. Deletion is
**optional** — whether you delete depends on your retention needs. A sensible order is download → confirm the content
is safely persisted → delete, so a failed download never loses data. Deletion is irreversible, so if a document can't
be confidently classified (e.g. an unrecognised sender), prefer flagging it for review over deleting it.

## Request signing for GET/DELETE

Both requests have **no body**, a case the
[security spec](https://digipost.github.io/digipost-technical-docs/API/security.md) covers explicitly: the
content-hash header is not used, and the spec's canonical-string examples include a request without a body. For the
client-vs-hand-rolled decision and how to verify a signer, see the **digipost-auth-and-signing** skill.
