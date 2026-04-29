---
name: primekg
description: 查询精准医学知识图谱（PrimeKG），获取包括基因、药物、疾病、表型等多尺度生物数据。
license: 未知
metadata:
    skill-author: K-Dense Inc. (PrimeKG原始数据来自哈佛MIMS)
---

# PrimeKG 知识图谱技能

## 概述

PrimeKG 是一个精准医学知识图谱，整合了超过20个主要数据库和高质量科学文献资源。它包含超过10万个节点和400万条边，涵盖29种关系类型，包括药物-靶点、疾病-基因和表型-疾病关联。

**核心功能：**
- 搜索节点（基因、蛋白质、药物、疾病、表型）
- 检索直接邻居节点（关联实体与临床证据）
- 分析局部疾病背景（相关基因、药物、表型）
- 识别药物-疾病路径（潜在药物重定位机会）

**数据访问：** 通过 `query_primekg.py` 进行程序化访问。数据存储于 `C:\Users\eamon\Documents\Data\PrimeKG\kg.csv`。

## 何时使用此技能

该技能适用于以下场景：

- **知识驱动的药物发现：** 识别疾病的靶点和作用机制
- **药物重定位：** 寻找可能具有新适应症证据的现有药物
- **表型分析：** 理解症状/表型与疾病和基因的关联
- **多尺度生物学研究：** 连接分子靶点（基因）与临床结果（疾病）
- **网络药理学：** 研究药物-靶点相互作用的网络级效应

## 核心工作流程

### 1. 搜索实体

查找基因、药物或疾病的标识符

```python
from scripts.query_primekg import search_nodes

# 搜索阿尔茨海默病节点
results = search_nodes("Alzheimer", node_type="disease")
# 返回：[{"id": "EFO_0000249", "type": "disease", "name": "Alzheimer's disease", ...}]
```

### 2. 获取邻居节点（直接关联）

检索所有连接节点及关系类型

```python
from scripts.query_primekg import get_neighbors

# 获取特定疾病ID的所有邻居节点
neighbors = get_neighbors("EFO_0000249")
# 返回：邻居节点列表，如 {"neighbor_name": "APOE", "relation": "disease_gene", ...}
```

### 3. 分析疾病背景

汇总疾病关联信息的高级函数

```python
from scripts.query_primekg import get_disease_context

# 获取疾病的综合摘要
context = get_disease_context("Alzheimer's disease")
# 访问：context['associated_genes'], context['associated_drugs'], context['phenotypes']
```

## PrimeKG 中的关系类型

图谱包含多种关键关系类型：
- `protein_protein`：蛋白质物理相互作用
- `drug_protein`：药物靶点/机制关联
- `disease_gene`：遗传关联
- `drug_disease`：适应症与禁忌症
- `disease_phenotype`：临床体征与症状
- `gwas`：全基因组关联研究证据

## 最佳实践

1. **使用精确ID：** 调用 `get_neighbors` 时确保使用 `search_nodes` 获取的正确ID
2. **先看背景：** 深入分析特定基因或药物前，优先使用 `get_disease_context` 获取全局概览
3. **过滤关系：** 在 `get_neighbors` 中使用 `relation_type` 筛选特定证据（如仅限 `drug_protein`）
4. **多尺度整合：** 结合 `OpenTargets` 获取深度遗传证据，或使用 `Semantic Scholar` 补充最新文献背景

## 资源

### 脚本
- `scripts/query_primekg.py`：知识图谱搜索与查询的核心函数

### 数据路径
- 数据：`/mnt/c/Users/eamon/Documents/Data/PrimeKG/kg.csv`
- 总节点数：约129,000
- 总边数：约4,000,000
- 数据库：基于CSV格式，针对pandas查询优化
