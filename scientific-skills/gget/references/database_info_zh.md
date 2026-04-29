# gget 数据库信息

gget 模块查询的数据库概览，包括更新频率和重要注意事项。

## 重要提示

gget 查询的数据库持续更新，有时会改变其结构。gget 模块每两周自动测试一次，必要时会更新以匹配新的数据库结构。请始终保持 gget 为最新版本：

```bash
pip install --upgrade gget
```

## 数据库目录

### 基因组参考数据库

#### Ensembl
- **使用模块：** gget ref, gget search, gget info, gget seq
- **描述：** 包含脊椎动物和无脊椎物种注释的综合基因组数据库
- **更新频率：** 定期发布（带版本号）；约每 3 个月发布新版本
- **访问方式：** FTP 下载, REST API
- **网站：** https://www.ensembl.org/
- **注意事项：**
  - 支持脊椎动物和无脊椎动物基因组
  - 可指定版本号确保可复现性
  - 常用物种提供快捷名称（'human', 'mouse'）

#### UCSC 基因组浏览器
- **使用模块：** gget blat
- **描述：** 集成 BLAT 比对工具的基因组浏览器数据库
- **更新频率：** 随新组装版本定期更新
- **访问方式：** Web 服务 API
- **网站：** https://genome.ucsc.edu/
- **注意事项：**
  - 提供多种基因组组装版本（hg38, mm39 等）
  - BLAT 针对脊椎动物基因组优化

### 蛋白质与结构数据库

#### UniProt
- **使用模块：** gget info, gget seq (氨基酸序列), gget elm
- **描述：** 通用蛋白质资源库，包含全面的蛋白质序列与功能信息
- **更新频率：** 定期发布（Swiss-Prot 每周更新，TrEMBL 每月更新）
- **访问方式：** REST API
- **网站：** https://www.uniprot.org/
- **注意事项：**
  - Swiss-Prot：人工注释并审核
  - TrEMBL：自动注释

#### NCBI（美国国家生物技术信息中心）
- **使用模块：** gget info, gget bgee (用于非 Ensembl 物种)
- **描述：** 具有广泛交叉引用的基因与蛋白质数据库
- **更新频率：** 持续更新
- **访问方式：** E-utilities API
- **网站：** https://www.ncbi.nlm.nih.gov/
- **包含数据库：** Gene, Protein, RefSeq

#### RCSB PDB（蛋白质数据库）
- **使用模块：** gget pdb
- **描述：** 蛋白质与核酸三维结构数据存储库
- **更新频率：** 每周更新
- **访问方式：** REST API
- **网站：** https://www.rcsb.org/
- **注意事项：**
  - 实验确定的结构（X射线、核磁共振、冷冻电镜）
  - 包含实验与出版物元数据

#### ELM（真核生物线性模体）
- **使用模块：** gget elm
- **描述：** 真核蛋白质功能位点数据库
- **更新频率：** 周期性更新
- **访问方式：** 本地下载（通过 gget setup elm）
- **网站：** http://elm.eu.org/
- **注意事项：**
  - 首次使用前需本地下载
  - 包含已验证的模体与模式

### 序列相似性数据库

#### BLAST 数据库（NCBI）
- **使用模块：** gget blast
- **描述：** 预格式化 BLAST 搜索数据库
- **更新频率：** 定期更新
- **访问方式：** NCBI BLAST API
- **包含数据库：**
  - **核苷酸：** nt（全部 GenBank）、refseq_rna、pdbnt
  - **蛋白质：** nr（非冗余）、swissprot、pdbaa、refseq_protein
- **注意事项：**
  - nt 和 nr 数据库规模庞大
  - 考虑使用专用数据库以加速聚焦搜索

### 表达与相关性数据库

#### ARCHS4
- **使用模块：** gget archs4
- **描述：** 海量公开 RNA-seq 数据挖掘库
- **更新频率：** 随新增样本周期性更新
- **访问方式：** HTTP API
- **网站：** https://maayanlab.cloud/archs4/
- **数据类型：**
  - 人类和小鼠 RNA-seq 数据
  - 相关性矩阵
  - 组织表达图谱
- **引用文献：** Lachmann et al., Nature Communications, 2018

#### CZ CELLxGENE Discover
- **使用模块：** gget cellxgene
- **描述：** 整合多研究的单细胞 RNA-seq 数据
- **更新频率：** 持续新增数据集
- **访问方式：** Census API（通过 cellxgene-census 包）
- **网站：** https://cellxgene.cziscience.com/
- **数据类型：**
  - 单细胞 RNA-seq 计数矩阵
  - 细胞类型注释
  - 组织与疾病元数据
- **注意事项：**
  - 需执行 gget setup cellxgene
  - 基因符号区分大小写
  - 可能不支持最新 Python 版本

#### Bgee
- **使用模块：** gget bgee
- **描述：** 基因表达与直系同源关系数据库
- **更新频率：** 定期发布
- **访问方式：** REST API
- **网站：** https://www.bgee.org/
- **数据类型：**
  - 跨组织与发育阶段的基因表达
  - 跨物种直系同源关系
- **引用文献：** Bastian et al., 2021

### 功能与通路数据库

#### Enrichr / modEnrichr
- **使用模块：** gget enrichr
- **描述：** 基因集富集分析网络服务
- **更新频率：** 底层数据库定期更新
- **访问方式：** REST API
- **网站：** https://maayanlab.cloud/Enrichr/
- **包含数据库：**
  - KEGG 通路
  - 基因本体（GO）
  - 转录因子靶标（ChEA）
  - 疾病关联（GWAS Catalog）
  - 细胞类型标记（PanglaoDB）
- **注意事项：**
  - 支持多模式生物
  - 可提供背景基因列表进行定制富集分析

### 疾病与药物数据库

#### Open Targets
- **使用模块：** gget opentargets
- **描述：** 疾病-靶标关联综合平台
- **更新频率：** 季度性定期发布
- **访问方式：** GraphQL API
- **网站：** https://www.opentargets.org/
- **数据类型：**
  - 疾病关联
  - 药物信息与临床试验
  - 靶标可成药性
  - 药物基因组学
  - 基因表达
  - DepMap 基因-疾病效应
  - 蛋白质相互作用

#### cBioPortal
- **使用模块：** gget cbio
- **描述：** 癌症基因组学数据门户
- **更新频率：** 持续新增研究数据
- **访问方式：** Web API, 可下载数据集
- **网站：** https://www.cbioportal.org/
- **数据类型：**
  - 突变、拷贝数变异、结构变异
  - 基因表达
  - 临床数据
- **注意事项：**
  - 数据集庞大，建议缓存
  - 支持多种癌症类型与研究

#### COSMIC（癌症体细胞突变目录）
- **使用模块：** gget cosmic
- **描述：** 综合性癌症突变数据库
- **更新频率：** 定期发布
- **访问方式：** 下载（商业用途需账户许可）
- **网站：** https://cancer.sanger.ac.uk/cosmic
- **数据类型：**
  - 癌症体细胞突变
  - 基因普查
  - 细胞系数据
  - 耐药突变
- **重要提示：**
  - 学术用途免费
  - 商业用途需支付许可费
  - 需 COSMIC 账户凭证
  - 查询前必须下载数据库

### AI 与预测服务

#### AlphaFold2（DeepMind）
- **使用模块：** gget alphafold
- **描述：** 蛋白质结构预测深度学习模型
- **模型版本：** 本地执行的简化版本
- **访问方式：** 本地计算（需通过 gget setup 下载模型）
- **网站：** https://alphafold.ebi.ac.uk/
- **注意事项：**
  - 需下载约 4GB 模型参数
  - 需安装 OpenMM
  - 计算密集
  - 存在 Python 版本特定要求

#### OpenAI API
- **使用模块：** gget gpt
- **描述：** 大型语言模型 API
- **更新频率：** 周期性发布新模型
- **访问方式：** REST API（需 API 密钥）
- **网站：** https://openai.com/
- **注意事项：**
  - 默认模型：gpt-3.5-turbo
  - 免费额度限账户创建后 3 个月
  - 设置账单限额控制成本

## 数据一致性与可复现性

### 版本控制
为确保分析可复现：

1. **指定数据库版本/发布号：**
   ```python
   # 使用特定 Ensembl 版本
   gget.ref("homo_sapiens", release=110)

   # 使用特定 Census 版本
   gget.cellxgene(gene=["PAX7"], census_version="2023-07-25")
   ```

2. **记录 gget 版本：**
   ```python
   import gget
   print(gget.__version__)
   ```

3. **保存原始数据：**
   ```python
   # 始终保存结果以确保可复现性
   results = gget.search(["ACE2"], species="homo_sapiens")
   results.to_csv("search_results_2025-01-15.csv", index=False)
   ```

### 处理数据库更新

1. **定期更新 gget：**
   - 每两周更新 gget 以匹配数据库结构变更
   - 检查发布说明了解重大变更

2. **错误处理：**
   - 数据库结构变更可能导致临时故障
   - 查看 GitHub 问题：https://github.com/pachterlab/gget/issues
   - 若出现错误请更新 gget

3. **API 速率限制：**
   - 大规模查询时实施延迟
   - 尽可能使用本地数据库（DIAMOND, COSMIC）
   - 缓存结果避免重复查询

## 数据库特定最佳实践

### Ensembl
- 使用快捷物种名（'human', 'mouse'）简化操作
- 指定版本号确保可复现性
- 通过 `gget ref --list_species` 查看可用物种

### UniProt
- UniProt ID 比基因名更稳定
- Swiss-Prot 注释经人工审核更可靠
- 仅在必要时使用 gget info 的 PDB 标志（增加运行时间）

### BLAST/BLAT
- 从默认参数开始，逐步优化
- 使用专用数据库（swissprot, refseq_protein）进行聚焦搜索
- 根据查询长度调整 E 值阈值

### 表达数据库
- CELLxGENE 中基因符号区分大小写
- ARCHS4 相关性数据基于共表达模式
- 解释结果时考虑组织特异性

### 癌症数据库
- cBioPortal：对重复分析进行本地缓存
- COSMIC：按需下载数据库子集
- 商业用途遵守许可协议

## 引用说明

使用 gget 时请同时引用 gget 出版物和底层数据库：

**gget：**
Luebbert, L. & Pachter, L. (2023). Efficient querying of genomic reference databases with gget. Bioinformatics. https://doi.org/10.1093/bioinformatics/btac836

**数据库特定引用：** 查看 references/ 目录或数据库官网获取引用信息。
