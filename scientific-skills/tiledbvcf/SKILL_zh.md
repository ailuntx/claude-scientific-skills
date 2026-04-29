---
name: tiledbvcf
description: 利用TileDB高效存储和检索基因组变异数据。支持可扩展的VCF/BCF数据导入、增量样本添加、压缩存储、并行查询以及群体基因组学数据导出能力。
license: MIT许可证
metadata:
    skill-author: Jeremy Leipzig
---

# TileDB-VCF

## 概述

TileDB-VCF是一个高性能C++库，提供Python和CLI接口，用于高效存储和检索基因组变异调用数据。基于TileDB的稀疏数组技术构建，支持可扩展的VCF/BCF文件导入、无需昂贵合并操作的增量样本添加，以及本地或云端存储变异数据的高效并行查询。

## 适用场景

本技能适用于以下场景：
- 学习TileDB-VCF概念和工作流程
- 基因组学分析和流程原型设计
- 处理中小规模数据集（<1000个样本）
- 需向现有数据集增量添加新样本
- 需跨多个样本高效查询特定基因组区域
- 处理云端存储的变异数据（S3、Azure、GCS）
- 需导出大型VCF数据集的子集
- 为队列研究构建变异数据库
- 教育项目和方法开发
- 变异数据操作对性能要求严苛的场景

## 快速入门

### 安装

**推荐方法：Conda/Mamba**
```bash
# 若使用M1 Mac请执行以下两行
CONDA_SUBDIR=osx-64
conda config --env --set subdir osx-64

# 创建conda环境
conda create -n tiledb-vcf "python<3.10"
conda activate tiledb-vcf

# Mamba是比conda更快更可靠的替代工具
conda install -c conda-forge mamba

# 安装TileDB-Py和TileDB-VCF，并集成其他实用库
mamba install -y -c conda-forge -c bioconda -c tiledb tiledb-py tiledbvcf-py pandas pyarrow numpy
```

**替代方案：Docker镜像**
```bash
docker pull tiledb/tiledbvcf-py     # Python接口
docker pull tiledb/tiledbvcf-cli    # 命令行接口
```

### 基础示例

**创建并填充数据集：**
```python
import tiledbvcf

# 创建新数据集
ds = tiledbvcf.Dataset(uri="my_dataset", mode="w",
                      cfg=tiledbvcf.ReadConfig(memory_budget=1024))

# 导入VCF文件（必须为单样本且含索引）
# 要求：
# - VCF文件必须为单样本（非多样本）
# - 必须包含索引：.csi（bcftools）或.tbi（tabix）
ds.ingest_samples(["sample1.vcf.gz", "sample2.vcf.gz"])
```

**查询变异数据：**
```python
# 打开现有数据集进行读取
ds = tiledbvcf.Dataset(uri="my_dataset", mode="r")

# 查询特定区域和样本
df = ds.read(
    attrs=["sample_name", "pos_start", "pos_end", "alleles", "fmt_GT"],
    regions=["chr1:1000000-2000000", "chr2:500000-1500000"],
    samples=["sample1", "sample2", "sample3"]
)
print(df.head())
```

**导出为VCF：**
```python
import os

# 导出两个VCF样本
ds.export(
    regions=["chr21:8220186-8405573"],
    samples=["HG00101", "HG00097"],
    output_format="v",
    output_dir=os.path.expanduser("~"),
)
```

## 核心功能

### 1. 数据集创建与导入

创建TileDB-VCF数据集并增量导入多个VCF/BCF文件的变异数据，适用于构建群体基因组学数据库和队列研究。

**要求：**
- **仅支持单样本VCF**：不支持多样本VCF
- **需索引文件**：VCF/BCF文件必须包含索引（.csi或.tbi）

**常用操作：**
- 创建具有优化数组模式的新数据集
- 并行导入单个或多个VCF/BCF文件
- 增量添加新样本而无需重新处理现有数据
- 配置内存使用和压缩设置
- 处理各类VCF格式及INFO/FORMAT字段
- 恢复中断的导入过程
- 导入期间验证数据完整性

### 2. 高效查询与过滤

跨基因组区域、样本和变异属性执行高性能变异数据查询，适用于关联研究、变异发现和群体分析。

**常用操作：**
- 查询特定基因组区域（单个或多个）
- 按样本名称或样本组过滤
- 提取特定变异属性（位置、等位基因、基因型、质量）
- 高效访问INFO和FORMAT字段
- 结合空间和基于属性的过滤
- 流式处理大型查询结果
- 跨样本或区域执行聚合分析

### 3. 数据导出与互操作性

以多种格式导出数据用于下游分析或与其他基因组学工具集成，适用于共享数据集、创建分析子集或对接其他流程。

**常用操作：**
- 导出为标准VCF/BCF格式
- 生成含选定字段的TSV文件
- 创建样本/区域特定子集
- 维护数据溯源和元数据
- 保留所有注释的无损数据导出
- 压缩输出格式
- 大型数据集的流式导出

### 4. 群体基因组学工作流

TileDB-VCF擅长处理需要跨大量样本和基因组区域高效访问变异数据的大规模群体基因组学分析。

**常用工作流：**
- 全基因组关联研究（GWAS）数据准备
- 罕见变异负荷检验
- 群体分层分析
- 跨群体等位基因频率计算
- 大型队列的质量控制
- 变异注释与过滤
- 跨群体比较分析

## 关键概念

### 数组模式与数据模型

**TileDB-VCF数据模型：**
- 变异以稀疏数组形式存储，基因组坐标作为维度
- 样本作为属性存储，支持高效样本特定查询
- INFO和FORMAT字段保留原始数据类型
- 自动压缩和分块实现最优存储

**模式配置示例：**
```python
# 含特定分块范围的自定义模式
config = tiledbvcf.ReadConfig(
    memory_budget=2048,  # 单位MB
    region_partition=(0, 3095677412),  # 全基因组范围
    sample_partition=(0, 10000)  # 支持最多1万个样本
)
```

### 坐标系与区域定义

**关键点：** TileDB-VCF遵循VCF标准使用**1-based基因组坐标**：
- 位置采用1-based计数（首个碱基为位置1）
- 范围两端均包含
- 区域"chr1:1000-2000"包含1000-2000位置（共1001个碱基）

**区域定义格式：**
```python
# 单区域
regions = ["chr1:1000000-2000000"]

# 多区域
regions = ["chr1:1000000-2000000", "chr2:500000-1500000"]

# 整条染色体
regions = ["chr1"]

# BED格式（内部自动转换为1-based）
regions = ["chr1:999999-2000000"]  # 等价于1-based的chr1:1000000-2000000
```

### 内存管理

**性能优化要点：**
1. 根据可用系统内存**设置合理内存预算**
2. 处理超大型结果集时**使用流式查询**
3. **分批处理大型导入任务**避免内存耗尽
4. 重复区域访问时**配置分块缓存**
5. 多文件处理时**采用并行导入**
6. **合并邻近区域**优化区域查询效率

### 云端存储集成

TileDB-VCF无缝支持云端存储：
```python
# S3数据集
ds = tiledbvcf.Dataset(uri="s3://bucket/dataset", mode="r")

# Azure Blob存储
ds = tiledbvcf.Dataset(uri="azure://container/dataset", mode="r")

# Google云存储
ds = tiledbvcf.Dataset(uri="gcs://bucket/dataset", mode="r")
```

## 常见问题

1. **导入期间内存耗尽**：大型VCF文件需设置合理内存预算并分批处理
2. **低效区域查询**：合并邻近区域而非执行多次独立查询
3. **样本名称缺失**：确保VCF头文件中的样本名与查询指定样本匹配
4. **坐标系混淆**：牢记TileDB-VCF采用与VCF标准一致的1-based坐标
5. **大型结果集**：返回数百万变异的查询需使用流式或分页处理
6. **云端权限问题**：确保云存储访问权限配置正确
7. **并发访问冲突**：多写入器操作同一数据集可能导致损坏——使用适当锁机制

## CLI使用指南

TileDB-VCF命令行接口提供以下子命令：

**可用子命令：**
- `create` - 创建空TileDB-VCF数据集
- `store` - 将样本导入TileDB-VCF数据集
- `export` - 从TileDB-VCF数据集导出数据
- `list` - 列出数据集内所有样本名称
- `stat` - 打印数据集高级统计信息
- `utils` - 数据集操作工具集
- `version` - 打印版本信息并退出

```bash
# 创建空数据集
tiledbvcf create --uri my_dataset

# 导入样本（需单样本VCF及索引）
tiledbvcf store --uri my_dataset --samples sample1.vcf.gz,sample2.vcf.gz

# 导出数据
tiledbvcf export --uri my_dataset \
  --regions "chr1:1000000-2000000" \
  --sample-names "sample1,sample2"

# 列出所有样本
tiledbvcf list --uri my_dataset

# 显示数据集统计
tiledbvcf stat --uri my_dataset
```

## 高级功能

### 等位基因频率分析
```python
# 计算等位基因频率
af_df = tiledbvcf.read_allele_frequency(
    uri="my_dataset",
    regions=["chr1:1000000-2000000"],
    samples=["sample1", "sample2", "sample3"]
)
```

### 样本质量控制
```python
# 执行样本QC
qc_results = tiledbvcf.sample_qc(
    uri="my_dataset",
    samples=["sample1", "sample2"]
)
```

### 自定义配置
```python
# 高级配置
config = tiledbvcf.ReadConfig(
    memory_budget=4096,
    tiledb_config={
        "sm.tile_cache_size": "1000000000",
        "vfs.s3.region": "us-east-1"
    }
)
```

## 资源

## 获取帮助

### 开源TileDB-VCF资源

**开源文档：**
- TileDB学院：https://cloud.tiledb.com/academy/
- 群体基因组学指南：https://cloud.tiledb.com/academy/structure/life-sciences/population-genomics/
- TileDB-VCF GitHub：https://github.com/TileDB-Inc/TileDB-VCF

### TileDB-Cloud资源

**大规模/生产级基因组学：**
- TileDB-Cloud平台：https://cloud.tiledb.com
- TileDB学院（完整文档）：https://cloud.tiledb.com/academy/

**入门指引：**
- 免费账户注册：https://cloud.tiledb.com
- 企业需求请联系：sales@tiledb.com

## 扩展至TileDB-Cloud

当基因组学工作负载超出单节点处理能力时，TileDB-Cloud为企业级生产流程提供扩展能力。

**注意**：本节基于现有文档介绍TileDB-Cloud功能。完整API细节及最新功能请查阅官方文档。

### 配置TileDB-Cloud

**1. 创建账户并获取API令牌**
```bash
# 在https://cloud.tiledb.com注册
# 账户设置中生成API令牌
```

**2. 安装TileDB-Cloud Python客户端**
```bash
# 基础安装
pip install tiledb-cloud

# 安装基因组学专用功能
pip install tiledb-cloud[life-sciences]
```

**3. 配置认证**
```bash
# 设置环境变量存储API令牌
export TILEDB_REST_TOKEN="your_api_token"
```

```python
import tiledb.cloud

# 通过TILEDB_REST_TOKEN自动认证
# 代码中无需显式登录
```

### 从开源版迁移至TileDB-Cloud

**大规模数据导入**
```python
# TileDB-Cloud：分布式VCF导入
import tiledb.cloud.vcf

# 使用专用VCF导入模块
# 注：具体API需参考TileDB-Cloud文档
# 此处展示功能结构
tiledb.cloud.vcf.ingestion.ingest_vcf_dataset(
    source="s3://my-bucket/vcf-files/",
    output="tiledb://my-namespace/large-dataset",
    namespace="my-namespace",
    acn="my-s3-credentials",
    ingest_resources={"cpu": "16", "memory": "64Gi"}
)
```

**分布式查询处理**
```python
# TileDB-Cloud：跨分布式存储的VCF查询
import tiledb.cloud.vcf
import tiledbvcf

# 定义数据集URI
dataset_uri = "tiledb://TileDB-Inc/gvcf-1kg-dragen-v376"

# 获取数据集所有样本
ds = tiledbvcf.Dataset(dataset_uri, tiledb_config=cfg)
samples = ds.samples()

# 定义查询属性和范围
attrs = ["sample_name", "fmt_GT", "fmt_AD", "fmt_DP"]
regions = ["chr13:32396898-32397044", "chr13:32398162-32400268"]

# 执行分布式读取
df = tiledb.cloud.vcf.read(
    dataset_uri=dataset_uri,
    regions=regions,
    samples=samples,
    attrs=attrs,
    namespace="my-namespace",  # 指定计费账户
)
df.to_pandas()
```

### 企业级功能

**数据共享与协作**
```python
# TileDB-Cloud通过命名空间权限和群组管理
# 提供企业级数据共享能力

# 通过TileDB-Cloud URI访问共享数据集
dataset_uri = "tiledb://shared-namespace/population-study"

# 通过共享笔记本和计算资源协作
# （具体API需参考TileDB-Cloud文档）
```

**成本优化**
- **无服务器计算**：按实际计算时间计费
- **自动扩缩容**：根据工作负载自动调整规模
- **竞价实例**：批处理作业使用成本优化计算资源
- **数据分层**：自动热/冷存储管理

**安全与合规**
- **端到端加密**：传输和静态数据加密
- **访问控制**：细粒度权限与审计日志
- **HIPAA/SOC2合规**：企业级安全标准
- **VPC支持**：私有云环境部署

### 迁移时机检查表

✅ **符合以下条件时迁移至TileDB-Cloud：**
- [ ] 数据集>1000个样本
- [ ] 需处理>100GB的VCF数据
- [ ] 需要分布式计算
- [ ] 多团队成员需访问权限
- [ ] 需要企业级安全/合规
- [ ] 期望成本优化的无服务器计算
- [ ] 要求24/7生产级运行

### TileDB-Cloud入门指引

1. **免费开始**：TileDB-Cloud提供评估用免费层级
2. **迁移支持**：TileDB团队提供迁移协助
3. **培训资源**：获取基因组学专用教程和示例
4. **专业服务**：定制化部署与优化

**后续步骤：**
- 访问 https://cloud.tiledb.com 创建账户
- 查阅文档 https://cloud.tiledb.com/academy/
- 企业需求联系 sales@tiledb.com
