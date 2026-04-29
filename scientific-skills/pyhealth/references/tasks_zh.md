# PyHealth 临床预测任务

## 概述
PyHealth 为常见医疗人工智能应用提供了 20 多个预定义的临床预测任务。每个任务函数将原始患者数据转换为结构化输入-输出对，用于模型训练。

## 任务函数结构
所有任务函数继承自 `BaseTask` 并提供：
- **input_schema**：定义输入特征（诊断、药物、实验室检查等）
- **output_schema**：定义预测目标（标签、数值）
- **pre_filter()**：可选的患者/就诊记录过滤逻辑

**使用模式：**
```python
from pyhealth.datasets import MIMIC4Dataset
from pyhealth.tasks import mortality_prediction_mimic4_fn

dataset = MIMIC4Dataset(root="/path/to/data")
sample_dataset = dataset.set_task(mortality_prediction_mimic4_fn)
```

## 电子健康记录（EHR）任务

### 死亡率预测
**目的：** 预测患者下次就诊或指定时间窗内的死亡风险

**MIMIC-III 死亡率** (`mortality_prediction_mimic3_fn`)
- 预测下次住院期间的死亡
- 二分类任务
- 输入：历史诊断、手术、药物
- 输出：二分类标签（死亡/存活）

**MIMIC-IV 死亡率** (`mortality_prediction_mimic4_fn`)
- MIMIC-IV 数据集更新版本
- 增强的特征集
- 改进的标签质量

**eICU 死亡率** (`mortality_prediction_eicu_fn`)
- 多中心 ICU 死亡率预测
- 考虑医院级别差异

**OMOP 死亡率** (`mortality_prediction_omop_fn`)
- 标准化死亡率预测
- 兼容 OMOP 通用数据模型

**院内死亡率** (`inhospital_mortality_prediction_mimic4_fn`)
- 预测当前住院期间的死亡
- 实时风险评估
- 预测窗口早于下次就诊死亡率

**StageNet 死亡率** (`mortality_prediction_mimic4_fn_stagenet`)
- 专为 StageNet 模型架构设计
- 时序阶段感知预测

### 再入院预测
**目的：** 识别指定时间窗（通常 30 天）内再入院高风险患者

**MIMIC-III 再入院** (`readmission_prediction_mimic3_fn`)
- 30 天再入院预测
- 二分类
- 输入：诊断史、药物、人口统计
- 输出：二分类标签（再入院/未再入院）

**MIMIC-IV 再入院** (`readmission_prediction_mimic4_fn`)
- 增强的再入院特征
- 改进的时序建模

**eICU 再入院** (`readmission_prediction_eicu_fn`)
- ICU 特定再入院风险
- 多中心数据

**OMOP 再入院** (`readmission_prediction_omop_fn`)
- 标准化再入院预测

### 住院时长预测
**目的：** 估算住院时长用于资源规划和患者管理

**MIMIC-III 住院时长** (`length_of_stay_prediction_mimic3_fn`)
- 回归任务
- 输入：入院诊断、生命体征、人口统计
- 输出：连续值（天数）

**MIMIC-IV 住院时长** (`length_of_stay_prediction_mimic4_fn`)
- 增强的住院时长预测特征
- 更精细的时序粒度

**eICU 住院时长** (`length_of_stay_prediction_eicu_fn`)
- ICU 停留时长预测
- 多医院数据

**OMOP 住院时长** (`length_of_stay_prediction_omop_fn`)
- 标准化住院时长预测

### 药物推荐
**目的：** 根据患者病史和当前状况推荐合适药物

**MIMIC-III 药物推荐** (`drug_recommendation_mimic3_fn`)
- 多标签分类
- 输入：诊断、既往用药、人口统计
- 输出：推荐药物代码集合
- 考虑药物相互作用

**MIMIC-IV 药物推荐** (`drug_recommendation_mimic4_fn`)
- 更新的药物数据
- 增强的相互作用建模

**eICU 药物推荐** (`drug_recommendation_eicu_fn`)
- 重症监护药物推荐

**OMOP 药物推荐** (`drug_recommendation_omop_fn`)
- 标准化药物推荐

**关键考量：**
- 处理多重用药场景
- 多标签预测（每位患者多种药物）
- 可集成 SafeDrug/GAMENet 模型实现安全感知推荐

## 专业临床任务

### 医疗编码
**MIMIC-III ICD-9 编码** (`icd9_coding_mimic3_fn`)
- 为临床笔记分配 ICD-9 诊断/手术编码
- 多标签文本分类
- 输入：临床文本/文档
- 输出：ICD-9 编码集合
- 同时支持诊断和手术编码

### 患者关联
**MIMIC-III 患者链接** (`patient_linkage_mimic3_fn`)
- 记录匹配与去重
- 二分类（是否同一患者）
- 输入：两条记录的人口统计和临床特征
- 输出：匹配概率

## 生理信号任务

### 睡眠分期
**目的：** 通过 EEG/生理信号分类睡眠阶段用于睡眠障碍诊断

**ISRUC 睡眠分期** (`sleep_staging_isruc_fn`)
- 多分类（清醒期、N1、N2、N3、REM）
- 输入：多通道 EEG 信号
- 输出：每时段睡眠阶段（通常 30 秒）

**SleepEDF 睡眠分期** (`sleep_staging_sleepedf_fn`)
- 标准睡眠分期任务
- PSG 信号处理

**SHHS 睡眠分期** (`sleep_staging_shhs_fn`)
- 大规模睡眠研究数据
- 群体水平睡眠分析

**标准化标签：**
- 清醒期 (W)
- 非快速眼动阶段 1 (N1)
- 非快速眼动阶段 2 (N2)
- 非快速眼动阶段 3 (N3/深睡期)
- 快速眼动期 (REM)

### EEG 分析
**异常检测** (`abnormality_detection_tuab_fn`)
- 二分类（正常/异常 EEG）
- 临床筛查应用
- 输入：多通道 EEG 记录
- 输出：二分类标签

**事件检测** (`event_detection_tuev_fn`)
- 识别特定 EEG 事件（尖波、癫痫发作）
- 多分类
- 输入：EEG 时间序列
- 输出：事件类型与时间

**癫痫发作检测** (`seizure_detection_tusz_fn`)
- 专业癫痫发作检测
- 癫痫监测关键应用
- 输入：连续 EEG
- 输出：癫痫发作/非癫痫发作分类

## 医学影像任务

### COVID-19 胸部 X 光分类
**COVID-19 CXR** (`covid_classification_cxr_fn`)
- 多分类图像识别
- 类别：COVID-19、细菌性肺炎、病毒性肺炎、正常
- 输入：胸部 X 光影像
- 输出：疾病分类

## 文本任务

### 医疗转录分类
**医疗专科分类** (`medical_transcription_classification_fn`)
- 按医疗专科分类临床笔记
- 多分类文本识别
- 输入：临床转录文本
- 输出：医疗专科（心脏病学、神经学等）

## 自定义任务创建

### 创建自定义任务
通过指定输入/输出模式定义自定义预测任务：
```python
from pyhealth.tasks import BaseTask

def custom_task_fn(patient):
    """自定义预测任务"""

    # 定义输入特征
    samples = []

    for i, visit in enumerate(patient.visits):
        # 跳过历史不足的记录
        if i < 2:
            continue

        # 从历史就诊创建输入
        input_info = {
            "diagnoses": [],
            "medications": [],
            "procedures": []
        }

        # 收集既往就诊特征
        for past_visit in patient.visits[:i]:
            for event in past_visit.events:
                if event.vocabulary == "ICD10CM":
                    input_info["diagnoses"].append(event.code)
                elif event.vocabulary == "NDC":
                    input_info["medications"].append(event.code)

        # 定义预测目标
        # 示例：预测当前就诊特定结果
        output_info = {
            "label": 1 if some_condition else 0
        }

        samples.append({
            "patient_id": patient.patient_id,
            "visit_id": visit.visit_id,
            "input_info": input_info,
            "output_info": output_info
        })

    return samples

# 应用自定义任务
sample_dataset = dataset.set_task(custom_task_fn)
```

### 任务函数组件
1. **输入模式定义**
   - 指定提取特征
   - 定义特征类型（编码、序列、数值）
   - 设置时间窗口

2. **输出模式定义**
   - 定义预测目标
   - 设置标签类型（二分类、多分类、多标签、回归）
   - 指定评估指标

3. **过滤逻辑**
   - 排除数据不足的患者/就诊
   - 应用纳入/排除标准
   - 处理缺失数据

4. **样本生成**
   - 创建输入-输出对
   - 保留患者/就诊标识符
   - 维持时序顺序

## 任务选择指南

### 临床预测任务
**适用场景：** 处理结构化 EHR 数据（诊断、药物、手术）

**数据集：** MIMIC-III、MIMIC-IV、eICU、OMOP

**常见任务：**
- 死亡率预测用于风险分层
- 再入院预测用于护理过渡规划
- 住院时长预测用于资源分配
- 药物推荐用于临床决策支持

### 信号处理任务
**适用场景：** 处理生理时序数据

**数据集：** SleepEDF、SHHS、ISRUC、TUEV、TUAB、TUSZ

**常见任务：**
- 睡眠分期用于睡眠障碍诊断
- EEG 异常检测用于筛查
- 癫痫发作检测用于癫痫监测

### 影像任务
**适用场景：** 处理医学影像

**数据集：** COVID-19 CXR

**常见任务：**
- 放射影像疾病分类
- 异常检测

### 文本任务
**适用场景：** 处理临床笔记和文档

**数据集：** 医疗转录文本、MIMIC-III（含笔记）

**常见任务：**
- 临床文本医疗编码
- 专科分类
- 临床信息抽取

## 任务输出结构
所有任务函数返回包含以下结构的 `SampleDataset`：
```python
sample = {
    "patient_id": "unique_patient_id",
    "visit_id": "unique_visit_id",  # 如适用
    "input_info": {
        # 输入特征（诊断、药物等）
    },
    "output_info": {
        # 预测目标（标签、数值）
    }
}
```

## 模型集成
任务定义模型的输入/输出契约：
```python
from pyhealth.datasets import MIMIC4Dataset
from pyhealth.tasks import mortality_prediction_mimic4_fn
from pyhealth.models import Transformer

# 1. 创建任务特定数据集
dataset = MIMIC4Dataset(root="/path/to/data")
sample_dataset = dataset.set_task(mortality_prediction_mimic4_fn)

# 2. 模型自动适配任务模式
model = Transformer(
    dataset=sample_dataset,
    feature_keys=["diagnoses", "medications"],
    mode="binary",  # 匹配任务输出
)
```

## 最佳实践
1. **任务匹配临床问题**：优先选择预定义任务进行标准化基准测试
2. **考虑时间窗口**：确保足够历史数据支撑有效预测
3. **处理类别不平衡**：多数临床结局罕见（死亡、再入院）
4. **验证临床相关性**：确保预测窗口与临床决策时间线对齐
5. **使用合适指标**：不同任务需不同评估指标（二分类用 AUROC，多分类用 macro-F1）
6. **记录排除标准**：追踪过滤的患者/就诊及原因
7. **保护患者隐私**：始终使用去标识化数据并遵循 HIPAA/GDPR 规范
