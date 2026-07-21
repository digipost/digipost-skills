### digipost (entry) — the scoping batch

| Decision | Mode | Implication | Resolved when |
| --- | --- | --- | --- |
| Language / stack | 🟡 | Decides client-library vs hand-rolled signing | Inferred from context; announced |
| Test & production | 🔴 | Production means real mail and real charges, this tool is ONLY to be used towards the test environment. Test is a separate host with its own account (issued by Digipost support) — and no real personal data may be sent there | Confirmed test environment; sticky for the session |
| Own account vs broker (on-behalf-of) | 🟡 | Decides which sender-id rides each request and whose inbox/requests you touch; a broker needs a granted right to act for the sender | Inferred from who owns the account; asked once if ambiguous — sticky |
| Overarching functionality | 🔴 | Which flow(s) — and therefore which skill(s) — the integration needs; also the moment to flag high-level functionality the developer hasn't asked for but plausibly wants | Named once; sticky. Proactively mention adjacent functionality they may not know exists |
| Business domain / sector | 🟡 | The relevance signal that fires the 🔵 structured-`data-type` offers downstream — a *regnskapsbyrå* implies invoices and reminders, an HR system implies employment documents | Inferred from what they describe; announced. Feeds the data-type offer in [send-post-helper.md](send-post-helper.md) |
| System shape (fixed vs per-send) | 🟡 | Decides fixed-value vs configurable-input for every material field below | Inferred when the surface makes it obvious; otherwise asked once, alongside the first 🔴 field |
| Prerequisites (sender-id, certificate, test access) | 🔴 | Nothing runs without them; onboarding is manual via Digipost support | Checked once; if missing, routed to the onboarding docs |
