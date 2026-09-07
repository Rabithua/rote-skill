---
name: rote
description: Manage Rote notes, articles, attachments, reactions, settings, and note share links. Use when an agent should operate Rote through an already-connected OAuth HTTP MCP server or through rote-toolkit CLI, SDK, or stdio MCP with a local OpenKey workflow.
---

# Rote

Choose one authenticated path and keep its trust boundary explicit.

## Connection choice

- Prefer an already-connected Rote Server HTTP MCP for remote agent work. It uses OAuth scopes such as `notes:share`.
- Use `rote-toolkit` 0.6.0 or later for local terminal, Node/TypeScript, or stdio MCP workflows backed by OpenKey permissions such as `SHAREROTE`.
- Do not silently switch between OAuth and OpenKey. If the selected path lacks access, report the missing scope or permission and let the user choose how to proceed.
- Use Toolkit CLI for one-off local operations, `RoteClient` for application code, and Toolkit stdio MCP for local agent integrations.
- Use public explore reads without authentication only when the task genuinely concerns public discovery.

## Safety rules

- Resolve loosely described notes before mutation. Preserve fields the user did not ask to change.
- Create or revoke a share link only when the user explicitly requests that action. Reading share status is non-mutating.
- Treat a share URL as a bearer credential: show it only to the intended user when needed, and do not repeat it in logs, summaries, or diagnostics.
- Toolkit attachment upload may read only file paths explicitly supplied by the user. Toolkit stdio MCP has no local-file upload tool and must not read arbitrary files.
- Prefer Toolkit or connected MCP tools over handwritten OpenKey HTTP calls.

## Reference

Read [references/commands.md](./references/commands.md) when exact CLI commands, SDK methods, MCP tool names, attachment steps, permissions, or failure handling are needed.
