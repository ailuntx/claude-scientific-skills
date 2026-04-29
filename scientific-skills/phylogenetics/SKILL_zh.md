```markdown
---
name: phylogenetics
description: 使用MAFFT（多序列比对）、IQ-TREE 2（最大似然法）和FastTree（快速NJ/ML）构建并分析系统发育树。通过ETE3或FigTree实现可视化。适用于进化分析、微生物基因组学、病毒系统动力学、蛋白质家族分析和分子钟研究。
license: Unknown
metadata:
    skill-author: Kuan-lin Huang
---

# 系统发育分析

## 概述

系统发育分析通过推断生物序列（基因、蛋白质、基因组）的进化分支模式来重建其进化历史。本技能涵盖标准流程：

1. **MAFFT** — 多序列比对
2. **IQ-TREE 2** — 带模型选择的最大似然树推断
3. **FastTree** — 快速近似最大似然法（适用于大型数据集）
4. **ETE3** — 用于树操作和可视化的Python库

**安装方法：**
```bash
# Conda（推荐用于CLI工具）
conda install -c bioconda mafft iqtree fasttree
pip install ete3
```

## 适用场景

在以下情况使用系统发育分析：

- **进化关系**：哪些生物体/基因与我的序列亲缘关系最近？
- **病毒系统动力学**：追踪疫情传播并估算传播日期
- **蛋白质家族分析**：推断基因家族内的进化关系
- **水平基因转移检测**：识别物种树/基因树不一致的基因
- **祖先序列重建**：推断祖先蛋白质序列
- **分子钟分析**：利用时间采样估算分化时间
- **GWAS辅助**：在进化背景下定位变异（如SARS-CoV-2变体）
- **微生物学**：基于16S rRNA或核心基因组的物种系统发育

## 标准工作流程

### 1. MAFFT多序列比对

```python
import subprocess
import os

def run_mafft(input_fasta: str, output_fasta: str, method: str = "auto",
               n_threads: int = 4) -> str:
    """
    使用MAFFT进行序列比对。

    参数：
        input_fasta: 未比对FASTA文件路径
        output_fasta: 比对结果输出路径
        method: 'auto'（自动选择）、'einsi'（高精度）、'linsi'（高精度但慢速）、
                'fftnsi'（中等）、'fftns'（快速）、'retree2'（极速）
        n_threads: CPU线程数

    返回：
        比对后的FASTA文件路径
    """
    methods = {
        "auto": ["mafft", "--auto"],
        "einsi": ["mafft", "--genafpair", "--maxiterate", "1000"],
        "linsi": ["mafft", "--localpair", "--maxiterate", "1000"],
        "fftnsi": ["mafft", "--fftnsi"],
        "fftns": ["mafft", "--fftns"],
        "retree2": ["mafft", "--retree", "2"],
    }

    cmd = methods.get(method, methods["auto"])
    cmd += ["--thread", str(n_threads), "--inputorder", input_fasta]

    with open(output_fasta, 'w') as out:
        result = subprocess.run(cmd, stdout=out, stderr=subprocess.PIPE, text=True)

    if result.returncode != 0:
        raise RuntimeError(f"MAFFT失败:\n{result.stderr}")

    # 统计比对序列数
    with open(output_fasta) as f:
        n_seqs = sum(1 for line in f if line.startswith('>'))
    print(f"MAFFT: 完成{n_seqs}条序列比对 → {output_fasta}")

    return output_fasta

# MAFFT方法选择指南：
# 少量序列(<200)高精度：linsi或einsi
# 中等规模(<1000)：fftnsi
# 大型数据集(>1000)：fftns或auto
# 超大规模(>10000)：mafft --retree 1
```

### 2. 比对修剪（可选但推荐）

```python
def trim_alignment_trimal(aligned_fasta: str, output_fasta: str,
                            method: str = "automated1") -> str:
    """
    使用TrimAl修剪低质量比对列。

    方法：
    - 'automated1': 自动启发式（推荐）
    - 'gappyout': 移除高间隙列
    - 'strict': 严格间隙阈值
    """
    cmd = ["trimal", f"-{method}", "-in", aligned_fasta, "-out", output_fasta, "-fasta"]
    result = subprocess.run(cmd, capture_output=True, text=True)
    if result.returncode != 0:
        print(f"TrimAl警告: {result.stderr}")
        # 回退到未修剪的比对
        import shutil
        shutil.copy(aligned_fasta, output_fasta)
    return output_fasta
```

### 3. IQ-TREE 2 — 最大似然树

```python
def run_iqtree(aligned_fasta: str, output_prefix: str,
                model: str = "TEST", bootstrap: int = 1000,
                n_threads: int = 4, extra_args: list = None) -> dict:
    """
    使用IQ-TREE 2构建最大似然树。

    参数：
        aligned_fasta: 比对后的FASTA文件
        output_prefix: 输出文件前缀
        model: 'TEST'自动选择模型，或指定模型（如DNA用'GTR+G'，
               蛋白质用'LG+G4'或'JTT+G'）
        bootstrap: 超快自举重复次数（推荐1000）
        n_threads: 线程数（'AUTO'自动检测）
        extra_args: 额外IQ-TREE参数

    返回：
        包含输出文件路径的字典
    """
    cmd = [
        "iqtree2",
        "-s", aligned_fasta,
        "--prefix", output_prefix,
        "-m", model,
        "-B", str(bootstrap),   # 超快自举
        "-T", str(n_threads),
        "--redo"                # 覆盖现有结果
    ]

    if extra_args:
        cmd.extend(extra_args)

    result = subprocess.run(cmd, capture_output=True, text=True)

    if result.returncode != 0:
        raise RuntimeError(f"IQ-TREE失败:\n{result.stderr}")

    # 打印模型选择结果
    log_file = f"{output_prefix}.log"
    if os.path.exists(log_file):
        with open(log_file) as f:
            for line in f:
                if "Best-fit model" in line:
                    print(f"IQ-TREE: {line.strip()}")

    output_files = {
        "tree": f"{output_prefix}.treefile",
        "log": f"{output_prefix}.log",
        "iqtree": f"{output_prefix}.iqtree",  # 完整报告
        "model": f"{output_prefix}.model.gz",
    }

    print(f"IQ-TREE: 树文件保存至 {output_files['tree']}")
    return output_files

# IQ-TREE模型选择指南：
# DNA:     TEST → GTR+G, HKY+G, TrN+G
# 蛋白质: TEST → LG+G4, WAG+G, JTT+G, Q.pfam+G
# 密码子:   TEST → MG+F3X4

# 时间（分子钟）分析需添加：
# extra_args = ["--date", "dates.txt", "--clock-test", "--date-CI", "95"]
```

### 4. FastTree — 快速近似最大似然法

适用于IQ-TREE处理缓慢的大型数据集(>1000序列)：

```python
def run_fasttree(aligned_fasta: str, output_tree: str,
                  sequence_type: str = "nt", model: str = "gtr",
                  n_threads: int = 4) -> str:
    """
    使用FastTree构建快速近似最大似然树。

    参数：
        sequence_type: 'nt'（核苷酸）或'aa'（氨基酸）
        model: nt用'gtr'（推荐）或'jc'；aa用'lg'、'wag'、'jtt'
    """
    if sequence_type == "nt":
        cmd = ["FastTree", "-nt", "-gtr"]
    else:
        cmd = ["FastTree", f"-{model}"]

    cmd += [aligned_fasta]

    with open(output_tree, 'w') as out:
        result = subprocess.run(cmd, stdout=out, stderr=subprocess.PIPE, text=True)

    if result.returncode != 0:
        raise RuntimeError(f"FastTree失败:\n{result.stderr}")

    print(f"FastTree: 树文件保存至 {output_tree}")
    return output_tree
```

### 5. ETE3树分析与可视化

```python
from ete3 import Tree, TreeStyle, NodeStyle, TextFace, PhyloTree
import matplotlib.pyplot as plt

def load_tree(tree_file: str) -> Tree:
    """加载Newick树文件"""
    t = Tree(tree_file)
    print(f"树统计: {len(t)}个叶节点, {len(list(t.traverse()))}个总节点")
    return t

def basic_tree_stats(t: Tree) -> dict:
    """计算基础树统计量"""
    leaves = t.get_leaves()
    distances = [t.get_distance(l1, l2) for l1 in leaves[:min(50, len(leaves))]
                 for l2 in leaves[:min(50, len(leaves))] if l1 != l2]

    stats = {
        "n_leaves": len(leaves),
        "n_internal_nodes": len(t) - len(leaves),
        "total_branch_length": sum(n.dist for n in t.traverse()),
        "max_leaf_distance": max(distances) if distances else 0,
        "mean_leaf_distance": sum(distances)/len(distances) if distances else 0,
    }
    return stats

def find_mrca(t: Tree, leaf_names: list) -> Tree:
    """查找叶节点集合的最近共同祖先"""
    return t.get_common_ancestor(*leaf_names)

def visualize_tree(t: Tree, output_file: str = "tree.png",
                    show_branch_support: bool = True,
                    color_groups: dict = None,
                    width: int = 800) -> None:
    """
    将系统发育树渲染为图像

    参数：
        t: ETE3树对象
        color_groups: 叶节点名称→颜色的映射字典（用于分类着色）
        show_branch_support: 显示自举值
    """
    ts = TreeStyle()
    ts.show_leaf_name = True
    ts.show_branch_support = show_branch_support
    ts.mode = "r"  # 'r'=矩形, 'c'=圆形

    if color_groups:
        for node in t.traverse():
            if node.is_leaf() and node.name in color_groups:
                nstyle = NodeStyle()
                nstyle["fgcolor"] = color_groups[node.name]
                nstyle["size"] = 8
                node.set_style(nstyle)

    t.render(output_file, tree_style=ts, w=width, units="px")
    print(f"树图像保存至: {output_file}")

def midpoint_root(t: Tree) -> Tree:
    """中点定根（当外类群未知时使用）"""
    t.set_outgroup(t.get_midpoint_outgroup())
    return t

def prune_tree(t: Tree, keep_leaves: list) -> Tree:
    """修剪树仅保留指定叶节点"""
    t.prune(keep_leaves, preserve_branch_length=True)
    return t
```

### 6. 完整分析脚本

```python
import subprocess, os
from ete3 import Tree

def full_phylogenetic_analysis(
    input_fasta: str,
    output_dir: str = "phylo_results",
    sequence_type: str = "nt",
    n_threads: int = 4,
    bootstrap: int = 1000,
    use_fasttree: bool = False
) -> dict:
    """
    完整系统发育流程：比对→修剪→建树→可视化

    参数：
        input_fasta: 未比对FASTA文件
        sequence_type: 'nt'（核苷酸）或'aa'（氨基酸/蛋白质）
        use_fasttree: 使用FastTree替代IQ-TREE（大型数据集更快）
    """
    os.makedirs(output_dir, exist_ok=True)
    prefix = os.path.join(output_dir, "phylo")

    print("=" * 50)
    print("步骤1: 多序列比对 (MAFFT)")
    aligned = run_mafft(input_fasta, f"{prefix}_aligned.fasta",
                         method="auto", n_threads=n_threads)

    print("\n步骤2: 树推断")
    if use_fasttree:
        tree_file = run_fasttree(
            aligned, f"{prefix}.tree",
            sequence_type=sequence_type,
            model="gtr" if sequence_type == "nt" else "lg"
        )
    else:
        model = "TEST" if sequence_type == "nt" else "TEST"
        iqtree_files = run_iqtree(
            aligned, prefix,
            model=model,
            bootstrap=bootstrap,
            n_threads=n_threads
        )
        tree_file = iqtree_files["tree"]

    print("\n步骤3: 树分析")
    t = Tree(tree_file)
    t = midpoint_root(t)

    stats = basic_tree_stats(t)
    print(f"树统计量: {stats}")

    print("\n步骤4: 可视化")
    visualize_tree(t, f"{prefix}_tree.png", show_branch_support=True)

    # 保存定根树
    rooted_tree_file = f"{prefix}_rooted.nwk"
    t.write(format=1, outfile=rooted_tree_file)

    results = {
        "aligned_fasta": aligned,
        "tree_file": tree_file,
        "rooted_tree": rooted_tree_file,
        "visualization": f"{prefix}_tree.png",
        "stats": stats
    }

    print("\n" + "=" * 50)
    print("系统发育分析完成!")
    print(f"结果目录: {output_dir}/")
    return results
```

## IQ-TREE模型指南

### DNA模型

| 模型 | 描述 | 适用场景 |
|-------|-------------|---------|
| `GTR+G4` | 通用时间可逆模型 + Gamma分布 | 最灵活的DNA模型 |
| `HKY+G4` | Hasegawa-Kishino-Yano + Gamma | 双速率模型（常用） |
| `TrN+G4` | Tamura-Nei | 不等转换率 |
| `JC` | Jukes-Cantor | 最简模型；所有速率相等 |

### 蛋白质模型

| 模型 | 描述 | 适用场景 |
|-------|-------------|---------|
| `LG+G4` | Le-Gascuel + Gamma | 最佳平均蛋白质模型 |
| `WAG+G4` | Whelan-Goldman | 广泛使用 |
| `JTT+G4` | Jones-Taylor-Thornton | 经典模型 |
| `Q.pfam+G4` | pfam训练模型 | 适用于Pfam类蛋白质家族 |
| `Q.bird+G4` | 鸟类特化模型 | 脊椎动物蛋白质 |

**提示：** 使用 `-m TEST` 让IQ-TREE自动选择最优模型。

## 最佳实践

- **优先保证比对质量**：低质量比对导致不可靠树结构；需手动检查比对
- **小型比对(<200序列)用`linsi`，大型比对用`fftns`或`auto`**
- **模型选择**：除非有特殊原因，始终使用`-m TEST`
- **自举分析**：使用≥1000次超快自举(`-B 1000`)支持分支
- **树定根**：未定根树可能误导；使用外类群或中点定根
- **>5000序列用FastTree**：IQ-TREE速度下降；FastTree快10–100倍
- **修剪长比对**：TrimAl移除不可靠列；提升树准确性
- **病毒/细菌序列建树前检查重组**（使用`RDP4`、`GARD`）

## 附加资源

- **MAFFT**: https://mafft.cbrc.jp/alignment/software/
- **IQ-TREE 2**: http://www.iqtree.org/ | 教程: https://www.iqtree.org/workshop/molevol2022
- **FastTree**: http://www.microbesonline.org/fasttree/
- **ETE3**: http://etetoolkit.org/
- **FigTree** (GUI可视化): https://tree.bio.ed.ac.uk/software/figtree/
- **iTOL** (在线可视化): https://itol.embl.de/
- **MUSCLE** (替代比对工具): https://www.drive5.com/muscle/
- **TrimAl** (比对修剪): https://vicfero.github.io/trimal/
```
