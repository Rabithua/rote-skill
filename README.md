<p align="right">English | <a href="./README.zh-CN.md">中文</a></p>

# Rote Skill

This repository contains a reusable skill for AI agents to work with Rote through the OAuth HTTP MCP or `rote-toolkit` OpenKey workflows.

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
