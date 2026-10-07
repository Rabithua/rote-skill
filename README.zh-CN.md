<p align="right"><a href="./README.md">English</a> | 中文</p>

# Rote Skill

这个仓库提供了一个通过 OAuth HTTP MCP 或 `rote-toolkit` OpenKey 工作流操作 Rote 的通用 AI agent skill。

## 安装

使用 [skills CLI](https://github.com/vercel-labs/skills) 安装，并选择要使用的 agent：

```bash
npx skills add Rabithua/rote-skill
```

默认安装到当前项目。如果希望在 Codex 中全局使用：

```bash
npx skills add Rabithua/rote-skill -g -a codex
```

此命令安装 skill 的操作说明和参考文档。实际操作 Rote 还需要连接 OAuth HTTP MCP，或单独配置 `rote-toolkit`。

## 更新

在项目目录中更新该项目安装的 skill：

```bash
npx skills update rote -p
```

更新全局安装的 skill：

```bash
npx skills update rote -g
```

`rote` 对应 `SKILL.md` 中声明的名称。如果此前手动复制过 skill，先执行一次安装命令，让 `skills` 记录来源并管理后续更新。

维护者将修改合并到本仓库默认分支 `main` 即可分发更新。Skill 分发不需要执行 `npm publish`，也不要求创建 GitHub Release；版本标签仅用于可选的版本标记。`rote-toolkit` 是独立的软件包，有自己的发布流程。

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
