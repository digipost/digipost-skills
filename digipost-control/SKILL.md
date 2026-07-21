---
name: digipost-control
description: >-
  Use when helping a developer use Digipost Control to request access to a
  document a user holds in Digipost — for example asking a volunteer for
  insight into their valid police certificate (politiattest) before they take
  on a role. The user sends no copy; if they accept the request, you gain
  time-limited insight into the document where it sits in Digipost. Covers the
  ShareDocumentsRequest lifecycle: sending the request (which reuses the normal
  send and signing machinery from the digipost-send-post skill), discovering
  shared documents via the event feed, reading request state, fetching shared
  content before it expires, and stopping the share. Defers exact field lists and
  schema to the official Digipost technical documentation.
---

# Digipost Control — requesting documents *from* a user

This skill helps you guide a developer through **Digipost Control**: instead of sending a document *to* someone, you ask a recipient for **access to a document they already hold in Digipost**. They send you no copy — if they accept your request, you gain time-limited insight into that document where it sits in Digipost. (The document is one they hold in Digipost — received from an organisation or another person, or uploaded by the user themselves.) The canonical case is requesting a police certificate (*politiattest*) from a volunteer before they take on a role. The user stays in control — they see your stated purpose and decide whether to grant access.

It is one *flow* in a larger Digipost integration. This skill gives the **correct mental model and the shape of the flow**, then points to the canonical docs for exact fields. Do not invent field names or values — when a specific schema detail is needed, link the developer to the relevant doc page listed below.

## Prerequisite: this flow builds on `digipost-send-post`

The single most useful thing to understand is that **sending a ShareDocumentsRequest is a normal send.** It is the same `POST /messages`, the same multipart assembly, the same request signing, and the same `message-delivery` response covered by the **digipost-send-post** skill. The *only* difference is what rides inside the message: a `share-documents-request` data-type on the primary document.

So everything about building and signing a send transfers directly — reuse it, don't relearn it:

- Multipart request assembly and the message/document model → the **digipost-send-post** skill (its `references/request-anatomy.md`)
- Security headers and request signing → the **digipost-auth-and-signing** skill (shared mechanics)
- sender-id vs org-number, test vs production, client libraries → the **digipost** entry skill (shared conventions)

What is genuinely **new** in this skill is the **read side** — discovering, reading, fetching, and stopping a share.

## The mental model (read this first)

- **Sending the request is an ordinary send.** A real message with a recipient and a **real primary-document** (a covering letter the user reads), carrying a `share-documents-request` data-type. A successful send means the *request* was delivered — **not** that anything has been shared yet.
- **Sharing is the user's asynchronous action.** It happens in Digipost, on the user's initiative, outside your integration. You cannot force it and must never assume it — it may happen minutes or days later, or never.
- **Access is time-boxed, and the ceiling is hard.** A shared document is not a permanent copy; it expires at `expiry-time`, at which point Digipost **automatically closes the request**. You can end it *earlier* with `stop_sharing`, but you cannot extend it beyond the `max-share-duration-seconds` the user granted.

## The lifecycle, end to end

1. **Send the ShareDocumentsRequest** — `POST /messages` with a `share-documents-request` on the primary document, stating your `purpose` and a `max-share-duration-seconds`. (An ordinary send — see the prerequisite above.)
2. **The user shares (or not)** — asynchronous, on their initiative, in Digipost.
3. **Discover a share** — poll `GET /documents/events` for a `SHARE_DOCUMENTS_REQUEST_DOCUMENTS_SHARED` event.
4. **Read state** — `GET /{sender-id}/share-documents-requests/uuid/{uuid}` to list shared documents, their expiry, and the HATEOAS links.
5. **Fetch content before it expires, then stop sharing** — follow the `rel` links; respect `expiry-time`; use `stop_sharing` when done.

Two principles worth stating up front to a developer: the `purpose` field is a short consent prompt shown to the user (the fuller explanation belongs in the covering letter — both are read), and access is time-boxed. The Java/.NET client libraries have **first-class support for the whole lifecycle** (building the request, reading state, fetching content, stopping the share) — prefer them over direct integration.

See `references/share-lifecycle.md` for the full flow with the exact XML, event and link names, get-state elements, and the bodiless-GET signing note.

## Common snags

| Symptom | Likely cause | Where to look |
| --- | --- | --- |
| "Sent a Control request but nothing is shared" | Sharing is the user's async action — poll `/documents/events`; it may never come | `references/share-lifecycle.md` |
| Content fetch fails / returns nothing | Share expired (`expiry-time`) or a one-time content link was reused; fetch promptly | `references/share-lifecycle.md` |
| Not sure where the request data goes | It's a `data-type` on the **primary-document**, in the datatypes namespace — not an attachment | `references/share-lifecycle.md` |
| Constructing the content/stop URLs by hand | Use the `rel` links from the get-state response instead | `references/share-lifecycle.md` |
| Signing the GET calls like a POST | The GETs are bodiless — different signing input than the send | **digipost-auth-and-signing** skill |

For error HTTP statuses at send time (400, 403, 404, …), see https://digipost.github.io/digipost-technical-docs/api-spec/response-codes.md.

> Sending a document *to* a recipient (the message/document model, multipart assembly, signing, delivery response) is the **digipost-send-post** skill — this flow depends on it.

## Out of scope

- The core send mechanics (multipart, signing, delivery response) — owned by the *digipost-send-post* skill; this flow reuses them.
- Reading or managing the organisation's own inbox — owned by the *digipost-manage-inbox* skill.
- Getting an account / certificate issued / test access — manual onboarding via Digipost support: https://digipost.github.io/digipost-technical-docs/index.md
- Pricing — see the public price list: https://www.digipost.no/bedrift/priser (Control is sold in subscription tiers). Contracts and custom / enterprise terms still go through Digipost sales.

## Canonical documentation

- Send ShareDocumentsRequest: https://digipost.github.io/digipost-technical-docs/api-spec/control/send.md
- Discover shared documents: https://digipost.github.io/digipost-technical-docs/api-spec/control/get-shared.md
- Get state: https://digipost.github.io/digipost-technical-docs/api-spec/control/get-state.md
- Data types & schema: https://digipost.github.io/digipost-technical-docs/d · https://digipost.github.io/digipost-technical-docs/api/schema.md
- Security / signing: https://digipost.github.io/digipost-technical-docs/api/security.md
- Response codes: https://digipost.github.io/digipost-technical-docs/api-spec/response-codes.md
- Client libraries: [Java](https://digipost.github.io/digipost-api-client-java/) · [.NET](https://digipost.github.io/digipost-api-client-dotnet/)
