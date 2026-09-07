<p align="right"><a href="./README.md">English</a> | 中文</p>

# Rote Skill

这个仓库提供了一个通过 OAuth HTTP MCP 或 `rote-toolkit` OpenKey 工作流操作 Rote 的通用 AI agent skill。

## 包含文件

- `SKILL.md`：skill 的触发描述与决策规则
- `references/commands.md`：命令参考、高阶 playbook 与失败处理约定
- `agents/openai.yaml`：skill 的 UI 元数据

## 相关仓库

- [Rote](https://github.com/Rabithua/Rote)
- [rote-toolkit](https://github.com/Rabithua/rote-toolkit)

## 兼容性

- 分享链接工作流要求 Rote Server 2.4.0 或更高版本。
- 本文档中的本地 CLI、SDK 与 stdio MCP 工作流要求 `rote-toolkit` 0.6.0 或更高版本。
- OAuth 与 OpenKey 是独立认证路径，skill 不会静默切换。
