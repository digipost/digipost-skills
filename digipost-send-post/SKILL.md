---
name: digipost-send-document
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
2. Load the relevant `reference/` file(s) only for the part the developer is stuck on — they are written to be read independently.
3. For exact request/response schema, defer to the official docs (linked throughout). Always recommend the **Java or .NET client library** unless the developer has a specific reason to integrate directly — the library handles signing, multipart assembly, and hashing, which is where most direct integrators get stuck.

## The mental model (read this first)

The single most common misunderstanding in support is **what a "sending" actually is**. Get this right before writing any code:

- A **message** is the *envelope*: it carries the recipient and metadata, plus a reference to every document being sent.
- Every message has exactly **one `primary-document`** and **zero or more `attachment`s**. The primary document is what the recipient sees first.
- A document entry in the message XML is only **metadata** (UUID, subject, file-type, authentication-level, sensitivity-level). The **actual file bytes are sent separately**, in their own part of a multipart request.
- The link between metadata and bytes is the **UUID**: the `filename` of each content part must be **identical** to the `uuid` in the XML. This is the detail people miss when they "send a message" and "attach a file" as if they were two separate calls — they are two *parts of one* `POST /messages` request.

So "sending a document" = build one message (recipient + document metadata) → attach the file bytes as additional multipart parts keyed by UUID → POST it, signed, to `/messages`.

Sending multiple files is *not* a separate mechanism — it is just the same message with one `primary-document` plus one `attachment` per extra file. (See `reference/request-anatomy.md`.)

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

1. **Decide delivery target.** Digital mail goes to a Digipost user; if the recipient is not a user, sending can fall back to physical mail (print) *if* your account is approved for it. See `reference/physical-mail-and-fallback.md`.
2. **(Optional) Identify the recipient** ahead of time with `POST /identification` to learn whether they are a Digipost user. See `reference/recipient-identification.md`.
3. **Build the message XML** — recipient + `primary-document` (+ `attachment`s). Set `authentication-level` and `sensitivity-level` on each document (see section above). See `reference/request-anatomy.md`.
4. **Assemble the multipart request** — the message XML as the first part, then one content part per document, each `filename` = the document's UUID.
5. **Add the security headers and sign the request.** This is the other big snag area. See `reference/signing-and-auth.md`.
6. **POST to `/messages`** (test or production endpoint — see below) and **read the response**: a `message-delivery` with a `status`, or an error. See `reference/errors-and-status.md`.

## Test vs. production

- Test base URL and production base URL are **different hosts**. Point at the test environment until sending works end to end, then switch. Confirm current hostnames in the docs — do not hardcode from memory.
- A production account is enabled separately (by emailing Digipost support) after test integration works.

Test environment details: https://digipost.github.io/digipost-technical-docs/process/test-environment.md

## Common snags

| Symptom | Likely cause | Where to look                                                                                                                                        |
| --- | --- |------------------------------------------------------------------------------------------------------------------------------------------------------|
| "What do I send — a message or a file?" | Treating message and file as separate calls | Mental model above; `reference/request-anatomy.md`                                                                                                   |
| `messages` vs `/messages` confusion | Wrong path / base URL; test vs prod host | `reference/request-anatomy.md`                                                                                                                       |
| 400 Bad Request | Missing header, SHA256 mismatch, bad date, blank subject, reused message-id, unsupported file type, XML fails XSD | `reference/errors-and-status.md`                                                                                                                     |
| 403 Forbidden / "No certificate found" | Certificate not uploaded, wrong cert, or signature string built incorrectly | `reference/signing-and-auth.md`                                                                                                                      |
| 404 on send | Recipient is not a Digipost user (and no physical fallback) | `reference/recipient-identification.md`, `reference/physical-mail-and-fallback.md`                                                                   |
| æøå garbled | Body not encoded as UTF-8 | `reference/request-anatomy.md`                                                                                                                       |
| Which ID goes where? | `X-Digipost-UserId` is the **sender id**, not the organisation number | `reference/signing-and-auth.md`                                                                                                                      |
| `Content-Type` for the request "not in docs" | It's a *multipart* type with a boundary, and each part has its own headers | `reference/request-anatomy.md`                                                                                                                       |
| No authentication-level or sensitivity-level set | Relying on library defaults instead of making explicit security policy choices | "Security and compliance fields" section above.                                                                                                      |
| Matching inbox senders by display name string | Display names are user-editable and not unique; correlating messages by display name is unreliable and will break with user edits or duplicates | **Do not implement display-name-based correlation.** Reference the official inbox API schema for the documented sender correlation fields.           |
| Storing inbox document without checking content type | Raw content may not be PDF; validation and type handling are developer responsibility | **Always validate `Content-Type` and `file-type` before storing.** Do not assume PDF or any specific format.                                         |
| Auto-deleting inbox messages with unidentified senders | Ambiguity in identifying a sender is not a reason to destroy data | **Never auto-delete messages.** Flag unidentified senders for review; let the user decide action. This is especially critical in production systems. |

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
