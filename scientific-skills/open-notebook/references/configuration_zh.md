# Open Notebook 配置指南

## Docker 部署

Open Notebook 以 Docker Compose 栈形式部署，包含两个主要服务：应用服务器和 SurrealDB。

### 最小化 docker-compose.yml

```yaml
version: "3.8"

services:
  surrealdb:
    image: surrealdb/surrealdb:latest
    command: start --user root --pass root rocksdb://data/database.db
    volumes:
      - surrealdb_data:/data
    ports:
      - "8000:8000"

  open-notebook:
    image: ghcr.io/lfnovo/open-notebook:latest
    depends_on:
      - surrealdb
    environment:
      - OPEN_NOTEBOOK_ENCRYPTION_KEY=${OPEN_NOTEBOOK_ENCRYPTION_KEY}
      - SURREAL_URL=ws://surrealdb:8000/rpc
      - SURREAL_NAMESPACE=open_notebook
      - SURREAL_DATABASE=open_notebook
    ports:
      - "8502:8502"   # 前端界面
      - "5055:5055"   # REST API
    volumes:
      - on_uploads:/app/uploads

volumes:
  surrealdb_data:
  on_uploads:
```

### 启动服务栈

```bash
# 设置加密密钥（必需）
export OPEN_NOTEBOOK_ENCRYPTION_KEY="your-secure-random-key"

# 启动服务
docker-compose up -d

# 查看日志
docker-compose logs -f open-notebook

# 停止服务
docker-compose down

# 停止并删除数据
docker-compose down -v
```

## 环境变量

### 必需变量

| 变量名 | 描述 |
|----------|-------------|
| `OPEN_NOTEBOOK_ENCRYPTION_KEY` | 用于加密存储API凭证的密钥。首次启动前必须设置并保持一致性。 |

### 数据库配置

| 变量名 | 默认值 | 描述 |
|----------|---------|-------------|
| `SURREAL_URL` | `ws://surrealdb:8000/rpc` | SurrealDB WebSocket连接URL |
| `SURREAL_NAMESPACE` | `open_notebook` | SurrealDB命名空间 |
| `SURREAL_DATABASE` | `open_notebook` | SurrealDB数据库名称 |
| `SURREAL_USER` | `root` | SurrealDB用户名 |
| `SURREAL_PASS` | `root` | SurrealDB密码 |

### 应用配置

| 变量名 | 默认值 | 描述 |
|----------|---------|-------------|
| `OPEN_NOTEBOOK_PASSWORD` | 无 | Web界面的可选密码保护 |
| `UPLOAD_DIR` | `/app/uploads` | 上传文件存储目录 |

### AI供应商密钥（传统方式）

API密钥也可通过环境变量设置以保持向后兼容，推荐使用凭证API或界面配置。

| 变量名 | 供应商 |
|----------|----------|
| `OPENAI_API_KEY` | OpenAI |
| `ANTHROPIC_API_KEY` | Anthropic |
| `GOOGLE_API_KEY` | Google GenAI |
| `GROQ_API_KEY` | Groq |
| `MISTRAL_API_KEY` | Mistral |
| `ELEVENLABS_API_KEY` | ElevenLabs |

## AI供应商配置

### 通过界面配置

1. 进入 **设置 > API密钥**
2. 点击 **添加凭证**
3. 选择供应商，输入API密钥和可选基础URL
4. 点击 **测试连接** 进行验证
5. 点击 **发现模型** 查找可用模型
6. 选择要注册的模型

### 通过API配置

```python
import requests

BASE_URL = "http://localhost:5055/api"

# 1. 创建凭证
cred = requests.post(f"{BASE_URL}/credentials", json={
    "provider": "anthropic",
    "name": "Anthropic Production",
    "api_key": "sk-ant-..."
}).json()

# 2. 测试连接
test = requests.post(f"{BASE_URL}/credentials/{cred['id']}/test").json()
assert test["success"]

# 3. 发现并注册模型
discovered = requests.post(
    f"{BASE_URL}/credentials/{cred['id']}/discover"
).json()

requests.post(
    f"{BASE_URL}/credentials/{cred['id']}/register-models",
    json={"model_ids": [m["id"] for m in discovered["models"]]}
)

# 4. 自动分配默认模型
requests.post(f"{BASE_URL}/models/auto-assign")
```

### 使用 Ollama（免费本地推理）

通过 Ollama 实现无需API费用的免费AI推理：

```yaml
# docker-compose-ollama.yml 补充配置
services:
  ollama:
    image: ollama/ollama:latest
    volumes:
      - ollama_data:/root/.ollama
    ports:
      - "11434:11434"
```

配置 Ollama 供应商时使用基础 URL `http://ollama:11434`。

## 安全配置

### 密码保护

设置 `OPEN_NOTEBOOK_PASSWORD` 启用身份验证：

```bash
export OPEN_NOTEBOOK_PASSWORD="your-ui-password"
```

### 反向代理（Nginx示例）

```nginx
server {
    listen 443 ssl;
    server_name notebook.example.com;

    ssl_certificate /etc/ssl/certs/cert.pem;
    ssl_certificate_key /etc/ssl/private/key.pem;

    location / {
        proxy_pass http://localhost:8502;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }

    location /api/ {
        proxy_pass http://localhost:5055/api/;
        proxy_set_header Host $host;
    }
}
```

## 备份与恢复

### 备份 SurrealDB 数据

```bash
# 导出数据库
docker exec surrealdb surreal export \
  --conn ws://localhost:8000 \
  --user root --pass root \
  --ns open_notebook --db open_notebook \
  /tmp/backup.surql

# 从容器复制备份
docker cp surrealdb:/tmp/backup.surql ./backup.surql
```

### 备份上传文件

```bash
# 复制上传卷内容
docker cp open-notebook:/app/uploads ./uploads_backup/
```

### 恢复操作

```bash
# 导入数据库备份
docker cp ./backup.surql surrealdb:/tmp/backup.surql
docker exec surrealdb surreal import \
  --conn ws://localhost:8000 \
  --user root --pass root \
  --ns open_notebook --db open_notebook \
  /tmp/backup.surql
```
