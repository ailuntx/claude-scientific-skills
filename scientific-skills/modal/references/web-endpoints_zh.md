# 模态 Web 端点

## 目录

- [简单端点](#simple-endpoints)
- [部署](#deployment)
- [ASGI 应用](#asgi-apps-fastapi-starlette-fasthtml)
- [WSGI 应用](#wsgi-apps-flask-django)
- [自定义 Web 服务器](#custom-web-servers)
- [WebSockets](#websockets)
- [认证](#authentication)
- [流式传输](#streaming)
- [并发](#concurrency)
- [限制](#limits)

## 简单端点

创建 Web 端点的最简单方式：

```python
import modal

app = modal.App("api-service")

@app.function()
@modal.fastapi_endpoint()
def hello(name: str = "World"):
    return {"message": f"Hello, {name}!"}
```

### POST 端点

```python
@app.function()
@modal.fastapi_endpoint(method="POST")
def predict(data: dict):
    result = model.predict(data["text"])
    return {"prediction": result}
```

### 查询参数

参数会自动从查询字符串解析：

```python
@app.function()
@modal.fastapi_endpoint()
def search(query: str, limit: int = 10):
    return {"results": do_search(query, limit)}
```

访问方式：`https://your-app.modal.run?query=hello&limit=5`

## 部署

### 开发模式

```bash
modal serve script.py
```

- 创建临时公共 URL
- 文件更改时热重载
- 适用于开发和测试
- 停止命令后 URL 失效

### 生产部署

```bash
modal deploy script.py
```

- 创建永久 URL
- 在云端持久运行
- 基于流量自动扩缩容
- URL 格式：`https://<workspace>--<app-name>-<function-name>.modal.run`

## ASGI 应用 (FastAPI, Starlette, FastHTML)

完整框架应用使用 `@modal.asgi_app`：

```python
from fastapi import FastAPI

web_app = FastAPI()

@web_app.get("/")
async def root():
    return {"status": "ok"}

@web_app.post("/predict")
async def predict(request: dict):
    return {"result": model.run(request["input"])}

@app.function(image=image, gpu="L40S")
@modal.asgi_app()
def fastapi_app():
    return web_app
```

### 类生命周期管理

```python
@app.cls(gpu="L40S", image=image)
class InferenceService:
    @modal.enter()
    def load_model(self):
        self.model = load_model()

    @modal.asgi_app()
    def serve(self):
        from fastapi import FastAPI
        app = FastAPI()

        @app.post("/generate")
        async def generate(request: dict):
            return self.model.generate(request["prompt"])

        return app
```

## WSGI 应用 (Flask, Django)

```python
from flask import Flask

flask_app = Flask(__name__)

@flask_app.route("/")
def index():
    return {"status": "ok"}

@app.function(image=image)
@modal.wsgi_app()
def flask_server():
    return flask_app
```

WSGI 是同步模式——并发请求在独立线程中运行。

## 自定义 Web 服务器

适用于非标准框架 (aiohttp, Tornado, TGI)：

```python
@app.function(image=image, gpu="H100")
@modal.web_server(port=8000)
def serve():
    import subprocess
    subprocess.Popen([
        "python", "-m", "vllm.entrypoints.openai.api_server",
        "--model", "meta-llama/Llama-3-70B",
        "--host", "0.0.0.0",  # 必须绑定到 0.0.0.0，而非 localhost
        "--port", "8000",
    ])
```

应用必须绑定到 `0.0.0.0`（而非 `127.0.0.1`）。

## WebSockets

通过 `@modal.asgi_app`、`@modal.wsgi_app` 和 `@modal.web_server` 支持：

```python
from fastapi import FastAPI, WebSocket

web_app = FastAPI()

@web_app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        result = process(data)
        await websocket.send_text(result)

@app.function()
@modal.asgi_app()
def ws_app():
    return web_app
```

- 完整 WebSocket 协议 (RFC 6455)
- 每条消息最大 2 MiB
- 暂不支持 RFC 8441 或 RFC 7692

## 认证

### 代理认证令牌（内置）

Modal 通过代理认证令牌提供原生端点保护：

```python
@app.function()
@modal.fastapi_endpoint()
def protected(text: str):
    return {"result": process(text)}
```

客户端需包含 `Modal-Key` 和 `Modal-Secret` 请求头进行认证。

### 自定义 Bearer 令牌

```python
from fastapi import Header, HTTPException

@app.function(secrets=[modal.Secret.from_name("auth-secret")])
@modal.fastapi_endpoint(method="POST")
def secure_predict(data: dict, authorization: str = Header(None)):
    import os
    expected = os.environ["AUTH_TOKEN"]
    if authorization != f"Bearer {expected}":
        raise HTTPException(status_code=401, detail="Unauthorized")
    return {"result": model.predict(data["text"])}
```

### 客户端 IP 访问

可用于地理位置识别、速率限制和访问控制。

## 流式传输

### 服务器推送事件 (SSE)

```python
from fastapi.responses import StreamingResponse

@app.function(gpu="H100")
@modal.fastapi_endpoint()
def stream_generate(prompt: str):
    def generate():
        for token in model.stream(prompt):
            yield f"data: {token}\n\n"
    return StreamingResponse(generate(), media_type="text/event-stream")
```

## 并发

使用 `@modal.concurrent` 实现单容器多请求处理：

```python
@app.function(gpu="L40S")
@modal.concurrent(max_inputs=10)
@modal.fastapi_endpoint(method="POST")
async def batch_predict(data: dict):
    return {"result": await model.predict_async(data["text"])}
```

## 限制

- 请求体：最大 4 GiB
- 响应体：无限制
- 速率限制：200 请求/秒（新账户支持 5 秒突发）
- 无活跃容器时出现冷启动（可通过 `min_containers` 避免）
