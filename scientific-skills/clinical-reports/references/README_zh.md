# 临床报告撰写技能

## 概述

用于撰写临床报告的综合性技能，涵盖病例报告、诊断报告、临床试验报告及患者文档。提供模板支持、法规合规性指导和验证工具。

## 包含内容

### 📋 四大报告类型

1. **临床病例报告** - 符合CARE标准的医学期刊发表用病例报告
2. **诊断报告** - 放射学(ACR)、病理学(CAP)及实验室报告
3. **临床试验报告** - SAE报告、临床研究报告(ICH-E3)、DSMB报告
4. **患者文档** - SOAP病程记录、入院记录(H&P)、出院小结、会诊记录

### 📚 参考文件（8份综合指南）

- `case_report_guidelines.md` - CARE指南、去标识化处理、期刊要求
- `diagnostic_reports_standards.md` - ACR/CAP/CLSI标准、结构化报告系统
- `clinical_trial_reporting.md` - ICH-E3/CONSORT/SAE报告标准、MedDRA编码
- `patient_documentation.md` - SOAP病程记录/H&P/出院小结标准
- `regulatory_compliance.md` - HIPAA/21 CFR Part 11/ICH-GCP/FDA法规
- `medical_terminology.md` - SNOMED-CT/LOINC/ICD-10/CPT编码
- `data_presentation.md` - 临床表格/图示/Kaplan-Meier曲线
- `peer_review_standards.md` - 临床稿件评审标准

### 📄 模板（12个专业模板）

- `case_report_template.md` - 遵循CARE指南的结构化病例报告
- `soap_note_template.md` - SOAP病程记录格式
- `history_physical_template.md` - 完整入院体检模板
- `discharge_summary_template.md` - 出院医疗文书
- `consult_note_template.md` - 专科会诊记录格式
- `radiology_report_template.md` - 含结构化报告的影像报告
- `pathology_report_template.md` - 含CAP摘要要素的手术病理报告
- `lab_report_template.md` - 临床实验室检测报告
- `clinical_trial_sae_template.md` - 严重不良事件报告表
- `clinical_trial_csr_template.md` - 临床研究报告框架(ICH-E3)
- `quality_checklist.md` - 全类型报告质量核查表
- `hipaa_compliance_checklist.md` - 隐私与去标识化验证表

### 🔧 验证脚本（8个自动化工具）

- `validate_case_report.py` - 检查CARE指南合规性与完整性
- `check_deidentification.py` - 扫描报告中18项HIPAA标识符
- `validate_trial_report.py` - 验证ICH-E3结构与必备要素
- `format_adverse_events.py` - 从CSV数据生成AE汇总表
- `generate_report_template.py` - 交互式模板选择与生成
- `extract_clinical_data.py` - 解析提取结构化临床数据
- `compliance_checker.py` - 验证法规合规要求
- `terminology_validator.py` - 校验医学术语与禁用缩写

## 快速入门

### 生成模板

```bash
cd .claude/skills/clinical-reports/scripts
python generate_report_template.py

# 或直接指定类型
python generate_report_template.py --type case_report --output my_case_report.md
```

### 验证病例报告

```bash
python validate_case_report.py my_case_report.md
```

### 检查去标识化

```bash
python check_deidentification.py my_case_report.md
```

### 验证临床试验报告

```bash
python validate_trial_report.py my_csr.md
```

## 核心功能

### CARE指南合规性
- 完整覆盖CARE核查清单
- 去标识化验证
- 知情同意文件管理
- 时间线创建辅助
- 文献综述整合

### 法规合规性
- **HIPAA** - 隐私保护、18项标识符清除、安全港方法
- **FDA** - 21 CFR Parts 11/50/56/312合规
- **ICH-GCP** - 临床试验质量管理规范
- **ALCOA-CCEA** - 数据完整性原则

### 专业标准
- **ACR** - 美国放射学会报告标准
- **CAP** - 美国病理学家学会摘要报告
- **CLSI** - 临床实验室标准协会
- **CONSORT** - 临床试验报告标准
- **ICH-E3** - 临床研究报告结构

### 医疗编码系统
- **ICD-10-CM** - 诊断编码
- **CPT** - 操作编码
- **SNOMED-CT** - 临床术语系统
- **LOINC** - 实验室观测编码
- **MedDRA** - 监管活动医学词典

## 典型应用场景

### 1. 发表临床病例报告

```
> 为65岁非典型急性阑尾炎患者创建临床病例报告

> 检查该报告是否符合HIPAA要求
> 验证是否符合CARE指南
```

### 2. 撰写诊断报告

```
> 生成胸部CT影像报告模板
> 创建结肠切除标本腺癌病理报告
> 编写全血细胞计数实验室报告
```

### 3. 临床试验文档

```
> 撰写因肺炎住院的严重不良事件报告
> 创建III期糖尿病试验临床研究报告框架
> 从试验数据生成不良事件汇总表
```

### 4. 患者临床记录

```
> 创建随访SOAP病程记录
> 为胸痛入院患者生成H&P记录
> 撰写心衰住院患者出院小结
> 创建心脏病学会诊记录
```

## 工作流示例

### 病例报告工作流

1. 获取患者**知情同意书**
2. **生成模板**：`python generate_report_template.py --type case_report`
3. 按CARE结构**撰写报告**
4. **合规验证**：`python validate_case_report.py case_report.md`
5. **去标识化检查**：`python check_deidentification.py case_report.md`
6. 附CARE清单**提交期刊**

### 临床试验SAE工作流

1. **生成SAE模板**：`python generate_report_template.py --type sae`
2. 事件发生后24小时内**完成SAE表格**
3. 使用WHO-UMC或Naranjo标准**评估因果关系**
4. **完整性验证**：`python validate_trial_report.py sae_report.md`
5. 按法规时限(7/15天)**提交申办方**
6. 依机构政策**通知伦理委员会(IRB)**

## 最佳实践

### 隐私与伦理
✓ 病例报告必须获取知情同意  
✓ 发表前清除所有18项HIPAA标识符  
✓ 使用去标识化验证脚本  
✓ 在稿件中记录同意过程  
✓ 评估罕见病的再识别风险  

### 临床质量
✓ 使用专业医学术语  
✓ 遵循结构化报告模板  
✓ 包含所有必备要素  
✓ 清晰记录时间线  
✓ 提供诊断依据  

### 法规合规
✓ 遵守SAE报告时限(7天/15天)  
✓ CSR报告遵循ICH-E3结构  
✓ 保持ALCOA-CCEA数据完整性  
✓ 记录方案依从性  
✓ 采用MedDRA编码不良事件  

### 文档规范
✓ 所有临床记录需签名并注明日期  
✓ 记录医疗必要性  
✓ 仅使用标准缩写  
✓ 禁用JCAHO"禁止使用"缩写列表  
✓ 保持清晰完整  

## 集成能力

本临床报告技能可与以下模块无缝集成：

- **科研写作** - 专业医学写作支持
- **同行评审** - 病例报告质量评估
- **文献管理** - 病例报告参考文献处理
- **科研基金** - 临床试验方案开发

## 资源

### 外部标准
- CARE指南：https://www.care-statement.org/
- ICH-E3指南：https://database.ich.org/sites/default/files/E3_Guideline.pdf
- CONSORT声明：http://www.consort-statement.org/
- HIPAA：https://www.hhs.gov/hipaa/
- ACR实践参数：https://www.acr.org/Clinical-Resources/Practice-Parameters-and-Technical-Standards
- CAP癌症协议：https://www.cap.org/protocols-and-guidelines

### 专业组织
- 美国医学会(AMA)
- 美国放射学会(ACR)
- 美国病理学家学会(CAP)
- 临床实验室标准协会(CLSI)
- 国际人用药品注册技术协调会(ICH)

## 技术支持

临床报告技能相关问题处理流程：
1. 查阅综合参考文件
2. 查看模板示例
3. 运行验证脚本定位问题
4. 参考SKILL.md获取详细指南

## 许可协议

Claude科研写作项目组成部分，详见主LICENSE文件。
