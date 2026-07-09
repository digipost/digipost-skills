# Physical mail and digital-to-physical fallback

Authoritative docs: [Physical Mail](https://digipost.github.io/digipost-technical-docs/physical-mail/index.md) and
[C4 & C5 Envelopes](https://digipost.github.io/digipost-technical-docs/physical-mail/c4-c5.md).

## When this comes up

**The recipient is not a Digipost user**, but the sender still wants the document delivered → fall back to **physical
print and post**.

If you are unsure whether a recipient is a Digipost user, you can check first with `POST /identification` —
not required, even for physical post, but it can come in handy (see `recipient-identification.md`).

Physical print means Digipost prints your document and sends it as ordinary mail to the recipient's postal address. This will only happen if the user is not a Digipost user.

## What stays the same vs. what changes

- **Same flow shape:** you still build a message with a `primary-document` (+ attachments) and POST it to `/messages`
  signed. The conceptual model in `../SKILL.md` does not change.
- **What changes** is the recipient/print configuration in the message and the document formatting requirements for
  print. The exact elements, envelope sizes (C4/C5), and PDF constraints are in the physical-mail docs and the
  [PDF format](https://digipost.github.io/digipost-technical-docs/other/pdf-format.md) page — defer there for the
  specifics rather than reconstructing them.

## Additional information required for print

To have a letter printed and sent physically you must — **in addition to ordinary recipient identification** — provide:

- the recipient's **name and physical postal address**, and
- the **physical return address** (returadresse) to use if the letter cannot be delivered by ordinary post.

Both must be complete/valid postal addresses per Norway Post's requirements; they are printed on a cover sheet and
shown in the envelope window. The exact XML elements live in the
[physical-mail docs](https://digipost.github.io/digipost-technical-docs/physical-mail/index.md) — defer there rather
than guessing element names.

## PDF/print formatting

Documents destined for print have stricter formatting rules (margins, fonts/embedding, page size) than purely digital
ones. Point developers to the [PDF format](https://digipost.github.io/digipost-technical-docs/other/pdf-format.md)
page before they generate print PDFs, to avoid rejected documents.

## Example/test material

Developers often ask for an example PDF for physical mail. Point them to the official samples on the
[physical-mail docs](https://digipost.github.io/digipost-technical-docs/physical-mail/index.md) rather than fabricating one:

- [Valid example PDF](https://digipost.github.io/digipost-technical-docs/assets/documents/gyldig-for-print.pdf) — meets the print formatting requirements.
- [Invalid example PDF](https://digipost.github.io/digipost-technical-docs/assets/documents/ikke-gyldig-for-print.pdf) — illustrates common formatting mistakes that get a document rejected for print.
