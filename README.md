# Digipost Skills

A set of [Agent Skills](https://agentskills.io) that model the Digipost API as conceptual **flows**. Each skill gives an
AI agent the correct mental model and the shape of a flow, then defers exact fields, schema, and endpoints to the
official [Digipost technical documentation](https://digipost.github.io/digipost-technical-docs/) (every page is also
served as `.md` for easy ingestion).

Skills follow the open [Agent Skills](https://agentskills.io) standard (`SKILL.md` with `name` + `description`
frontmatter), so they work in Claude Code, Codex CLI, Gemini CLI, Cursor, GitHub Copilot, OpenCode, and other
skills-compatible agents — not just one tool.

## Structure

Each skill is a **self-contained folder** — its `SKILL.md` plus a `references/` directory of flow-specific detail.
There is no shared top-level `references/` directory: cross-cutting mechanics are themselves skills, and skills route to
each other **by name** (never by relative file path), which is what keeps them portable across agents.

```
digipost-skills/
  digipost/                         ← entry point: routes to the others; shared conventions
    SKILL.md
    references/conventions.md       ← sender-id vs org-number, broker rule, media type, test vs prod, client libs
  digipost-auth-and-signing/                 ← shared prerequisite: security headers, request signing, 403 diagnosis
    SKILL.md
  digipost-decision-helper/         ← shared rubric: when & how heavily to involve the developer in a choice
    SKILL.md
    references/                     ← per-flow decision catalogues + the DECISIONS.md convention
      entry-helper.md
      send-post-helper.md
      control-helper.md
      manage-inbox-helper.md
      auth-and-signing-helper.md
  digipost-send-post/               ← flow: send a document to a recipient
    SKILL.md
    references/
      request-anatomy.md
      recipient-identification.md
      physical-mail-fallback.md
  digipost-control/                 ← flow: request access to a document a user holds (ShareDocumentsRequest)
    SKILL.md
    references/
      share-lifecycle.md            ← wire-level lifecycle: XML, events, links, signing
  digipost-manage-inbox/            ← flow: read & manage the organisation's inbox
    SKILL.md
    references/
      inbox-anatomy.md
      document-retrieval-and-delete.md
      scope-and-boundaries.md
```

## Skills

- **digipost** — the entry point. Start here when integrating against Digipost without a specific flow in mind, or for
  cross-cutting questions (which client library, test vs production, sender-id vs organisation number). Routes to the
  flow-specific skills and the signing prerequisite.
- **digipost-auth-and-signing** — the shared prerequisite every flow depends on: security headers, request signing, response
  verification, and diagnosing a `403` / signature error. On the JVM or .NET the client library handles it; other
  languages hand-roll it against the spec and verify before trusting.
- **digipost-decision-helper** — the shared decision rubric every flow uses. Two gates decide *whether* to involve the
  developer in a choice they haven't made, four response modes (ask / announce / offer / silent) decide *how heavily*,
  and per-flow catalogues in its `references/` list each decision point with its qualitative implication. Also surfaces
  value-add features the developer may not know to ask for, and records resolved choices in a `DECISIONS.md` at the
  integration repo's root so they stay sticky and drift is visible.
- **digipost-send-post** — sending a document (digital mail / letter): the message/document model, recipient
  identification, multipart request assembly, and reading the delivery response.
- **digipost-control** — using **Digipost Control** to request access to a document a user holds in Digipost
  (ShareDocumentsRequest): the canonical *politiattest* case, then discovering, reading, fetching, and stopping the
  share. Builds on *digipost-send-post* — the request itself is a normal send — and adds the read side.
- **digipost-manage-inbox** — reading and managing the **organisation's own** inbox: listing received documents,
  downloading content, and deleting after retrieval. (Not a way to read an end-user's personal mailbox — see that
  skill's `references/scope-and-boundaries.md`.)

Further flows can be added as sibling folders following the same pattern: a self-contained `SKILL.md` + `references/`,
routing to `digipost` and `digipost-auth-and-signing` by name.

## Installing

These are standard Agent Skills, so the easiest path is the cross-agent installer
[`npx skills`](https://github.com/vercel-labs/skills), which drops each skill folder into your agent's skills directory
(Claude Code, Codex, Cursor, Gemini CLI, OpenCode, and more):

```
npx skills add digipost/digipost-skills
```

Or install manually by placing (or symlinking) the skill folders into a skills directory your agent scans:

- **Claude Code** (also read by Cursor & OpenCode for compatibility): `~/.claude/skills/` or a project's `.claude/skills/`
- **Codex CLI, Gemini CLI, Cursor, OpenCode**: `~/.agents/skills/` or a project's `.agents/skills/` (the tool-neutral
  location), or each tool's own dir (`~/.codex/skills/`, `~/.gemini/skills/`, `.cursor/skills/`, `.opencode/skills/`)

Keep the folders together — the skills reference each other by name and expect the whole set to be installed.
