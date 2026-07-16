### digipost-send-post

Rows ordered by weight: 🔴 gates first, then announcements, offers, and silent defaults.

| Decision | Mode | Implication | Resolved when |
| --- | --- | --- | --- |
| `authentication-level` | 🔴 | How strongly the recipient must authenticate; the schema default is the weakest level, two-factor is expected for financial/personal content and adds a per-message charge; the ID-porten levels are permission-gated and for government agencies only | Fixed per surface, or a per-document input — options + implication shown |
| `sensitivity-level` | 🔴 | Whether metadata (e.g. the subject) shows in notifications and lower-security sessions, or stays hidden until the required login | Fixed per surface, or a per-document input |
| Non-user fallback policy | 🔴 | What happens when the recipient isn't a Digipost user: fail digital-only (404), print fallback via `print-details` (several times the digital price; needs recipient postal address + return address and a print agreement), print only if unread (`print-if-unread`), or invite registration (`request-for-registration`) | A stated policy — fixed or per-send |
| Print sub-choices (when printing) | 🟡 | `color` (monochrome default; colour is a priced add-on) and `nondeliverable-handling` (return-to-sender default vs shred — an ops/privacy call); print PDFs have stricter format rules | Announced with defaults when print is enabled |
| Recipient addressing method | 🟡 | fødselsnummer / Digipost address / name + address (mind the secure-matching rules — misdelivery risk) / organisation number for B2B; the schema defines further variants | Inferred from the data they hold |
| `message-id` & retries | 🟡 | Optional sender-set id; a reused id is rejected, which makes retries safe against double-delivery — fresh id per message, the same id when retrying that message | Announced as the practice |
| Extra attachments | 🟡 | Charged per attachment, and every file counts toward the size-tiered message price | Announced when files are added |
| Notifications (`sms-notification`, `email-notification`) | 🔵 | Recipients already get Digipost's own free notifications; these add scheduled if-unread reminders — SMS carries a per-message charge | Offered; off unless taken |
| Structured `data-type` (invoice, payslip, inkasso, appointment, …) | 🔵 | Unlocks richer mailbox views/actions (e.g. an invoice payable in Digipost); some types are also billed as their own message types, so the correct type matters for billing too | Offered when the content matches a type; exact shape from the data-types docs |
| Document format (PDF vs HTML) | 🔵 | HTML renders inline in Digipost (whitelist-validated); PDF opens separately and is the only print-compatible format | Offered when relevant |
| Scheduled delivery (`delivery-time`) | 🔵 | Deliver at a future time; permission-gated | Offered when the flow suggests it |
| Opening receipt (åpningskvittering) | 🔵 | A separate read-confirmation service; requiring it means the recipient must consent to notifying you before they can read the letter | Offered once; availability confirmed with Digipost |
| Identify-first (`POST /identification`) | 🔵 | Optional pre-check; not mandatory before sending | Offered when user-status is uncertain |
| Encoding (UTF-8) | 🟢 | One correct answer | Always UTF-8 |