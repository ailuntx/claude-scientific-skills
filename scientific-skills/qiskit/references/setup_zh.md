# Qiskit 设置与安装

## 安装

使用 uv 安装 Qiskit：

```bash
uv pip install qiskit
```

如需可视化功能：

```bash
uv pip install "qiskit[visualization]" matplotlib
```

## Python 环境设置

创建并激活虚拟环境以隔离依赖项：

```bash
# macOS/Linux
python3 -m venv .venv
source .venv/bin/activate

# Windows
python -m venv .venv
.venv\Scripts\activate
```

## 支持的 Python 版本

请查看 [Qiskit PyPI 页面](https://pypi.org/project/qiskit/)获取当前支持的 Python 版本。截至 2025 年，Qiskit 通常支持 Python 3.8 及以上版本。

## IBM Quantum 账户设置

要在真实的 IBM Quantum 硬件上运行量子电路，您需要 IBM Quantum 账户和 API 令牌。

### 创建账户

1. 访问 [IBM Quantum 平台](https://quantum.ibm.com/)
2. 注册免费账户
3. 进入账户设置页面获取 API 令牌

### 配置认证

保存 IBM Quantum 凭证：

```python
from qiskit_ibm_runtime import QiskitRuntimeService

# 保存凭证（仅首次需要）
QiskitRuntimeService.save_account(
    channel="ibm_quantum",
    token="YOUR_IBM_QUANTUM_TOKEN"
)

# 后续会话 - 加载已保存凭证
service = QiskitRuntimeService()
```

### 环境变量方法

也可将 API 令牌设为环境变量：

```bash
export QISKIT_IBM_TOKEN="YOUR_IBM_QUANTUM_TOKEN"
```

## 本地开发（无需账户）

无需 IBM Quantum 账户即可使用模拟器本地构建和测试量子电路：

```python
from qiskit import QuantumCircuit
from qiskit.primitives import StatevectorSampler

qc = QuantumCircuit(2)
qc.h(0)
qc.cx(0, 1)
qc.measure_all()

# 使用模拟器本地运行
sampler = StatevectorSampler()
result = sampler.run([qc], shots=1024).result()
```

## 验证安装

测试安装是否成功：

```python
import qiskit
print(qiskit.__version__)

from qiskit import QuantumCircuit
qc = QuantumCircuit(2)
print("Qiskit 安装成功！")
```
