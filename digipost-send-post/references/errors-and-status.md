# Reading the send response

This covers the response shape that is **specific to a send**. For the generic HTTP status table, the 400 checklist,
provisioning-vs-code triage, and logging discipline (shared across all flows), see
`../../references/response-codes.md`. For 403 / signature diagnosis, see `../../references/signing-and-auth.md`.

## Success response

A successful send returns `201 Created` with a `message-delivery` body, e.g.:

```xml
<message-delivery xmlns="http://api.digipost.no/schema/v8">
    <delivery-method>DIGIPOST</delivery-method>
    <sender-id>497013</sender-id>
    <status>DELIVERED</status>
    <delivery-time>2016-12-07T08:45:32.675+01:00</delivery-time>
    <primary-document> ... <content-hash .../> </primary-document>
</message-delivery>
```

- `delivery-method` tells you whether it went `DIGIPOST` (digital) or to physical print.
- `status` (e.g. `DELIVERED`) reflects the outcome.
- `content-hash` lets you confirm Digipost received the bytes you sent.

## Send-specific status codes to watch

Read these in the context of the shared status table (`../../references/response-codes.md`):

- `404` on create → the recipient is **not a Digipost user** / deactivated. Identify the recipient; consider physical
  fallback (`physical-mail-fallback.md`).
- `409 Conflict` → you sent file content when the message status was not `NOT_COMPLETE`. Don't re-send content for an
  already-completed message.
- `415` → the multipart request `Content-Type` or a per-part `Content-Type` is wrong (`request-anatomy.md`).
