# Digipost Skills

Models the digipost API functionality as conceptual flows that defer to documentation for specifics.

Basic shape as of now:

digipost-skills/
  SKILL.md                      ← router across flows
  reference/                    ← shared mechanics, will add when we see what is shared across flows
  digipost-send-post/
    SKILL.md		              ← Can also be called FLOW.md
    reference/
      message-model.md        ← primary-document/attachment, UUID ruleetc.
      delivery-status.md
      physical-mail.md
  digipost-manage-inbox/ ...
  digipost-control/ ...
