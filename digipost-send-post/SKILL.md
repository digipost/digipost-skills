---
name: digipost-send-post
description: >-
  Use when helping a developer send a document (digital mail / letter) to a
  recipient through the Digipost API. Covers the conceptual model of a
  "sending", how to identify a recipient, how to assemble the multipart send
  request, the required security headers and request signing, and how to read
  the delivery response and send-time errors. Defers exact field lists and schema to the
  official Digipost technical documentation.
---

# Digipost — Send a Document

This skill helps you (an AI agent) guide a developer through **sending a document to a recipient** via the Digipost API. It is one *flow* in a larger Digipost integration; sibling flows (e.g. *manage inbox*, *Digipost Control*) are out of scope here.

The skill's job is to give the **correct mental model and the shape of the flow**, then point to the canonical docs for exact fields. Do not invent field names or values — when a specific schema detail is needed, link the developer to the relevant doc page listed below.

## How to use this skill

1. Read this file to orient on the flow and the mental model.
2. Load the relevant `references/` file(s) only for the part the developer is stuck on — they are written to be read independently. Files under `references/` are specific to this flow; files under `../references/` (repo root) are shared across all Digipost flows.
3. For exact request/response schema, defer to the official docs (linked throughout). Always recommend the **Java or .NET client library** unless the developer has a specific reason to integrate directly — the library handles signing, multipart assembly, and hashing, which is where most direct integrators get stuck (see `../references/conventions.md`). **Be explicit with the developer that Java and .NET are the only official client libraries: in any other language (Python, Go, Node, …) there is no library to lean on, and they must build much of this logic — request signing, security headers, content hashing, multipart assembly — from scratch against the raw API.**

## The mental model (read this first)

The single most common misunderstanding in support is **what a "sending" actually is**. Get this right before writing any code:

- A **message** is the *envelope*: it carries the recipient and metadata, plus a reference to every document being sent.
- Every message has exactly **one `primary-document`** and **zero or more `attachment`s**. The primary document is what the recipient sees first.
- A document entry in the message XML is only **metadata** (UUID, subject, file-type, authentication-level, sensitivity-level). The **actual file bytes are sent separately**, in their own part of a multipart request.
- The link between metadata and bytes is the **UUID**: the `filename` of each content part must be **identical** to the `uuid` in the XML. This is the detail people miss when they "send a message" and "attach a file" as if they were two separate calls — they are two *parts of one* `POST /messages` request.

So "sending a document" = build one message (recipient + document metadata) → attach the file bytes as additional multipart parts keyed by UUID → POST it, signed, to `/messages`.

Sending multiple files is *not* a separate mechanism — it is just the same message with one `primary-document` plus one `attachment` per extra file. (See `references/request-anatomy.md`.)

Sending to **multiple recipients** is likewise not a separate mechanism: a message addresses exactly **one recipient**, so to reach several people you simply loop over them and send one ordinary message per recipient. Give each message its own fresh `message-id` and document UUIDs — reusing a `message-id` across sends is rejected.

## Security and compliance fields (required)

The `authentication-level` and `sensitivity-level` fields on each document are **not optional defaults** — they are security properties that affect how the recipient accesses the document. **Always set these explicitly; do not rely on library defaults:**

- **`authentication-level`**: Controls how strongly the recipient must authenticate to read the document. Common values:
  - `PASSWORD` — sufficient for general correspondence; lowest security.
  - `TWO_FACTOR` — for documents with financial or personal information (invoices, statements, health records). Recommended for any business-to-consumer document from a regulated industry.
  - `BANKID` or equivalent — for highly sensitive documents (contracts, legal documents).

- **`sensitivity-level`**: Indicates the content's sensitivity for audit and compliance purposes. Guides recipient notification and storage behavior.
  - The official docs for enum values and usage patterns can be found at https://digipost.github.io/digipost-technical-docs/assets/documents/api_v8.xsd.

**Default or missing values are a compliance risk.** Choose values appropriate for the content type and sender's regulatory obligations — a bank sending invoices should use at least `TWO_FACTOR` authentication.

## The flow, end to end

1. **Decide delivery target.** Digital mail goes to a Digipost user; if the recipient is not a user, sending can fall back to physical mail (print) *if* your account is approved for it — falling back is a decision made in your own code (see `references/physical-mail-fallback.md`). Optionally, a separate `POST /identification` can tell you ahead of time whether the recipient is a Digipost user — that is the only question it answers — handy if you are unsure, but not required for either digital or physical sending. See `references/recipient-identification.md`.
2. **Build the message XML** — recipient + `primary-document` (+ `attachment`s). Set `authentication-level` and `sensitivity-level` on each document (see section above). See `references/request-anatomy.md`.
3. **Assemble the multipart request** — the message XML as the first part, then one content part per document, each `filename` = the document's UUID.
4. **Add the security headers and sign the request.** This is the other big snag area. See `../references/signing-and-auth.md` (shared).
5. **POST to `/messages`** (test or production endpoint — see below) and **read the response**: a `message-delivery` with a `status`, or an error. See `references/errors-and-status.md` and the shared `../references/response-codes.md`.

## Test vs. production

Test and production are **different hosts**. Point at the test environment until sending works end to end, then switch; confirm current hostnames in the docs rather than hardcoding from memory. A production account is enabled separately (by emailing Digipost support) after test integration works. See `../references/conventions.md` and the [test environment docs](https://digipost.github.io/digipost-technical-docs/process/test-environment.md).

## Common snags

| Symptom | Likely cause | Where to look |
| --- | --- | --- |
| "What do I send — a message or a file?" | Treating message and file as separate calls | Mental model above; `references/request-anatomy.md` |
| `messages` vs `/messages` confusion | Wrong path / base URL; test vs prod host | `references/request-anatomy.md` |
| æøå garbled | Body not encoded as UTF-8 | `references/request-anatomy.md` |
| Which ID goes where? | `X-Digipost-UserId` is the **sender id**, not the organisation number | `../references/conventions.md` |
| `Content-Type` for the request "not in docs" | It's a *multipart* type with a boundary, and each part has its own headers | `references/request-anatomy.md` |
| No authentication-level or sensitivity-level set | Relying on library defaults instead of making explicit security policy choices | "Security and compliance fields" section above. |

For error HTTP statuses at send time (400, 403, 404, …), see `references/errors-and-status.md`.

> Reading or managing the organisation's inbox (downloading received documents, sender correlation, deletion, "never auto-delete") is a **different flow** — see the *digipost-manage-inbox* skill.

## Out of scope

- Getting an account / certificate issued / test access — manual onboarding via Digipost support: https://digipost.github.io/digipost-technical-docs/index.md
- Reading a user's inbox or documents — different flow (and note: third-party inbox reading on behalf of users is not offered).
- Digipost Control (share documents request) — different flow.
- Pricing and contractual setup — not a technical-docs topic; refer to Digipost sales/support.

## Canonical documentation

- API overview: https://digipost.github.io/digipost-technical-docs/api/index.md
- Send: https://digipost.github.io/digipost-technical-docs/api-spec/send.md
- Attachments: https://digipost.github.io/digipost-technical-docs/api-spec/attachment.md
- Identify recipient: https://digipost.github.io/digipost-technical-docs/api-spec/identify.md
- HTTP headers: https://digipost.github.io/digipost-technical-docs/api-spec/header.md
- Security / signing: https://digipost.github.io/digipost-technical-docs/API/security.md
- Response codes: https://digipost.github.io/digipost-technical-docs/api-spec/response-codes.md
- Data types & schema: https://digipost.github.io/digipost-technical-docs/data-types/index.md · https://digipost.github.io/digipost-technical-docs/api/schema.md
- Physical mail: https://digipost.github.io/digipost-technical-docs/physical-mail/index.md
