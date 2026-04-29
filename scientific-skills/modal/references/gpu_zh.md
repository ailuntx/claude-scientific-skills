# 模态 GPU 计算

## 目录

- [可用 GPU](#可用-gpu)
- [申请 GPU](#申请-gpu)
- [GPU 选择指南](#gpu-选择指南)
- [多 GPU](#多-gpu)
- [GPU 回退链](#gpu-回退链)
- [自动升级](#自动升级)
- [多 GPU 训练](#多-gpu-训练)

## 可用 GPU

| GPU | 显存 | 单容器上限 | 最佳用途 |
|-----|------|-------------------|----------|
| T4 | 16 GB | 8 | 低成本推理，小型模型 |
| L4 | 24 GB | 8 | 推理，视频处理 |
| A10 | 24 GB | 4 | 推理，微调小型模型 |
| L40S | 48 GB | 8 | 推理（最佳性价比），中型模型 |
| A100-40GB | 40 GB | 8 | 训练，大型模型推理 |
| A100-80GB | 80 GB | 8 | 训练，大型模型 |
| RTX-PRO-6000 | 48 GB | 8 | 渲染，推理 |
| H100 | 80 GB | 8 | 大规模训练，快速推理 |
| H200 | 141 GB | 8 | 超大型模型训练 |
| B200 | 192 GB | 8 | 最大规模模型，最高吞吐量 |
| B200+ | 192 GB | 8 | B200 或 B300，按 B200 计费 |

## 申请 GPU

### 基础申请

```python
@app.function(gpu="H100")
def train():
    import torch
    assert torch.cuda.is_available()
    print(f"Using: {torch.cuda.get_device_name(0)}")
```

### 字符串简写

```python
gpu="T4"           # 单张 T4
gpu="A100-80GB"    # 单张 A100 80GB
gpu="H100:4"       # 四张 H100
```

### GPU 对象（高级）

```python
@app.function(gpu=modal.gpu.H100(count=2))
def multi_gpu():
    ...
```

## GPU 选择指南

### 推理场景

| 模型规模 | 推荐 GPU | 原因 |
|-----------|----------------|-----|
| < 70亿参数 | T4, L4 | 成本效益高，显存充足 |
| 70-130亿参数 | L40S | 最佳性价比，48GB 显存 |
| 130-700亿参数 | A100-80GB, H100 | 大显存，高内存带宽 |
| 700亿+参数 | H100:2+, H200, B200 | 多 GPU 或超大显存 |

### 训练场景

| 任务 | 推荐 GPU |
|------|----------------|
| 微调（LoRA） | L40S, A100-40GB |
| 小型模型全量微调 | A100-80GB |
| 大型模型全量微调 | H100:4+, H200 |
| 预训练 | H100:8, B200:8 |

### 通用建议

L40S 是推理任务的最佳默认选择——48GB 显存提供了卓越的成本性能平衡。

## 多 GPU

通过添加 `:数量` 申请多 GPU：

```python
@app.function(gpu="H100:4")
def distributed():
    import torch
    print(f"可用 GPU 数量: {torch.cuda.device_count()}")
    # 所有 4 张 GPU 位于同一物理机
```

- 多数类型支持最多 8 张 GPU（A10 上限为 4 张）
- 所有 GPU 挂载在同一物理机
- 申请超过 2 张 GPU 可能增加等待时间
- 最大显存：8 × B200 = 1,536 GB

## GPU 回退链

指定优先级 GPU 列表：

```python
@app.function(gpu=["H100", "A100-80GB", "L40S"])
def flexible():
    # Modal 优先尝试 H100，其次 A100-80GB，最后 L40S
    ...
```

当特定 GPU 不可用时，可有效减少排队时间。

## 自动升级

### H100 → H200

Modal 可能免费将 H100 请求自动升级至 H200。如需禁用：

```python
@app.function(gpu="H100!")  # 感叹号阻止自动升级
def must_use_h100():
    ...
```

### A100 → A100-80GB

A100-40GB 请求可能免费升级至 80GB 版本。

### B200+

`gpu="B200+"` 允许 Modal 在 B200 或 B300 GPU 上以 B200 价格运行。需 CUDA 13.0+。

## 多 GPU 训练

Modal 支持单节点多 GPU 训练。多节点训练处于内测阶段。

### PyTorch DDP 示例

```python
@app.function(gpu="H100:4", image=image, timeout=86400)
def train_distributed():
    import torch
    import torch.distributed as dist

    dist.init_process_group(backend="nccl")
    local_rank = int(os.environ.get("LOCAL_RANK", 0))
    device = torch.device(f"cuda:{local_rank}")
    # ... 使用 DDP 的训练循环 ...
```

### PyTorch Lightning

使用会重新执行 Python 入口点的框架（如 PyTorch Lightning）时：

1. 设置策略为 `ddp_spawn` 或 `ddp_notebook`
2. 或通过子进程运行训练

```python
@app.function(gpu="H100:4", image=image)
def train():
    import subprocess
    subprocess.run(["python", "train_script.py"], check=True)
```

### Hugging Face Accelerate

```python
@app.function(gpu="A100-80GB:4", image=image)
def finetune():
    import subprocess
    subprocess.run([
        "accelerate", "launch",
        "--num_processes", "4",
        "train.py"
    ], check=True)
```
