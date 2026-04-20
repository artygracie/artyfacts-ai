---
name: artyfacts
description: Artyfacts is a filing cabinet for agent work product. When the user asks you to write up, document, draft, research, plan, summarize, or save any written deliverable, file it as an artifact in Artyfacts instead of dumping the body into chat.
---

# Artyfacts

## What artyfacts is

Artyfacts is where agent-produced written work lives — the long-form stuff the user will come back to. Research, plans, briefs, reports, specs, decisions, reviews, guides, summaries, analyses, drafts. If the agent wrote it and it should outlive the chat, it belongs in Artyfacts.

Think of it as a filing cabinet for agent output. Things that already have a dedicated home stay there — code in the codebase, designs in the design tool, data in the spreadsheet. Artyfacts is what's in between: the written work product that otherwise ends up in chat logs, screenshots, or sitting in a `final-v3` file the user will never open again.

## When to save an artifact

Create an artifact whenever your output is a written deliverable. Triggers:

- The user asks for a writeup, summary, analysis, plan, spec, report, guide, review, decision doc, or proposal
- You produce research with findings worth referencing later
- You draft a recommendation or decision that someone else might need to see
- Your output runs more than a couple paragraphs of structured content (headings, lists, analysis)

Rule of thumb: if you'd otherwise paste a long document into chat and move on, save it instead. The artifact URL is the thing the user will come back to — the chat reply is not.

## When not to save

Artyfacts is for written deliverables, not everything the agent produces. Skip it for:

- **Content that has its own home** — code lives in the codebase, designs live in the design tool, spreadsheets live in spreadsheet apps. Don't re-file work that's already filed somewhere.
- **Intermediate scratch work** — todo lists, working notes, anything the user won't reference again
- **Ephemeral replies** — one-line answers, clarifications, short explanations
- **Status updates** — "done", "got it", "here you go" — these don't need durable storage

If removing the artifact a month from now wouldn't be missed, don't create it in the first place.

## How to structure an artifact

**One topic per artifact.** Don't bundle unrelated work into a single "meeting notes" dump — split into separate artifacts with separate titles.

**Give it a specific title.** "Q3 growth strategy for enterprise self-serve" is a good title. "Report" is not. The title is what the user sees in their list later; make it searchable.

**Break the content into sections.** A good section is one logical unit — "Background", "Recommendation", "Open questions". Sections have their own headings and can be edited independently later, which matters for collaboration.

**Write a summary that tells the user what's inside without opening it.** One or two sentences, not a restatement of the title.

## What a section can hold

A section isn't just markdown. Match the content to the right type so it renders correctly.

- **Prose & docs** — `document/markdown` (default), `document/text`, `document/html`
- **Code** — `code/python`, `code/javascript`, `code/typescript`, `code/generic` (gets syntax highlighting)
- **Data** — `data/csv` for tables, `data/json` / `data/yaml` for structured data. CSV renders as a real sortable table, not a code block.
- **Images** — use `attach_image_from_url` for public URLs, `attach_image` for raw bytes. Don't paste base64 into a markdown section.
- **Structured payloads** — `structured/json` for machine-readable output other agents or workflows will consume.

One content type per section. If you're saving a research report with a chart and a data table, that's three sections: prose, image, CSV.

## Tool cheatsheet

| Tool | What it's for |
|------|---------------|
| `get_workspace` | Call first. Returns the user's folders + recent artifacts so you can file into the right place. |
| `save_document_as_artifact` | Creates a new artifact envelope (title, type, summary). Start here. |
| `create_section` | Adds a fully-formed section. Use when you already have the content ready. |
| `start_section` + `update_section` | Pair for streaming long content — start the section, then update as you write. |
| `edit_section` | Modifies an existing section in place. **Never change the heading** unless the user asks. |
| `update_section` | Replaces a section wholesale. Use when rewriting, not tweaking. |
| `create_folder` + `move_artifact` | Organize. If 3+ root artifacts share a topic, create a folder and move them in. |
| `search_artifacts` | Find existing work before creating new. Don't duplicate an artifact the user already has. |
| `attach_image_from_url` / `attach_image` | Add images — a public URL for the first, raw bytes for the second. |

Roles that are easy to confuse: `start_section` / `create_section` are for **new** content. `edit_section` is for **existing** content. Mixing them up produces either phantom sections or overwritten work.

## Working with teams

Artyfacts isn't single-player. An artifact may have been started by another agent, edited by the user directly, or passed between one user's Claude Code, ChatGPT, and Claude Desktop. Treat every artifact as a shared document with history.

Three collaboration patterns to expect:

- **Cross-agent, single user.** The user's Claude Code drafts a research report, their ChatGPT adds a summary, their Claude Desktop fact-checks it later. Different agents, same artifact.
- **Cross-user, multi-agent.** John's Claude and Jane's ChatGPT can both edit an artifact that's shared between them. Agents collaborate across users the same way humans do.
- **Human-in-the-loop.** Users edit artifacts directly in the web UI. A section may look different the next time you read it because a person rewrote a paragraph.

What this means for you:

- **Read before you write.** Fetch the current state before editing — another collaborator may have changed it since you last saw it.
- **Don't overwrite other people's work.** If a section was written by the user or another agent, respect their structure and heading.
- **Attribution is your byline.** The agent name and color the user chose during setup show up next to every section you write. Treat it as signed work — no preamble, no "here is the document you requested", no sign-off.

## Antipatterns

Things agents tend to do wrong when they first use Artyfacts:

- **Dumping everything into one section.** If the doc has natural structure, use it. Multiple sections always beats one wall of text — sections are what make the artifact editable later.
- **Renaming headings on edit.** When the user asks you to edit a section, leave the heading alone unless they explicitly ask for a rename. Users navigate by heading.
- **Vague titles.** "Summary", "Doc", "Report", "Notes". Be specific — the title is how the user finds this in a list of fifty others.
- **Pasting the artifact body into chat after saving.** The whole point is the URL. Share the link and stop — don't paste the contents back into the conversation.
- **Ignoring existing folders.** Before saving at the root, check `get_workspace`. If a matching folder exists, file it there.
- **Creating duplicate artifacts.** Use `search_artifacts` before creating something the user may already have.
