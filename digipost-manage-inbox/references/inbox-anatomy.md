# Inbox listing: what you get back and how to track it

Authoritative spec: [Get Inbox](https://digipost.github.io/digipost-technical-docs/api-spec/inbox.md). Use that page
for the exact element names and the current API version; do not reconstruct the schema from memory.

## What the endpoint returns

`GET` against the organisation's inbox resource (`.../{sender-id}/inbox`) returns a list of **document** entries —
**metadata only** (id, subject, sender, reference-from-sender, delivery/first-accessed times, content-type, …) plus a
**`content-uri`** (where to fetch the bytes) and a **`delete-uri`**. A document entry may contain nested
**`attachment`** entries, each with its own id and `content-uri`. For the canonical element set, read the `inbox-document`
type in the [XSD](https://digipost.github.io/digipost-technical-docs/assets/documents/api_v8.xsd) (v8 at the time of
writing — confirm the current version via the [schema page](https://digipost.github.io/digipost-technical-docs/api/schema.md)).
Note which elements are optional there: e.g. `subject`, `first-accessed`, and `content-type` may be absent — don't
assume them when parsing.

## Pagination and ordering

- The path is keyed by the **sender id** — your Digipost account id, **not** the organisation number. See
  the **digipost** entry skill's shared conventions.
- Listing is paged with **`offset`** and **`limit`** query parameters; the client docs cap `limit` at **1000**. These
  are list positions and have no connection to document ids.
- Set the versioned XML media type in the `Accept` header — see the **digipost** entry skill's shared conventions.
- Documents are ordered by **delivery time** (per the client library docs), so new arrivals shift offset-based pages:
  two consecutive requests with the same `offset`/`limit` may return a different list. Position is not an identifier.
- A practical polling pattern from the Java client docs: page through the inbox and stop as soon as you reach a
  document you have already processed — everything older is unchanged.

## Tracking what you've handled

- **Track by document `id`.** Keep your own record of processed ids and treat it as the source of truth.
- The `first-accessed` element is set the first time the document's **content is fetched** (a download link is
  requested) — **listing the inbox does not set it**, so polling never marks anything as accessed. An absent
  `first-accessed` means nobody has downloaded the content yet; note it flips on *any* content fetch, not just your
  system's, which is why your own id record remains the source of truth for "processed by us".

## Identifying the sender

The `sender` element is the sender's **Digipost name** — full name for a private individual, organisation name for a
business — and it is the **only sender identification an inbox document carries**. It is not necessarily unique, and it
changes if the sender renames itself, so name-based matching is inherently approximate: design for collisions and
renames rather than treating the name as an identifier.

For robust correlation, use the optional **`reference-from-sender`** element when present: it is set by the sender and
carries *their* external reference for the document (e.g. a case or invoice number), which makes it a proper join key.
For senders you can coordinate with, agree that they set it when sending.

## On `authentication-level`

The `authentication-level` on an inbox document is the level required when the document was originally sent to the
organisation. Digipost enforces it when the document is accessed at digipost.no/bedrift, but not over the API. Which level your own
retrieving system applies is up to you.

## Broker vs. on behalf of itself

The sender id in the path is **whose inbox you are reading**. The identity rule is the same across all flows — see
"Broker: acting on behalf of another sender" in the **digipost** entry skill's shared conventions.
