# PyHealth 医疗编码翻译

## 概述

医疗健康数据使用多种编码系统和标准。PyHealth 的 MedCode 模块通过本体查询和跨系统映射，实现医疗编码系统间的翻译与映射。

## 核心类

### InnerMap
处理系统内部本体查询和层级导航。

**关键能力：**
- 带属性（名称、描述）的编码查询
- 祖先/后代层级遍历
- 编码标准化与转换
- 父子关系导航

### CrossMap
管理不同编码标准间的跨系统映射。

**关键能力：**
- 编码系统间翻译
- 多对多关系处理
- 层级规范（针对药物）
- 双向映射支持

## 支持的编码系统

### 诊断代码

**ICD-9-CM（国际疾病分类第九版临床修订版）**
- 传统诊断编码系统
- 3-5位数字的层级结构
- 2015年前美国医疗系统使用
- 用法：`from pyhealth.medcode import InnerMap`
  - `icd9_map = InnerMap.load("ICD9CM")`

**ICD-10-CM（国际疾病分类第十版临床修订版）**
- 当前诊断编码标准
- 字母数字混合编码（3-7位）
- 比ICD-9更精细
- 用法：`from pyhealth.medcode import InnerMap`
  - `icd10_map = InnerMap.load("ICD10CM")`

**CCSCM（ICD-CM临床分类软件）**
- 将ICD代码分组为临床意义类别
- 降低分析维度
- 单层级与多层级结构
- 用法：`from pyhealth.medcode import CrossMap`
  - `icd_to_ccs = CrossMap.load("ICD9CM", "CCSCM")`

### 操作代码

**ICD-9-PROC（ICD-9操作代码）**
- 住院操作分类
- 3-4位数字编码
- 传统系统（2015年前）
- 用法：`from pyhealth.medcode import InnerMap`
  - `icd9proc_map = InnerMap.load("ICD9PROC")`

**ICD-10-PROC（ICD-10操作编码系统）**
- 当前操作编码标准
- 7位字母数字编码
- 比ICD-9-PROC更详细
- 用法：`from pyhealth.medcode import InnerMap`
  - `icd10proc_map = InnerMap.load("ICD10PROC")`

**CCSPROC（操作临床分类软件）**
- 将操作代码分组为类别
- 简化操作分析
- 用法：`from pyhealth.medcode import CrossMap`
  - `proc_to_ccs = CrossMap.load("ICD9PROC", "CCSPROC")`

### 药物代码

**NDC（国家药品代码）**
- 美国FDA药品标识系统
- 10或11位数字编码
- 产品级粒度（厂商、规格、包装）
- 用法：`from pyhealth.medcode import InnerMap`
  - `ndc_map = InnerMap.load("NDC")`

**RxNorm**
- 标准化药品术语
- 规范化药品名称与关系
- 链接多药品词库
- 用法：`from pyhealth.medcode import CrossMap`
  - `ndc_to_rxnorm = CrossMap.load("NDC", "RXNORM")`

**ATC（解剖治疗化学分类系统）**
- WHO药品分类系统
- 5级层级结构：
  - **1级**：解剖主组（1字母）
  - **2级**：治疗亚组（2数字）
  - **3级**：药理亚组（1字母）
  - **4级**：化学亚组（1字母）
  - **5级**：化学物质（2数字）
- 示例："C03CA01" = 呋塞米
  - C = 心血管系统
  - C03 = 利尿剂
  - C03C = 高效利尿剂
  - C03CA = 磺胺类
  - C03CA01 = 呋塞米

**用法：**
```python
from pyhealth.medcode import CrossMap
ndc_to_atc = CrossMap.load("NDC", "ATC")
atc_codes = ndc_to_atc.map("00074-3799-13", level=3)  # 获取ATC 3级
```

## 常用操作

### InnerMap 操作

**1. 编码查询**
```python
from pyhealth.medcode import InnerMap

icd9_map = InnerMap.load("ICD9CM")
info = icd9_map.lookup("428.0")  # 心力衰竭
# 返回：名称、描述、附加属性
```

**2. 祖先遍历**
```python
# 获取层级中所有父编码
ancestors = icd9_map.get_ancestors("428.0")
# 返回：["428", "420-429", "390-459"]
```

**3. 后代遍历**
```python
# 获取所有子编码
descendants = icd9_map.get_descendants("428")
# 返回：["428.0", "428.1", "428.2", ...]
```

**4. 编码标准化**
```python
# 规范化编码格式
standard_code = icd9_map.standardize("4280")  # 返回 "428.0"
```

### CrossMap 操作

**1. 直接翻译**
```python
from pyhealth.medcode import CrossMap

# ICD-9-CM 转 CCS
icd_to_ccs = CrossMap.load("ICD9CM", "CCSCM")
ccs_codes = icd_to_ccs.map("82101")  # 冠状动脉粥样硬化
# 返回：["101"]  # 冠状动脉粥样硬化的CCS类别
```

**2. 层级药物映射**
```python
# 不同级别的NDC转ATC映射
ndc_to_atc = CrossMap.load("NDC", "ATC")

# 获取特定ATC级别
atc_level_1 = ndc_to_atc.map("00074-3799-13", level=1)  # 解剖组
atc_level_3 = ndc_to_atc.map("00074-3799-13", level=3)  # 药理组
atc_level_5 = ndc_to_atc.map("00074-3799-13", level=5)  # 化学物质
```

**3. 双向映射**
```python
# 双向映射
rxnorm_to_ndc = CrossMap.load("RXNORM", "NDC")
ndc_codes = rxnorm_to_ndc.map("197381")  # 获取RxNorm对应的所有NDC编码
```

## 工作流示例

### 示例1：标准化并分组诊断
```python
from pyhealth.medcode import InnerMap, CrossMap

# 加载映射
icd9_map = InnerMap.load("ICD9CM")
icd_to_ccs = CrossMap.load("ICD9CM", "CCSCM")

# 处理诊断编码
raw_codes = ["4280", "428.0", "42800"]

standardized = [icd9_map.standardize(code) for code in raw_codes]
# 全部转为 "428.0"

ccs_categories = [icd_to_ccs.map(code)[0] for code in standardized]
# 全部映射到CCS类别 "108"（心力衰竭）
```

### 示例2：药物分类分析
```python
from pyhealth.medcode import CrossMap

# 将NDC映射到ATC进行药物类别分析
ndc_to_atc = CrossMap.load("NDC", "ATC")

patient_drugs = ["00074-3799-13", "00074-7286-01", "00456-0765-01"]

# 获取治疗亚组（ATC 2级）
drug_classes = []
for ndc in patient_drugs:
    atc_codes = ndc_to_atc.map(ndc, level=2)
    if atc_codes:
        drug_classes.append(atc_codes[0])

# 分析药物类别分布
```

### 示例3：ICD-9到ICD-10迁移
```python
from pyhealth.medcode import CrossMap

# 加载ICD-9到ICD-10映射
icd9_to_icd10 = CrossMap.load("ICD9CM", "ICD10CM")

# 转换历史ICD-9编码
icd9_code = "428.0"
icd10_codes = icd9_to_icd10.map(icd9_code)
# 返回：["I50.9", "I50.1", ...]  # 多个可能的ICD-10编码

# 处理一对多映射
for icd10_code in icd10_codes:
    print(f"ICD-9 {icd9_code} -> ICD-10 {icd10_code}")
```

## 与数据集集成

医疗编码翻译与PyHealth数据集无缝集成：

```python
from pyhealth.datasets import MIMIC4Dataset
from pyhealth.medcode import CrossMap

# 加载数据集
dataset = MIMIC4Dataset(root="/path/to/data")

# 加载编码映射
icd_to_ccs = CrossMap.load("ICD10CM", "CCSCM")

# 处理患者诊断
for patient in dataset.iter_patients():
    for visit in patient.visits:
        diagnosis_events = [e for e in visit.events if e.vocabulary == "ICD10CM"]

        for event in diagnosis_events:
            ccs_codes = icd_to_ccs.map(event.code)
            print(f"诊断 {event.code} -> CCS {ccs_codes}")
```

## 用例

### 临床研究
- 跨编码系统标准化诊断
- 分组相关病症以识别队列
- 协调使用不同标准的多中心研究

### 药物安全分析
- 按治疗类别分类药物
- 识别类别级药物相互作用
- 分析多重用药模式

### 医疗健康分析
- 降低诊断/操作维度
- 创建有临床意义的类别
- 支持跨编码系统变更的纵向分析

### 机器学习
- 创建一致的特征表示
- 处理训练/测试数据中的词库不匹配
- 生成层级嵌入

## 最佳实践

1. **始终先标准化编码** 以确保格式一致
2. **正确处理一对多映射**（部分编码映射到多个目标）
3. **明确指定ATC级别** 以避免药物映射歧义
4. **使用CCS类别** 降低诊断/操作维度
5. **验证映射关系**（部分编码可能无直接翻译）
6. **记录编码版本**（ICD-9 vs ICD-10）以保持数据溯源
