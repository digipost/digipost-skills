# Reading the response and triaging errors

Authoritative spec: [Response Codes](https://digipost.github.io/digipost-technical-docs/api-spec/response-codes.md).
Always read the **error message in the response body** — Digipost includes a human-readable explanation.

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

## HTTP status → meaning → first action

| Code | Meaning | First thing to check |
| --- | --- | --- |
| `200 OK` | Request succeeded (non-create). | — |
| `201 Created` | Message resource created — the send worked. | — |
| `400 Bad Request` | Malformed request (see causes below). | Body encoding, headers, message-id, file-type, XSD validity. |
| `403 Forbidden` | Auth/permission problem. | Certificate uploaded? Signature string correct? Sender id permitted? See `signing-and-auth.md`. |
| `404 Not Found` | When creating a message: recipient is **not a Digipost user** / deactivated. Otherwise a normal not-found. | Identify the recipient; consider physical fallback. |
| `409 Conflict` | You sent file content when the message status was not `NOT_COMPLETE`. | Don't re-send content for an already-completed message. |
| `415` | Unsupported media type. | The request/part `Content-Type` is wrong — check the multipart type and per-part types (`request-anatomy.md`). |
| `500 Internal Server Error` | Error on Digipost's side. | Retry / contact Digipost; not your bug. |

## What a 400 specifically can mean

Per the docs, `400` covers any of:

- A required HTTP header is missing.
- `X-Content-SHA256` does not match the actual body content.
- `Date` is malformed or off from server time by more than five minutes.
- `subject` is blank.
- The `message-id` was used before (reusing an id is rejected — generate a fresh one per message).
- The document/file type is not supported.
- The XML does not validate against the XSD.

Work down that list against the error body; the message usually names the specific cause.

## Is it me or is it Digipost? (provisioning vs. code)

Some failures are **account permissions not yet enabled**, not code bugs — e.g. a 403 where auth is otherwise correct,
or being not approved for a capability such as direct print. When the request is well-formed and signed correctly but
still forbidden/blocked, treat it as a provisioning question for Digipost rather than continuing to change code.

## Logging

Log the full error body on every non-2xx. Digipost deliberately returns detailed messages (including, for signature
errors, the canonical string it computed) — capturing them turns most "why is this failing" loops into a quick diff.
