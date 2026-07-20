---
name: digipost-decision-helper
description: >-
  Use when building a Digipost integration and you are about to settle a choice
  the developer has not explicitly made — authentication or sensitivity level,
  print fallback and its return address, share purpose and duration, retention
  of fetched content, delete-after-retrieval, notifications, or anything that
  adds cost, affects compliance, sends something real-world, or is irreversible.
  Also use when unsure whether something the developer said already resolves a
  question, or whether to ask, assume-and-announce, offer, or proceed silently.
  The shared decision rubric for every Digipost flow skill (send, inbox,
  Control, auth): two gates decide whether the developer must be involved, four
  response modes decide how heavily, and per-flow catalogues in references/
  list each decision point with its implication. Decisions the developer has
  already made are sticky — never re-ask them.
---

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

The catalogue is split per flow in this skill's `references/` folder — load the one(s) for the flow at hand:

- [references/entry-helper.md](references/entry-helper.md) — the **digipost** entry scoping batch
- [references/send-post-helper.md](references/send-post-helper.md) — **digipost-send-post**
- [references/control-helper.md](references/control-helper.md) — **digipost-control** (the send leg inherits the whole send-post table)
- [references/manage-inbox-helper.md](references/manage-inbox-helper.md) — **digipost-manage-inbox**
- [references/auth-and-signing-helper.md](references/auth-and-signing-helper.md) — **digipost-auth-and-signing**