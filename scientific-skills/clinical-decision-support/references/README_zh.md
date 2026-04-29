# 临床决策支持技能

面向制药和临床研究领域医疗专业人员的专业临床决策支持文档。

## 快速入门

本技能支持生成三类临床文档：

1. **个体患者治疗计划** - 针对特定患者的个性化治疗方案
2. **患者队列分析** - 基于生物标志物分层的群体结果分析
3. **治疗推荐报告** - 循证临床指南

所有文档均生成为紧凑的专业LaTeX/PDF文件。

## 目录结构

```
clinical-decision-support/
├── SKILL.md                     # 主技能定义
├── README.md                    # 本文件
│
├── references/                  # 临床指导文档
│   ├── patient_cohort_analysis.md
│   ├── treatment_recommendations.md
│   ├── clinical_decision_algorithms.md
│   ├── biomarker_classification.md
│   ├── outcome_analysis.md
│   └── evidence_synthesis.md
│
├── assets/                      # 模板和示例
│   ├── cohort_analysis_template.tex
│   ├── treatment_recommendation_template.tex
│   ├── clinical_pathway_template.tex
│   ├── biomarker_report_template.tex
│   ├── example_gbm_cohort.md
│   ├── recommendation_strength_guide.md
│   └── color_schemes.tex
│
└── scripts/                     # 分析和生成工具
    ├── generate_survival_analysis.py
    ├── create_cohort_tables.py
    ├── build_decision_tree.py
    ├── biomarker_classifier.py
    └── validate_cds_document.py
```

## 应用示例

### 创建患者队列分析
```
> 分析包含45名NSCLC患者的队列，按PD-L1表达分层
  (<1%、1-49%、≥50%)，包含ORR、PFS和OS结局指标
```

### 生成治疗推荐
```
> 使用GRADE方法为HER2阳性转移性乳腺癌
  创建循证治疗推荐
```

### 构建临床路径
```
> 使用TIMI风险评分生成急性胸痛管理的
  临床决策算法
```

## 核心功能

- **GRADE方法学**：证据质量分级（高/中/低/极低）
- **推荐强度**：强推荐（1级）与条件推荐（2级）
- **生物标志物整合**：基因组学、表达谱和分子亚型分类
- **统计分析**：Kaplan-Meier、Cox回归、log-rank检验
- **指南一致性**：整合NCCN、ASCO、ESMO、AHA/ACC标准
- **专业输出**：0.5英寸页边距、颜色编码框、达到发表质量

## 依赖项

Python脚本需要：
- `pandas`, `numpy`, `scipy`：数据分析和统计
- `lifelines`：生存分析（Kaplan-Meier、Cox回归）
- `matplotlib`：可视化
- `pyyaml`（可选）：决策树YAML输入

安装命令：
```bash
pip install pandas numpy scipy lifelines matplotlib pyyaml
```

## 包含参考文献

1. **患者队列分析**：分层方法、生物标志物相关性、统计比较
2. **治疗推荐**：证据分级、治疗排序、特殊人群考量
3. **临床决策算法**：风险评分、决策树、TikZ流程图
4. **生物标志物分类**：基因组变异、分子亚型、伴随诊断
5. **结局分析**：生存分析方法、疗效标准（RECIST）、效应量
6. **证据综合**：指南整合、系统评价、荟萃分析

## 提供模板

1. **队列分析**：人口统计表、生物标志物谱、结局指标、统计结果、建议
2. **治疗推荐**：证据综述、GRADE分级选项、监测方案、决策算法
3. **临床路径**：含风险分层和紧急程度编码的TikZ流程图
4. **生物标志物报告**：基于分级可操作性的基因组分析及疗法匹配

## 包含脚本

1. **`generate_survival_analysis.py`**：创建含风险比的Kaplan-Meier曲线
2. **`create_cohort_tables.py`**：生成基线、疗效和安全性表格
3. **`build_decision_tree.py`**：将文本/JSON转换为TikZ流程图
4. **`biomarker_classifier.py`**：按PD-L1、HER2、分子亚型对患者分层
5. **`validate_cds_document.py`**：完整性与合规性质量检查

## 集成能力

与现有技能集成：
- **scientific-writing**：文献管理、统计报告
- **clinical-reports**：医学术语、HIPAA合规性
- **scientific-schematics**：TikZ流程图

## 版本信息

版本 1.0 - 初始发布
创建时间：2024年11月
最后更新：2024年11月5日

## 问题反馈

本技能专为制药和临床研究领域创建临床决策支持文档的专业人员设计。有关使用问题或改进建议，请联系科学写作开发团队。
