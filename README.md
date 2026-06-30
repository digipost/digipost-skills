# Digipost Skills

Models the digipost API functionality as conceptual flows that defer to documentation for specifics.

Basic shape as of now:

digipost/
  SKILL.md                      ← router across flows
  reference/                    ← shared mechanic, will add when we see what is shared across flows
  send-document/
    SKILL.md		      ←Can also be called FLOW.md
    reference/
      message-model.md        ← primary-document/attachment, UUID ruleetc.
      delivery-status.md
      physical-mail.md
  manage-inbox/ ...
  control/ ...
