---
name: rote
description: Use rote-toolkit to create, update, delete, search, list, and explore Rote notes and related resources through the Rote API. Trigger this skill when Codex needs to operate a Rote instance from terminal workflows or AI-agent flows, especially for note CRUD, public explore-note discovery, article creation, reactions, profile updates, permission checks, or MCP server setup via the rote-toolkit CLI, SDK, or MCP server.
---

# Rote

Use `rote-toolkit` instead of reimplementing Rote API calls.

## Decision Rules

- Prefer the CLI for one-off operations.
- Prefer `RoteClient` for Node/TypeScript integrations.
- Prefer MCP only when the user explicitly needs MCP tool access.
- Use `explore`/`exploreNotes`/`rote_explore_notes` for public explore-page notes; this path does not require OpenKey auth.
- Check `~/.rote-toolkit/config.json` before authenticated operations. If missing, run `rote config`.
- Do not hand-roll `fetch` calls to the Rote OpenKey API if `RoteClient` already covers the operation.
- Normalize tags as arrays for SDK/MCP usage and comma-separated strings for CLI usage.
- When modifying notes, require a `noteId` and at least one updated field.
- For one-shot user tasks, prefer executing the CLI over writing new helper code.
- For repeated or embedded workflows, import `RoteClient` from `rote-toolkit`.

## References

- Read [references/commands.md](./references/commands.md) only when you need exact commands, batch-operation patterns, failure handling, or MCP tool names.
