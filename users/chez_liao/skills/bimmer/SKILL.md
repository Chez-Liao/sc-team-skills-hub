---
name: bimmer
description: Internal Bimmer project knowledge, retrieved live by topic from the company Skill Hub.
---

# Bimmer Internal Skill

**Visibility: INTERNAL** - full content and history are not stored in this public repository.

1. Call `get_internal_skill` from `sc-internal-skills` with `skill_id` set to
   `bimmer` and omit `topic` to retrieve the current Topic Catalog.
2. Select the topic or topics that best match the user's request.
3. Call `get_internal_skill` again for each selected `topic`.
4. Follow the returned topic content as the authoritative latest workflow.

If the MCP tool is unavailable, stop and reply exactly with
`INTERNAL_SKILL_UNAVAILABLE`. Do not guess or reconstruct internal content.
