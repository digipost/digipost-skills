### digipost-control

The send leg inherits the whole **digipost-send-post** table — the covering letter is a real document
with its own `authentication-level`/`sensitivity-level`. These rows are what Control adds:

| Decision | Mode | Implication | Resolved when |
| --- | --- | --- | --- |
| `purpose` (and the covering letter) | 🔴 | `purpose` is shown prominently as the consent prompt; the primary document carries the fuller explanation — both are read, and a vague ask loses trust and lowers the chance they share | Both written meaningfully; templated or per-request |
| `max-share-duration-seconds` | 🔴 | Data-minimisation; the clock starts when the user shares, and the ceiling is hard once granted | Fixed per use-case, or a configurable input |
| Post-fetch retention | 🟡 | Whether you persist the document itself or only your assessment of it — storing less is better data-minimisation, and some document types carry their own retention rules | Announced as a policy; developer confirms |
| Event feed vs per-request polling | 🟡 | Inferable from how many requests are outstanding; correlating events to requests means persisting sent message UUIDs (and events follow the broker identity) | Announced from volume |
| `stop_sharing` when done | 🔵 | Good data-minimisation once you have what you need; active requests can also be cancelled | Offered when relevant |
| Subscription tier | 🔵 | Control is sold in subscription tiers | Flagged early so they aren't blocked mid-integration |
