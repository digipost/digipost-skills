# Physical mail and digital-to-physical fallback

Authoritative docs: [Physical Mail](https://digipost.github.io/digipost-technical-docs/physical-mail/index.md) and
[C4 & C5 Envelopes](https://digipost.github.io/digipost-technical-docs/physical-mail/c4-c5.md).

## When this comes up

Two recurring developer needs:

1. **The recipient is not a Digipost user**, but the sender still wants the document delivered → fall back to **physical
   print and post**.
2. **The sender always wants to send physically** (e.g. invitations, formal letters), bypassing the digital path.

Both depend on the account being **approved for physical print**. A well-formed send can still be rejected with a
permission error (e.g. *not approved for direct print*) — that is a provisioning matter to resolve with Digipost, not a
code fix. See `errors-and-status.md`.

## What stays the same vs. what changes

- **Same flow shape:** you still build a message with a `primary-document` (+ attachments) and POST it to `/messages`
  signed. The conceptual model in `SKILL.md` does not change.
- **What changes** is the recipient/print configuration in the message and the document formatting requirements for
  print. The exact elements, envelope sizes (C4/C5), and PDF constraints are in the physical-mail docs and the
  [PDF format](https://digipost.github.io/digipost-technical-docs/other/pdf-format.md) page — defer there for the
  specifics rather than reconstructing them.

## PDF/print formatting

Documents destined for print have stricter formatting rules (margins, fonts/embedding, page size) than purely digital
ones. Point developers to the [PDF format](https://digipost.github.io/digipost-technical-docs/other/pdf-format.md)
page before they generate print PDFs, to avoid rejected documents.

## Example/test material

Developers often ask for an example PDF for physical mail. Check the
[test environment](https://digipost.github.io/digipost-technical-docs/process/test-environment.md) and physical-mail
docs for current sample resources rather than fabricating one.

## Avoiding the digital path entirely

If the requirement is "always physical, never check Digipost first", that is a configuration of how the message is
addressed/forced to print — confirm the supported mechanism in the physical-mail docs. Do not assume a flag name;
verify it.
