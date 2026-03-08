<p align="right"><a href="./README.md">English</a> | 中文</p>

# Rote Skill

这个仓库提供了一个通过 `rote-toolkit` 操作 Rote 的通用 AI agent skill。

## 包含文件

- `SKILL.md`：skill 的触发描述与决策规则
- `references/commands.md`：命令参考、高阶 playbook 与失败处理约定
- `agents/openai.yaml`：skill 的 UI 元数据

## 相关仓库

- [Rote](https://github.com/Rabithua/Rote)
- [rote-toolkit](https://github.com/Rabithua/rote-toolkit)

## 版本提醒

更新这个 skill 时，请同时保持 `rote-toolkit` 为最新版本。

skill 的新能力可能依赖 `rote-toolkit` 新增的 CLI、SDK 或 MCP 特性，因此在使用 README 或 skill 中新增的能力前，应先升级 `rote-toolkit`。
