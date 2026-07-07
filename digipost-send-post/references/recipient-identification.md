# Identifying the recipient

Authoritative spec: [Identify Recipient](https://digipost.github.io/digipost-technical-docs/api-spec/identify.md).

## Why identify first

You can send without identifying first, but a separate `POST /identification` lets you check **whether a person is a
Digipost user** before sending — that is the only question it answers. It says nothing about delivery preferences or
physical mail; whether to fall back to physical mail is a decision your own code makes, with the identification result
as one possible input. Useful for updating a customer database with Digipost addresses.

Identifying first is **optional in all cases — including sends that fall back to physical mail**. It can still come in
handy when you are unsure whether a recipient is a Digipost user, so your code can decide whether to send digitally or
fall back to print (see `physical-mail-fallback.md`).

## How a recipient can be addressed

A recipient can be specified by any of:

- **Personal identification number** (fødselsnummer) — the most common and most reliable.
- **Digipost address** (e.g. `ola.nordmann#1234`).
- **Name and street address.**

The exact XML for each variant is in the [XSD schema](https://digipost.github.io/digipost-technical-docs/api/schema.md);
defer there rather than guessing element names.

## Identification results

`POST /identification` returns one of:

- `DIGIPOST` — the person is a Digipost user (safe to send digital mail).
- `IDENTIFIED` — the person was identified but is **not** a Digipost user. A returned `person-alias` can be reused in
  later identification.
- `UNIDENTIFIED` — Digipost could not identify the person from the supplied information.

You can rely on the returned Digipost user being the same person specified in the request.

## On secure identification by address + birth date

Some integrators ask how a recipient is *securely* matched (e.g. by name/address or by fødselsnummer). The matching
rules and what each identifier guarantees live in the spec and schema — point developers there rather than describing
matching behaviour from memory, since this touches correctness and privacy.

## Sender id vs. organisation number (frequent confusion)

Don't confuse the identifier that addresses the *recipient* (fødselsnummer / Digipost address / name+address, above)
with the identifier that authenticates *you* the sender. The API header wants your **sender id**, not your organisation
number — full explanation in `../../references/conventions.md`.

## If the recipient is not a Digipost user

A send to a non-user returns **404** unless physical-mail fallback applies. See `physical-mail-fallback.md`.
