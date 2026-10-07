<p align="right">English | <a href="./README.zh-CN.md">中文</a></p>

# Rote Skill

This repository contains a reusable skill for AI agents to work with Rote through the OAuth HTTP MCP or `rote-toolkit` OpenKey workflows.

## Install

Install with the [skills CLI](https://github.com/vercel-labs/skills) and choose your agent:

```bash
npx skills add Rabithua/rote-skill
```

By default, the skill is installed in the current project. To install it globally for Codex:

```bash
npx skills add Rabithua/rote-skill -g -a codex
```

This installs the skill instructions and references. Connect the OAuth HTTP MCP or set up `rote-toolkit` separately to operate Rote.

## Update

Update a project installation from that project directory:

```bash
npx skills update rote -p
```

Update a global installation:

```bash
npx skills update rote -g
```

`rote` is the name declared in `SKILL.md`. If you previously copied the skill manually, run the installation command once so `skills` can track its source and manage updates.

Maintainers publish updates by merging changes into this repository's default branch, `main`. No `npm publish` or GitHub Release is required for skill distribution; release tags are optional version markers. `rote-toolkit` is a separate package with its own release process.

## Included Files

- `SKILL.md`: Skill trigger description and decision rules
- `references/commands.md`: Command reference, playbooks, and failure-handling guidance
- `agents/openai.yaml`: UI metadata for the skill

## Related Repositories

- [Rote](https://github.com/Rabithua/Rote)
- [rote-toolkit](https://github.com/Rabithua/rote-toolkit)

## Compatibility

- Share-link workflows require Rote Server 2.4.0 or later.
- Local CLI, SDK, and stdio MCP workflows documented here require `rote-toolkit` 0.6.0 or later.
- OAuth and OpenKey are separate authentication paths and are never switched silently.
