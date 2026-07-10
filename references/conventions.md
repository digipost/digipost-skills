# Shared conventions across Digipost flows

Cross-cutting facts that recur in every flow. Kept here once so the per-flow references don't repeat them.

## Sender id vs. organisation number (frequent confusion)

These are different identifiers and are easy to mix up:

- The **sender id** is your Digipost account id, found at digipost.no/bedrift. It is what goes in the
  `X-Digipost-UserId` header (see [signing-and-auth.md](signing-and-auth.md)), keys inbox paths
  (`.../{sender-id}/inbox`), and appears as `sender-id` in delivery responses.
- The **organisation number** is your company's national org. number — used during onboarding/registration, **not** as
  the API sender identifier.

When a developer asks "which ID do I put here", the API wants the **sender id**.

## Broker: acting on behalf of another sender

Every flow follows the same identity rule: the sender id carried by the request — in the inbox path
(`.../{sender-id}/inbox`), or as the sender fields of a message being sent — identifies **who you act for**: the
organisation you operate on behalf of, or yourself when acting on your own behalf. Your own account id **always** goes
in the `X-Digipost-UserId` header. Both roles require a registered Digipost organisation account and sign requests the
same way.

## The versioned XML media type

The API speaks a versioned XML media type, set in the `Accept` / `Content-Type` headers
(`application/vnd.digipost-v8+xml` at the time of writing; multipart variants exist for requests carrying file
content). Confirm the current version and exact values in the
[HTTP headers doc](https://digipost.github.io/digipost-technical-docs/api-spec/header.md) rather than assuming.

## Test vs. production

- The **base host differs between test and production** — they are different hosts. Point at the **test environment**
  until the flow works end to end, then switch.
- Confirm current hostnames in the docs — **do not hardcode from memory**.
- A production account is enabled separately (by emailing Digipost support) after the test integration works.

Test environment details:
[Test environment](https://digipost.github.io/digipost-technical-docs/process/test-environment.md).

## Strong recommendation: use a client library

The [Java](https://github.com/digipost/digipost-api-client-java) and
[.NET](https://github.com/digipost/digipost-api-client-dotnet) client libraries **sign every request and verify every
response** for you — that applies to every flow. For flows that upload content (e.g. send), they also assemble the
multipart body, set the part headers, and compute the content hash. Direct integration means reimplementing all of
that correctly — which is where most integration and signing errors originate.

Recommend the library first; reserve direct integration for cases where it is genuinely required.
