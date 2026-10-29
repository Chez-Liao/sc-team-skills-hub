---
name: private-test
description: Validate retrieval of an internal-only Team Skill through the existing public team-skills plugin.
---

# Internal Skill POC

This public file is only a safe routing stub. The internal workflow is not stored
in the public Git repository.

Before doing anything else, call the MCP tool `get_internal_skill` from
`sc-internal-skills` with exactly:

- `skill_id`: `private-test`

Follow the returned instructions exactly.

If the MCP server is unavailable or the tool returns an error, stop and reply
exactly with `INTERNAL_SKILL_UNAVAILABLE`. Do not guess or reconstruct the
internal workflow.
