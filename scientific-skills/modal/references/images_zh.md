# 模态容器镜像

## 目录

- [概述](#概述)
- [基础镜像](#基础镜像)
- [安装软件包](#安装软件包)
- [系统软件包](#系统软件包)
- [Shell 命令](#shell-命令)
- [构建期间运行 Python](#构建期间运行-python)
- [添加本地文件](#添加本地文件)
- [环境变量](#环境变量)
- [Dockerfiles](#dockerfiles)
- [替代包管理器](#替代包管理器)
- [镜像缓存](#镜像缓存)
- [处理仅远程可用的导入](#处理仅远程可用的导入)

## 概述

每个 Modal 函数都在由 `Image` 构建的容器内运行。默认情况下，Modal 使用与本地解释器相同 Python 次要版本的 Debian Linux 镜像。

镜像采用惰性构建——仅当首次调用使用该镜像的函数时，Modal 才会构建/拉取镜像。各层会被缓存以实现快速重建。

## 基础镜像

```python
# 默认：使用本地 Python 版本的 Debian slim
image = modal.Image.debian_slim()

# 指定 Python 版本
image = modal.Image.debian_slim(python_version="3.11")

# 从 Docker Hub 获取
image = modal.Image.from_registry("nvidia/cuda:12.4.0-devel-ubuntu22.04")

# 从 Dockerfile 构建
image = modal.Image.from_dockerfile("./Dockerfile")
```

## 安装软件包

### uv（推荐）

`uv_pip_install` 使用 uv 包管理器实现快速可靠的安装：

```python
image = (
    modal.Image.debian_slim(python_version="3.11")
    .uv_pip_install(
        "torch==2.8.0",
        "transformers>=4.40",
        "accelerate",
        "scipy",
    )
)
```

固定版本以保证可复现性。uv 的依赖解析速度比 pip 更快。

### pip（备用方案）

```python
image = modal.Image.debian_slim().pip_install(
    "numpy==1.26.0",
    "pandas==2.1.0",
)
```

### 从 requirements.txt 安装

```python
image = modal.Image.debian_slim().pip_install_from_requirements("requirements.txt")
```

### 私有软件包

```python
image = (
    modal.Image.debian_slim()
    .pip_install_private_repos(
        "github.com/org/private-repo",
        git_user="username",
        secrets=[modal.Secret.from_name("github-token")],
    )
)
```

## 系统软件包

通过 apt 安装 Linux 软件包：

```python
image = (
    modal.Image.debian_slim()
    .apt_install("ffmpeg", "libsndfile1", "git", "curl")
    .uv_pip_install("librosa", "soundfile")
)
```

## Shell 命令

在镜像构建期间执行任意命令：

```python
image = (
    modal.Image.debian_slim()
    .run_commands(
        "wget https://example.com/data.tar.gz",
        "tar -xzf data.tar.gz -C /opt/data",
        "rm data.tar.gz",
    )
)
```

### 使用 GPU

某些构建步骤需要 GPU 访问（例如编译 CUDA 内核）：

```python
image = (
    modal.Image.debian_slim()
    .uv_pip_install("torch")
    .run_commands("python -c 'import torch; torch.cuda.is_available()'", gpu="A100")
)
```

## 构建期间运行 Python

将 Python 函数作为构建步骤执行——适用于下载模型权重：

```python
def download_model():
    from huggingface_hub import snapshot_download
    snapshot_download("meta-llama/Llama-3-8B", local_dir="/models/llama3")

image = (
    modal.Image.debian_slim(python_version="3.11")
    .uv_pip_install("huggingface_hub", "torch", "transformers")
    .run_function(download_model, secrets=[modal.Secret.from_name("huggingface")])
)
```

生成的文件系统（包括下载的文件）会被快照到镜像中。

## 添加本地文件

### 本地目录

```python
image = modal.Image.debian_slim().add_local_dir(
    local_path="./config",
    remote_path="/root/config",
)
```

默认情况下，文件在容器启动时添加（不烘焙到镜像层）。使用 `copy=True` 将其烘焙到镜像层。

### 本地 Python 模块

```python
image = modal.Image.debian_slim().add_local_python_source("my_module")
```

此方法利用 Python 的导入系统查找并包含模块。

### 单个文件

```python
image = modal.Image.debian_slim().add_local_file(
    local_path="./model_config.json",
    remote_path="/root/config.json",
)
```

## 环境变量

```python
image = (
    modal.Image.debian_slim()
    .env({
        "TRANSFORMERS_CACHE": "/cache",
        "TOKENIZERS_PARALLELISM": "false",
        "HF_HOME": "/cache/huggingface",
    })
)
```

名称和值必须是字符串类型。

## Dockerfiles

从现有 Dockerfile 构建：

```python
image = modal.Image.from_dockerfile("./Dockerfile")

# 带构建上下文
image = modal.Image.from_dockerfile("./Dockerfile", context_mount=modal.Mount.from_local_dir("."))
```

## 替代包管理器

### Micromamba / Conda

适用于需要协调系统与 Python 软件包安装的场景：

```python
image = (
    modal.Image.micromamba(python_version="3.11")
    .micromamba_install("cudatoolkit=11.8", "cudnn=8.6", channels=["conda-forge"])
    .uv_pip_install("torch")
)
```

## 镜像缓存

Modal 按层（每个方法调用）缓存镜像。某一层的缓存失效会导致所有后续层失效。

### 优化技巧

1. **按变更频率排序层级**：将稳定依赖放在前面，频繁变更的代码放在最后
2. **固定版本**：未固定版本可能导致解析结果不同并破坏缓存
3. **分离大型安装**：将重型软件包（torch, tensorflow）放在早期层级

### 强制重建

```python
# 单层重建
image = modal.Image.debian_slim().apt_install("git", force_build=True)
```

```bash
# 重建运行中的所有镜像
MODAL_FORCE_BUILD=1 modal run script.py

# 忽略缓存重建
MODAL_IGNORE_CACHE=1 modal run script.py
```

## 处理仅远程可用的导入

当软件包仅在容器中可用（本地不可用）时，使用条件导入：

```python
@app.function(image=image)
def process():
    import torch  # 仅在容器中可用
    return torch.cuda.device_count()
```

对于跨函数共享的模块级导入，使用 `Image.imports()` 上下文管理器：

```python
with image.imports():
    import torch
    import transformers
```

这可以避免本地出现 `ImportError`，同时使导入在容器中可用。
