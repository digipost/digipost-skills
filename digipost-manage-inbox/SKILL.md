---
name: digipost-manage-inbox
description: >-
  Use when helping a developer read and manage an ORGANISATION'S Digipost
  inbox via the API — listing documents that have been sent to the
  organisation, downloading the actual document content, and deleting
  documents after retrieval. Use this for questions like "how do I fetch
  documents sent to our organisation", "how do I download/poll our inbox",
  "how do I delete a document after reading it". Defers all exact fields, schema, and endpoint
  details to the official Digipost technical documentation.
---

# Digipost — Manage Inbox

This skill helps you (an AI agent) guide a developer through **reading and managing the organisation's own inbox** via the Digipost API. It is one *flow* in a larger Digipost integration; sibling flows (e.g. *send document*, *Digipost Control*) are out of scope here.

The skill gives the **correct mental model and the shape of the flow**, then points to the canonical docs for exact fields and endpoints. Do not invent endpoint paths, element names, or parameter names — when a specific detail is needed, link the developer to the relevant doc page listed below.

## Read this first: whose inbox is this?

The **single most important clarification** — and a recurring source of confusion in support — is *whose* inbox the API exposes:

- This flow is about the **organisation's own inbox**: documents that other parties have **sent to your organisation** as a Digipost recipient (or that you receive as a broker / on behalf of yourself). You are the receiver.
- It is **not** a way to read an end-user's personal Digipost mailbox on their behalf. Requests of the form "let our system read a user's inbox with their consent" are a different thing and are **not offered** by this API. See `references/scope-and-boundaries.md` before designing anything around end-user mailboxes.

If a developer's goal is "read someone else's mailbox", stop and read the scope file first — most of those requests do not map to this flow at all.

> Files under `references/` are specific to this flow; files under `../references/` (repo root) are shared across all Digipost flows.

## The mental model

- An **inbox** is a list of **document** entries that have been delivered to the organisation. Each entry is **metadata only** (id, subject, sender, delivery-time, content-type, etc.) plus **URIs** that point to the actual content and to the delete operation.
- A document entry can itself carry one or more nested **attachment** entries, each with its own id and content URI.
- The **actual bytes are fetched in a separate call** via the document's content URI — listing the inbox does not download content. (Same separation-of-metadata-from-content idea as the send flow.)
- Tracking what you've already handled is **your responsibility**: identify documents by their **document id** (not by list position), and use the metadata that indicates first access to tell whether you've seen a document before. See `references/inbox-anatomy.md`.

## The flow, end to end

1. **List the inbox** — `GET` the inbox resource for your sender id, using pagination parameters. You get back document metadata + content/delete URIs. See `references/inbox-anatomy.md`.
2. **Decide what's new** — use the document id and the first-access indicator to avoid re-processing. Do **not** rely on ordering. See `references/inbox-anatomy.md`.
3. **Download content** — request the document's content URI. This returns a short-lived, one-time redirect to the actual file; follow it immediately. See `references/document-retrieval-and-delete.md`.
4. **(Optional) Delete** — after you've safely retrieved a document, you may delete it from the inbox via its delete URI. Deletion is not mandatory. See `references/document-retrieval-and-delete.md`.

Every request is signed with the same security mechanism as the rest of the API. See the shared `../references/signing-and-auth.md` (the inbox flow's requests are bodiless `GET`/`DELETE` — see the "without a body" canonical-string rules there). For response status codes and reading error bodies, see the shared `../references/response-codes.md`.

## Test vs. production

The inbox endpoints live under the same API host as the rest of the API, and that host **differs between test and production**. Point at the test environment until the flow works end to end, then switch; confirm current hostnames in the docs rather than hardcoding from memory. See `../references/conventions.md` and the [test environment docs](https://digipost.github.io/digipost-technical-docs/process/test-environment.md).

## Areas that need special attention

These are the points where developers have actually gotten stuck or confused. Handle them explicitly.

| Topic | What to watch for | Where to look |
| --- | --- | --- |
| "Read a user's mailbox on their behalf" | Not offered by this API. Don't design around it; don't conflate with Digipost Control. | `references/scope-and-boundaries.md` |
| Org inbox vs. personal mailbox | "Inbox" here = the organisation's received documents, not an end-user's postbox. | This file; `references/scope-and-boundaries.md` |
| "Turn off receiving mail" | Whether/how an org can stop receiving is account configuration, not an API call described in these docs — confirm with Digipost rather than inventing an endpoint. | `references/scope-and-boundaries.md` |
| Read / opening receipts ("åpningskvittering", status for unopened letters) | This is a **send-side** concept and is shown in the web admin UI, not the inbox API. Do not promise it via this flow. | `references/scope-and-boundaries.md` |
| Unstable list ordering | List order is not stable and must not be used as an identifier; track by document id. | `references/inbox-anatomy.md` |
| One-time, time-limited content link | The content URI yields a single-use, short-lived redirect — don't cache or reuse it. | `references/document-retrieval-and-delete.md` |
| Correlating senders by display name | Display names are user-editable and not unique; matching documents by display-name string is unreliable and breaks on edits or duplicates. **Don't** implement display-name correlation — use the documented sender fields. | `references/inbox-anatomy.md` |
| Storing content without checking its type | Raw content may not be PDF; validation and type handling are the developer's responsibility. **Always** validate `content-type` / `file-type` before storing — don't assume a format. | `references/document-retrieval-and-delete.md` |
| Auto-deleting messages with unidentified senders | Ambiguity in identifying a sender is not a reason to destroy data. **Never** auto-delete; flag for review and let the user decide. Especially critical in production. | `references/document-retrieval-and-delete.md` |
| sender-id vs. organisation number | The inbox path is keyed by the **sender id**, not the org number. | `../references/conventions.md` |

## Out of scope (point elsewhere)

- Sending documents → that's the *send-document* flow.
- Digipost Control / shared-documents requests → that's the *digipost-control* flow.
- Getting an account / certificate issued / test access → manual onboarding via Digipost support: https://digipost.github.io/digipost-technical-docs/index.md
- Pricing and contractual setup → not a technical-docs topic; refer to Digipost sales/support.

## Canonical documentation

- API overview: https://digipost.github.io/digipost-technical-docs/api/index.md
- API specification index: https://digipost.github.io/digipost-technical-docs/api-spec/index.md
- Get Inbox: https://digipost.github.io/digipost-technical-docs/api-spec/inbox.md
- Get Document: https://digipost.github.io/digipost-technical-docs/api-spec/get-document.md
- Delete Document: https://digipost.github.io/digipost-technical-docs/api-spec/delete-document.md
- Security / signing: https://digipost.github.io/digipost-technical-docs/API/security.md
- HTTP headers: https://digipost.github.io/digipost-technical-docs/api-spec/header.md
- Response codes: https://digipost.github.io/digipost-technical-docs/api-spec/response-codes.md
- Data types & schema: https://digipost.github.io/digipost-technical-docs/data-types/index.md · https://digipost.github.io/digipost-technical-docs/api/schema.md
- FAQ (Norwegian): https://digipost.github.io/digipost-technical-docs/other/faq.md