# Scope and boundaries

Where developer expectations and what this API offers diverge. When a request matches one of these, state the boundary
plainly, cite the relevant doc, and route capability questions to Digipost contact — never fabricate an endpoint,
parameter, or feature to satisfy the ask.

## "Read a user's mailbox on their behalf" — not part of this API

The technical documentation exposes no API for reading end-users' personal mailboxes on their behalf, with or without
consent. Don't design an integration around this capability, and don't present the organisation-inbox endpoints as a
way to achieve it.

## Digipost Control is a different flow — not a substitute

**Digipost Control** (the share-documents-request capability) has its own model and is not a general "read someone's
inbox" mechanism. Route questions about it to its documentation:
https://digipost.github.io/digipost-technical-docs/api-spec/control/index.md

## Turning off receipt of mail

Whether and how an organisation can stop receiving mail into its inbox is **account configuration**, not an operation in the inbox API spec. That's managed in the business admin UI at digipost.no/bedrift.

## Read / opening receipts are a send-side concept

Questions like "can I get a status for unopened letters?" or "can I get opening receipts (åpningskvittering)?" are
about documents the organisation has **sent**, not its inbox. Per the
[FAQ](https://digipost.github.io/digipost-technical-docs/other/faq.md) (Norwegian): by default a sender only gets
delivery confirmation, not read status; a separate opening-receipt service exists (the recipient must accept that the
sender is notified), with status visible in the business admin UI at digipost.no/bedrift. Point this need at the
send-side documentation and the FAQ, not the inbox endpoints.
