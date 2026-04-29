# 共识峰集：基因组宇宙构建

## 概述

Geniml 提供构建基因组"宇宙"的工具——即从 BED 文件集合中创建标准化的共识峰参考集。这些宇宙代表多个分析数据集显示显著覆盖重叠的基因组区域，可作为标记化和分析的参考词汇表。

## 适用场景

在以下情况使用共识峰创建：
- 从多个实验构建参考峰集
- 为 Region2Vec 或 BEDspace 标记化创建宇宙文件
- 跨数据集标准化基因组区域
- 定义具有统计显著性的感兴趣区域

## 工作流程

### 步骤 1：合并 BED 文件

将所有 BED 文件合并为单一文件：

```bash
cat /path/to/bed/files/*.bed > combined_files.bed
```

### 步骤 2：生成覆盖轨迹

使用 uniwig 创建带平滑窗口的 bigWig 覆盖轨迹：

```bash
uniwig -m 25 combined_files.bed chrom.sizes coverage/
```

**参数说明：**
- `-m 25`：平滑窗口大小（染色质可及性分析通常为 25bp）
- `chrom.sizes`：参考基因组的染色体尺寸文件
- `coverage/`：bigWig 文件输出目录

平滑窗口有助于降低噪声并创建更稳健的峰边界。

### 步骤 3：构建宇宙

使用以下四种方法之一构建共识峰：

## 宇宙构建方法

### 1. 覆盖截断法 (CC)

使用固定覆盖阈值的最简方法：

```bash
geniml universe build cc \
  --coverage-folder coverage/ \
  --output-file universe_cc.bed \
  --cutoff 5 \
  --merge 100 \
  --filter-size 50
```

**参数说明：**
- `--cutoff`：覆盖阈值（1=全集；文件数=交集）
- `--merge`：相邻峰合并距离（bp）
- `--filter-size`：最小峰尺寸过滤（bp）

**适用场景：** 简单阈值选择即可满足需求时

### 2. 柔性覆盖截断法 (CCF)

为边界和区域核心创建似然截断的置信区间：

```bash
geniml universe build ccf \
  --coverage-folder coverage/ \
  --output-file universe_ccf.bed \
  --cutoff 5 \
  --confidence 0.95 \
  --merge 100 \
  --filter-size 50
```

**新增参数：**
- `--confidence`：柔性边界的置信水平（0-1）

**适用场景：** 需要捕获峰边界不确定性时

### 3. 最大似然法 (ML)

构建考虑区域起止位置的概率模型：

```bash
geniml universe build ml \
  --coverage-folder coverage/ \
  --output-file universe_ml.bed \
  --merge 100 \
  --filter-size 50 \
  --model-type gaussian
```

**参数说明：**
- `--model-type`：似然估计分布类型（高斯分布/泊松分布）

**适用场景：** 需要统计建模峰位置时

### 4. 隐马尔可夫模型法 (HMM)

将基因组区域建模为隐藏状态，覆盖度作为观测值：

```bash
geniml universe build hmm \
  --coverage-folder coverage/ \
  --output-file universe_hmm.bed \
  --states 3 \
  --merge 100 \
  --filter-size 50
```

**参数说明：**
- `--states`：HMM 隐藏状态数量（通常 2-5）

**适用场景：** 需要捕获复杂基因组状态模式时

## Python API

```python
from geniml.universe import build_universe

# 使用覆盖截断法构建
universe = build_universe(
    coverage_folder='coverage/',
    method='cc',
    cutoff=5,
    merge_distance=100,
    min_size=50,
    output_file='universe.bed'
)
```

## 方法对比

| 方法   | 复杂度 | 灵活性 | 计算成本 | 最佳适用场景         |
|--------|--------|--------|----------|----------------------|
| CC     | 低     | 低     | 低       | 快速参考集构建       |
| CCF    | 中     | 中     | 中       | 边界不确定性分析     |
| ML     | 高     | 高     | 高       | 统计严谨性要求高     |
| HMM    | 高     | 高     | 极高     | 复杂模式建模         |

## 最佳实践

### 方法选择指南

1. **从 CC 开始**：快速可解释，适合初步探索
2. **使用 CCF**：当峰边界存在噪声或不确定性时
3. **应用 ML**：需要发表级统计分析时
4. **部署 HMM**：建模复杂染色质状态时

### 参数选择建议

**覆盖截断值：**
- `cutoff = 1`：全集（最宽松）
- `cutoff = n_files`：交集（最严格）
- `cutoff = 0.5 * n_files`：中等共识（典型选择）

**合并距离：**
- ATAC-seq：100-200bp
- ChIP-seq（窄峰）：50-100bp
- ChIP-seq（宽峰）：500-1000bp

**尺寸过滤：**
- 最小 30bp 避免假阳性
- 多数检测推荐 50-100bp
- 宽组蛋白标记需更大值

### 质量评估

构建后执行质量检查：

```python
from geniml.evaluation import assess_universe

metrics = assess_universe(
    universe_file='universe.bed',
    coverage_folder='coverage/',
    bed_files='bed_files/'
)

print(f"区域数量: {metrics['n_regions']}")
print(f"平均区域大小: {metrics['mean_size']:.1f}bp")
print(f"原始峰覆盖度: {metrics['coverage']:.1%}")
```

**关键指标：**
- **区域数量**：应捕获主要特征且避免过度碎片化
- **尺寸分布**：需符合生物学预期（如 ATAC-seq 约 500bp）
- **输入覆盖度**：原始峰被代表的比例（通常 >80%）

## 输出格式

共识峰以 BED 格式保存，包含三列必需数据：

```
chr1    1000    1500
chr1    2000    2800
chr2    500     1000
```

根据方法不同，可能额外包含置信度分数或状态注释列。

## 典型工作流

### Region2Vec 流程

1. 使用优选方法构建宇宙
2. 将宇宙作为标记化参考
3. 对 BED 文件进行标记化
4. 训练 Region2Vec 模型

### BEDspace 流程

1. 从所有数据集构建宇宙
2. 在预处理步骤使用宇宙
3. 结合元数据训练 BEDspace
4. 跨区域和标签进行查询

### scEmbed 流程

1. 从批量或聚合 scATAC-seq 创建宇宙
2. 用于细胞标记化
3. 训练 scEmbed 模型
4. 生成细胞嵌入

## 故障排除

**区域过少：** 降低截断阈值或减小过滤尺寸  
**区域过多：** 提高截断阈值、增加合并距离或增大过滤尺寸  
**边界噪声大：** 改用 CCF 或 ML 方法替代 CC  
**计算耗时过长：** 先用 CC 方法快速获取结果，再按需使用 ML/HMM 优化
