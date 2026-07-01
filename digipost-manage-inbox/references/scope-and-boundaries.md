# Scope and boundaries (read before designing)

This flow is the place where developer expectations and what the API actually offers diverge most often. The items
below come straight from recurring support questions. When a developer's goal touches one of these, address the
boundary **first** — before discussing endpoints.

## 1. Organisation inbox ≠ end-user personal mailbox

The inbox API exposes the **organisation's own inbox**: documents that have been **sent to your organisation** (as a
Digipost recipient, or received as a broker / on behalf of itself). You are the receiver.

It is **not** a window into an individual end-user's personal Digipost mailbox. Listing "the inbox" never means listing
some other person's postbox.

## 2. "Read a user's mailbox on their behalf" — not part of this API

A recurring request is some variant of: *"we want our system to read end-users' Digipost inboxes, with their consent."*
The Digipost technical documentation does **not** expose an API for reading end-users' personal mailboxes on their
behalf. Do not design an integration around this capability, and do not present the organisation-inbox endpoints as a
way to achieve it — they are a different thing (see point 1).

If a developer specifically needs this, treat it as a **product/capability question for Digipost**, not an
implementation detail to work around. Do not assert what may or may not exist beyond what the docs state — point them
to Digipost contact: https://digipost.github.io/digipost-technical-docs/other/contact.md

## 3. Digipost Control is a different flow — not a substitute

Developers sometimes ask whether **Digipost Control** (the share-documents-request capability) can be used to read a
user's mailbox. Control is its own flow with its own model and is **not** a general "read someone's inbox" mechanism.
If the developer is asking about Control, route them to that flow's documentation rather than improvising here:
https://digipost.github.io/digipost-technical-docs/api-spec/control/index.md

## 4. Turning off receipt of mail

Some organisations ask to **stop receiving mail** into their inbox. Whether and how an organisation can disable receipt
is **account configuration**, not an operation described in the inbox API spec. Do not invent an endpoint or a flag for
it — confirm the supported approach with Digipost:
https://digipost.github.io/digipost-technical-docs/other/contact.md

## 5. Read / opening receipts are a send-side concept, not inbox

Questions like *"can I get a status for unopened letters?"* or *"can I get opening receipts (åpningskvittering)?"* are
about documents an organisation has **sent**, not about its inbox — so they do not belong to this flow. Per the FAQ:

- By default a sender is **not** told whether an individual document has been read; Digipost only confirms delivery.
- There is a **separate opening-receipt service**; if the sender requires it, the recipient must accept that the sender
  is notified, and the sender can see the status in the **business admin web interface** at digipost.no/bedrift.

So: do not promise read/opening status through the inbox API. If this is the developer's real need, point them to the
send-side documentation and the FAQ rather than the inbox endpoints.

FAQ (Norwegian; the page links a Google Translate version):
https://digipost.github.io/digipost-technical-docs/other/faq.md

## How to use this file

When a request matches one of the above, say plainly what the API does and does not do, cite the relevant doc, and —
for capability questions (points 2, 4) — route the developer to Digipost contact instead of constructing a workaround.
Never fabricate an endpoint, parameter, or feature to satisfy one of these asks.