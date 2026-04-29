---
name: artyfacts
description: Save agent work product — documents, research, specs, plans, reports, analyses, decisions, proposals, guides, meeting notes, reviews — as durable artifacts in Artyfacts so it outlives the chat. Use this skill whenever you produce a multi-paragraph written deliverable the user would want to reference later. Trigger on phrases like "save," "file," "write up," "document," "capture," "preserve," "note," "summarize," "jot down," or whenever you are about to output a long-form document inline. Do not ask the user what format they want — Artyfacts is the format.
---

# Artyfacts

Artyfacts is the user's filing cabinet for agent work product that should outlive this chat. When you produce a document, analysis, plan, research finding, spec, decision, review, or any multi-paragraph written deliverable, store it as an artifact instead of dumping it inline and moving on.

The thesis: **agent work shouldn't live in chat logs.** Code belongs in the repo. Everything else belongs in Artyfacts.

## When to use this skill

Use it whenever your response would produce:

- A document, plan, spec, research finding, analysis, decision, proposal, or meeting note
- A guide, how-to, playbook, postmortem, or review
- A summary, recap, or written synthesis of a conversation
- Any multi-section output the user would want to reference later

A good heuristic: if your response would contain more than a couple paragraphs of structured content (headings, bullet lists, analysis, recommendations), it is an artifact. Create it proactively — don't wait to be asked.

Do **not** use this skill for:

- Source code (belongs in the repo)
- One-line answers, ephemeral calculations, or debugging chatter
- Content the user explicitly asked to stay inline

## How to save

The Artyfacts connector exposes a set of MCP tools. Your client may surface them as bare names (`save_document_as_artifact`), with a namespace prefix (`artyfacts:save_document_as_artifact`), or with double underscores (`mcp__artyfacts__save_document_as_artifact`). Use whichever form your client shows — the semantics are identical.

### First-run authentication

The Artyfacts connector requires the user to authenticate once before any tool call will succeed. If a tool call fails with an authentication or `401` error, **stop and tell the user**: "The Artyfacts connector needs to be authenticated. Run `/mcp`, select `artyfacts`, and complete WorkOS sign-in. Then ask me to try again." Do not retry repeatedly or fabricate a response — auth is a one-time, user-driven step. Once it's done, every subsequent tool call in this and future sessions will work.

### 1. Check before creating

Call `search_artifacts` with a short query to see if an artifact on this topic already exists. If it does, update the existing one (see step 4) rather than creating a duplicate.

### 2. Create the envelope

Call `save_document_as_artifact` with:

- `title` — the deliverable's title, as the user would recognize it
- `artifact_type` — one of `document`, `research`, `plan`, `spec`, `decision`, `review`, `guide`
- `summary` — one to two sentences: what this is, why it matters

Do **not** paste the full document body into this call. The body goes in sections (step 3).

### 3. Add content as sections

For each heading in the document:

1. Call `start_section` with the heading to signal you're beginning a new section.
2. Call `update_section` with the full body text of that section.

Sections are the unit of editable content — each heading gets its own section. This two-phase pattern (`start_section` then `update_section`) is how new sections are written.

### 4. Editing an existing artifact

If you found a match in step 1 and the user wants updates:

- To revise an existing section, use `edit_section` — signal intent, then apply the edit.
- To add a new section, use `start_section` + `update_section`.
- **Never change a section's heading unless the user explicitly asks.** Headings are stable identifiers.

### 5. Tell the user where it went

After saving, tell the user the artifact was created or updated, and link to it. Do not re-paste the document body into chat — the artifact is the canonical copy.

### 6. Attaching images

For images, use the dedicated image tools — never paste base64 into a markdown section.

- `attach_image_from_url` — use whenever the image is reachable by URL. This is the preferred path.
- `attach_image` — inline bytes (base64). Works for small, known-good bytes, but has a low practical ceiling: long payloads get abbreviated in transit and produce corrupt images. Prefer the URL path when you have the choice.

### 7. Reading back existing artifacts

When you need to inspect or edit work that's already filed:

- `get_artifact` — returns the artifact's structure (title, section headings, metadata). **Note:** by default this does not include section bodies. Pass `include_bodies: true` to get the full text, or call `get_section` for a single section's body.
- `list_artifacts` / `list_sections` — enumerate without fetching content.
- Always read the current state before editing — another collaborator may have changed it since you last saw it.

## Sharing & access

Artyfacts isn't single-player. Artifacts can be shared with specific people, entire folders can be shared, and anything can be surfaced via public link. When the user asks to share, invite, grant access, or publish an artifact, use the sharing tools rather than describing the action.

**Read before you share.** Call `list_artifact_access` first to see who already has access and at what permission level. Don't re-invite someone who already has access, and don't downgrade or change someone's permission without confirming.

### Sharing tools

- `create_share_link` — generate a public share URL. Use when the user says "share this" without naming a person. `revoke_share_link` kills an existing link.
- `share_artifact_with_user` — invite a specific person by email to a single artifact.
- `share_folder_with_user` — invite a person to an entire folder (all current and future artifacts in it).
- `list_team_members` — enumerate org members when you need to look up who to share with.
- `list_artifact_access` — see current collaborators and permission levels on an artifact.
- `update_collaborator_permission` — change an existing collaborator's role.
- `remove_artifact_collaborator` — revoke a user's access.
- `set_artifact_visibility` — flip an artifact between private, org-visible, and public. Use sparingly — prefer share links for one-off external sharing.

### Collaboration expectations

Treat every artifact as a shared document with history:

- Another agent, or the user themselves, may have edited an artifact since the last time you read it. Fetch the current state before editing.
- Respect existing structure. If a section was written by someone else, don't rewrite it wholesale unless the user asks.
- The agent name and color the user chose during setup show up next to every section you write. Treat it as a byline — no preamble, no "here is the document you requested", no sign-off.

## Voice when writing artifacts

Artifacts are durable. Write so someone reading in six months can pick up cold.

- **research** — evidence-forward, sourced, with a clear finding stated up top
- **plan / spec** — action-forward, dated, with owners where known
- **decision / review** — state the call, then the reasoning, then the alternatives considered
- **guide / document** — structured, skimmable, examples over prose

Complete sentences. No shorthand from the conversation that produced it. No "as we discussed" — the reader may not have been there.

## Do not

- Do **not** ask "would you like this as Word / PDF / Markdown?" Artyfacts is the format.
- Do **not** output the full document inline and then also save it. Save it and link to it.
- Do **not** create an artifact for ephemeral work — one-off queries, debugging output, quick answers.
- Do **not** change section headings on updates unless the user asks.
- Do **not** create a new artifact if one on the same topic already exists. Update it.
- Do **not** paste base64-encoded images into markdown sections. Use `attach_image_from_url` (preferred) or `attach_image` for raw bytes.
- Do **not** re-invite someone who already has access, or change a collaborator's permission without confirming. Check `list_artifact_access` first.

## Quick reference

| Situation | Tool |
|---|---|
| Looking for an existing artifact | `search_artifacts` |
| Creating a new artifact | `save_document_as_artifact` |
| Adding a new section | `start_section` then `update_section` |
| Revising an existing section | `edit_section` |
| Grouping artifacts | `create_folder`, `move_artifact` |
| Reading back what's saved | `list_artifacts`, `get_artifact` (pass `include_bodies: true` for content), `list_sections`, `get_section` |
| Attaching an image | `attach_image_from_url` (preferred), `attach_image` (small inline bytes only) |
| Sharing via public link | `create_share_link`, `revoke_share_link` |
| Sharing with a specific person | `share_artifact_with_user`, `share_folder_with_user` |
| Looking up org members | `list_team_members` |
| Checking or changing who has access | `list_artifact_access`, `update_collaborator_permission`, `remove_artifact_collaborator` |
| Changing artifact visibility | `set_artifact_visibility` |
