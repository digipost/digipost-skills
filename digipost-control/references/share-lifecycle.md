# Digipost Control: the ShareDocumentsRequest lifecycle

This is the wire-level walkthrough of Digipost Control — the exact XML, event and link names, get-state elements, and signing notes. Read `../SKILL.md` first for the mental model and the shape of the flow.

Authoritative specs:
[Send ShareDocumentsRequest](https://digipost.github.io/digipost-technical-docs/api-spec/control/send.md) ·
[Discover shared documents](https://digipost.github.io/digipost-technical-docs/api-spec/control/get-shared.md) ·
[Get state](https://digipost.github.io/digipost-technical-docs/api-spec/control/get-state.md). Defer exact fields and
the current API version to these pages — do not guess.

See `../SKILL.md` for the send-flow relationship (sending a ShareDocumentsRequest is an ordinary send) and the
lifecycle at a glance. This page picks up from there with the wire detail of each step; step numbers match the
lifecycle in SKILL.md (step 2 — the user shares — is their asynchronous action and has no API of its own).

## Step 1 — Send the ShareDocumentsRequest

The message is an ordinary send: a recipient plus a **primary-document** — a covering letter the
user sees — assembled as multipart and signed, exactly like any send (see
the **digipost-send-post** skill's `references/request-anatomy.md`). You still add the file bytes; the request is not a bodiless
API call. The one thing that makes it a Control request is a `share-documents-request` `data-type` on that
**`primary-document`** (not an attachment), in the datatypes namespace `http://api.digipost.no/schema/datatypes`:

```xml
<primary-document>
    ...
    <data-type>
        <share-documents-request xmlns="http://api.digipost.no/schema/datatypes">
            <max-share-duration-seconds>1209600</max-share-duration-seconds>
            <purpose>Why you need the documentation — shown to the recipient.</purpose>
        </share-documents-request>
    </data-type>
</primary-document>
```

Two fields shape the request:

- **`purpose`** — a short free-text line, shown to the user as the consent prompt explaining *why* you need the
  documentation. Keep it short: the fuller explanation belongs in the **covering letter** you attach. Write both as
  if the recipient will read them, because they will — a vague ask undermines trust and reduces the chance they share.
- **`max-share-duration-seconds`** — how long the share stays valid once granted (the example uses `1209600` = 14 days).
  Ask for what you actually need; a short window is better data-minimisation practice.

Defer the exact `share-documents-request` schema to the
[ShareDocumentsRequest data type](https://digipost.github.io/digipost-technical-docs/data-types/index.md) and the
[Send](https://digipost.github.io/digipost-technical-docs/api-spec/control/send.md) page — don't guess field names.
The Digipost **Java/.NET client libraries have first-class support** for the whole lifecycle (building the request,
reading state, fetching shared content, stopping the share) — prefer them over direct integration; see the
[Java](https://digipost.github.io/digipost-api-client-java/) and
[.NET](https://digipost.github.io/digipost-api-client-dotnet/) client docs.

The response is the same `message-delivery` as any send — see the delivery response and send-time errors in the
**digipost-send-post** skill, and
[response codes](https://digipost.github.io/digipost-technical-docs/api-spec/response-codes.md).

## Step 3 — Discover that documents were shared

Sharing happens asynchronously and on the user's initiative, so you **poll** for it. `GET /documents/events` returns
events; carry the `X-Digipost-UserId` header (the **sender id**, as everywhere else — see
the **digipost** entry skill's shared conventions, not the organisation number) and page through a time window:

- Query params: `from` and `to` (ISO 8601 with timezone), plus `offset` and `maxResults` for paging.
- The response is a `document-events` document with `event` elements. The event whose `type` is
  `SHARE_DOCUMENTS_REQUEST_DOCUMENTS_SHARED` signals that the user shared documents in response to a request.

```xml
<document-events xmlns="http://api.digipost.no/schema/v8"
                 xmlns:data-types="http://api.digipost.no/schema/datatypes">
    <event uuid="9150909b-b041-4744-9491-74bf129194bd"
           type="SHARE_DOCUMENTS_REQUEST_DOCUMENTS_SHARED"
           created="2026-01-26T13:57:28.443+01:00"
           document-created="2026-01-26T13:56:59.140+01:00"/>
</document-events>
```

Practical polling advice to give a developer:

- **Track a cursor instead of polling a fixed interval.** On each poll, set the next `from` to just after the newest
  `created` timestamp you have already processed, so each window continues where the last one ended — no gaps and no
  re-processing. Don't assume events arrive on a regular schedule: a user may share minutes or days after the request,
  or never.
- **Never assume a share will come.** The request itself doesn't expire, and a user may never share — accepting or
  declining is entirely on their initiative. `max-share-duration-seconds` only starts counting down once documents
  are actually shared; it caps how long that share stays open.
- **With only a few outstanding requests, you can skip the event feed entirely** and instead poll each request's state
  directly (step 4): its shared-documents list starts empty and becomes non-empty once the user shares. The event feed
  is worth the extra cursor bookkeeping only when you have many requests outstanding, since one feed call surfaces
  shares across all of them at once.

## Step 4 — Get the state of a request

Once you know something was shared (or any time you want the current picture), read the request state:

`GET /{sender-id}/share-documents-requests/uuid/{uuid}` — substitute your `sender-id` and the request's `uuid`.

The response (`share-documents-request-state`) lists each `shared-document` with metadata and, crucially, **HATEOAS
links** for the actions you can take. Expect elements like `delivery-time`, `subject`, `file-type`,
`file-size-bytes`, an `origin` (either `private-person` or `organisation` as nested XML with attributes), plus `shared-at-time` and `expiry-time`.

The link relations (`rel` values) to look for:

- `https://api.digipost.no/relations/get_shared_document_content_stream` — stream the document bytes.
- `https://api.digipost.no/relations/get_shared_document_content` — get the document content.
- `https://api.digipost.no/relations/stop_sharing` — end the share early.

Follow the links from the response rather than constructing these URLs yourself — that's the point of the links being
there, and it keeps you resilient to path changes. Defer the exact element list to the
[Get state](https://digipost.github.io/digipost-technical-docs/api-spec/control/get-state.md) page.

## Step 5 — Fetch content, mind the expiry, then stop sharing

- **Fetch before `expiry-time`.** Access is time-boxed by the duration the user granted; a shared document is not a
  permanent copy. **When the duration expires, Digipost automatically closes the request** — the window is a hard
  ceiling you cannot extend.
- **Validate the content type** before storing — don't assume PDF. Same discipline as any content download.
- **(Optional) Use `stop_sharing` when you're done.** Ending the share early once you have what you need is good
  data-minimisation and respects the user who granted access. Follow the `stop_sharing` link rather than deleting
  anything. Note this only ever *shortens* the window — it ends the request before `expiry-time`; it cannot push the
  expiry out.

## Signing note

The `POST /messages` in step 1 signs exactly like a normal send (it has a body). The `GET` calls in steps 3–5 are
**bodiless** requests, which signs slightly differently (no content-hash header). See the bodiless-request rules in
the **digipost-auth-and-signing** skill and defer to the
[security docs](https://digipost.github.io/digipost-technical-docs/api/security.md) for the canonical details.

For common snags (nothing shared, content fetch fails, where the request data goes, signing the GETs), see the
**Common snags** table in `../SKILL.md`.
