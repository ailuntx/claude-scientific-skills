---
name: diffdock
description: 基于扩散的分子对接。从PDB/SMILES预测蛋白质-配体结合构象、置信度分数、虚拟筛选，用于基于结构的药物设计。不用于亲和力预测。
license: MIT license
metadata:
    skill-author: K-Dense Inc.
---

# DiffDock：基于扩散模型的分子对接工具

## 概述

DiffDock是一种基于扩散的深度学习分子对接工具，可预测小分子配体与蛋白质靶标的3D结合构象。它代表了计算对接领域的最先进技术，对基于结构的药物发现和化学生物学至关重要。

**核心功能：**
- 利用深度学习高精度预测配体结合构象
- 支持蛋白质结构（PDB文件）或序列（通过ESMFold）
- 处理单一复合物或批量虚拟筛选任务
- 生成置信度分数评估预测可靠性
- 支持多样化配体输入（SMILES、SDF、MOL2）

**关键区别：** DiffDock预测**结合构象**（3D结构）和**置信度**（预测确定性），而非结合亲和力（ΔG, Kd）。进行亲和力评估时需结合评分函数（GNINA, MM/GBSA）。

## 适用场景

本工具适用于以下场景：

- "将此配体对接到蛋白质"或"预测结合构象"
- "运行分子对接"或"执行蛋白质-配体对接"
- "虚拟筛选"或"筛选化合物库"
- "该分子在何处结合？"或"预测结合位点"
- 基于结构的药物设计或先导化合物优化任务
- 涉及PDB文件+SMILES字符串或配体结构的任务
- 多组蛋白质-配体对的批量对接

## 安装与环境配置

### 检查环境状态

执行DiffDock任务前，请验证环境配置：

```bash
# 使用提供的环境检查脚本
python scripts/setup_check.py
```

此脚本将验证Python版本、带CUDA的PyTorch、PyTorch Geometric、RDKit、ESM及其他依赖项。

### 安装选项

**选项1：Conda（推荐）**
```bash
git clone https://github.com/gcorso/DiffDock.git
cd DiffDock
conda env create --file environment.yml
conda activate diffdock
```

**选项2：Docker**
```bash
docker pull rbgcsail/diffdock
docker run -it --gpus all --entrypoint /bin/bash rbgcsail/diffdock
micromamba activate diffdock
```

**重要提示：**
- 强烈推荐使用GPU（比CPU快10-100倍）
- 首次运行需预计算SO(2)/SO(3)查找表（约2-5分钟）
- 模型检查点（约500MB）缺失时将自动下载

## 核心工作流程

### 工作流1：单组蛋白质-配体对接

**场景：** 单配体对单蛋白质靶标对接

**输入要求：**
- 蛋白质：PDB文件或氨基酸序列
- 配体：SMILES字符串或结构文件（SDF/MOL2）

**命令：**
```bash
python -m inference \
  --config default_inference_args.yaml \
  --protein_path protein.pdb \
  --ligand "CC(=O)Oc1ccccc1C(=O)O" \
  --out_dir results/single_docking/
```

**替代方案（蛋白质序列）：**
```bash
python -m inference \
  --config default_inference_args.yaml \
  --protein_sequence "MSKGEELFTGVVPILVELDGDVNGHKF..." \
  --ligand ligand.sdf \
  --out_dir results/sequence_docking/
```

**输出结构：**
```
results/single_docking/
├── rank_1.sdf          # 排名第一的构象
├── rank_2.sdf          # 排名第二的构象
├── ...
├── rank_10.sdf         # 第10个构象（默认采样10次）
└── confidence_scores.txt
```

### 工作流2：批量处理多组复合物

**场景：** 多配体对多蛋白质对接，虚拟筛选任务

**步骤1：准备批量CSV文件**

使用脚本创建或验证批量输入：

```bash
# 创建模板
python scripts/prepare_batch_csv.py --create --output batch_input.csv

# 验证现有CSV
python scripts/prepare_batch_csv.py my_input.csv --validate
```

**CSV格式：**
```csv
complex_name,protein_path,ligand_description,protein_sequence
complex1,protein1.pdb,CC(=O)Oc1ccccc1C(=O)O,
complex2,,COc1ccc(C#N)cc1,MSKGEELFT...
complex3,protein3.pdb,ligand3.sdf,
```

**必填列：**
- `complex_name`：唯一标识符
- `protein_path`：PDB文件路径（使用序列时留空）
- `ligand_description`：SMILES字符串或配体文件路径
- `protein_sequence`：氨基酸序列（使用PDB时留空）

**步骤2：运行批量对接**

```bash
python -m inference \
  --config default_inference_args.yaml \
  --protein_ligand_csv batch_input.csv \
  --out_dir results/batch/ \
  --batch_size 10
```

**大型虚拟筛选（>100化合物）：**

预计算蛋白质嵌入以加速处理：
```bash
# 预计算嵌入
python datasets/esm_embedding_preparation.py \
  --protein_ligand_csv screening_input.csv \
  --out_file protein_embeddings.pt

# 使用预计算嵌入运行
python -m inference \
  --config default_inference_args.yaml \
  --protein_ligand_csv screening_input.csv \
  --esm_embeddings_path protein_embeddings.pt \
  --out_dir results/screening/
```

### 工作流3：结果分析

对接完成后，分析置信度分数并排序预测结果：

```bash
# 分析所有结果
python scripts/analyze_results.py results/batch/

# 显示每组前5名
python scripts/analyze_results.py results/batch/ --top 5

# 按置信度阈值筛选
python scripts/analyze_results.py results/batch/ --threshold 0.0

# 导出至CSV
python scripts/analyze_results.py results/batch/ --export summary.csv

# 显示所有复合物中前20名预测
python scripts/analyze_results.py results/batch/ --best 20
```

分析脚本功能：
- 解析所有预测的置信度分数
- 分类为高（>0）、中（-1.5至0）、低（<-1.5）
- 在复合物内部和跨复合物排序预测
- 生成统计摘要
- 导出CSV供下游分析

## 置信度分数解读

**分数含义：**

| 分数区间 | 置信等级 | 解释 |
|------------|------------------|----------------|
| **> 0** | 高 | 强预测，结果可能准确 |
| **-1.5至0** | 中 | 合理预测，需谨慎验证 |
| **< -1.5** | 低 | 不确定预测，需实验验证 |

**关键说明：**
1. **置信度≠亲和力**：高置信度表示结构确定性，非强结合力
2. **考虑背景因素**：
   - 大分子配体（>500 Da）：预期置信度较低
   - 多蛋白链：可能降低置信度
   - 新型蛋白家族：性能可能受限
3. **多重采样**：检查前3-5名预测，寻找一致性

**详细指南：** 使用Read工具查阅`references/confidence_and_limitations.md`

## 参数自定义

### 使用自定义配置

为特定场景创建配置：

```bash
# 复制模板
cp assets/custom_inference_config.yaml my_config.yaml

# 编辑参数（模板含预设值）
# 使用自定义配置运行
python -m inference \
  --config my_config.yaml \
  --protein_ligand_csv input.csv \
  --out_dir results/
```

### 关键可调参数

**采样密度：**
- `samples_per_complex: 10` → 困难案例增至20-40
- 更多样本=更好覆盖但延长运行时间

**推理步数：**
- `inference_steps: 20` → 更高精度可增至25-30
- 更多步数=潜在质量提升但速度降低

**温度参数（控制多样性）：**
- `temp_sampling_tor: 7.04` → 柔性配体增至8-10
- `temp_sampling_tor: 7.04` → 刚性配体降至5-6
- 更高温度=更多样化构象

**模板预设方案：**
1. 高精度：更多样本+步数，更低温度
2. 快速筛选：较少样本，更快速度
3. 柔性配体：增加扭转温度
4. 刚性配体：降低扭转温度

**完整参数参考：** 使用Read工具查阅`references/parameters_reference.md`

## 高级技巧

### 集成对接（蛋白质柔性）

对已知柔性蛋白进行多构象对接：

```python
# 创建集成CSV
import pandas as pd

conformations = ["conf1.pdb", "conf2.pdb", "conf3.pdb"]
ligand = "CC(=O)Oc1ccccc1C(=O)O"

data = {
    "complex_name": [f"ensemble_{i}" for i in range(len(conformations))],
    "protein_path": conformations,
    "ligand_description": [ligand] * len(conformations),
    "protein_sequence": [""] * len(conformations)
}

pd.DataFrame(data).to_csv("ensemble_input.csv", index=False)
```

增加采样运行对接：
```bash
python -m inference \
  --config default_inference_args.yaml \
  --protein_ligand_csv ensemble_input.csv \
  --samples_per_complex 20 \
  --out_dir results/ensemble/
```

### 与评分函数集成

DiffDock生成构象，需结合其他工具评估亲和力：

**GNINA（快速神经网络评分）：**
```bash
for pose in results/*.sdf; do
    gnina -r protein.pdb -l "$pose" --score_only
done
```

**MM/GBSA（更精确但较慢）：**
能量最小化后使用AmberTools MMPBSA.py或gmx_MMPBSA

**自由能计算（最精确）：**
使用OpenMM + OpenFE或GROMACS进行FEP/TI计算

**推荐流程：**
1. DiffDock → 生成带置信度分数的构象
2. 可视化检查 → 验证结构合理性
3. GNINA或MM/GBSA → 重新评分并按亲和力排序
4. 实验验证 → 生化检测

## 限制与适用范围

**DiffDock适用对象：**
- 小分子配体（通常100-1000 Da）
- 类药有机化合物
- 小肽段（<20残基）
- 单链或多链蛋白质

**DiffDock不适用对象：**
- 大型生物分子（蛋白-蛋白对接）→ 使用DiffDock-PP或AlphaFold-Multimer
- 大肽段（>20残基）→ 使用替代方法
- 共价对接 → 使用专用共价对接工具
- 结合亲和力预测 → 需结合评分函数
- 膜蛋白 → 未专门训练，谨慎使用

**完整限制说明：** 使用Read工具查阅`references/confidence_and_limitations.md`

## 故障排除

### 常见问题

**问题：所有预测置信度均低**
- 原因：大分子/特殊配体、结合位点不明确、蛋白质柔性
- 解决：增加`samples_per_complex`(20-40)，尝试集成对接，验证蛋白结构

**问题：内存溢出错误**
- 原因：GPU内存不足
- 解决：降低`--batch_size 2`或减少单次处理量

**问题：运行缓慢**
- 原因：使用CPU而非GPU
- 解决：通过`python -c "import torch; print(torch.cuda.is_available())"`验证CUDA，启用GPU

**问题：不合理的结合构象**
- 原因：蛋白准备不足、配体过大、错误结合位点
- 解决：检查蛋白缺失残基，移除远端水分子，考虑指定结合位点

**问题："Module not found"错误**
- 原因：依赖缺失或环境错误
- 解决：运行`python scripts/setup_check.py`诊断

### 性能优化

**最佳实践：**
1. 使用GPU（关键）
2. 重复使用的蛋白质预计算ESM嵌入
3. 批量处理复合物
4. 从默认参数开始，按需调整
5. 验证蛋白结构（补全缺失残基）
6. 配体使用规范SMILES格式

## 图形用户界面

启动交互式Web界面：

```bash
python app/main.py
# 访问 http://localhost:7860
```

或免安装使用在线演示：
- https://huggingface.co/spaces/reginabarzilaygroup/DiffDock-Web

## 资源

### 辅助脚本（`scripts/`）

**`prepare_batch_csv.py`**：创建和验证批量输入CSV
- 创建含示例条目的模板
- 验证文件路径和SMILES字符串
- 检查必填列和格式问题

**`analyze_results.py`**：分析置信度分数并排序预测
- 解析单次或批量运行结果
- 生成统计摘要
- 导出CSV供下游分析
- 识别跨复合物的最佳预测

**`setup_check.py`**：验证DiffDock环境配置
- 检查Python版本和依赖项
- 验证PyTorch和CUDA可用性
- 测试RDKit和PyTorch Geometric安装
- 提供缺失组件的安装指导

### 参考文档（`references/`）

**`parameters_reference.md`**：完整参数文档
- 所有命令行选项和配置参数
- 默认值及有效范围
- 控制多样性的温度参数
- 模型检查点位置和版本标识

适用场景：
- 需要详细参数说明
- 特定体系的微调指导
- 替代采样策略

**`confidence_and_limitations.md`**：置信度解读与工具限制
- 置信度分数详细解释
- 何时信任预测结果
- DiffDock的适用范围和限制
- 与其他工具的集成方案
- 预测质量故障排除

适用场景：
- 解读置信度分数
- 了解DiffDock的不适用场景
- 工具组合使用指导
- 验证策略制定

**`workflows_examples.md`**：完整工作流示例
- 详细安装指南
- 全工作流分步示例
- 高级集成模式
- 常见问题排查
- 最佳实践与优化技巧

适用场景：
- 含代码的完整工作流示例
- 与GNINA、OpenMM等工具的集成
- 虚拟筛选工作流
- 集成对接流程

### 资源文件（`assets/`）

**`batch_template.csv`**：批量处理模板
- 预格式化含必填列的CSV
- 展示不同输入类型的示例条目
- 可直接填充实际数据

**`custom_inference_config.yaml`**：配置模板
- 含完整参数的注释版YAML
- 四种常见场景预设配置
- 逐参数详细说明
- 可直接定制使用

## 最佳实践

1. **始终通过`setup_check.py`验证环境**，再启动作业
2. **用`prepare_batch_csv.py`验证批量CSV**，提前拦截错误
3. **从默认参数开始**，再根据体系需求调整
4. **生成多组样本**(10-40)确保预测稳健性
5. **可视化检查**重要构象后再进行下游分析
6. **结合评分函数**评估亲和力
7. **置信度分数仅用于初筛**，非最终决策
8. **虚拟筛选中预计算嵌入**提升效率
9. **记录所用参数**确保可复现性
10. **尽可能实验验证**结果

## 引用文献

使用DiffDock时请引用相关论文：

**DiffDock-L（当前默认模型）：**
```
Stärk et al. (2024) "DiffDock-L: Improving Molecular Docking with Diffusion Models"
arXiv:2402.18396
```

**原始DiffDock：**
```
Corso et al. (2023) "DiffDock: Diffusion Steps, Twists, and Turns for Molecular Docking"
ICLR 2023, arXiv:2210.01776
```

## 附加资源

- **GitHub仓库
