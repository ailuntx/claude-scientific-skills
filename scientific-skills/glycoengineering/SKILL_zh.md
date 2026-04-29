---
name: glycoengineering
description: 分析与设计蛋白质糖基化。扫描序列中的N-糖基化序列子（N-X-S/T），预测O-糖基化热点区域，并访问精选的糖工程工具（NetOGlyc、GlycoShield、GlycoWorkbench）。适用于糖蛋白工程、治疗性抗体优化和疫苗设计。
license: 未知
metadata:
    skill-author: Kuan-lin Huang
---

# 糖工程学

## 概述

糖基化是最常见且最复杂的蛋白质翻译后修饰（PTM），影响超过50%的人类蛋白质。聚糖调控蛋白质折叠、稳定性、免疫识别、受体相互作用以及治疗性蛋白质的药代动力学。糖工程学涉及对糖基化模式进行理性修饰，以提升治疗效力、稳定性或免疫逃逸能力。

**两种主要糖基化类型：**
- **N-糖基化**：附着于天冬酰胺（N）的序列子N-X-[S/T]（X≠脯氨酸）；发生于内质网/高尔基体
- **O-糖基化**：附着于丝氨酸（S）或苏氨酸（T）；无严格保守基序；主要由GalNAc起始

## 适用场景

在以下场景使用本技能：

- **抗体工程**：优化Fc糖基化以增强ADCC、CDC或降低免疫原性
- **治疗性蛋白质设计**：识别影响半衰期、稳定性或免疫原性的糖基化位点
- **疫苗抗原设计**：构建聚糖屏障以聚焦免疫应答于保守表位
- **生物类似药表征**：对比原研药与生物类似药的聚糖模式
- **药物靶点分析**：糖基化是否影响受体与靶点的结合？
- **蛋白质稳定性**：N-聚糖常稳定蛋白质结构；识别可引入稳定突变的位点

## N-糖基化序列子分析

### N-糖基化位点扫描

N-糖基化发生于序列子**N-X-[S/T]**（X≠脯氨酸）。

```python
import re
from typing import List, Tuple

def find_n_glycosylation_sequons(sequence: str) -> List[dict]:
    """
    扫描蛋白质序列中的经典N-连接糖基化序列子。
    基序：N-X-[S/T]，其中X≠脯氨酸。

    参数：
        sequence: 单字母氨基酸序列

    返回：
        包含位置（1起始）、基序及上下文信息的字典列表
    """
    seq = sequence.upper()
    results = []
    i = 0
    while i <= len(seq) - 3:
        triplet = seq[i:i+3]
        if triplet[0] == 'N' and triplet[1] != 'P' and triplet[2] in {'S', 'T'}:
            context = seq[max(0, i-3):i+6]  # ±3残基上下文
            results.append({
                'position': i + 1,   # 1起始
                'motif': triplet,
                'context': context,
                'sequon_type': 'NXS' if triplet[2] == 'S' else 'NXT'
            })
            i += 3
        else:
            i += 1
    return results

def summarize_glycosylation_sites(sequence: str, protein_name: str = "") -> str:
    """生成N-糖基化位点的研究日志摘要"""
    sequons = find_n_glycosylation_sequons(sequence)

    lines = [f"# N-糖基化序列子分析：{protein_name or '蛋白质'}"]
    lines.append(f"序列长度：{len(sequence)}")
    lines.append(f"总N-糖基化序列子数：{len(sequons)}")

    if sequons:
        lines.append(f"\nN-X-S位点数：{sum(1 for s in sequons if s['sequon_type'] == 'NXS')}")
        lines.append(f"N-X-T位点数：{sum(1 for s in sequons if s['sequon_type'] == 'NXT')}")
        lines.append(f"\n位点详情：")
        for s in sequons:
            lines.append(f"  位置 {s['position']}: {s['motif']} (上下文：...{s['context']}...)")
    else:
        lines.append("未检测到经典N-糖基化序列子")

    return "\n".join(lines)

# 示例：IgG1 Fc区
fc_sequence = "APELLGGPSVFLFPPKPKDTLMISRTPEVTCVVVDVSHEDPEVKFNWYVDGVEVHNAKTKPREEQYNSTYRVVSVLTVLHQDWLNGKEYKCKVSNKALPAPIEKTISKAKGQPREPQVYTLPPSREEMTKNQVSLTCLVKGFYPSDIAVEWESNGQPENNYKTTPPVLDSDGSFFLYSKLTVDKSRWQQGNVFSCSVMHEALHNHYTQKSLSLSPGK"
print(summarize_glycosylation_sites(fc_sequence, "IgG1 Fc"))
```

### N-糖基化位点突变

```python
def eliminate_glycosite(sequence: str, position: int, replacement: str = "Q") -> str:
    """
    通过Asn→Gln保守替换消除N-糖基化位点。

    参数：
        sequence: 蛋白质序列
        position: 待突变Asn的1起始位置
        replacement: 替换氨基酸（默认Q=谷氨酰胺；体积相似且不糖基化）

    返回：
        突变后序列
    """
    seq = list(sequence.upper())
    idx = position - 1
    assert seq[idx] == 'N', f"位置 {position} 为 '{seq[idx]}'，非 'N'"
    seq[idx] = replacement.upper()
    return ''.join(seq)

def add_glycosite(sequence: str, position: int, flanking_context: str = "S") -> str:
    """
    通过将残基突变为Asn引入N-糖基化位点，
    并确保X≠脯氨酸且+2位为S/T。

    参数：
        position: 引入Asn的1起始位置
        flanking_context: 位置+2处的'S'或'T'（如需修饰）
    """
    seq = list(sequence.upper())
    idx = position - 1

    # 突变为Asn
    seq[idx] = 'N'

    # 确保X+1≠脯氨酸（必要时突变为Ala）
    if idx + 1 < len(seq) and seq[idx + 1] == 'P':
        seq[idx + 1] = 'A'

    # 确保X+2为S或T
    if idx + 2 < len(seq) and seq[idx + 2] not in ('S', 'T'):
        seq[idx + 2] = flanking_context

    return ''.join(seq)
```

## O-糖基化分析

### 启发式O-糖基化热点预测

```python
def predict_o_glycosylation_hotspots(
    sequence: str,
    window: int = 7,
    min_st_fraction: float = 0.4,
    disallow_proline_next: bool = True
) -> List[dict]:
    """
    基于局部S/T密度的启发式O-糖基化热点评分。
    不可替代NetOGlyc；作为快速基线使用。

    规则：
    - O-GalNAc糖基化聚集于富含Ser/Thr的片段
    - 标记S/T富集窗口中的Ser/Thr残基
    - 避免S/T后紧跟脯氨酸（TP/SP基序抑制GalNAc-T）

    参数：
        window: 局部S/T密度的奇数窗口大小
        min_st_fraction: 标记位点所需窗口内S/T最小占比
    """
    if window % 2 == 0:
        window = 7
    seq = sequence.upper()
    half = window // 2
    candidates = []

    for i, aa in enumerate(seq):
        if aa not in ('S', 'T'):
            continue
        if disallow_proline_next and i + 1 < len(seq) and seq[i+1] == 'P':
            continue

        start = max(0, i - half)
        end = min(len(seq), i + half + 1)
        segment = seq[start:end]
        st_count = sum(1 for c in segment if c in ('S', 'T'))
        frac = st_count / len(segment)

        if frac >= min_st_fraction:
            candidates.append({
                'position': i + 1,
                'residue': aa,
                'st_fraction': round(frac, 3),
                'window': f"{start+1}-{end}",
                'segment': segment
            })

    return candidates
```

## 外部糖工程工具

### 1. NetOGlyc 4.0（O-糖基化预测）

高精度O-GalNAc位点预测网络服务：
- **URL**: https://services.healthtech.dtu.dk/services/NetOGlyc-4.0/
- **输入**：FASTA蛋白质序列
- **输出**：单残基O-糖基化概率评分
- **方法**：基于实验验证O-GalNAc位点训练的神经网络

```python
import requests

def submit_netoglycv4(fasta_sequence: str) -> str:
    """
    向NetOGlyc 4.0网络服务提交序列。
    返回结果获取的任务URL。

    注意：使用DTU Health Tech网络服务，结果需1~5分钟。
    """
    url = "https://services.healthtech.dtu.dk/cgi-bin/webface2.cgi"
    # NetOGlyc提交参数（随服务版本变化）
    # 推荐直接使用网页界面
    print("提交序列至：https://services.healthtech.dtu.dk/services/NetOGlyc-4.0/")
    return url

# 另：NetNGlyc用于N-糖基化预测
# URL: https://services.healthtech.dtu.dk/services/NetNGlyc-1.0/
```

### 2. GlycoShield-MD（聚糖屏障分析）

GlycoShield-MD分析分子动力学模拟中聚糖对蛋白质表面的屏障作用：
- **URL**: https://gitlab.mpcdf.mpg.de/dioscuri-biophysics/glycoshield-md/
- **用途**：在MD轨迹上映射蛋白质表面聚糖屏障
- **输出**：单残基屏障覆盖率、可视化结果

```bash
# 安装
pip install glycoshield

# 基础用法：分析糖蛋白MD轨迹的聚糖屏障
glycoshield \
    --topology glycoprotein.pdb \
    --trajectory glycoprotein.xtc \
    --glycan_resnames BGLCNA FUC \
    --output shielding_analysis/
```

### 3. GlycoWorkbench（聚糖结构绘制/分析）

- **URL**: https://www.eurocarbdb.org/project/glycoworkbench
- **用途**：绘制聚糖结构、计算质量、注释质谱图谱
- **格式**：GlycoCT、IUPAC缩略聚糖命名法

### 4. GlyConnect（聚糖-蛋白质数据库）

- **URL**: https://glyconnect.expasy.org/
- **用途**：查找实验验证的糖蛋白及糖基化位点
- **查询**：按蛋白质（UniProt ID）、聚糖结构或组织检索

```python
import requests

def query_glyconnect(uniprot_id: str) -> dict:
    """通过GlyConnect查询蛋白质糖基化数据"""
    url = f"https://glyconnect.expasy.org/api/proteins/uniprot/{uniprot_id}"
    response = requests.get(url, headers={"Accept": "application/json"})
    if response.status_code == 200:
        return response.json()
    return {}

# 示例：查询EGFR糖基化
egfr_glyco = query_glyconnect("P00533")
```

### 5. UniCarbKB（聚糖结构数据库）

- **URL**: https://unicarbkb.org/
- **用途**：浏览聚糖结构、按质量或组成搜索
- **格式**：GlycoCT或IUPAC命名法

## 关键糖工程策略

### 治疗性抗体应用

| 目标 | 策略 | 说明 |
|------|----------|-------|
| 增强ADCC | Fc区Asn297去岩藻糖基化 | 去岩藻糖化IgG1的FcγRIIIa结合力提升约50倍 |
| 降低免疫原性 | 去除非人源聚糖 | 消除α-Gal、NGNA表位 |
| 延长PK半衰期 | 唾液酸化 | 唾液酸化聚糖延长半衰期 |
| 减轻炎症反应 | 高唾液酸化 | IVIG抗炎机制 |
| 构建聚糖屏障 | 在表面添加N-糖基化位点 | 遮蔽易损表位（疫苗设计） |

### 常用突变

| 突变 | 效应 |
|----------|--------|
| N297A/Q (IgG1) | 消除Fc糖基化（无糖基化） |
| N297D (IgG1) | 消除Fc糖基化 |
| S298A/E333A/K334A | 增强FcγRIIIa结合 |
| F243L (IgG1) | 促进去岩藻糖化 |
| T299A | 消除Fc糖基化 |

## 聚糖命名法

### IUPAC缩略命名法（单糖缩写）

| 符号 | 全称 | 类型 |
|--------|-----------|------|
| Glc | 葡萄糖 | 己糖 |
| GlcNAc | N-乙酰葡糖胺 | 己糖胺 |
| Man | 甘露糖 | 己糖 |
| Gal | 半乳糖 | 己糖 |
| Fuc | 岩藻糖 | 脱氧己糖 |
| Neu5Ac | N-乙酰神经氨酸（唾液酸） | 唾液酸 |
| GalNAc | N-乙酰半乳糖胺 | 己糖胺 |

### 复合N-聚糖结构

```
典型复合双天线N-聚糖：
Neu5Ac-Gal-GlcNAc-Man\
                       Man-GlcNAc-GlcNAc-[Asn]
Neu5Ac-Gal-GlcNAc-Man/
(±核心岩藻糖修饰于最内侧GlcNAc)
```

## 最佳实践

- **优先使用NetNGlyc/NetOGlyc**进行预测，再实验验证
- **质谱验证**：采用糖蛋白质组学（Byonic、Mascot）进行位点特异性聚糖分析
- **考虑位点环境**：非所有预测序列子均被糖基化（可及性、细胞类型、蛋白质构象）
- **抗体领域**：Fc区N297聚糖至关重要——始终优先表征此位点
- **使用GlyConnect**：检查目标蛋白是否存在实验验证的糖基化数据

## 补充资源

- **GlyTouCan**（聚糖结构库）：https://glytoucan.org/
- **GlyConnect**：https://glyconnect.expasy.org/
- **CFG功能糖组学**：http://www.functionalglycomics.org/
- **DTU Health Tech服务器**（NetNGlyc、NetOGlyc）：https://services.healthtech.dtu.dk/
- **GlycoWorkbench**：https://glycoworkbench.software.informer.com/
- **综述**：Apweiler R et al. (1999) Biochim Biophys Acta. PMID: 10564035
- **治疗性糖工程综述**：Jefferis R (2009) Nature Reviews Drug Discovery. PMID: 19448661
