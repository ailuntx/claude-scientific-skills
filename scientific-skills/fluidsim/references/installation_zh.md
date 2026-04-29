# FluidSim 安装指南

## 环境要求

- Python >= 3.9
- 推荐使用虚拟环境

## 安装方法

### 基础安装

使用 uv 安装 fluidsim：

```bash
uv pip install fluidsim
```

### 支持 FFT（伪谱求解器必需）

大多数 fluidsim 求解器使用基于傅里叶的方法，需要 FFT 库支持：

```bash
uv pip install "fluidsim[fft]"
```

此命令将安装 fluidfft 和 pyfftw 依赖项。

### 支持 MPI 和 FFT（用于并行模拟）

用于高性能并行计算：

```bash
uv pip install "fluidsim[fft,mpi]"
```

注意：此操作将触发 mpi4py 的本地编译。

## 环境配置

### 输出目录设置

通过环境变量控制模拟数据存储位置：

```bash
export FLUIDSIM_PATH=/path/to/simulation/outputs
export FLUIDDYN_PATH_SCRATCH=/path/to/working/directory
```

### FFT 方法选择

指定 FFT 实现方式（可选）：

```bash
export FLUIDSIM_TYPE_FFT2D=fft2d.with_fftw
export FLUIDSIM_TYPE_FFT3D=fft3d.with_fftw
```

## 安装验证

运行测试验证安装：

```bash
pytest --pyargs fluidsim
```

## 无需认证

FluidSim 不需要 API 密钥或身份验证令牌。
