### digipost-manage-inbox

| Decision | Mode | Implication | Resolved when |
| --- | --- | --- | --- |
| Delete after retrieval | 🔴 | Irreversible; deletion is optional | A stated retention policy; download → confirm persisted → delete |
| Access gating for retrieved documents | 🟡 | A document's original `authentication-level` is not enforced over the API — the gate your own system puts in front of downloaded documents is your design | Announced; confirmed for sensitive inboxes |
| Track by document id / cursor | 🟢 | Position is not an identifier; one correct approach | Always by id, never list position |
| Validate downloaded content type | 🟢 | Bytes aren't guaranteed to be PDF; one correct approach | Always checked before persisting |