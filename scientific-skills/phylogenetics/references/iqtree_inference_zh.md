# IQ-TREE 2 系统发育推断参考指南

## 基本命令语法

```bash
iqtree2 -s alignment.fasta --prefix output -m TEST -B 1000 -T AUTO --redo
```

## 关键参数

| 标志 | 描述 | 默认值 |
|------|-------------|---------|
| `-s` | 输入比对文件 | 必需 |
| `--prefix` | 输出文件前缀 | 比对文件名 |
| `-m` | 替换模型（或TEST） | GTR+G |
| `-B` | 超快自举重复次数 | 关闭 |
| `-b` | 标准自举重复次数（较慢） | 关闭 |
| `-T` | 线程数（或AUTO） | 1 |
| `-o` | 外类群名称 | 无（无根树） |
| `--redo` | 覆盖现有结果 | 关闭 |
| `-alrt` | SH-aLRT检验重复次数 | 关闭 |

## 模型选择

```bash
# 完整模型测试（自动选择最优模型）
iqtree2 -s alignment.fasta -m TEST --prefix test_run -B 1000 -T 4

# 显式指定模型
iqtree2 -s alignment.fasta -m GTR+G4 --prefix gtr_run -B 1000

# 蛋白质序列
iqtree2 -s protein.fasta -m TEST --prefix prot_tree -B 1000

# 基于密码子的分析
iqtree2 -s codon.fasta -m GY --prefix codon_tree -B 1000
```

## 自举方法

### 超快自举（UFBoot，推荐）
```bash
iqtree2 -s alignment.fasta -B 1000  # 1000次重复
# ≥95的值表示可靠支持
# 比标准自举快约10倍
```

### 标准自举
```bash
iqtree2 -s alignment.fasta -b 100  # 100次重复（非常慢）
```

### SH-aLRT检验（快速替代方案）
```bash
iqtree2 -s alignment.fasta -alrt 1000 -B 1000  # 同时使用SH-aLRT和UFBoot
# SH-aLRT ≥80 且 UFBoot ≥95 = 强支持分支
```

## 分支支持度解读

| 自举值 | 解读 |
|----------------|----------------|
| ≥ 95 | 强支持 |
| 70–94 | 中等支持 |
| 50–69 | 弱支持 |
| < 50 | 不可靠（无支持） |

## 输出文件

| 文件 | 描述 |
|------|-------------|
| `{prefix}.treefile` | 最大似然树（Newick格式） |
| `{prefix}.iqtree` | 完整分析报告 |
| `{prefix}.log` | 计算日志 |
| `{prefix}.contree` | 自举共识树 |
| `{prefix}.splits.nex` | 网络分割 |
| `{prefix}.bionj` | BioNJ初始树 |
| `{prefix}.model.gz` | 保存的模型参数 |

## 进阶分析

### 分子钟（定年）

```bash
# 含采样时间的时间分析
iqtree2 -s alignment.fasta -m GTR+G \
        --date dates.tsv \           # 制表符分隔：物种名称 YYYY-MM-DD
        --clock-test \               # 检验分子钟进化
        --date-CI 95 \              # 节点定年95%置信区间
        --prefix dated_tree
```

### 一致性因子

```bash
# 基因一致性因子（gCF）- 需要多基因比对
iqtree2 --gcf gene_trees.nwk \
        --tree main_tree.treefile \
        --cf-verbose \
        --prefix cf_analysis
```

### 祖先序列重建

```bash
iqtree2 -s alignment.fasta -m LG+G4 \
        -asr \                      # 边缘祖先状态重建
        --prefix anc_tree
# 输出：{prefix}.state（各节点祖先序列）
```

### 分区模型（多基因）

```bash
# 创建分区文件（partitions.txt）：
# DNA, gene1 = 1-500
# DNA, gene2 = 501-1000

iqtree2 -s concat_alignment.fasta \
        -p partitions.txt \
        -m TEST \
        -B 1000 \
        --prefix partition_tree
```

## IQ-TREE日志解析

```python
def parse_iqtree_log(log_file: str) -> dict:
    """从IQ-TREE日志文件中提取关键结果"""
    results = {}
    with open(log_file) as f:
        for line in f:
            if "Best-fit model" in line:
                results["best_model"] = line.split(":")[1].strip()
            elif "Log-likelihood of the tree:" in line:
                results["log_likelihood"] = float(line.split(":")[1].strip())
            elif "Number of free parameters" in line:
                results["free_params"] = int(line.split(":")[1].strip())
            elif "Akaike information criterion" in line:
                results["AIC"] = float(line.split(":")[1].strip())
            elif "Bayesian information criterion" in line:
                results["BIC"] = float(line.split(":")[1].strip())
            elif "Total CPU time used" in line:
                results["cpu_time"] = line.split(":")[1].strip()
    return results

# 示例：
# results = parse_iqtree_log("output.log")
# print(f"最优模型: {results['best_model']}")
# print(f"对数似然值: {results['log_likelihood']:.2f}")
```

## 常见问题与解决方案

| 问题 | 可能原因 | 解决方案 |
|-------|-------------|---------|
| 所有自举值=0 | 分类单元过少 | 自举需要≥4个分类单元 |
| 分支过长 | 比对伪影 | 重新修剪比对；检查异常序列 |
| 内存错误 | 序列过多 | 使用FastTree；或设置-T 1 |
| 模型拟合差 | 字母表错误 | 检查核酸/蛋白质类型指定 |
| 序列完全相同 | 重复序列 | 比对前移除重复序列 |

## MAFFT比对指南

```bash
# 高精度（<200条序列）
mafft --localpair --maxiterate 1000 input.fasta > aligned.fasta

# 中等规模（200-1000条序列）
mafft --auto input.fasta > aligned.fasta

# 快速模式（>1000条序列）
mafft --fftns input.fasta > aligned.fasta

# 超大规模（>10000条序列）
mafft --retree 1 input.fasta > aligned.fasta

# 使用多线程
mafft --thread 8 --auto input.fasta > aligned.fasta
```
