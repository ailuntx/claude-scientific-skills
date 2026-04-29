# Modal 密钥管理

## 概述

Modal Secrets 将凭证和敏感数据作为环境变量安全地传递给函数。密钥以加密形式存储，并且仅对您的工作区可用。

## 创建密钥

### 通过 CLI

```bash
# 通过键值对创建
modal secret create my-api-keys API_KEY=sk-xxx DB_PASSWORD=hunter2

# 从现有环境变量创建
modal secret create my-env-keys API_KEY=$API_KEY

# 列出所有密钥
modal secret list

# 删除密钥
modal secret delete my-api-keys
```

### 通过仪表板

访问 https://modal.com/secrets 创建和管理密钥。提供常见服务（Postgres、MongoDB、Hugging Face、Weights & Biases 等）的模板。

### 编程方式（内联）

```python
# 从字典创建（适用于开发）
secret = modal.Secret.from_dict({"API_KEY": "sk-xxx"})

# 从 .env 文件创建
secret = modal.Secret.from_dotenv()

# 从命名密钥创建（通过 CLI 或仪表板创建）
secret = modal.Secret.from_name("my-api-keys")
```

## 在函数中使用密钥

### 基本用法

```python
@app.function(secrets=[modal.Secret.from_name("my-api-keys")])
def call_api():
    import os
    api_key = os.environ["API_KEY"]
    # 使用密钥
    response = requests.get(url, headers={"Authorization": f"Bearer {api_key}"})
    return response.json()
```

### 多个密钥

```python
@app.function(secrets=[
    modal.Secret.from_name("openai-keys"),
    modal.Secret.from_name("database-creds"),
])
def process():
    import os
    openai_key = os.environ["OPENAI_API_KEY"]
    db_url = os.environ["DATABASE_URL"]
    ...
```

密钥按顺序应用 —— 如果两个密钥定义了相同的键，则后一个会覆盖前一个。

### 在类中使用

```python
@app.cls(secrets=[modal.Secret.from_name("huggingface")])
class ModelService:
    @modal.enter()
    def load(self):
        import os
        token = os.environ["HF_TOKEN"]
        self.model = AutoModel.from_pretrained("model-name", token=token)
```

### 从 .env 文件

```python
# 从当前目录读取 .env 文件
@app.function(secrets=[modal.Secret.from_dotenv()])
def local_dev():
    import os
    api_key = os.environ["API_KEY"]
```

`.env` 文件格式：

```
API_KEY=sk-xxx
DATABASE_URL=postgres://user:pass@host/db
DEBUG=false
```

## 常见密钥模板

| 服务 | 典型键名 |
|---------|-------------|
| OpenAI | `OPENAI_API_KEY` |
| Hugging Face | `HF_TOKEN` |
| AWS | `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` |
| Postgres | `PGHOST`, `PGPORT`, `PGUSER`, `PGPASSWORD`, `PGDATABASE` |
| Weights & Biases | `WANDB_API_KEY` |
| GitHub | `GITHUB_TOKEN` |

## 安全注意事项

- 密钥在存储和传输过程中均加密
- 仅对您工作区中的函数可访问
- 切勿记录或打印密钥值
- 在生产环境中使用 `.from_name()`（而非 `.from_dict()`）
- 通过仪表板或 CLI 定期轮换密钥
