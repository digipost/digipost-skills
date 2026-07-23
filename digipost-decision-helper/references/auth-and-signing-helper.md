### digipost-auth-and-signing

| Decision | Mode | Implication | Resolved when |
| --- | --- | --- | --- |
| Client library vs hand-roll | 🟡 | The library signs and verifies for you; hand-rolling is error-prone | Inferred from stack; library recommended |
| Certificate: bought vs Digipost-generated | 🟡 | A bought enterprise certificate (i.e. Buypass) means Digipost never holds your private key — required for full non-repudiation; a Digipost-generated one is the quicker start. The docs make this trade-off explicitly the integrator's responsibility | Announced with the trade-off; recorded once — sticky |
| Keystore + password storage | 🟡 | The private key is a production secret and the basis of your identity to Digipost — keep the keystore out of the repo and the password in a secret store or environment config, never in source. If it's ever exposed, treat it as compromised and rotate | Announced as practice; exposed-key handling in the auth-and-signing skill |
| Verify response signatures | 🟢 | The libraries verify automatically; hand-rolled integrations verify per the security spec before trusting a response | Always verified |
