# TimesFM 系统要求

## 硬件层级

TimesFM 可在多种硬件配置上运行。本指南帮助您选择合适的设置并优化机器性能。

### 层级 1：最低配置（仅 CPU，4–8 GB 内存）

- **使用场景**：轻量级探索、单序列预测、原型开发
- **模型**：仅限 TimesFM 2.5 (200M)
- **批大小**：`per_core_batch_size=4`
- **上下文长度**：限制 `max_context=512`
- **预期速度**：每 100 点序列约 2–5 秒

```python
model.compile(timesfm.ForecastConfig(
    max_context=512,
    max_horizon=128,
    per_core_batch_size=4,
    normalize_inputs=True,
    use_continuous_quantile_head=True,
    fix_quantile_crossing=True,
))
```

### 层级 2：标准配置（CPU 16 GB 或 GPU 4–8 GB 显存）

- **使用场景**：批量预测（数十序列）、评估、生产原型
- **模型**：TimesFM 2.5 (200M)
- **批大小**：`per_core_batch_size=32` (CPU) 或 `64` (GPU)
- **上下文长度**：`max_context=1024`
- **预期速度**：每 100 点序列约 0.5–1 秒 (GPU)

```python
model.compile(timesfm.ForecastConfig(
    max_context=1024,
    max_horizon=256,
    per_core_batch_size=64,
    normalize_inputs=True,
    use_continuous_quantile_head=True,
    fix_quantile_crossing=True,
))
```

### 层级 3：生产配置（GPU 16+ GB 显存或 Apple Silicon 32+ GB）

- **使用场景**：大规模批量预测（数千序列）、长上下文场景
- **模型**：TimesFM 2.5 (200M)
- **批大小**：`per_core_batch_size=128–256`
- **上下文长度**：`max_context=4096` 或更高
- **预期速度**：每 100 点序列约 0.1–0.3 秒

```python
model.compile(timesfm.ForecastConfig(
    max_context=4096,
    max_horizon=256,
    per_core_batch_size=128,
    normalize_inputs=True,
    use_continuous_quantile_head=True,
    fix_quantile_crossing=True,
))
```

### 层级 4：旧版模型（v1.0/v2.0 — 500M 参数）

- **⚠️ 警告**：TimesFM v2.0 (500M) 需要 **≥ 16 GB 内存** (CPU) 或 **≥ 8 GB 显存** (GPU)
- **⚠️ 警告**：TimesFM v1.0 旧版 JAX 实现可能需 **≥ 32 GB 内存**
- **建议**：除非明确需要旧版检查点，否则请使用 TimesFM 2.5

## 内存估算

### CPU 内存 (RAM)

推理期间近似内存使用：

| 组件 | TimesFM 2.5 (200M) | TimesFM 2.0 (500M) |
| --------- | ------------------- | ------------------- |
| 模型权重 | ~800 MB | ~2 GB |
| 运行时开销 | ~500 MB | ~1 GB |
| 输入/输出缓冲区 | 每 1000 序列 ~200 MB | 每 1000 序列 ~500 MB |
| **小批量总计** | **~1.5 GB** | **~3.5 GB** |
| **大批量总计** | **~3 GB** | **~6 GB** |

**公式**：`内存 ≈ 模型权重 + 0.5 GB + (0.2 MB × 序列数 × 上下文长度 / 1000)`

### GPU 显存 (VRAM)

| 组件 | TimesFM 2.5 (200M) |
| --------- | ------------------- |
| 模型权重 | ~800 MB |
| KV缓存 + 激活值 | ~200–500 MB (随上下文长度增加) |
| 批处理缓冲区 | context=1024 时每 100 序列 ~100 MB |
| **批大小=32** | **~1.2 GB** |
| **批大小=128** | **~1.8 GB** |
| **批大小=256** | **~2.5 GB** |

### 磁盘空间

| 项目 | 大小 |
| ---- | ---- |
| TimesFM 2.5 safetensors | ~800 MB |
| Hugging Face 缓存开销 | ~200 MB |
| **总下载量** | **~1 GB** |

模型权重从 Hugging Face Hub 下载一次后缓存于
`~/.cache/huggingface/` (或 `$HF_HOME`)。

## GPU 选择指南

### NVIDIA GPU (CUDA)

| GPU | 显存 | 推荐批大小 | 备注 |
| --- | ---- | ----------------- | ----- |
| RTX 3060 | 12 GB | 64 | 良好入门级 |
| RTX 3090 / 4090 | 24 GB | 256 | 生产环境优异 |
| A100 (40 GB) | 40 GB | 512 | 云/HPC |
| A100 (80 GB) | 80 GB | 1024 | 云/HPC |
| T4 | 16 GB | 128 | 云端 (Colab, AWS) |
| V100 | 16–32 GB | 128–256 | 云端 |

### Apple Silicon (MPS)

| 芯片 | 统一内存 | 推荐批大小 | 备注 |
| ---- | -------------- | ----------------- | ----- |
| M1 | 8–16 GB | 16–32 | 可用，慢于 CUDA |
| M1 Pro/Max | 16–64 GB | 32–128 | 性能良好 |
| M2/M3/M4 Pro/Max | 18–128 GB | 64–256 | 优异 |

### 仅 CPU

可在满足内存要求的任意 CPU 运行。速度比 GPU 慢 5–20 倍。

## Python 与依赖包要求

| 要求 | 最低版本 | 推荐版本 |
| ----------- | ------- | ----------- |
| Python | 3.10 | 3.12+ |
| numpy | 1.26.4 | 最新 |
| torch | 2.0.0 | 最新 |
| huggingface_hub | 0.23.0 | 最新 |
| safetensors | 0.5.3 | 最新 |

### 可选依赖

| 包 | 用途 | 安装命令 |
| ------- | ------- | ------- |
| jax | Flax 后端 | `pip install jax[cuda]` |
| flax | Flax 后端 | `pip install flax` |
| scikit-learn | XReg 协变量 | `pip install scikit-learn` |

## 操作系统兼容性

| 系统 | 状态 | 备注 |
| -- | ------ | ----- |
| Linux (Ubuntu 20.04+) | ✅ 完全支持 | CUDA 下性能最佳 |
| macOS 13+ (Ventura) | ✅ 完全支持 | Apple Silicon 支持 MPS 加速 |
| Windows 11 + WSL2 | ✅ 支持 | 推荐使用 WSL2 |
| Windows (原生) | ⚠️ 部分支持 | PyTorch 可用，存在边界情况 |

## 故障排除

### 内存不足 (OOM)

```python
# 减小批大小
model.compile(timesfm.ForecastConfig(
    per_core_batch_size=4,  # 从极小值开始
    max_context=512,        # 缩短上下文
    ...
))

# 分块处理
for i in range(0, len(inputs), 50):
    chunk = inputs[i:i+50]
    p, q = model.forecast(horizon=H, inputs=chunk)
```

### CPU 推理缓慢

```python
# 确保矩阵乘法精度设置
import torch
torch.set_float32_matmul_precision("high")

# 缩短上下文长度
model.compile(timesfm.ForecastConfig(
    max_context=256,  # 上下文越短越快
    ...
))
```

### 模型下载失败

```bash
# 设置其他缓存目录
export HF_HOME=/path/with/more/space

# 或手动下载
huggingface-cli download google/timesfm-2.5-200m-pytorch
```
