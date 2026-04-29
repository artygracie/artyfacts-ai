# Artyfacts Skill

An agent skill that teaches an LLM to proactively save agent work product — documents, research, plans, specs, decisions — as artifacts in [Artyfacts](https://artyfacts.ai), instead of burying them in the chat log.

Install this alongside the [Artyfacts MCP connector](https://artyfacts.ai/docs) to turn "I produced a deliverable" into "the deliverable is filed."

## What this skill does

The Artyfacts MCP connector gives your agent the *ability* to save artifacts. This skill gives it the *habit*. Without the skill, the agent knows the tools exist; with it, the agent knows **when to reach for them** — whenever a response would otherwise produce a multi-paragraph document inline.

Ship both together:

- **Connector** → tools the agent can call
- **Skill** → instructions telling the agent to call them

## Install

Skills don't sync across agent surfaces. Install on each surface where you want proactive saving.

### Claude Code (recommended)

This package ships as a Claude Code plugin that bundles the MCP connector **and** the skill in a single install. From inside Claude Code:

```
/plugin marketplace add artygracie/artyfacts-ai
/plugin install artyfacts@artyfacts
```

That wires up both the remote MCP connector (`https://artyfacts.ai/mcp`) and this skill. No separate `claude mcp add` step needed. Sign in with WorkOS the first time the connector is invoked.

### Other CLI agents

```bash
mkdir -p ~/.claude/skills
cp -r skills/artyfacts ~/.claude/skills/
```

Or for a single project, copy `skills/artyfacts/` into `.claude/skills/` at the project root. For non-Claude CLI agents, use the equivalent skills directory.

You'll also need to register the MCP connector separately on these surfaces — see the [Artyfacts docs](https://artyfacts.ai/docs).

### Desktop & web clients

1. Zip the `skills/artyfacts/` folder (the one containing `SKILL.md`).
2. In your agent's client, open **Settings → Skills → Upload** (or the equivalent).
3. Select the zip.

### API workspaces

Upload via your provider's workspace console — the skill becomes available to every agent run in that workspace.

## Verify it's working

After installing on any surface, try:

> "Draft a one-pager on our Q3 pricing experiment."

The agent should call `save_document_as_artifact`, add sections via `start_section` + `update_section`, and tell you where the artifact lives — without being asked to save.

If the agent produces the document inline instead, the skill isn't loaded. Re-check the install path for your surface.

## Requirements

- The [Artyfacts MCP connector](https://artyfacts.ai/docs) must be connected to the same surface as this skill. The skill instructs the agent to call tools the connector provides; without the connector, the tool calls will fail.
- An agent runtime that supports the `SKILL.md` format (with frontmatter `name` / `description` and markdown body). If your runtime uses a different skill format, the body is portable — port the frontmatter to match.

## Versioning

This skill is versioned independently of the Artyfacts MCP server. Tool names it references:

- `search_artifacts`
- `save_document_as_artifact`
- `start_section`, `update_section`, `edit_section`
- `list_artifacts`, `get_artifact`, `list_sections`, `get_section`
- `create_folder`, `move_artifact`
- `attach_image_from_url`, `attach_image`
- `create_share_link`, `revoke_share_link`
- `share_artifact_with_user`, `share_folder_with_user`
- `list_team_members`, `list_artifact_access`
- `update_collaborator_permission`, `remove_artifact_collaborator`
- `set_artifact_visibility`

If these tool names change on the server, update `SKILL.md` in lockstep.

## License

MIT.
