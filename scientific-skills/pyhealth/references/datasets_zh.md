# PyHealth 数据集与数据结构

## 核心数据结构

### 事件（Event）
单次医疗发生项，包含以下属性：
- **code**：医疗编码（诊断、药物、手术、实验室检验）
- **vocabulary**：编码体系（ICD-9-CM、NDC、LOINC等）
- **timestamp**：事件发生时间
- **value**：数值（用于实验室检验、生命体征）
- **unit**：测量单位

### 患者（Patient）
按时间顺序组织的就诊事件集合，包含：
- **patient_id**：唯一标识符
- **birth_datetime**：出生日期
- **gender**：患者性别
- **ethnicity**：患者种族
- **visits**：就诊对象列表

### 就诊（Visit）
医疗接触事件，包含：
- **visit_id**：唯一标识符
- **encounter_time**：就诊时间戳
- **discharge_time**：出院时间戳
- **visit_type**：接触类型（住院、门诊、急诊）
- **events**：本次就诊期间的事件列表

## BaseDataset 基类

**核心方法：**
- `get_patient(patient_id)`：获取单条患者记录
- `iter_patients()`：遍历所有患者
- `stats()`：获取数据集统计信息（患者数、就诊数、事件数）
- `set_task(task_fn)`：定义预测任务

## 可用数据集

### 电子健康记录（EHR）数据集

**MIMIC-III 数据集** (`MIMIC3Dataset`)
- 来自贝斯以色列女执事医疗中心的重症监护数据
- 40,000+ 重症患者
- 包含诊断、手术、药物、实验室结果
- 用法：`from pyhealth.datasets import MIMIC3Dataset`

**MIMIC-IV 数据集** (`MIMIC4Dataset`)
- 升级版本包含 70,000+ 患者
- 改进的数据质量和覆盖范围
- 增强的人口统计和临床细节
- 用法：`from pyhealth.datasets import MIMIC4Dataset`

**eICU 数据集** (`eICUDataset`)
- 多中心重症监护数据库
- 200+ 医院的 200,000+ 入院记录
- 跨机构的标准化ICU数据
- 用法：`from pyhealth.datasets import eICUDataset`

**OMOP 数据集** (`OMOPDataset`)
- 观察性医疗结果合作组织格式
- 标准化通用数据模型
- 支持跨医疗系统互操作
- 用法：`from pyhealth.datasets import OMOPDataset`

**EHRShot 数据集** (`EHRShotDataset`)
- 少样本学习基准数据集
- 专为测试模型泛化能力设计
- 用法：`from pyhealth.datasets import EHRShotDataset`

### 生理信号数据集

**睡眠脑电图数据集：**
- `SleepEDFDataset`：用于睡眠分期的Sleep-EDF数据库
- `SHHSDataset`：睡眠心脏健康研究数据
- `ISRUCDataset`：ISRUC-Sleep数据库

**天普大学脑电图数据集：**
- `TUEVDataset`：异常脑电事件检测
- `TUABDataset`：异常/正常脑电分类
- `TUSZDataset`：癫痫发作检测

**所有信号数据集支持：**
- 多通道脑电信号
- 标准化采样率
- 专家标注
- 睡眠分期或异常标签

### 医学影像数据集

**COVID-19 胸部X光数据集** (`COVID19CXRDataset`)
- 用于COVID-19分类的胸部X光影像
- 多类别标签（COVID-19、肺炎、正常）
- 用法：`from pyhealth.datasets import COVID19CXRDataset`

### 文本数据集

**医疗转录数据集** (`MedicalTranscriptionsDataset`)
- 临床记录与转录文本
- 医疗专科分类
- 基于文本的预测任务
- 用法：`from pyhealth.datasets import MedicalTranscriptionsDataset`

**心脏病学数据集** (`CardiologyDataset`)
- 心脏病患者记录
- 心血管疾病预测
- 用法：`from pyhealth.datasets import CardiologyDataset`

### 预处理数据集

**MIMIC 提取数据集** (`MIMICExtractDataset`)
- 预提取的MIMIC特征
- 开箱即用的基准数据
- 减少预处理需求
- 用法：`from pyhealth.datasets import MIMICExtractDataset`

## SampleDataset 类

将原始数据集转换为任务特定的格式化样本。

**用途：** 将患者级数据转换为模型就绪的输入/输出对

**核心属性：**
- `input_schema`：定义输入数据结构
- `output_schema`：定义目标标签/预测值
- `samples`：处理后的样本列表

**使用模式：**
```python
# 在BaseDataset上设置任务后
sample_dataset = dataset.set_task(task_fn)
```

## 数据划分函数

**患者级划分** (`split_by_patient`)
- 确保患者不跨划分组出现
- 防止数据泄露
- 临床预测任务推荐使用

**就诊级划分** (`split_by_visit`)
- 按单次就诊划分
- 允许同一患者跨划分组（需谨慎使用）

**样本级划分** (`split_by_sample`)
- 随机样本划分
- 最灵活但可能导致泄露

**参数：**
- `dataset`：待划分的SampleDataset
- `ratios`：划分比例元组（如 [0.7, 0.1, 0.2]）
- `seed`：保证可复现性的随机种子

## 通用工作流

```python
from pyhealth.datasets import MIMIC4Dataset
from pyhealth.tasks import mortality_prediction_mimic4_fn
from pyhealth.datasets import split_by_patient

# 1. 加载数据集
dataset = MIMIC4Dataset(root="/path/to/data")

# 2. 设置预测任务
sample_dataset = dataset.set_task(mortality_prediction_mimic4_fn)

# 3. 划分数据
train, val, test = split_by_patient(sample_dataset, [0.7, 0.1, 0.2])

# 4. 获取统计信息
print(dataset.stats())
```

## 性能说明

- PyHealth 处理医疗数据的速度**比 pandas 快 3 倍**
- 针对大规模EHR数据集优化
- 内存高效的患者遍历
- 特征提取的向量化操作
