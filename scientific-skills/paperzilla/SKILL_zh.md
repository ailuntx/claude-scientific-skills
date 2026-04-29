---
name: paperzilla
description: 与您的代理就 Paperzilla 中的项目、推荐和权威论文进行对话。当用户询问近期项目推荐、权威论文详情、基于 Markdown 的摘要、推荐反馈、信息流导出或 Atom 订阅链接时使用此技能。
license: MIT
metadata:
  skill-author: "Paperzilla Inc"
---

# Paperzilla

当您想与代理讨论 Paperzilla 中的项目、推荐和权威论文时，请使用此技能。

## 您可以询问的内容

- "给我项目 X 的最新推荐。"
- "打开推荐 Y 并解释其重要性。"
- "获取权威论文 Z 的 Markdown 版本并进行总结。"
- "告诉我这篇论文与我的研究有何关联。"
- "显示项目 X 的信息流。"
- "对推荐项目留下反馈。"
- "将这篇论文、推荐或信息流导出为 JSON。"

这是 Paperzilla 的核心技能。它让您的代理能够直接访问 Paperzilla 数据，但不会强加工作流程或外部交付集成。

## 访问方式

此仓库中的大多数配置都使用 `pz` CLI。

如果当前配置附带了额外的代理特定指令，请同时遵循这些指令。

## 安装

### macOS
```bash
brew install paperzilla-ai/tap/pz
```

### Windows (Scoop)
```bash
scoop bucket add paperzilla-ai https://github.com/paperzilla-ai/scoop-bucket
scoop install pz
```

### Linux
使用官方 Linux 安装指南：

- https://docs.paperzilla.ai/guides/cli-getting-started

### 从源代码构建 (Go 1.23+)
请参阅 CLI 仓库获取源代码构建方法：

- https://github.com/paperzilla-ai/pz

## 更新

检查您的 CLI 是否是最新版本，并获取特定于安装方式的升级步骤：

```bash
pz update
```

如果检测不明确，可以显式覆盖：

```bash
pz update --install-method homebrew
pz update --install-method scoop
pz update --install-method release
pz update --install-method source
```

支持的值包括 `auto`、`homebrew`、`scoop`、`release` 和 `source`。

## 认证

```bash
pz login
```

## CLI 参考

如果当前配置使用 `pz`，以下是核心命令。

### 列出项目
```bash
pz project list
```

### 显示单个项目
```bash
pz project <project-id>
```

### 浏览项目信息流
```bash
pz feed <project-id>
```

有用的标志：
- `--must-read`
- `--since YYYY-MM-DD`
- `--limit N`
- `--json`
- `--atom`

示例：
```bash
pz feed <project-id> --must-read --since 2026-03-01 --limit 5
pz feed <project-id> --json
pz feed <project-id> --atom
```

信息流输出可以包含现有的推荐反馈标记：

- `[↑]` 点赞
- `[↓]` 点踩
- `[★]` 收藏

### 阅读权威论文
```bash
pz paper <paper-id>
pz paper <paper-id> --json
pz paper <paper-id> --markdown
pz paper <paper-id> --project <project-id>
```

### 打开您某个项目中的推荐项目
```bash
pz rec <project-paper-id>
pz rec <project-paper-id> --json
pz rec <project-paper-id> --markdown
```

### 留下推荐反馈
```bash
pz feedback <project-paper-id> upvote
pz feedback <project-paper-id> star
pz feedback <project-paper-id> downvote --reason not_relevant
pz feedback clear <project-paper-id>
```

## 输出与自动化

- 对于机器解析，优先使用 `--json`。
- `pz paper --markdown` 仅在 Markdown 已预先准备好时返回。
- `pz rec --markdown` 可以排队生成 Markdown，并在生成过程中打印友好的重试消息。
- `--atom` 返回一个用于阅读器订阅的个人信息流 URL。

## 配置

```bash
export PZ_API_URL="https://paperzilla.ai"
```

## 参考

- 文档：https://docs.paperzilla.ai/guides/cli
- 快速入门：https://docs.paperzilla.ai/guides/cli-getting-started
- 仓库：https://github.com/paperzilla-ai/pz
