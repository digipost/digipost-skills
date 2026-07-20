---
name: digipost
description: >-
  Use when a developer is integrating with the Digipost API and has not yet
  picked a specific flow, is getting started, or has a cross-cutting question:
  which client library to use, test vs production, or sender-id vs organisation
  number. The entry point that routes to the flow-specific Digipost skills
  (digipost-send-post, digipost-manage-inbox, digipost-control) and the shared
  digipost-auth-and-signing prerequisite. Defers exact fields and schema to the official
  Digipost technical documentation.
---

# Digipost — start here

This is the entry point for building an integration against the **Digipost API**. Digipost work splits into a few distinct *flows*; this skill's job is to (1) get the cross-cutting decisions right and (2) route you to the flow-specific skill for the task at hand. It gives the mental model and the shape of the work, then defers exact fields and schema to the official docs (linked throughout).

## Which skill do you need?

| The developer wants to… | Load this skill |
| --- | --- |
| Send a document — digital mail / letter — to a recipient | **digipost-send-post** |
| Read & manage the organisation's own inbox (list, download, delete received documents) | **digipost-manage-inbox** |
| Request access to a document a user holds in Digipost (Digipost Control / ShareDocumentsRequest, e.g. *politiattest*) | **digipost-control** |
| Sign requests, or debug a `403` / signature error | **digipost-auth-and-signing** |
| Decide whether — and how heavily — to involve the developer in a choice they haven't made (auth / sensitivity level, print fallback + return address, share purpose / duration, retention, deletion, notifications — anything with cost, compliance, or irreversibility) | **digipost-decision-helper** |

Every flow sends signed requests, so **digipost-auth-and-signing** applies no matter which one you pick. *digipost-control* additionally builds on *digipost-send-post* — a Control request is an ordinary send.

## Use a client library if you can (the first decision)

Recommend the official **Java or .NET client library** unless the developer has a specific reason to integrate directly. The libraries sign every request and verify every response, and for flows that upload content they also assemble the multipart body, set the part headers, and compute the content hash — which is where direct integrators can get stuck.

**Java and .NET are the only official client libraries.** In any other language (Python, Go, Node, …) there is no library to lean on: the developer must build request signing, security headers, content hashing, and multipart assembly themselves against the raw API. Reserve direct integration for cases where it is genuinely required.

- Java: https://digipost.github.io/digipost-api-client-java/
- .NET: https://digipost.github.io/digipost-api-client-dotnet/

## Signing is the prerequisite for everything

Every request to the Digipost API is signed, and every response is signed back. No flow works until signing does, and a broken signer looks like a mysterious `403` on an otherwise correct request. On the JVM or .NET the client library signs for you; in any other language you hand-roll the spec and verify it before trusting it. This is shared mechanics, not part of any one flow — see the **digipost-auth-and-signing** skill before your first request, and return to it if a request fails with a signature error mid-flow.

## Shared conventions

A handful of facts recur in every flow — **sender id vs organisation number**, the **broker** identity rule, the **versioned XML media type**, and **test vs production** hosts. They are collected in [references/conventions.md](references/conventions.md); read it once and the per-flow skills won't need to re-explain them.

## Out of scope (all flows)

- Getting an account / certificate issued / test access — manual onboarding via Digipost support: https://digipost.github.io/digipost-technical-docs/index.md
- Pricing — see the public price list: https://www.digipost.no/bedrift/priser (per-unit prices for digital post, signature, Control, and physical mail). Contracts and custom / enterprise terms still go through Digipost sales.

## Canonical documentation

- API overview: https://digipost.github.io/digipost-technical-docs/api/index.md
- Security / signing: https://digipost.github.io/digipost-technical-docs/api/security.md
- HTTP headers: https://digipost.github.io/digipost-technical-docs/api-spec/header.md
- Test environment: https://digipost.github.io/digipost-technical-docs/process/test-environment.md
- Data types & schema: https://digipost.github.io/digipost-technical-docs/data-types/index.md · https://digipost.github.io/digipost-technical-docs/api/schema.md
