---
name: gtars
description: 用于基因组区间分析的高性能Rust工具包，提供Python绑定。适用于计算基因组学和机器学习应用中的基因组区域操作、BED文件处理、覆盖度轨迹分析、重叠检测、ML模型标记化或片段分析。
license: 未知
metadata:
    skill-author: K-Dense Inc.
---

# Gtars：Rust基因组工具与算法

## 概述

Gtars是用于操作、分析和处理基因组区间数据的高性能Rust工具包。它提供专用于重叠检测、覆盖度分析、机器学习标记化及参考序列管理的工具。

在以下场景中使用本工具：
- 基因组区间文件（BED格式）
- 基因组区域间重叠检测
- 覆盖度轨迹生成（WIG, BigWig）
- 基因组机器学习预处理与标记化
- 单细胞基因组学中的片段分析
- 参考序列检索与验证

## 安装

### Python安装

安装gtars Python绑定：

```bash
uv uv pip install gtars
```

### CLI安装

安装命令行工具（需Rust/Cargo环境）：

```bash
# 安装全部功能
cargo install gtars-cli --features "uniwig overlaprs igd bbcache scoring fragsplit"

# 或仅安装特定功能
cargo install gtars-cli --features "uniwig overlaprs"
```

### Rust库

在Rust项目的Cargo.toml中添加：

```toml
[dependencies]
gtars = { version = "0.1", features = ["tokenizers", "overlaprs"] }
```

## 核心功能

Gtars按基因组分析任务划分为专用模块：

### 1. 重叠检测与IGD索引

使用集成基因组数据库（IGD）数据结构高效检测基因组区间重叠。

**适用场景：**
- 查找重叠调控元件
- 变异注释
- 比较ChIP-seq峰
- 识别共享基因组特征

**快速示例：**
```python
import gtars

# 构建IGD索引并查询重叠
igd = gtars.igd.build_index("regions.bed")
overlaps = igd.query("chr1", 1000, 2000)
```

完整重叠检测文档见`references/overlap.md`。

### 2. 覆盖度轨迹生成

通过uniwig模块从测序数据生成覆盖度轨迹。

**适用场景：**
- ATAC-seq可及性图谱
- ChIP-seq覆盖度可视化
- RNA-seq读段覆盖度
- 差异覆盖度分析

**快速示例：**
```bash
# 生成BigWig覆盖度轨迹
gtars uniwig generate --input fragments.bed --output coverage.bw --format bigwig
```

详细覆盖度分析流程见`references/coverage.md`。

### 3. 基因组标记化

将基因组区域转化为离散标记，用于机器学习应用（特别是基因组数据的深度学习模型）。

**适用场景：**
- 基因组ML模型预处理
- 与geniml库集成
- 创建位置编码
- 基因组序列上的Transformer模型训练

**快速示例：**
```python
from gtars.tokenizers import TreeTokenizer

tokenizer = TreeTokenizer.from_bed_file("training_regions.bed")
token = tokenizer.tokenize("chr1", 1000, 2000)
```

标记化文档见`references/tokenizers.md`。

### 4. 参考序列管理

处理参考基因组序列并遵循GA4GH refget协议计算摘要。

**适用场景：**
- 验证参考基因组完整性
- 提取特定基因组序列
- 计算序列摘要
- 交叉参考比对

**快速示例：**
```python
# 加载参考序列并提取片段
store = gtars.RefgetStore.from_fasta("hg38.fa")
sequence = store.get_subsequence("chr1", 1000, 2000)
```

参考序列操作见`references/refget.md`。

### 5. 片段处理

分割与分析片段文件（特别适用于单细胞基因组学数据）。

**适用场景：**
- 处理单细胞ATAC-seq数据
- 按细胞条形码分割片段
- 基于聚类的片段分析
- 片段质量控制

**快速示例：**
```bash
# 按聚类分割片段
gtars fragsplit cluster-split --input fragments.tsv --clusters clusters.txt --output-dir ./by_cluster/
```

片段处理命令见`references/cli.md`。

### 6. 片段评分

针对参考数据集进行片段重叠评分。

**适用场景：**
- 评估片段富集度
- 实验数据与参考数据比对
- 质量指标计算
- 跨样本批量评分

**快速示例：**
```bash
# 对片段进行参考数据集评分
gtars scoring score --fragments fragments.bed --reference reference.bed --output scores.txt
```

## 典型工作流

### 工作流1：峰重叠分析

识别重叠基因组特征：

```python
import gtars

# 加载两组区域集
peaks = gtars.RegionSet.from_bed("chip_peaks.bed")
promoters = gtars.RegionSet.from_bed("promoters.bed")

# 查找重叠区域
overlapping_peaks = peaks.filter_overlapping(promoters)

# 导出结果
overlapping_peaks.to_bed("peaks_in_promoters.bed")
```

### 工作流2：覆盖度轨迹流程

生成可视化覆盖度轨迹：

```bash
# 步骤1：生成覆盖度
gtars uniwig generate --input atac_fragments.bed --output coverage.wig --resolution 10

# 步骤2：转换为基因组浏览器兼容的BigWig格式
gtars uniwig generate --input atac_fragments.bed --output coverage.bw --format bigwig
```

### 工作流3：机器学习预处理

为机器学习准备基因组数据：

```python
from gtars.tokenizers import TreeTokenizer
import gtars

# 步骤1：加载训练区域
regions = gtars.RegionSet.from_bed("training_peaks.bed")

# 步骤2：创建标记器
tokenizer = TreeTokenizer.from_bed_file("training_peaks.bed")

# 步骤3：标记化区域
tokens = [tokenizer.tokenize(r.chromosome, r.start, r.end) for r in regions]

# 步骤4：将标记用于ML流程
# (与geniml或自定义模型集成)
```

## Python与CLI使用场景

**使用Python API的场景：**
- 集成分析流程
- 需要程序化控制
- 结合NumPy/Pandas使用
- 构建定制工作流

**使用CLI的场景：**
- 快速单次分析
- Shell脚本编写
- 批量文件处理
- 工作流原型设计

## 参考文档

完整模块文档：

- **`references/python-api.md`** - 完整Python API参考（含RegionSet操作、NumPy集成及数据导出）
- **`references/overlap.md`** - IGD索引、重叠检测与集合运算
- **`references/coverage.md`** - uniwig覆盖度轨迹生成
- **`references/tokenizers.md`** - 面向ML应用的基因组标记化
- **`references/refget.md`** - 参考序列管理与摘要计算
- **`references/cli.md`** - 命令行接口完整参考

## 与geniml集成

Gtars是geniml Python包的基础设施，为机器学习工作流提供核心基因组区间操作。进行geniml相关任务时，可使用gtars进行数据预处理和标记化。

## 性能特性

- **原生Rust性能**：低内存开销的快速执行
- **并行处理**：支持大数据集的多线程操作
- **内存高效**：流式处理与内存映射文件支持
- **零拷贝操作**：NumPy集成实现最小化数据复制

## 数据格式

Gtars支持标准基因组格式：

- **BED**：基因组区间（3列或扩展格式）
- **WIG/BigWig**：覆盖度轨迹
- **FASTA**：参考序列
- **片段TSV**：含条形码的单细胞片段文件

## 错误处理与调试

启用详细日志进行故障排查：

```python
import gtars

# 启用调试日志
gtars.set_log_level("DEBUG")
```

```bash
# CLI详细模式
gtars --verbose <command>
```
