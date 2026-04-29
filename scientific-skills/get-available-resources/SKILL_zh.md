---
name: get-available-resources
description: 该技能应在启动任何计算密集型科学任务时使用，用于检测并报告可用系统资源（CPU核心、GPU、内存、磁盘空间）。它会创建包含资源信息和战略建议的JSON文件，为计算决策提供依据，例如是否使用并行处理（joblib、multiprocessing）、核外计算（Dask、Zarr）、GPU加速（PyTorch、JAX）或内存优化策略。在运行分析、训练模型、处理大型数据集或任何资源受限的任务前使用此技能。
license: MIT license
metadata:
    skill-author: K-Dense Inc.
---

# 获取可用资源

## 概述

检测可用计算资源并为科学计算任务生成战略建议。该技能自动识别CPU能力、GPU可用性（NVIDIA CUDA、AMD ROCm、Apple Silicon Metal）、内存限制和磁盘空间，帮助制定合理的计算方案决策。

## 使用场景

在任何计算密集型任务前主动使用此技能：

- **数据分析前**：判断数据集能否载入内存或需核外处理
- **模型训练前**：检查GPU加速可用性及后端选择
- **并行处理前**：确定joblib、multiprocessing或Dask的最佳工作线程数
- **大型文件操作前**：验证磁盘空间充足性及存储策略
- **项目初始化时**：了解基础能力以制定架构决策

**典型场景示例：**
- "帮我分析这个50GB基因组数据集" → 先使用此技能判断是否需要Dask/Zarr
- "基于此数据训练神经网络" → 使用此技能检测可用GPU及后端
- "并行处理10,000个文件" → 使用此技能确定最佳工作线程数
- "运行计算密集型模拟" → 使用此技能了解资源限制

## 实现原理

### 资源检测

该技能运行 `scripts/detect_resources.py` 脚本自动检测：

1. **CPU信息**
   - 物理核心与逻辑核心数
   - 处理器架构与型号
   - CPU频率信息

2. **GPU信息**
   - NVIDIA GPU：通过nvidia-smi检测，报告显存/驱动版本/计算能力
   - AMD GPU：通过rocm-smi检测
   - Apple Silicon：检测支持Metal的M1/M2/M3/M4芯片及统一内存

3. **内存信息**
   - 总内存与可用内存
   - 当前内存使用率
   - 交换空间可用性

4. **磁盘空间信息**
   - 工作目录的总磁盘空间与可用空间
   - 当前使用率

5. **操作系统信息**
   - 系统类型（macOS/Linux/Windows）
   - 系统版本与发行号
   - Python版本

### 输出格式

技能在当前工作目录生成 `.claude_resources.json` 文件，包含：

```json
{
  "timestamp": "2025-10-23T10:30:00",
  "os": {
    "system": "Darwin",
    "release": "25.0.0",
    "machine": "arm64"
  },
  "cpu": {
    "physical_cores": 8,
    "logical_cores": 8,
    "architecture": "arm64"
  },
  "memory": {
    "total_gb": 16.0,
    "available_gb": 8.5,
    "percent_used": 46.9
  },
  "disk": {
    "total_gb": 500.0,
    "available_gb": 200.0,
    "percent_used": 60.0
  },
  "gpu": {
    "nvidia_gpus": [],
    "amd_gpus": [],
    "apple_silicon": {
      "name": "Apple M2",
      "type": "Apple Silicon",
      "backend": "Metal",
      "unified_memory": true
    },
    "total_gpus": 1,
    "available_backends": ["Metal"]
  },
  "recommendations": {
    "parallel_processing": {
      "strategy": "high_parallelism",
      "suggested_workers": 6,
      "libraries": ["joblib", "multiprocessing", "dask"]
    },
    "memory_strategy": {
      "strategy": "moderate_memory",
      "libraries": ["dask", "zarr"],
      "note": "处理>2GB数据集时建议分块"
    },
    "gpu_acceleration": {
      "available": true,
      "backends": ["Metal"],
      "suggested_libraries": ["pytorch-mps", "tensorflow-metal", "jax-metal"]
    },
    "large_data_handling": {
      "strategy": "disk_abundant",
      "note": "有充足空间存储大型中间文件"
    }
  }
}
```

### 战略建议

技能生成上下文感知建议：

**并行处理建议：**
- **高并行性（8+核心）**：使用Dask/joblib/multiprocessing，工作线程数=核心数-2
- **中等并行性（4-7核心）**：使用joblib/multiprocessing，工作线程数=核心数-1
- **低并行性（<4核心）**：优先顺序处理以避免开销

**内存策略建议：**
- **内存紧张（<4GB可用）**：使用Zarr/Dask/H5py进行核外处理
- **中等内存（4-16GB可用）**：处理>2GB数据集时使用Dask/Zarr
- **内存充足（>16GB可用）**：可直接加载多数数据集至内存

**GPU加速建议：**
- **检测到NVIDIA GPU**：使用PyTorch/TensorFlow/JAX/CuPy/RAPIDS
- **检测到AMD GPU**：使用PyTorch-ROCm/TensorFlow-ROCm
- **检测到Apple Silicon**：使用PyTorch(MPS后端)/TensorFlow-Metal/JAX-Metal
- **未检测到GPU**：使用CPU优化库

**大型数据处理建议：**
- **磁盘紧张（<10GB）**：采用流式处理或压缩策略
- **中等磁盘（10-100GB）**：使用Zarr/H5py/Parquet格式
- **磁盘充足（>100GB）**：可自由创建大型中间文件

## 使用指南

### 步骤1：运行资源检测

在启动计算密集型任务时执行检测脚本：

```bash
python scripts/detect_resources.py
```

可选参数：
- `-o, --output <路径>`：指定输出路径（默认：`.claude_resources.json`）
- `-v, --verbose`：在stdout打印完整资源信息

### 步骤2：读取并应用建议

检测完成后读取生成的 `.claude_resources.json` 文件指导计算决策：

```python
# 示例：在代码中使用建议
import json

with open('.claude_resources.json', 'r') as f:
    resources = json.load(f)

# 检查并行处理策略
if resources['recommendations']['parallel_processing']['strategy'] == 'high_parallelism':
    n_jobs = resources['recommendations']['parallel_processing']['suggested_workers']
    # 使用joblib/Dask/multiprocessing并设置n_jobs个工作线程

# 检查内存策略
if resources['recommendations']['memory_strategy']['strategy'] == 'memory_constrained':
    # 使用Dask/Zarr/H5py进行核外处理
    import dask.array as da
    # 分块加载数据

# 检查GPU可用性
if resources['recommendations']['gpu_acceleration']['available']:
    backends = resources['recommendations']['gpu_acceleration']['backends']
    # 根据可用后端选择GPU库
```

### 步骤3：制定决策

利用资源信息和建议进行战略选择：

**数据加载示例：**
```python
memory_available_gb = resources['memory']['available_gb']
dataset_size_gb = 10

if dataset_size_gb > memory_available_gb * 0.5:
    # 数据集相对内存较大，使用Dask
    import dask.dataframe as dd
    df = dd.read_csv('large_file.csv')
else:
    # 数据集可载入内存，使用pandas
    import pandas as pd
    df = pd.read_csv('large_file.csv')
```

**并行处理示例：**
```python
from joblib import Parallel, delayed

n_jobs = resources['recommendations']['parallel_processing'].get('suggested_workers', 1)

results = Parallel(n_jobs=n_jobs)(
    delayed(process_function)(item) for item in data
)
```

**GPU加速示例：**
```python
import torch

if 'CUDA' in resources['gpu']['available_backends']:
    device = torch.device('cuda')
elif 'Metal' in resources['gpu']['available_backends']:
    device = torch.device('mps')
else:
    device = torch.device('cpu')

model = model.to(device)
```

## 依赖项

检测脚本需要安装以下Python包：

```bash
uv pip install psutil
```

其他功能使用Python标准库模块（json, os, platform, subprocess, sys, pathlib）。

## 平台支持

- **macOS**：完整支持，包括Apple Silicon（M1/M2/M3/M4）GPU检测
- **Linux**：完整支持，包括NVIDIA（nvidia-smi）和AMD（rocm-smi）GPU检测
- **Windows**：完整支持，包括NVIDIA GPU检测

## 最佳实践

1. **尽早运行**：在项目启动或执行主要计算任务前执行资源检测
2. **定期重检**：系统资源随时间变化（内存使用率、磁盘空间）
3. **扩展前验证**：增加并行工作线程或数据规模前确认资源充足
4. **记录决策**：将`.claude_resources.json`文件保留在项目目录中，记录资源感知决策
5. **结合版本控制**：不同机器能力不同，资源文件可确保可移植性

## 故障排除

**GPU未检测到：**
- 确保安装GPU驱动（Apple Silicon用system_profiler，NVIDIA用nvidia-smi，AMD用rocm-smi）
- 检查GPU工具是否在系统PATH中
- 确认GPU未被其他进程占用

**脚本执行失败：**
- 确保已安装psutil：`uv pip install psutil`
- 检查Python版本兼容性（需Python 3.6+）
- 验证脚本有执行权限：`chmod +x scripts/detect_resources.py`

**内存读数不准确：**
- 内存读数为瞬时快照，实际可用内存持续变化
- 检测前关闭其他应用以获取准确"可用内存"数据
- 可多次运行检测并取平均值
