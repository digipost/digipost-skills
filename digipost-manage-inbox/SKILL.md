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

This skill helps you guide a developer through **reading and managing the organisation's own inbox** via the Digipost API. It is one *flow* in a larger Digipost integration; sibling flows (e.g. *send document*, *Digipost Control*) are out of scope here.

The skill's job is to give the **correct mental model and the shape of the flow**, then point to the canonical docs for exact fields. Do not invent endpoint paths, element names, or parameter names — when a specific schema detail is needed, link the developer to the relevant doc page listed below.

## How to use this skill

1. Read this file to orient on the flow and the mental model.
2. Load the relevant `references/` file(s) only for the part the developer is stuck on — they are written to be read independently. Files under `references/` are specific to this flow; files under `../references/` (repo root) are shared across all Digipost flows.

## The mental model (read this first)

- The inbox is the **organisation's own inbox**: documents that other parties have sent **to your organisation** (as a Digipost recipient, or received as a broker / on behalf of itself). It is *not* a way to read an end-user's personal mailbox on their behalf — that capability is not offered by this API. If the developer's goal touches end-user mailboxes, read `references/scope-and-boundaries.md` before designing anything.
- Listing the inbox returns **document metadata only** plus **URIs** pointing to the content and the delete operation. The **actual bytes are fetched in a separate call** per document, via its `content-uri`. A document can also carry nested **attachment** entries with their own content URIs, downloaded the same way.
- The content URI does not serve the bytes directly: it redirects to a **one-time, short-lived download URL**. Follow it immediately; re-request the content URI if you need the bytes again. See `references/document-retrieval-and-delete.md`.
- **Tracking what you've handled is your responsibility.** Identify documents by their **document id**, never by list position — the same page can change between requests as new documents arrive. See `references/inbox-anatomy.md`.

## The flow, end to end

1. **List the inbox** — `GET` the inbox resource for your sender id, paged with `offset`/`limit`. You get back document metadata + content/delete URIs. See `references/inbox-anatomy.md`.
2. **Decide what's new** — compare document ids against your own record of processed ids; you can stop paging once you hit a document you've already handled. See `references/inbox-anatomy.md`.
3. **Download content** — request the document's content URI and follow the redirect immediately. See `references/document-retrieval-and-delete.md`.
4. **(Optional) Delete** — once the content is safely persisted, you may delete the document via its delete URI. Deletion is not mandatory. See `references/document-retrieval-and-delete.md`.

Every request is signed with the same security mechanism as the rest of the API — see the shared `../references/signing-and-auth.md`. The inbox flow's requests are bodiless `GET`/`DELETE`, a case the [security spec](https://digipost.github.io/digipost-technical-docs/API/security.md) covers explicitly (the content-hash header is not used, and its canonical-string examples include a request without a body). For error statuses and error bodies, see [Response codes](https://digipost.github.io/digipost-technical-docs/api-spec/response-codes.md).

## Client libraries

The official [Java](https://digipost.github.io/digipost-api-client-java/v16.x/) and [.NET](https://digipost.github.io/digipost-api-client-dotnet/v14.0/) clients cover this entire flow — see "Receive messages" in each: fetching the inbox with offset/limit, downloading document and attachment content, and deleting. They also handle request signing. If the developer is on either stack, recommend the client first (see `../references/conventions.md`) and work from those docs' code examples rather than reconstructing raw requests.

## Test vs. production

The inbox endpoints live under the same API host as the rest of the API, and that host **differs between test and production**. Point at the test environment until the flow works end to end, then switch; confirm current hostnames in the docs rather than hardcoding from memory. See `../references/conventions.md` and the [test environment docs](https://digipost.github.io/digipost-technical-docs/process/test-environment.md).

## Common snags

| Symptom | Likely cause | Where to look |
| --- | --- | --- |
| Different results between polls with the same `offset`/`limit` | New arrivals shift offset-based pages; position is not an identifier — track by document id | `references/inbox-anatomy.md` |
| Content link fails on reuse or after a delay | The content URI redirect is one-time and short-lived; follow it immediately, re-request for a fresh one | `references/document-retrieval-and-delete.md` |
| Which ID goes where? | The inbox path is keyed by the **sender id**, not the organisation number | `../references/conventions.md` |
| "Can we see whether recipients opened what we sent?" (åpningskvittering) | Opening receipts are a send-side concept, not an inbox API feature | `references/scope-and-boundaries.md` |

## Out of scope (point elsewhere)

- Sending documents → that's the *send-document* flow.
- Digipost Control / shared-documents requests → that's the *digipost-control* flow.
- Reading an end-user's personal mailbox on their behalf → not offered by this API; see `references/scope-and-boundaries.md`.
- The client libraries' "Archive" functionality → availability is a product/commercial question, not something to design around from the client docs; refer interested developers to Digipost contact: https://digipost.github.io/digipost-technical-docs/other/contact.md
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
