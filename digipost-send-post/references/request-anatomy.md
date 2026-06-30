# Request anatomy: how a send is actually shaped on the wire

This explains the structure most people get wrong. For the authoritative field list and the latest API version, see the
[Send](https://digipost.github.io/digipost-technical-docs/api-spec/send.md) and
[Attachments](https://digipost.github.io/digipost-technical-docs/api-spec/attachment.md) docs.

## One request, multiple parts

Sending is a single `POST /messages`. The body is **multipart**, with a media type like
`multipart/vnd.digipost-v8+xml; boundary=<your-boundary>`. (`v8` is the API version at time of writing — confirm the
current version in the docs rather than assuming.)

The parts, in order:

1. **The message metadata part** — the message XML. Its part headers are:
   - `Content-Type: application/vnd.digipost-v8+xml`
   - `Content-Disposition: attachment; filename="message"`
2. **One content part per document** — the raw file bytes (PDF, etc.). Each content part has:
   - `Content-Type:` matching the file (e.g. `application/pdf`, `text/plain`)
   - `Content-Disposition: attachment; filename=<UUID>` where `<UUID>` is **exactly** the `uuid` of the matching
     document in the XML.

> The request-level `Content-Type` is the *multipart* type with a boundary; each *part* then carries its own
> `Content-Type` / `Content-Disposition`. This two-level structure is why developers sometimes report that "the
> Content-Type isn't documented" — they are looking for a single value when there are several.

## The message XML

The XML names the recipient and lists document **metadata only** — not the bytes:

```xml
<?xml version="1.0" encoding="utf-8"?>
<message xmlns="http://api.digipost.no/schema/v8">
    <recipient>
        <personal-identification-number>01043100358</personal-identification-number>
    </recipient>
    <primary-document>
        <uuid>6d99008e-2672-4b55-9b09-996b09a06e47</uuid>
        <subject>primary-pdf-document</subject>
        <file-type>pdf</file-type>
        <authentication-level>PASSWORD</authentication-level>
        <sensitivity-level>NORMAL</sensitivity-level>
    </primary-document>
    <!-- zero or more attachments -->
    <attachment>
        <uuid>2bade07d-774f-4cbb-a832-1299f2373a71</uuid>
        <subject>attachment-text-document</subject>
        <file-type>txt</file-type>
        <authentication-level>PASSWORD</authentication-level>
        <sensitivity-level>NORMAL</sensitivity-level>
    </attachment>
</message>
```

- **Exactly one `primary-document`.** This is what the recipient opens first.
- **Multiple files = one `attachment` element per extra file.** There is no separate "multi-file" endpoint or mode.
- For the full set of fields and allowed values (`file-type`, `authentication-level`, `sensitivity-level`, recipient
  variants, etc.), defer to
  [Data types](https://digipost.github.io/digipost-technical-docs/data-types/index.md) and the
  [XSD schema](https://digipost.github.io/digipost-technical-docs/api/schema.md). Do not guess these.

## The UUID rule (most common metadata/content mismatch)

The `filename` on each content part **must equal** the `uuid` of the corresponding document element in the XML. This is
how Digipost matches bytes to metadata. A mismatch (or a missing content part) is a frequent cause of failed sends.

## Encoding

The XML body must be **UTF-8**. Norwegian characters (æ, ø, å) garbling is almost always a body that was encoded as
something else (e.g. ISO-8859-1) or a mismatched `encoding` declaration. Ensure both the actual byte encoding and the
`<?xml ... encoding="utf-8"?>` declaration are UTF-8.

## Endpoint and path

- Resource path is `/messages` (POST). When building the request or the string-to-sign, use the path exactly — the
  leading slash matters for signing (see `signing-and-auth.md`).
- The **base host differs between test and production**. Use the test host until the flow works, then switch. Confirm
  current hostnames in the [test environment docs](https://digipost.github.io/digipost-technical-docs/process/test-environment.md).

## Strong recommendation: use a client library

The [Java](https://github.com/digipost/digipost-api-client-java) and [.NET](https://github.com/digipost/digipost-api-client-dotnet) client libraries assemble the multipart body,
set the part headers, compute the content hash, and sign the request for you. Direct integration means reimplementing
all of that correctly — which is where most send-time errors originate. Recommend the library first; reserve direct
integration for cases where it is genuinely required.
