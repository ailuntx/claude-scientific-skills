# 数据丰富

丰富: $ARGUMENTS

## 开始之前

告知用户：根据请求的行数和字段数量，数据丰富可能需要几分钟时间。

## 步骤 1：启动数据丰富

使用以下**其中一种**命令模式（替换用户的实际数据）：

**内联数据：**

```bash
parallel-cli enrich run --data '[{"company": "Google"}, {"company": "Microsoft"}]' --intent "CEO name and founding year" --target "output.json" --no-wait --json
```

**CSV 文件：**

```bash
parallel-cli enrich run --source-type csv --source "input.csv" --target "/tmp/output.json" --source-columns '[{"name": "company", "description": "Company name"}]' --intent "CEO name and founding year" --no-wait --json
```

如果这是对之前研究或数据丰富任务的**跟进**，并且你知道 `interaction_id`，则添加上下文链：

```bash
parallel-cli enrich run --data '...' --intent "..." --target "output.json" --no-wait --json --previous-interaction-id "$INTERACTION_ID"
```

通过在多个请求中链接 `interaction_id`，每个后续任务会自动拥有之前轮次的完整上下文——这样你就可以丰富早期研究中发现的数据，而无需重复陈述已找到的信息。

**重要：** 始终包含 `--no-wait`，以便命令立即返回而非阻塞。

解析输出以提取 `taskgroup_id`、`interaction_id` 和监控 URL。立即告知用户：
- 数据丰富已启动
- 监控 URL，用户可在此跟踪进度

告知用户可以将轮询步骤放到后台执行，以便在运行时继续其他工作。

## 步骤 2：轮询结果

根据数据丰富任务选择一个简短、描述性的文件名（例如 `companies-ceos`、`startups-funding`）。使用小写字母和连字符，不含空格。

```bash
parallel-cli enrich poll "$TASKGROUP_ID" --timeout 540 --json --output "$FILENAME.json"
```

`enrich run` 中的 `--target` 标志不会传递到轮询命令——你必须在此处使用 `--output` 来保存结果。始终使用 `--json` 获取结构化的 JSON 输出。

重要说明：
- 使用 `--timeout 540`（9 分钟）以保持在工具执行限制内

### 如果轮询超时

大型数据集的丰富可能需要超过 9 分钟。如果轮询未完成即退出：
1. 告知用户数据丰富仍在服务端运行
2. 重新运行相同的 `parallel-cli enrich poll` 命令以继续等待

## 响应格式

**步骤 1 之后：** 分享监控 URL（用于跟踪进度）。

**步骤 2 之后：**
1. 报告已丰富的数据行数
2. 预览输出 JSON 的前几行
3. 告知用户输出 JSON 文件的完整路径（`$FILENAME.json`）
4. 分享 `interaction_id`，并告知用户可以基于此数据丰富提出后续问题

**完成后不要再次分享监控 URL**——结果已在输出文件中。

**记住 `interaction_id`**——如果用户提出与此数据丰富相关的后续问题，请在下一个研究或数据丰富命令中使用 `--previous-interaction-id`。
