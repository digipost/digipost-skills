# Reading the send response

This covers the response shape and the status codes you are most likely to hit **when sending**. For the generic HTTP
status table, the full 400 checklist, provisioning-vs-code triage, and logging discipline (shared across all flows),
see `../../references/response-codes.md`. For deeper 403 / signature diagnosis, see
`../../references/signing-and-auth.md`.

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

## Status codes to watch when sending

Read these in the context of the shared status table (`../../references/response-codes.md`):

- `400 Bad Request` → missing header, `X-Content-SHA256` mismatch, bad date, blank subject, reused `message-id`,
  unsupported file type, or XML that fails the XSD. Work down the 400 checklist in `../../references/response-codes.md`.
- `403 Forbidden` / "No certificate found" → certificate not uploaded, wrong certificate, or the signature string built
  incorrectly. See `../../references/signing-and-auth.md`.
- `403 NOT_APPROVED_FOR_PREENCRYPT` → you called `encrypt()` / set `pre-encrypt`, but the account is not approved for
  **client-side pre-encryption**. This is a separate capability, distinct from print approval — being approved for print
  does not grant it. Pre-encryption is **not required for print** (the transport is already secured with TLS), so the
  usual fix is to **remove the pre-encryption** rather than add it; only ask Digipost for the capability if you have a real
  requirement to encrypt content before it leaves the sender. This is a provisioning matter, not a code bug.
- `404` on create → the recipient is **not a Digipost user** / deactivated. Identify the recipient
  (`recipient-identification.md`); consider physical fallback (`physical-mail-fallback.md`).
- `409 Conflict` → you sent file content when the message status was not `NOT_COMPLETE`. Don't re-send content for an
  already-completed message.
- `415` → the multipart request `Content-Type` or a per-part `Content-Type` is wrong (`request-anatomy.md`).
