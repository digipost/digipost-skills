# Digipost Control: the ShareDocumentsRequest lifecycle

This is the wire-level walkthrough of Digipost Control — the exact XML, event and link names, get-state elements, and signing notes. Read `../SKILL.md` first for the mental model and the shape of the flow.

Authoritative specs:
[Send ShareDocumentsRequest](https://digipost.github.io/digipost-technical-docs/api-spec/control/send.md) ·
[Discover shared documents](https://digipost.github.io/digipost-technical-docs/api-spec/control/get-shared.md) ·
[Get state](https://digipost.github.io/digipost-technical-docs/api-spec/control/get-state.md). Defer exact fields and
the current API version to these pages — do not guess.

## Relationship to the send flow

**Sending a ShareDocumentsRequest is a normal send.** It is the same `POST /messages`, the same multipart assembly,
the same signing, and the same `message-delivery` response documented by the **digipost-send-post** skill (see
`../../digipost-send-post/references/request-anatomy.md`). The *only* difference is what rides inside the message: a
`share-documents-request` data-type on the primary document. So everything you know about building and signing a send
transfers directly — reuse it, don't relearn it.

The read-back half (discover + get-state below) is what's genuinely new.

## The lifecycle, end to end

1. **Send the request** — `POST /messages` with a `share-documents-request` on the primary document, stating your
   `purpose` and a `max-share-duration-seconds`. The user receives it and sees the purpose.
2. **The user shares (or doesn't)** — this happens in Digipost, outside your integration. You cannot force or assume it.
3. **Discover that documents were shared** — poll `GET /documents/events` and watch for a
   `SHARE_DOCUMENTS_REQUEST_DOCUMENTS_SHARED` event.
4. **Read the state** — `GET /{sender-id}/share-documents-requests/uuid/{uuid}` to list the shared documents, their
   expiry, and the links to fetch content or stop sharing.
5. **Fetch content** while it is still valid, then optionally **stop sharing**.

## Step 1 — Send the ShareDocumentsRequest

The message is an ordinary send: a recipient plus a **primary-document that is a real PDF** — a covering letter the
user sees — assembled as multipart and signed, exactly like any send (see
`../../digipost-send-post/references/request-anatomy.md`). You still add the file bytes; the request is not a bodiless
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
  documentation. Keep it short: the fuller explanation belongs in the **PDF covering letter** you attach. Write both as
  if the recipient will read them, because they will — a vague ask undermines trust and reduces the chance they share.
- **`max-share-duration-seconds`** — how long the share stays valid once granted (the example uses `1209600` = 14 days).
  Ask for what you actually need; a short window is better data-minimisation practice.

Defer the exact `share-documents-request` schema to the
[ShareDocumentsRequest data type](https://digipost.github.io/digipost-technical-docs/data-types/index.md) and the
[Send](https://digipost.github.io/digipost-technical-docs/api-spec/control/send.md) page — don't guess field names.
The Digipost **Java/.NET client libraries have first-class support** for the whole lifecycle (building the request,
reading state, fetching shared content, stopping the share) — prefer them over direct integration; see the
[Java client docs](https://digipost.github.io/digipost-api-client-java/).

The response is the same `message-delivery` as any send — see the delivery response and send-time errors in the
**digipost-send-post** skill, and
[response codes](https://digipost.github.io/digipost-technical-docs/api-spec/response-codes.md). A successful send means
the *request* was delivered, **not** that anything has been shared yet.

## Step 3 — Discover that documents were shared

Sharing happens asynchronously and on the user's initiative, so you **poll** for it. `GET /documents/events` returns
events; carry the `X-Digipost-UserId` header (the **sender id**, as everywhere else — see
`../../references/conventions.md`, not the organisation number) and page through a time window:

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

- **Advance the window using the `created` timestamps you've already seen**, rather than assuming events arrive in a
  fixed cadence — sharing can happen minutes or days after the request, or never.
- **Never assume a share will come.** Respect `max-share-duration-seconds`: if the window passes with no event, the
  request has effectively lapsed. Don't build logic that blocks on a share that may not happen.
- **Low volume? You can skip the event feed** and just poll the request state (step 4) directly — its shared-documents
  list becomes non-empty once the user shares. The event feed scales better when you have many requests outstanding.

## Step 4 — Get the state of a request

Once you know something was shared (or any time you want the current picture), read the request state:

`GET /{sender-id}/share-documents-requests/uuid/{uuid}` — substitute your `sender-id` and the request's `uuid`.

The response (`share-documents-request-state`) lists each `shared-document` with metadata and, crucially, **HATEOAS
links** for the actions you can take. Expect elements like `delivery-time`, `subject`, `file-type`,
`file-size-bytes`, an `origin` (either `private-person` or `organisation`), plus `shared-at-time` and `expiry-time`.

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
- **Use `stop_sharing` when you're done.** Ending the share early once you have what you need is good
  data-minimisation and respects the user who granted access. Follow the `stop_sharing` link rather than deleting
  anything. Note this only ever *shortens* the window — it ends the request before `expiry-time`; it cannot push the
  expiry out.

## Signing note

The `POST /messages` in step 1 signs exactly like a normal send (it has a body). The `GET` calls in steps 3–5 are
**bodiless** requests, which signs slightly differently (no content-hash header). See the bodiless-request rules in
`../../references/signing-and-auth.md` and defer to the
[security docs](https://digipost.github.io/digipost-technical-docs/API/security.md) for the canonical details.

## Common snags

| Symptom | Likely cause |
| --- | --- |
| "I sent the request but nothing is shared" | Sharing is the user's action and is asynchronous — poll `/documents/events`; it may never come. |
| Content fetch returns nothing / fails later | The share expired (`expiry-time`) or you reused a one-time content link. Fetch promptly while access is live. |
| Not sure where the request data goes | It's a `data-type` on the **primary-document**, in the datatypes namespace — not an attachment. |
| Constructing the content/stop URLs by hand | Use the `rel` links from the get-state response instead. |
| Signing the GET calls like a POST | The GETs are bodiless — different signing input than the send. See `../../references/signing-and-auth.md`. |
