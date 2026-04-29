# 糖数据库与资源参考指南

## 核心数据库

### GlyTouCan
- **网址**: https://glytoucan.org/
- **内容**: 糖链结构的唯一标识符（GTC ID）
- **用途**: 跨数据库标准化糖链识别
- **格式**: GlycoCT, WURCS, IUPAC

```python
import requests

def lookup_glytoucan(glytoucan_id: str) -> dict:
    """从GlyTouCan获取糖链详细信息"""
    url = f"https://api.glytoucan.org/glycan/{glytoucan_id}"
    response = requests.get(url, headers={"Accept": "application/json"})
    return response.json() if response.ok else {}
```

### GlyConnect
- **网址**: https://glyconnect.expasy.org/
- **内容**: 包含位点特异性糖谱的蛋白质糖基化数据库
- **整合**: 关联UniProt蛋白质与实验验证的糖基化数据
- **用途**: 查询目标蛋白的已知糖基化位点

```python
import requests

def get_glycoprotein_info(uniprot_id: str) -> dict:
    """从GlyConnect获取蛋白质糖基化数据"""
    base_url = "https://glyconnect.expasy.org/api"
    response = requests.get(f"{base_url}/proteins/uniprot/{uniprot_id}")
    return response.json() if response.ok else {}

def get_glycan_compositions(glyconnect_protein_id: int) -> list:
    """获取GlyConnect蛋白质条目的所有糖链组成"""
    base_url = "https://glyconnect.expasy.org/api"
    response = requests.get(f"{base_url}/compositions/protein/{glyconnect_protein_id}")
    return response.json().get("data", []) if response.ok else []
```

### UniCarbKB
- **网址**: https://unicarbkb.org/
- **内容**: 含生物学背景的精选糖链结构
- **特性**: 组织/细胞类型特异性糖链数据，质谱数据

### KEGG Glycan
- **网址**: https://www.genome.jp/kegg/glycan/
- **内容**: KEGG格式糖链结构及生物合成通路
- **整合**: 链接至糖链生物合成的KEGG通路图谱

### CAZy（碳水化合物活性酶数据库）
- **网址**: http://www.cazy.org/
- **内容**: 构建、分解和修饰糖链的酶类
- **用途**: 识别糖工程应用相关酶

## 预测服务器

### NetNGlyc 1.0
- **网址**: https://services.healthtech.dtu.dk/services/NetNGlyc-1.0/
- **方法**: N-糖基化位点预测神经网络
- **输入**: 蛋白质FASTA序列
- **输出**: 天冬酰胺位点概率评分（阈值约0.5）

### NetOGlyc 4.0
- **网址**: https://services.healthtech.dtu.dk/services/NetOGlyc-4.0/
- **方法**: O-GalNAc糖基化预测神经网络
- **输入**: 蛋白质FASTA序列
- **输出**: 丝氨酸/苏氨酸位点概率（阈值约0.5）

### GlycoMine（机器学习）
- N/O/C型糖基化机器学习预测器
- 支持多种糖链类型：N-GlcNAc, O-GalNAc, O-GlcNAc, O-Man, O-Fuc, O-Glc, C-Man

### SymLink（糖基化位点及序列基序预测器）
- 物种特异性的N-糖基化预测
- 比简单序列基序扫描更精准

## 质谱糖蛋白质组学工具

### Byonic（Protein Metrics）
- 基于MS2谱图的从头糖肽鉴定
- 综合糖链数据库
- 位点特异性糖型分配

### Mascot Glycan Analysis
- 糖链特异性搜索参数
- 常用于自下而上糖蛋白质组学

### GlycoWorkbench
- **网址**: https://www.eurocarbdb.org/project/glycoworkbench
- 糖链结构绘制与质量计算
- 糖链碎片离子标注MS/MS谱图

### Skyline
- 糖肽靶向定量分析
- 集成糖链数据库

## 糖链命名体系

### Oxford命名法（适用于N-糖链）
用文本字符串编码复杂N-糖链：
```
G0F   = 核心岩藻糖基化，双天线，无半乳糖
G1F   = 核心岩藻糖基化，一个半乳糖
G2F   = 核心岩藻糖基化，两个半乳糖
G2FS1 = 核心岩藻糖基化，两个半乳糖，一个唾液酸
G2FS2 = 核心岩藻糖基化，两个半乳糖，两个唾液酸
M5    = 高甘露糖5型（Man5GlcNAc2）
M9    = 高甘露糖9型（Man9GlcNAc2）
```

### 糖链符号命名法（SNFG）
出版物标准彩色符号：
- 蓝色圆形 = 葡萄糖
- 绿色圆形 = 甘露糖
- 黄色圆形 = 半乳糖
- 蓝色方形 = N-乙酰葡糖胺
- 黄色方形 = N-乙酰半乳糖胺
- 紫色菱形 = N-乙酰神经氨酸（唾液酸）
- 红色三角形 = 岩藻糖

## 治疗性糖蛋白及关键糖基化位点

| 治疗性蛋白 | 靶点 | 关键糖基化位点 | 功能 |
|-------------|--------|------------------|---------|
| IgG1抗体 | 多种 | N297（Fc区） | ADCC/CDC效应功能 |
| 促红细胞生成素 | EPOR | N24, N38, N83, O-糖链 | 药代动力学 |
| 依那西普 | TNF | N420（IgG1 Fc区） | 半衰期 |
| tPA（阿替普酶） | 纤维蛋白 | N117, N184, N448 | 纤维蛋白结合 |
| 凝血因子VIII | VWF | 25个N-糖基化位点 | 清除率 |

## 批量分析示例

```python
from glycoengineering_tools import find_n_glycosylation_sequons, predict_o_glycosylation_hotspots
import pandas as pd

def analyze_glycosylation_landscape(sequences_dict: dict) -> pd.DataFrame:
    """
    多蛋白糖基化批量分析
    
    参数:
        sequences_dict: {蛋白名称: 序列}
    
    返回:
        含各蛋白糖基化概览的DataFrame
    """
    results = []
    for name, seq in sequences_dict.items():
        n_sites = find_n_glycosylation_sequons(seq)
        o_sites = predict_o_glycosylation_hotspots(seq)

        results.append({
            'protein': name,
            'length': len(seq),
            'n_glycosites': len(n_sites),
            'o_glyco_hotspots': len(o_sites),
            'n_glyco_density': len(n_sites) / len(seq) * 100,
            'n_glyco_positions': [s['position'] for s in n_sites]
        })

    return pd.DataFrame(results)
```
