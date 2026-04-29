```markdown
---
name: exploratory-data-analysis
description: 对200多种科学数据格式执行全面的探索性数据分析。该技能适用于分析任何科学数据文件以理解其结构、内容、质量和特征。自动检测文件类型并生成详细的Markdown报告，包含格式特定分析、质量指标和下游分析建议。涵盖化学、生物信息学、显微成像、光谱学、蛋白质组学、代谢组学及通用科学数据格式。
license: MIT license
metadata:
    skill-author: K-Dense Inc.
---

# 探索性数据分析

## 概述

对跨多个领域的科学数据文件执行全面的探索性数据分析（EDA）。该技能提供自动化文件类型检测、格式特定分析、数据质量评估，并生成适合文档记录和下游分析规划的详细Markdown报告。

**核心能力：**
- 自动检测和分析200+种科学文件格式
- 全面的格式特定元数据提取
- 数据质量和完整性评估
- 统计摘要与分布分析
- 可视化建议
- 下游分析推荐
- Markdown报告生成

## 使用场景

在以下情况使用本技能：
- 用户提供科学数据文件路径进行分析
- 用户要求"探索"、"分析"或"总结"数据文件
- 用户需要理解科学数据的结构和内容
- 用户在进行分析前需要完整的数据集报告
- 用户需要评估数据质量或完整性
- 用户询问适合某文件的分析方法

## 支持的文件类别

本技能全面覆盖科学文件格式，分为六大类别：

### 1. 化学与分子格式（60+扩展名）
结构文件、计算化学输出、分子动力学轨迹和化学数据库。

**文件类型包括：** `.pdb`, `.cif`, `.mol`, `.mol2`, `.sdf`, `.xyz`, `.smi`, `.gro`, `.log`, `.fchk`, `.cube`, `.dcd`, `.xtc`, `.trr`, `.prmtop`, `.psf` 等。

**参考文件：** `references/chemistry_molecular_formats.md`

### 2. 生物信息学与基因组学格式（50+扩展名）
序列数据、比对结果、注释信息、变异数据和表达数据。

**文件类型包括：** `.fasta`, `.fastq`, `.sam`, `.bam`, `.vcf`, `.bed`, `.gff`, `.gtf`, `.bigwig`, `.h5ad`, `.loom`, `.counts`, `.mtx` 等。

**参考文件：** `references/bioinformatics_genomics_formats.md`

### 3. 显微成像格式（45+扩展名）
显微图像、医学影像、全玻片成像和电子显微镜数据。

**文件类型包括：** `.tif`, `.nd2`, `.lif`, `.czi`, `.ims`, `.dcm`, `.nii`, `.mrc`, `.dm3`, `.vsi`, `.svs`, `.ome.tiff` 等。

**参考文件：** `references/microscopy_imaging_formats.md`

### 4. 光谱学与分析化学格式（35+扩展名）
核磁共振、质谱、红外/拉曼、紫外-可见光谱、X射线、色谱及其他分析技术。

**文件类型包括：** `.fid`, `.mzML`, `.mzXML`, `.raw`, `.mgf`, `.spc`, `.jdx`, `.xy`, `.cif`（晶体学）, `.wdf` 等。

**参考文件：** `references/spectroscopy_analytical_formats.md`

### 5. 蛋白质组学与代谢组学格式（30+扩展名）
质谱蛋白质组学、代谢组学、脂质组学及多组学数据。

**文件类型包括：** `.mzML`, `.pepXML`, `.protXML`, `.mzid`, `.mzTab`, `.sky`, `.mgf`, `.msp`, `.h5ad` 等。

**参考文件：** `references/proteomics_metabolomics_formats.md`

### 6. 通用科学数据格式（30+扩展名）
数组、表格、层次化数据、压缩归档及常见科学格式。

**文件类型包括：** `.npy`, `.npz`, `.csv`, `.xlsx`, `.json`, `.hdf5`, `.zarr`, `.parquet`, `.mat`, `.fits`, `.nc`, `.xml` 等。

**参考文件：** `references/general_scientific_formats.md`

## 工作流程

### 步骤1：文件类型检测

当用户提供文件路径时：
1. 提取文件扩展名
2. 在对应参考文件中查找扩展名
3. 确定文件类别和格式描述
4. 加载格式特定信息

**示例：**
```
用户："分析 data.fastq"
→ 扩展名：.fastq
→ 类别：bioinformatics_genomics
→ 格式：FASTQ格式（带质量评分的序列数据）
→ 参考文件：references/bioinformatics_genomics_formats.md
```

### 步骤2：加载格式特定信息

根据文件类型读取对应参考文件以了解：
- **典型数据：** 该格式包含的数据类型
- **使用场景：** 常见应用领域
- **Python库：** Python读取方法
- **EDA方法：** 适合该数据类型的分析方法

在参考文件中搜索特定扩展名（例如在`bioinformatics_genomics_formats.md`中搜索"### .fastq"）。

### 步骤3：执行数据分析

使用`scripts/eda_analyzer.py`脚本或实现自定义分析：

**选项A：使用分析脚本**
```python
# 脚本自动执行：
# 1. 检测文件类型
# 2. 加载参考信息
# 3. 执行格式特定分析
# 4. 生成Markdown报告

python scripts/eda_analyzer.py <文件路径> [输出文件名.md]
```

**选项B：在对话中执行自定义分析**
根据参考文件的格式信息执行适当分析：

表格数据（CSV, TSV, Excel）：
- 用pandas加载
- 检查维度和数据类型
- 分析缺失值
- 计算统计摘要
- 识别异常值
- 检查重复项

序列数据（FASTA, FASTQ）：
- 统计序列数量
- 分析长度分布
- 计算GC含量
- 评估质量评分（FASTQ）

图像数据（TIFF, ND2, CZI）：
- 检查维度（X, Y, Z, C, T）
- 分析位深和值域
- 提取元数据（通道、时间戳、空间校准）
- 计算强度统计量

数组数据（NPY, HDF5）：
- 检查形状和维度
- 分析数据类型
- 计算统计摘要
- 检查缺失/无效值

### 步骤4：生成综合报告

创建包含以下部分的Markdown报告：

#### 必需章节：
1. **标题与元数据**
   - 文件名和时间戳
   - 文件大小和位置

2. **基本信息**
   - 文件属性
   - 格式标识

3. **文件类型详情**
   - 参考文件中的格式描述
   - 典型数据内容
   - 常见应用场景
   - Python读取库

4. **数据分析**
   - 结构和维度
   - 统计摘要
   - 质量评估
   - 数据特征

5. **关键发现**
   - 显著模式
   - 潜在问题
   - 质量指标

6. **建议**
   - 预处理步骤
   - 适用分析方法
   - 工具与方法
   - 可视化方案

#### 模板位置
使用`assets/report_template.md`作为报告结构指南。

### 步骤5：保存报告

使用描述性文件名保存Markdown报告：
- 模式：`{原始文件名}_eda_report.md`
- 示例：`experiment_data.fastq` → `experiment_data_eda_report.md`

## 详细格式参考

每个参考文件包含数十种文件类型的全面信息。查找特定格式信息：
1. 通过扩展名确定类别
2. 阅读对应参考文件
3. 搜索匹配扩展名的章节标题（如"### .pdb"）
4. 提取格式信息

### 参考文件结构

每个格式条目包含：
- **描述：** 格式定义
- **典型数据：** 包含内容
- **使用场景：** 常见应用
- **Python库：** 读取方法（含代码示例）
- **EDA方法：** 需执行的特定分析

**示例查询：**
```markdown
### .pdb - 蛋白质数据库
**描述：** 生物大分子3D结构的标准格式
**典型数据：** 原子坐标、残基信息、二级结构
**使用场景：** 蛋白质结构分析、分子可视化、分子对接
**Python库：**
- `Biopython`: `Bio.PDB`
- `MDAnalysis`: `MDAnalysis.Universe('file.pdb')`
**EDA方法：**
- 结构验证（键长、角度）
- B因子分布分析
- 缺失残基检测
- 拉氏图绘制
```

## 最佳实践

### 参考文件使用

参考文件较大（每个10,000+词）。高效使用方法：
1. **按扩展名搜索：** 使用grep查找特定格式
   ```python
   import re
   with open('references/chemistry_molecular_formats.md', 'r') as f:
       content = f.read()
       pattern = r'### \.pdb[^#]*?(?=###|\Z)'
       match = re.search(pattern, content, re.IGNORECASE | re.DOTALL)
   ```

2. **提取相关章节：** 避免不必要地加载整个参考文件

3. **缓存格式信息：** 分析同类型多个文件时复用格式信息

### 数据分析

1. **大文件采样：** 对百万级记录文件分析代表性样本
2. **优雅处理错误：** 科学格式常需特定库，提供明确安装指引
3. **验证元数据：** 交叉检查元数据一致性（如声明维度与实际数据）
4. **考虑数据溯源：** 记录仪器、软件版本和处理步骤

### 报告生成

1. **全面性：** 包含下游分析所需全部相关信息
2. **具体化：** 基于文件类型提供具体建议
3. **可操作性：** 建议明确的后续步骤和工具
4. **包含代码示例：** 展示数据加载和处理方法

## 示例

### 示例1：分析FASTQ文件

```python
# 用户提供："分析 reads.fastq"

# 1. 检测文件类型
扩展名 = '.fastq'
类别 = 'bioinformatics_genomics'

# 2. 读取参考信息
# 在 references/bioinformatics_genomics_formats.md 搜索 "### .fastq"

# 3. 执行分析
from Bio import SeqIO
sequences = list(SeqIO.parse('reads.fastq', 'fastq'))
# 计算：序列数量、长度分布、质量评分、GC含量

# 4. 生成报告
# 包含：格式描述、分析结果、质控建议

# 5. 保存为：reads_eda_report.md
```

### 示例2：分析CSV数据集

```python
# 用户提供："探索 experiment_results.csv"

# 1. 检测：.csv → general_scientific

# 2. 加载CSV格式参考

# 3. 分析
import pandas as pd
df = pd.read_csv('experiment_results.csv')
# 维度、数据类型、缺失值、统计量、相关性

# 4. 生成报告：
# - 数据结构
# - 缺失值模式
# - 统计摘要
# - 相关矩阵
# - 异常值检测结果

# 5. 保存报告
```

### 示例3：分析显微数据

```python
# 用户提供："分析 cells.nd2"

# 1. 检测：.nd2 → microscopy_imaging（尼康格式）

# 2. 读取ND2格式参考
# 了解：多维数据（XYZCT），需nd2reader库

# 3. 分析
from nd2reader import ND2Reader
with ND2Reader('cells.nd2') as images:
    # 提取：维度、通道、时间点、元数据
    # 计算：强度统计量、帧信息

# 4. 生成报告：
# - 图像维度（XY、Z轴层叠、时间、通道）
# - 通道波长
# - 像素尺寸和校准
# - 图像分析建议

# 5. 保存报告
```

## 故障排除

### 缺失库

科学格式常需专用库：

**问题：** 读取文件时出现导入错误

**解决方案：** 提供明确安装指引
```python
try:
    from Bio import SeqIO
except ImportError:
    print("安装Biopython：uv pip install biopython")
```

按类别常见依赖：
- **生物信息学：** `biopython`, `pysam`, `pyBigWig`
- **化学：** `rdkit`, `mdanalysis`, `cclib`
- **显微成像：** `tifffile`, `nd2reader`, `aicsimageio`, `pydicom`
- **光谱学：** `nmrglue`, `pymzml`, `pyteomics`
- **通用：** `pandas`, `numpy`, `h5py`, `scipy`

### 未知文件类型

若扩展名不在参考文件中：
1. 询问用户文件格式
2. 检查是否为厂商特定变体
3. 基于文件结构尝试通用分析（文本/二进制）
4. 提供通用建议

### 大文件处理

超大文件策略：
1. 采样策略（前N条记录）
2. 使用内存映射访问（HDF5, NPY）
3. 分块处理（CSV, FASTQ）
4. 基于样本提供估算值

## 脚本使用

可直接使用`scripts/eda_analyzer.py`：

```bash
# 基础用法
python scripts/eda_analyzer.py data.csv

# 指定输出文件
python scripts/eda_analyzer.py data.csv output_report.md

# 脚本将：
# 1. 自动检测文件类型
# 2. 加载格式参考
# 3. 执行适当分析
# 4. 生成Markdown报告
```

该脚本支持多种常见格式的自动分析，但对话中的自定义分析能提供更灵活和领域特定的洞察。

## 高级用法

### 多文件分析

分析多个关联文件时：
1. 对每个文件执行独立EDA
2. 创建摘要对比报告
3. 识别关联关系和依赖项
4. 建议整合策略

### 质量控制

数据质量评估：
1. 检查格式合规性
2. 验证元数据一致性
3. 评估完整性
4. 识别异常值和异常
5. 与预期范围/分布对比

### 预处理建议

基于数据特征推荐：
1. 标准化策略
2. 缺失值填补
3. 异常值处理
4. 批次校正
5. 格式转换

## 资源

### scripts/
- `eda_analyzer.py`：可直接运行或导入的综合分析脚本

### references/
- `chemistry_molecular_formats.md`：60+化学/分子文件格式
- `bioinformatics_genomics_formats.md`：50+生物信息学格式
- `microscopy_imaging_formats.md`：45+成像格式
- `spectroscopy_analytical_formats.md`：35+光谱学格式
- `proteomics_metabolomics_formats.md`：30+组学格式
- `general_scientific_formats.md`：30+通用格式

### assets/
- `report_template.md`：EDA报告的完整Markdown模板
```
