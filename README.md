# Digipost Skills

A set of [Agent Skills](https://agentskills.io) that model the Digipost API as conceptual **flows**. Each skill gives an
AI agent the correct mental model and the shape of a flow, then defers exact fields, schema, and endpoints to the
official [Digipost technical documentation](https://digipost.github.io/digipost-technical-docs/) (every page is also
served as `.md` for easy ingestion).

## Structure

```
digipost-skills/
  references/                       ← shared mechanics, used by every flow
    signing-and-auth.md             ← security headers, request signing, certificates, 403 diagnosis
    response-codes.md               ← HTTP status triage, reading error bodies
    conventions.md                  ← sender-id vs org-number, test vs production, client libraries
  digipost-send-post/               ← flow: send a document to a recipient
    SKILL.md
    references/                      ← flow-specific
      request-anatomy.md
      recipient-identification.md
      physical-mail-fallback.md
      errors-and-status.md
      digipost-control.md          ← request documents back from a user (ShareDocumentsRequest)
  digipost-manage-inbox/            ← flow: read & manage the organisation's inbox
    SKILL.md
    references/                      ← flow-specific
      inbox-anatomy.md
      document-retrieval-and-delete.md
      scope-and-boundaries.md
```

## How references resolve

Within a skill, a link to `references/x.md` points at that skill's own flow-specific file, while `../references/x.md`
points at the repo-root **shared** references above. The shared files live outside the individual skill directories, so
these skills are designed to ship **together from this repo** rather than be copied out individually.

## Skills

- **digipost-send-post** — sending a document (digital mail / letter): the message/document model, recipient
  identification, multipart request assembly, signing, and reading the delivery response. Also covers **Digipost
  Control** — using the same send machinery to request documents *back* from a user (ShareDocumentsRequest), then
  discovering, reading, fetching, and stopping the share.
- **digipost-manage-inbox** — reading and managing the **organisation's own** inbox: listing received documents,
  downloading content, and deleting after retrieval. (Not a way to read an end-user's personal mailbox — see that
  skill's `references/scope-and-boundaries.md`.)

Further flows can be added as sibling directories following the same pattern.
