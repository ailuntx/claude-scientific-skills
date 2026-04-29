# TimesFM API 参考文档

## 模型类

### `timesfm.TimesFM_2p5_200M_torch`

TimesFM 2.5 的主模型类（2亿参数，PyTorch后端）。

#### `from_pretrained()`

```python
model = timesfm.TimesFM_2p5_200M_torch.from_pretrained(
    "google/timesfm-2.5-200m-pytorch",
    cache_dir=None,         # 可选：自定义缓存目录
    force_download=True,    # 即使已缓存也重新下载
)
```

| 参数 | 类型 | 默认值 | 描述 |
| --------- | ---- | ------- | ----------- |
| `model_id` | str | `"google/timesfm-2.5-200m-pytorch"` | Hugging Face 模型标识符 |
| `revision` | str \| None | None | 指定模型版本 |
| `cache_dir` | str \| Path \| None | None | 自定义缓存目录 |
| `force_download` | bool | True | 强制重新下载权重 |

**返回**：初始化的 `TimesFM_2p5_200M_torch` 实例（尚未编译）。

#### `compile()`

根据给定预测配置编译模型。**必须在调用 `forecast()` 前执行。**

```python
model.compile(
    timesfm.ForecastConfig(
        max_context=1024,
        max_horizon=256,
        normalize_inputs=True,
        per_core_batch_size=32,
        use_continuous_quantile_head=True,
        force_flip_invariance=True,
        infer_is_positive=True,
        fix_quantile_crossing=True,
    )
)
```

**异常**：无（但未编译时调用 `forecast()` 会引发 `RuntimeError`）。

#### `forecast()`

对一个或多个时间序列进行推理预测。

```python
point_forecast, quantile_forecast = model.forecast(
    horizon=24,
    inputs=[array1, array2, ...],
)
```

| 参数 | 类型 | 描述 |
| --------- | ---- | ----------- |
| `horizon` | int | 预测的未来步长 |
| `inputs` | list[np.ndarray] | 一维numpy数组列表（每个代表一个时间序列） |

**返回**：`tuple[np.ndarray, np.ndarray]`

- `point_forecast`：形状 `(batch_size, horizon)` — 中位数（0.5分位数）
- `quantile_forecast`：形状 `(batch_size, horizon, 10)` — [均值, q10, q20, ..., q90]

**异常**：未编译模型时抛出 `RuntimeError`。

**关键行为**：

- 自动去除前导NaN值
- 内部NaN值进行线性插值
- 长于 `max_context` 的序列被截断（使用最后 `max_context` 个点）
- 短于 `max_context` 的序列进行填充

#### `forecast_with_covariates()`

使用外生变量进行推理（需安装 `timesfm[xreg]`）。

```python
point, quantiles = model.forecast_with_covariates(
    inputs=inputs,
    dynamic_numerical_covariates={"temp": [temp_array1, temp_array2]},
    dynamic_categorical_covariates={"dow": [dow_array1, dow_array2]},
    static_categorical_covariates={"region": ["east", "west"]},
    xreg_mode="xreg + timesfm",
)
```

| 参数 | 类型 | 描述 |
| --------- | ---- | ----------- |
| `inputs` | list[np.ndarray] | 目标时间序列 |
| `dynamic_numerical_covariates` | dict[str, list[np.ndarray]] | 时变数值特征 |
| `dynamic_categorical_covariates` | dict[str, list[np.ndarray]] | 时变类别特征 |
| `static_categorical_covariates` | dict[str, list[str]] | 每个序列的固定类别特征 |
| `xreg_mode` | str | `"xreg + timesfm"` 或 `"timesfm + xreg"` |

**注意**：动态协变量每个序列长度必须为 `上下文长度 + 预测步长`。

---

## `timesfm.ForecastConfig`

控制所有预测行为的不可变数据类。

```python
@dataclasses.dataclass(frozen=True)
class ForecastConfig:
    max_context: int = 0
    max_horizon: int = 0
    normalize_inputs: bool = False
    per_core_batch_size: int = 1
    use_continuous_quantile_head: bool = False
    force_flip_invariance: bool = True
    infer_is_positive: bool = True
    fix_quantile_crossing: bool = False
    return_backcast: bool = False
    quantiles: list[float] = [0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9]
    decode_index: int = 5
```

### 参数详解

#### `max_context` (int, 默认=0)

用作上下文的历史时间点最大数量。

- **0**：使用模型支持的最大上下文（v2.5为16,384）
- **N**：截取序列最后N个点
- **最佳实践**：设为最长序列长度，或为速度设为512–2048

#### `max_horizon` (int, 默认=0)

最大预测步长。

- **0**：使用模型默认最大值
- **N**：预测最多N步（仍可调用 `forecast(horizon=M)` 其中 M ≤ N）
- **最佳实践**：设为预期最大预测长度

#### `normalize_inputs` (bool, 默认=False)

是否在输入模型前对每个序列进行z标准化。

- **True**（推荐）：归一化为零均值单位方差
- **False**：直接传递原始值
- **适用False场景**：序列已标准化或接近1.0量级

#### `per_core_batch_size` (int, 默认=1)

每个设备每批次处理的序列数。

- 增加可提升吞吐量，减少可避免OOM
- 硬件推荐值参见 `references/system_requirements.md`

#### `use_continuous_quantile_head` (bool, 默认=False)

使用300万参数连续分位数头提升区间校准精度。

- **True**（推荐）：预测区间更准确（尤其长步长）
- **False**：使用固定分位数桶（更快但区间精度较低）

#### `force_flip_invariance` (bool, 默认=True)

确保模型满足 `f(-x) = -f(x)` 数学特性。

- **True**（推荐）：数学一致性—预测结果符号翻转不变
- **False**：稍快但预测可能不对称

#### `infer_is_positive` (bool, 默认=True)

自动检测输入值是否全为正数并约束预测值≥0。

- **True**：适用于销售额/需求量/计数/价格/交易量
- **False**：适用于温度/收益率/PnL等可为负的序列

#### `fix_quantile_crossing` (bool, 默认=False)

后处理分位数确保单调性（q10 ≤ q20 ≤ ... ≤ q90）。

- **True**（推荐）：保证分位数有序
- **False**：稍快但分位数偶尔交叉

#### `return_backcast` (bool, 默认=False)

返回模型对输入的逆向重构（backcast）及预测结果。

- **True**：用于协变量流程和诊断
- **False**：仅返回预测结果

---

## 可用模型检查点

| 模型标识符 | 版本 | 参数 | 后端 | 上下文长度 |
| -------- | ------- | ------ | ------- | ------- |
| `google/timesfm-2.5-200m-pytorch` | 2.5 | 200M | PyTorch | 16,384 |
| `google/timesfm-2.5-200m-flax` | 2.5 | 200M | JAX/Flax | 16,384 |
| `google/timesfm-2.5-200m-transformers` | 2.5 | 200M | Transformers | 16,384 |
| `google/timesfm-2.0-500m-pytorch` | 2.0 | 500M | PyTorch | 2,048 |
| `google/timesfm-2.0-500m-jax` | 2.0 | 500M | JAX | 2,048 |
| `google/timesfm-1.0-200m-pytorch` | 1.0 | 200M | PyTorch | 2,048 |
| `google/timesfm-1.0-200m` | 1.0 | 200M | JAX | 2,048 |

---

## 输出形状参考

| 输出 | 形状 | 描述 |
| ------ | ----- | ----------- |
| `point_forecast` | `(B, H)` | B个序列H步长的中位数预测 |
| `quantile_forecast` | `(B, H, 10)` | 完整分位数分布 |
| `quantile_forecast[:,:,0]` | `(B, H)` | 均值 |
| `quantile_forecast[:,:,1]` | `(B, H)` | 10%分位数 |
| `quantile_forecast[:,:,5]` | `(B, H)` | 50%分位数（= point_forecast） |
| `quantile_forecast[:,:,9]` | `(B, H)` | 90%分位数 |

其中 `B` = 批次大小（输入序列数），`H` = 预测步长。

---

## 错误处理

| 错误 | 原因 | 解决方案 |
| ----- | ----- | --- |
| `RuntimeError: Model is not compiled` | 未编译即调用 `forecast()` | 先执行 `model.compile(ForecastConfig(...))` |
| `torch.cuda.OutOfMemoryError` | GPU批次过大 | 减小 `per_core_batch_size` |
| `ValueError: inputs must be list` | 传入数组而非列表 | 用列表包裹：`[array]` |
| `HfHubHTTPError` | 下载失败 | 检查网络，设置 `HF_HOME` 为可写目录 |
