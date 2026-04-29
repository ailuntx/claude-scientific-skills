# 模态资源配置

## CPU

### 申请 CPU

```python
@app.function(cpu=4.0)
def compute():
    ...
```

- 数值代表**物理核心数**，非虚拟CPU
- 默认值：0.125 核心
- Modal 会根据 CPU 申请自动设置 `OPENBLAS_NUM_THREADS`、`OMP_NUM_THREADS`、`MKL_NUM_THREADS`

### CPU 限制

- 默认软限制：在申请基础上额外增加 16 个物理核心
- 设置显式限制可避免资源争用问题：

```python
@app.function(cpu=4.0)  # 申请 4 核心
def bounded_compute():
    ...
```

## 内存

### 申请内存

```python
@app.function(memory=16384)  # 以 MiB 为单位的 16 GiB
def large_data():
    ...
```

- 单位为 **MiB** (兆字节)
- 默认值：128 MiB

### 内存限制

设置硬性内存限制可强制终止超限容器：

```python
@app.function(memory=8192)  # 8 GiB 申请与限制
def bounded_memory():
    ...
```

这能避免为内存泄漏支付额外费用。

## 临时磁盘

用于容器生命周期内的临时存储：

```python
@app.function(ephemeral_disk=102400)  # 以 MiB 为单位的 100 GiB
def process_dataset():
    # 临时文件位于 /tmp 或容器文件系统任意位置
    ...
```

- 单位为 **MiB**
- 默认配额：每个容器 512 GiB
- 上限：3,145,728 MiB (3 TiB)
- 容器关闭时数据将丢失
- 持久存储请使用 Volumes

出于计费考虑，磁盘申请量会按 20:1 比例增加内存申请量。

## 超时设置

```python
@app.function(timeout=3600)  # 以秒为单位的 1 小时
def long_running():
    ...
```

- 默认值：300 秒 (5 分钟)
- 上限：86,400 秒 (24 小时)
- 超时后函数将被终止

## 计费规则

按**申请量与实际使用量的较高值**计费：

| 资源类型 | 计费基准 |
|----------|--------------|
| CPU | max(申请量, 使用量) |
| 内存 | max(申请量, 使用量) |
| GPU | GPU 分配时长 |
| 磁盘 | 按 20:1 比例增加内存计费 |

### 成本优化建议

- 按需申请资源
- 选用合适 GPU 层级（推理任务优选 L40S 而非 H100）
- 设置 `scaledown_window` 减少闲置时间
- 可接受冷启动时使用 `min_containers=0`
- 使用 `.map()` 批量处理而非多次 `.remote()` 调用

## 完整示例

```python
@app.function(
    cpu=8.0,              # 8 物理核心
    memory=32768,         # 32 GiB
    gpu="L40S",           # L40S GPU
    ephemeral_disk=204800, # 200 GiB 临时磁盘
    timeout=7200,         # 2 小时
    max_containers=50,
    min_containers=1,
)
def full_pipeline(data_path: str):
    ...
```
