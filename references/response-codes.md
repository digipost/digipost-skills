# Reading responses and triaging errors (shared across flows)

The HTTP status semantics and error-handling discipline are the same across Digipost flows. Authoritative spec:
[Response Codes](https://digipost.github.io/digipost-technical-docs/api-spec/response-codes.md). Always read the
**error message in the response body** — Digipost includes a human-readable explanation, and for signature errors it
echoes the canonical string it computed.

## HTTP status → meaning → first action

| Code | Meaning | First thing to check |
| --- | --- | --- |
| `200 OK` | Request succeeded (non-create). Inbox list, content fetch, and `DELETE` all return `200`. | — |
| `201 Created` | A resource was created — e.g. a send worked (`message-delivery` body). | — |
| `307 Temporary Redirect` | Normal redirect. Inbox content fetch redirects to a one-time, short-lived URL — follow it immediately. | Flow-specific; for inbox see the flow's document-retrieval reference. |
| `400 Bad Request` | Malformed request (see causes below). | Body encoding, headers, message-id, file-type, XSD validity. |
| `403 Forbidden` | Auth/permission problem. | Certificate uploaded? Signature string correct? Sender id permitted? See [signing-and-auth.md](signing-and-auth.md). |
| `404 Not Found` | When creating a message: recipient is **not a Digipost user** / deactivated. Otherwise a normal not-found. | Identify the recipient; consider physical fallback. |
| `409 Conflict` | You sent file content when the message status was not `NOT_COMPLETE`. | Don't re-send content for an already-completed message. |
| `415` | Unsupported media type. | The request/part `Content-Type` is wrong — check the multipart type and per-part types. |
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
