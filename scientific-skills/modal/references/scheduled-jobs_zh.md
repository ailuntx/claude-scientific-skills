# 模态计划任务

## 概述

模态支持通过 cron 语法或固定间隔自动运行函数。使用 `modal deploy` 部署计划函数后，它们将在云端无人值守运行。

## 计划类型

### modal.Cron

标准 cron 语法——部署后保持稳定：

```python
import modal

app = modal.App("scheduled-tasks")

# 每天 UTC 时间上午9点
@app.function(schedule=modal.Cron("0 9 * * *"))
def daily_report():
    generate_and_send_report()

# 每周一午夜
@app.function(schedule=modal.Cron("0 0 * * 1"))
def weekly_cleanup():
    cleanup_old_data()

# 每15分钟
@app.function(schedule=modal.Cron("*/15 * * * *"))
def frequent_check():
    check_system_health()
```

#### Cron 语法参考

```
┌───────────── 分钟 (0-59)
│ ┌───────────── 小时 (0-23)
│ │ ┌───────────── 日期 (1-31)
│ │ │ ┌───────────── 月份 (1-12)
│ │ │ │ ┌───────────── 星期 (0-6, 周日=0)
│ │ │ │ │
* * * * *
```

| 模式 | 含义 |
|---------|---------|
| `0 9 * * *` | 每天 UTC 时间上午9点 |
| `0 */6 * * *` | 每6小时 |
| `*/30 * * * *` | 每30分钟 |
| `0 0 * * 1` | 每周一午夜 |
| `0 0 1 * *` | 每月第一天 |
| `0 9 * * 1-5` | 工作日每天上午9点 |

### modal.Period

固定间隔——每次部署时重置：

```python
# 每5小时
@app.function(schedule=modal.Period(hours=5))
def periodic_sync():
    sync_data()

# 每30分钟
@app.function(schedule=modal.Period(minutes=30))
def poll_updates():
    check_for_updates()

# 每天
@app.function(schedule=modal.Period(days=1))
def daily_task():
    ...
```

`modal.Period` 会在每次部署时重置计时器。若需不受部署影响的计划，请使用 `modal.Cron`。

## 部署计划函数

计划仅在部署时激活：

```bash
modal deploy script.py
```

`modal run` 和 `modal serve` 不会激活计划任务。

## 监控

- 在模态控制台的 **应用** 区域查看计划运行记录
- 每次运行显示状态、持续时间和日志
- 使用控制台 **"立即运行"** 按钮手动触发

## 管理

- 计划无法暂停——需移除计划并重新部署才能停止
- 修改计划：更新 `schedule` 参数后重新部署
- 完全停止：移除 `schedule` 参数或运行 `modal app stop <名称>`

## 常见模式

### ETL 管道

```python
@app.function(
    schedule=modal.Cron("0 2 * * *"),  # 每天 UTC 时间凌晨2点
    secrets=[modal.Secret.from_name("db-creds")],
    timeout=7200,
)
def etl_pipeline():
    import os
    data = extract(os.environ["SOURCE_DB_URL"])
    transformed = transform(data)
    load(transformed, os.environ["DEST_DB_URL"])
```

### 模型重训练

```python
@app.function(
    schedule=modal.Cron("0 0 * * 0"),  # 每周日
    gpu="H100",
    volumes={"/data": data_vol, "/models": model_vol},
    timeout=86400,
)
def retrain():
    model = train_on_latest_data("/data/training/")
    torch.save(model.state_dict(), "/models/latest.pt")
```

### 健康检查

```python
@app.function(
    schedule=modal.Period(minutes=5),
    secrets=[modal.Secret.from_name("slack-webhook")],
)
def health_check():
    import os, requests
    status = check_all_services()
    if not status["healthy"]:
        requests.post(os.environ["SLACK_URL"], json={"text": f"警报: {status}"})
```
