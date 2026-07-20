# Decision points: when (and how) a Digipost skill involves the developer

This is the shared rubric every Digipost flow skill routes to **by name**. It defines *when* an agent
should stop and involve the developer, and *how heavily* — so beginners stay in control and informed
of features they might want, while developers who already have a plan aren't asked things they've
answered. Read it once; each flow skill carries only a short "Decision points" block that points here.

Two framing facts shape everything below:

- **"The user" is a developer building an integration — not the end recipient.** A decision is rarely
  "pick one value for one send." It is "how should this integration handle this field," which usually
  depends on the shape of their system.
- **We can't know that shape up front.** They may have a central outbox with per-send dropdowns, or
  split functionality across purpose-specific surfaces (an *invoice* page, a *documents* page, a
  *Control* page) where a value is naturally fixed. So never silently hardcode a field a developer
  might want to vary.

## The two gates — ask only if BOTH are true

1. **Material** — getting it wrong costs money, breaks compliance, is irreversible, sends something
   real-world, or silently forgoes a feature the developer would plausibly want.
2. **Unresolved** — the developer hasn't already stated it, and it can't be safely inferred from their
   stated goal, their stack, their data, or the shape of their system.

If either gate is false, don't ask. A field with one obvious answer (gate 1 false) or one the
developer has already pinned down (gate 2 false) is simply used.

## Response modes — lightest touch first

| Mode | Use when | How it reads |
| --- | --- | --- |
| 🟢 **Silent default** | Trivial, reversible, one correct answer | *(no message — just proceed)* |
| 🟡 **Assume & announce** | Reversible; a safe default exists; the developer should know | "I'll use X — tell me if that's not right" |
| 🔴 **Ask before proceeding** | Material with no safe default: compliance, cost, irreversible, go-live | A real question; wait for the answer |
| 🔵 **Offer once** | A value-add the developer may not know exists | One line; mention once, don't nag |

**Lightweight default:** only 🔴 gates stop the agent. Everything else announces or offers and keeps
moving. (Easy to tighten later if developers want more hand-holding.)

## Resolving a material decision: a fixed value *or* a configurable input

This is the part that matters for fields like `sensitivity-level` and `authentication-level`, where a
developer may want to expose the choice in their own UI (a dropdown) rather than fix it.

A material decision is **resolved** when the developer understands the options and their implications
and has chosen **where the value comes from** — *not* necessarily when a single value has been
extracted. Two valid resolutions:

- **Fixed value** — the same for every send on this surface (e.g. an *invoice* page that is always
  `TWO_FACTOR`). Capture it once; it is then sticky for that surface.
- **Configurable input** — the value varies by document, recipient, or context, so it belongs as a
  labelled input in the developer's system (a dropdown, config, or a value derived from document
  type). Here the deliverable is a **well-labelled input with the implication surfaced to whoever
  picks it** — not a hardcoded constant.

So for a 🔴 field the obligation is discharged by making the options and their implications clear and
confirming fixed-vs-configurable — **never by quietly choosing a value the developer didn't see.**
When the system's shape is still unknown, ask once (it's cheap and resolves many downstream fields):
*"Is this fixed for the whole flow, or something your users pick per send?"* A purpose-specific
surface usually fixes the value; a general outbox usually exposes it.

## Balance levers

- **Front-load one scoping batch** in the **digipost** entry skill — stack, environment, own-account
  vs broker, flow, volume, prerequisites, and the system-shape question above — so downstream skills
  inherit the answers.
- **"If unspecified"** on every gate: the phrase that suppresses questions for developers who have
  already decided.
- **Sticky decisions** — never re-ask environment, stack, sender-id, broker role, certificate choice,
  or an already-resolved field within a session or a surface.
- **Inform ≠ ask** — 🔵 offers are one line, never a gate.

## How to phrase it

- 🔴 *fixed:* "`sensitivity-level` controls whether the subject shows in the email/SMS notification or
  stays hidden until the recipient logs in. For payslips you'll likely want the hidden option. Fix it
  to that for this flow?"
- 🔴 *configurable:* "Since this outbox sends mixed content, I'll expose `sensitivity-level` as a
  dropdown — one option leaves the subject visible in notifications, the other hides it until login —
  so your operators choose per send. Sound right?"
- 🟡 "I'll address recipients by fødselsnummer since that's what your data has — say if you need
  name + address instead."
- 🔵 "Digipost can also send an SMS reminder if the message goes unread (small extra charge) —
  want that on?"

Do **not** invent field names, values, or endpoints when surfacing a choice — name only what the flow
skills and the [schema](https://digipost.github.io/digipost-technical-docs/api/schema.md) already
define, and defer exact values there.

## The decision-point catalogue

Legend: 🔴 ask · 🟡 announce · 🔵 offer · 🟢 silent. **Resolved when** states what discharges the gate.

Implications are deliberately **qualitative** — "adds a charge", "subscription-gated" — because prices
and terms change; current amounts live in the [price list](https://www.digipost.no/bedrift/priser).
Field names below are the schema's own; defer exact values and structure to the linked docs.

### digipost (entry) — the scoping batch

| Decision | Mode | Implication | Resolved when |
| --- | --- | --- | --- |
| Language / stack | 🟡 | Decides client-library vs hand-rolled signing | Inferred from context; announced |
| Test & production | 🔴 | Production means real mail and real charges, this tool is ONLY to be used towards the test environment. Test is a separate host with its own account (issued by Digipost support) — and no real personal data may be sent there | Confirmed test environment; sticky for the session |
| Own account vs broker (on-behalf-of) | 🟡 | Decides which sender-id rides each request and whose inbox/requests you touch; a broker needs a granted right to act for the sender | Inferred from who owns the account; asked once if ambiguous — sticky |
**| Overarching functionality to be implemented  | 🔴 | Routes to the correct flow skill(s) | Inferred from the goal; ask if ambiguous |**
| [DELETE]Volume (one-off vs bulk) | 🟡 | Shapes looping and polling advice; the REST API handles large batches — the legacy SFTP batch interface is closed to new integrations | Inferred or announced |
| System shape (fixed vs per-send) | 🟡 | Decides fixed-value vs configurable-input for every material field below | Inferred when the surface makes it obvious; otherwise asked once, alongside the first 🔴 field |
| Prerequisites (sender-id, certificate, test access) | 🔴 | Nothing runs without them; onboarding is manual via Digipost support | Checked once; if missing, routed to the onboarding docs |

### digipost-send-post

Rows ordered by weight: 🔴 gates first, then announcements, offers, and silent defaults.

| Decision | Mode | Implication | Resolved when |
| --- | --- | --- | --- |
| `authentication-level` | 🔴 | How strongly the recipient must authenticate; the schema default is the weakest level, two-factor is expected for financial/personal content and adds a per-message charge; the ID-porten levels are permission-gated and for government agencies only | Fixed per surface, or a per-document input — options + implication shown |
| `sensitivity-level` | 🔴 | Whether metadata (e.g. the subject) shows in notifications and lower-security sessions, or stays hidden until the required login | Fixed per surface, or a per-document input |
| Non-user fallback policy | 🔴 | What happens when the recipient isn't a Digipost user: fail digital-only (404), print fallback via `print-details` (several times the digital price; needs recipient postal address + return address and a print agreement), print only if unread (`print-if-unread`), or invite registration (`request-for-registration`) | A stated policy — fixed or per-send |
| [DELETE]Test → production switch | 🔴 | Go-live: real mail, real charges; production access is enabled by Digipost support once test works | Confirmed once (sticky from entry) |
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

### digipost-control

The send leg inherits the whole **digipost-send-post** table — the covering letter is a real document
with its own `authentication-level`/`sensitivity-level`. These rows are what Control adds:

| Decision | Mode | Implication | Resolved when |
| --- | --- | --- | --- |
| `purpose` (and the covering letter) | 🔴 | `purpose` is shown prominently as the consent prompt; the primary document carries the fuller explanation — both are read, and a vague ask loses trust and lowers the chance they share | Both written meaningfully; templated or per-request |
| `max-share-duration-seconds` | 🔴 | Data-minimisation; the clock starts when the user shares, and the ceiling is hard once granted | Fixed per use-case, or a configurable input |
| [Jobbe litt med] Origin restriction (`allowed-origin-organisation-numbers`) | 🟡 | Limits sharing to documents actually received from the named organisations — without it, a self-uploaded file can satisfy the request; pair with checking each shared document's `origin` | Announced; restricted when the use-case targets official documents (e.g. politiattest) |
| Post-fetch retention | 🟡 | Whether you persist the document itself or only your assessment of it — storing less is better data-minimisation, and some document types carry their own retention rules | Announced as a policy; developer confirms |
| Event feed vs per-request polling | 🟡 | Inferable from how many requests are outstanding; correlating events to requests means persisting sent message UUIDs (and events follow the broker identity) | Announced from volume |
| `stop_sharing` when done | 🔵 | Good data-minimisation once you have what you need; active requests can also be cancelled | Offered when relevant |
| Subscription tier | 🔵 | Control is sold in subscription tiers | Flagged early so they aren't blocked mid-integration |

### digipost-manage-inbox

| Decision | Mode | Implication | Resolved when |
| --- | --- | --- | --- |
| Delete after retrieval | 🔴 | Irreversible; deletion is optional | A stated retention policy; download → confirm persisted → delete |
| Access gating for retrieved documents | 🟡 | A document's original `authentication-level` is not enforced over the API — the gate your own system puts in front of downloaded documents is your design | Announced; confirmed for sensitive inboxes |
| [DELETE?]`reference-from-sender` correlation | 🟡 | The robust join key when present — sender names are neither unique nor stable | Announced when correlating |
| Track by document id / cursor | 🟢 | Position is not an identifier; one correct approach | Always by id, never list position |
| Validate downloaded content type | 🟢 | Bytes aren't guaranteed to be PDF; one correct approach | Always checked before persisting |

### digipost-auth-and-signing

| Decision | Mode | Implication | Resolved when |
| --- | --- | --- | --- |
| Client library vs hand-roll | 🟡 | The library signs and verifies for you; hand-rolling is error-prone | Inferred from stack; library recommended |
| Certificate: bought vs Digipost-generated | 🟡 | A bought enterprise certificate (e.g. Buypass) means Digipost never holds your private key — required for full non-repudiation; a Digipost-generated one is the quicker start. The docs make this trade-off explicitly the integrator's responsibility | Announced with the trade-off; recorded once — sticky |
| Where the certificate + password live | 🟡 | The private key is a production secret: a secret store or environment config — never the repository | Announced as the practice |
| Verify response signatures | 🟢 | The libraries verify automatically; hand-rolled integrations verify per the security spec before trusting a response | Always verified |

---

**For skill authors:** a flow skill should carry only a compact block — *"Decision points (ask only if
unspecified) — see the **digipost** skill's `references/decision-points.md`"* — plus its own 🔴 gates
and 🔵 offers from the table above. Keep any named values illustrative and defer the exact schema to
the [data types](https://digipost.github.io/digipost-technical-docs/data-types/index.md) and
[XSD schema](https://digipost.github.io/digipost-technical-docs/api/schema.md) docs.
