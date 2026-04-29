# 模态卷（Modal Volumes）

## 目录

- [概述](#概述)
- [创建卷](#创建卷)
- [挂载卷](#挂载卷)
- [读写文件](#读写文件)
- [CLI访问](#cli访问)
- [提交与重载](#提交与重载)
- [并发访问](#并发访问)
- [卷v2版本](#卷v2版本)
- [常用模式](#常用模式)

## 概述

卷是Modal的分布式文件系统，专为写一次读多次（write-once, read-many）的工作负载优化，例如存储模型权重并跨容器分发。

核心特性：
- 在函数调用和部署间持久化
- 可被多个函数同时挂载
- 后台每几秒自动提交
- 容器关闭时执行最终提交

## 创建卷

### 代码创建（惰性创建）

```python
vol = modal.Volume.from_name("my-volume", create_if_missing=True)
```

### CLI创建

```bash
modal volume create my-volume

# v2卷（测试版）
modal volume create my-volume --version=2
```

### 编程式v2创建

```python
vol = modal.Volume.from_name("my-volume", create_if_missing=True, version=2)
```

## 挂载卷

通过`volumes`参数将卷挂载到函数：

```python
vol = modal.Volume.from_name("model-store", create_if_missing=True)

@app.function(volumes={"/models": vol})
def use_model():
    # 通过/models/访问文件
    with open("/models/config.json") as f:
        config = json.load(f)
```

挂载多个卷：

```python
weights_vol = modal.Volume.from_name("weights")
data_vol = modal.Volume.from_name("datasets")

@app.function(volumes={"/weights": weights_vol, "/data": data_vol})
def train():
    ...
```

## 读写文件

### 写入

```python
@app.function(volumes={"/data": vol})
def save_results(results):
    import json
    import os

    os.makedirs("/data/outputs", exist_ok=True)
    with open("/data/outputs/results.json", "w") as f:
        json.dump(results, f)
```

### 读取

```python
@app.function(volumes={"/data": vol})
def load_results():
    with open("/data/outputs/results.json") as f:
        return json.load(f)
```

### 大文件（模型权重）

```python
@app.function(volumes={"/models": vol}, gpu="L40S")
def save_model():
    import torch
    model = train_model()
    torch.save(model.state_dict(), "/models/checkpoint.pt")

@app.function(volumes={"/models": vol}, gpu="L40S")
def load_model():
    import torch
    model = MyModel()
    model.load_state_dict(torch.load("/models/checkpoint.pt"))
    return model
```

## CLI访问

```bash
# 列出文件
modal volume ls my-volume
modal volume ls my-volume /subdir/

# 上传文件
modal volume put my-volume local_file.txt
modal volume put my-volume local_file.txt /remote/path/file.txt

# 下载文件
modal volume get my-volume /remote/file.txt local_file.txt

# 删除卷
modal volume delete my-volume
```

## 提交与重载

Modal每几秒在后台自动提交卷更改，并在容器关闭时执行最终提交。

### 显式提交

强制立即提交：

```python
@app.function(volumes={"/data": vol})
def writer():
    with open("/data/file.txt", "w") as f:
        f.write("hello")
    vol.commit()  # 立即可被其他容器访问
```

### 重载

查看其他容器的更改：

```python
@app.function(volumes={"/data": vol})
def reader():
    vol.reload()  # 刷新以获取最新写入
    with open("/data/file.txt") as f:
        return f.read()
```

## 并发访问

### v1卷

- 建议最大5个并发提交
- 相同文件并发修改时"最后写入优先"
- 避免并发修改相同文件
- 最大500,000个文件（inode）

### v2卷

- 支持数百个并发写入（不同文件）
- 无文件数量限制
- 改进的随机访问性能
- 单文件最大1 TiB，单目录最多262,144个文件

## 卷v2版本

v2卷（测试版）提供显著改进：

| 特性 | v1 | v2 |
|------|----|----|
| 最大文件数 | 500,000 | 无限制 |
| 并发写入 | ~5 | 数百 |
| 最大文件大小 | 无限制 | 1 TiB |
| 随机访问 | 有限 | 完全支持 |
| HIPAA合规 | 否 | 是 |
| 硬链接 | 否 | 是 |

启用v2：

```python
vol = modal.Volume.from_name("my-vol-v2", create_if_missing=True, version=2)
```

## 常用模式

### 模型权重存储

```python
vol = modal.Volume.from_name("model-weights", create_if_missing=True)

# 在镜像构建时一次性下载
def download_weights():
    from huggingface_hub import snapshot_download
    snapshot_download("meta-llama/Llama-3-8B", local_dir="/models/llama3")

image = (
    modal.Image.debian_slim()
    .uv_pip_install("huggingface_hub")
    .run_function(download_weights, volumes={"/models": vol})
)
```

### 训练检查点

```python
@app.function(volumes={"/checkpoints": vol}, gpu="H100", timeout=86400)
def train():
    for epoch in range(100):
        train_one_epoch()
        torch.save(model.state_dict(), f"/checkpoints/epoch_{epoch}.pt")
        vol.commit()  # 立即保存检查点
```

### 函数间共享数据

```python
data_vol = modal.Volume.from_name("shared-data", create_if_missing=True)

@app.function(volumes={"/data": data_vol})
def preprocess():
    # 写入处理后的数据
    df.to_parquet("/data/processed.parquet")

@app.function(volumes={"/data": data_vol})
def analyze():
    data_vol.reload()  # 确保获取最新数据
    df = pd.read_parquet("/data/processed.parquet")
    return df.describe()
```

### 性能优化建议

- 卷针对大文件优化，不适用于大量小文件
- v1卷保持50,000个以下文件/目录可获得最佳性能
- 使用Parquet等列式格式替代多个小CSV
- 纯临时数据建议使用`ephemeral_disk`而非卷
