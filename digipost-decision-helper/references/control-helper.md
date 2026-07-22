### digipost-control

The send leg inherits the whole **digipost-send-post** table — the covering letter is a real document
with its own `authentication-level`/`sensitivity-level`. These rows are what Control adds:

| Decision | Mode | Implication | Resolved when |
| --- | --- | --- | --- |
| `purpose` (and the covering letter) | 🔴 | `purpose` is shown prominently as the consent prompt; the primary document carries the fuller explanation — both are read, and a vague ask loses trust and lowers the chance they share | Both written meaningfully; templated or per-request |
| `max-share-duration-seconds` | 🔴 | Data-minimisation; the clock starts when the user shares, and the ceiling is hard once granted | Fixed per use-case, or a configurable input |
| Post-fetch retention | 🔴 | Whether you persist the fetched document at all. The point of Control is that you *don't need to* store it — you request access, read or assess it, and let it go; keeping the document itself reintroduces exactly the data you were avoiding holding (and some document types carry their own retention rules). We generally do **not** want the document stored — keep only your assessment unless there's a specific reason not to | A deliberate developer choice, not an assumption — surfaced for the developer to decide either way |
| Event feed vs per-request polling | 🟡 | Inferable from how many requests are outstanding; correlating events to requests means persisting sent message UUIDs (and events follow the broker identity) | Announced from volume |
| `stop_sharing` when done | 🔵 | Good data-minimisation once you have what you need; active requests can also be cancelled | Offered when relevant |
| Subscription tier | 🔴 | Control is sold in subscription tiers, and you must choose one to use the service at all — that's the developer's call, not something the integration can pick for them | Confirmed the developer has chosen a tier — "using Control requires a subscription model; has that been set up?" |
