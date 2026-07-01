# Inbox listing: what you get back and how to track it

Authoritative spec: [Get Inbox](https://digipost.github.io/digipost-technical-docs/api-spec/inbox.md). Use that page
for the exact element names and the current API version; do not reconstruct the schema from memory.

## What the endpoint returns

`GET` against the organisation's inbox resource returns a list of **document** entries — **metadata only**, not file
content. Per the docs, a document entry carries fields such as an `id`, `subject`, `sender`, `delivery-time`,
`first-accessed`, `authentication-level`, `content-type`, plus a **`content-uri`** (where to fetch the bytes) and a
**`delete-uri`** (where to delete it). A document entry may also contain nested **`attachment`** entries, each with its
own `id` and `content-uri`.

> Treat the field list above as orientation. For the canonical, current set of elements and their meaning, read the
> Get Inbox doc and the [schema](https://digipost.github.io/digipost-technical-docs/api/schema.md).

## Endpoint and pagination

- The inbox resource is keyed by the **sender id** in the path (the docs show a path of the form
  `.../{sender-id}/inbox`). The sender id is your Digipost account id, **not** the organisation number — see
  `../../references/conventions.md`.
- Listing is paged with **`offset`** and **`limit`** query parameters (the docs example uses `offset=0&limit=100`).
- The response returns **metadata only**; downloading content is a separate call per document
  (`document-retrieval-and-delete.md`).

## Identifying the sender (do not correlate by display name)

A document carries sender information, but a human-readable **display name is user-editable and not unique**.
Correlating or grouping documents by the display-name *string* is unreliable — it breaks when a sender renames itself or
when two senders share a name. Do not build display-name matching. Use the documented sender identifier fields from the
Get Inbox doc and [schema](https://digipost.github.io/digipost-technical-docs/api/schema.md) instead, and treat the
exact field semantics as a docs matter rather than guessing.

## Tracking what you've handled (important correctness point)

The docs are explicit about this, and it has tripped people up:

- **Do not use list order as an identifier.** The order of returned documents is not stable — two consecutive requests
  with the same `offset`/`limit` may return a different list.
- **Track by the document `id`.** Use the actual document id to record what you have accessed or downloaded.
- **Use the first-access indicator.** The `first-accessed` element lets you tell whether a document is being accessed
  for the first time, which is useful for "process only new documents" logic.

A robust polling loop therefore looks like: list a page → for each document id you have not yet processed, fetch its
content → optionally delete → keep your own record of processed ids. Do not assume page N always contains the same
documents.

## On `authentication-level`

The `authentication-level` on an inbox document reflects the level that was required when the document was originally
sent to the organisation. Per the docs, Digipost enforces that level when the document is accessed at
digipost.no/bedrift, but it is up to the retrieving system which authentication level it applies on its own side. Treat
the precise semantics as a docs matter rather than asserting behaviour here.

## Broker vs. on behalf of itself

An organisation can interact with the inbox either as a **broker** or **on behalf of itself**. Both use the same
security mechanism and a registered Digipost organisation account. Where this changes the request, defer to the
Get Inbox doc and the security documentation rather than guessing.