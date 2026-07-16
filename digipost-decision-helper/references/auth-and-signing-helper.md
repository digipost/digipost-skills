### digipost-auth-and-signing

| Decision | Mode | Implication | Resolved when |
| --- | --- | --- | --- |
| Client library vs hand-roll | 🟡 | The library signs and verifies for you; hand-rolling is error-prone | Inferred from stack; library recommended |
| Certificate: bought vs Digipost-generated | 🟡 | A bought enterprise certificate (e.g. Buypass) means Digipost never holds your private key — required for full non-repudiation; a Digipost-generated one is the quicker start. The docs make this trade-off explicitly the integrator's responsibility | Announced with the trade-off; recorded once — sticky |
| Where the certificate + password live | 🟡 | The private key is a production secret: a secret store or environment config — never the repository | Announced as the practice |
| Verify response signatures | 🟢 | The libraries verify automatically; hand-rolled integrations verify per the security spec before trusting a response | Always verified |
