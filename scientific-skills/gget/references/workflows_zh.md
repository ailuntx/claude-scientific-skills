# gget 工作流示例

扩展的工作流示例，展示如何组合多个 gget 模块完成常见生物信息学任务。

## 目录
1. [完整基因分析流程](#complete-gene-analysis-pipeline)
2. [比较结构生物学](#comparative-structural-biology)
3. [癌症基因组学分析](#cancer-genomics-analysis)
4. [单细胞表达分析](#single-cell-expression-analysis)
5. [构建参考转录组](#building-reference-transcriptomes)
6. [突变影响评估](#mutation-impact-assessment)
7. [药物靶点发现](#drug-target-discovery)

---

## 完整基因分析流程

从基因发现到功能注释的全面分析流程。

```python
import gget
import pandas as pd

# 步骤1：搜索目标基因
print("Step 1: Searching for GABA receptor genes...")
search_results = gget.search(["GABA", "receptor", "alpha"],
                             species="homo_sapiens",
                             andor="and")
print(f"Found {len(search_results)} genes")

# 步骤2：获取详细信息
print("\nStep 2: Getting detailed information...")
gene_ids = search_results["ensembl_id"].tolist()[:5]  # 前5个基因
gene_info = gget.info(gene_ids, pdb=True)
print(gene_info[["ensembl_id", "gene_name", "uniprot_id", "description"]])

# 步骤3：获取序列
print("\nStep 3: Retrieving sequences...")
nucleotide_seqs = gget.seq(gene_ids)
protein_seqs = gget.seq(gene_ids, translate=True)

# 保存序列
with open("gaba_receptors_nt.fasta", "w") as f:
    f.write(nucleotide_seqs)
with open("gaba_receptors_aa.fasta", "w") as f:
    f.write(protein_seqs)

# 步骤4：获取表达数据
print("\nStep 4: Getting tissue expression...")
for gene_id, gene_name in zip(gene_ids, gene_info["gene_name"]):
    expr_data = gget.archs4(gene_name, which="tissue")
    print(f"\n{gene_name} expression:")
    print(expr_data.head())

# 步骤5：寻找相关基因
print("\nStep 5: Finding correlated genes...")
correlated = gget.archs4(gene_info["gene_name"].iloc[0], which="correlation")
correlated_top = correlated.head(20)
print(correlated_top)

# 步骤6：相关基因富集分析
print("\nStep 6: Performing enrichment analysis...")
gene_list = correlated_top["gene_symbol"].tolist()
enrichment = gget.enrichr(gene_list, database="ontology", plot=True)
print(enrichment.head(10))

# 步骤7：获取疾病关联
print("\nStep 7: Getting disease associations...")
for gene_id, gene_name in zip(gene_ids[:3], gene_info["gene_name"][:3]):
    diseases = gget.opentargets(gene_id, resource="diseases", limit=5)
    print(f"\n{gene_name} disease associations:")
    print(diseases)

# 步骤8：检查直系同源物
print("\nStep 8: Finding orthologs...")
orthologs = gget.bgee(gene_ids[0], type="orthologs")
print(orthologs)

print("\nComplete gene analysis pipeline finished!")
```

---

## 比较结构生物学

跨物种比较蛋白质结构并分析功能基序。

```python
import gget

# 定义比较基因
human_gene = "ENSG00000169174"  # PCSK9
mouse_gene = "ENSMUSG00000044254"  # Pcsk9

print("Comparative Structural Biology Workflow")
print("=" * 50)

# 步骤1：获取基因信息
print("\n1. Getting gene information...")
human_info = gget.info([human_gene])
mouse_info = gget.info([mouse_gene])

print(f"Human: {human_info['gene_name'].iloc[0]}")
print(f"Mouse: {mouse_info['gene_name'].iloc[0]}")

# 步骤2：获取蛋白质序列
print("\n2. Retrieving protein sequences...")
human_seq = gget.seq(human_gene, translate=True)
mouse_seq = gget.seq(mouse_gene, translate=True)

# 保存文件用于比对
with open("pcsk9_sequences.fasta", "w") as f:
    f.write(human_seq)
    f.write("\n")
    f.write(mouse_seq)

# 步骤3：序列比对
print("\n3. Aligning sequences...")
alignment = gget.muscle("pcsk9_sequences.fasta")
print("Alignment completed. Visualizing in ClustalW format:")
print(alignment)

# 步骤4：从PDB获取现有结构
print("\n4. Searching PDB for existing structures...")
# 使用BLAST按序列搜索
pdb_results = gget.blast(human_seq, database="pdbaa", limit=5)
print("Top PDB matches:")
print(pdb_results[["Description", "Max Score", "Query Coverage"]])

# 下载最佳结构
if len(pdb_results) > 0:
    # 从描述中提取PDB ID（通常格式："PDB|XXXX|..."）
    pdb_id = pdb_results.iloc[0]["Description"].split("|")[1]
    print(f"\nDownloading PDB structure: {pdb_id}")
    gget.pdb(pdb_id, save=True)

# 步骤5：预测AlphaFold结构
print("\n5. Predicting structures with AlphaFold...")
# 注意：需要配置gget alphafold且计算密集
# 取消注释运行：
# human_structure = gget.alphafold(human_seq, plot=True)
# mouse_structure = gget.alphafold(mouse_seq, plot=True)
print("(AlphaFold prediction skipped - uncomment to run)")

# 步骤6：识别功能基序
print("\n6. Identifying functional motifs with ELM...")
# 注意：需要配置gget elm
# 取消注释运行：
# human_ortholog_df, human_regex_df = gget.elm(human_seq)
# print("Human PCSK9 functional motifs:")
# print(human_regex_df)
print("(ELM analysis skipped - uncomment to run)")

# 步骤7：获取直系同源信息
print("\n7. Getting orthology information from Bgee...")
orthologs = gget.bgee(human_gene, type="orthologs")
print("PCSK9 orthologs:")
print(orthologs)

print("\nComparative structural biology workflow completed!")
```

---

## 癌症基因组学分析

分析癌症相关基因及其突变。

```python
import gget
import matplotlib.pyplot as plt

print("Cancer Genomics Analysis Workflow")
print("=" * 50)

# 步骤1：搜索癌症相关基因
print("\n1. Searching for breast cancer genes...")
genes = gget.search(["breast", "cancer", "BRCA"],
                    species="homo_sapiens",
                    andor="or",
                    limit=20)
print(f"Found {len(genes)} genes")

# 聚焦特定基因
target_genes = ["BRCA1", "BRCA2", "TP53", "PIK3CA", "ESR1"]
print(f"\nAnalyzing: {', '.join(target_genes)}")

# 步骤2：获取基因信息
print("\n2. Getting gene information...")
gene_search = []
for gene in target_genes:
    result = gget.search([gene], species="homo_sapiens", limit=1)
    if len(result) > 0:
        gene_search.append(result.iloc[0])

gene_df = pd.DataFrame(gene_search)
gene_ids = gene_df["ensembl_id"].tolist()

# 步骤3：获取疾病关联
print("\n3. Getting disease associations from OpenTargets...")
for gene_id, gene_name in zip(gene_ids, target_genes):
    print(f"\n{gene_name} disease associations:")
    diseases = gget.opentargets(gene_id, resource="diseases", limit=3)
    print(diseases[["disease_name", "overall_score"]])

# 步骤4：获取药物关联
print("\n4. Getting drug associations...")
for gene_id, gene_name in zip(gene_ids[:3], target_genes[:3]):
    print(f"\n{gene_name} drug associations:")
    drugs = gget.opentargets(gene_id, resource="drugs", limit=3)
    if len(drugs) > 0:
        print(drugs[["drug_name", "drug_type", "max_phase_for_all_diseases"]])

# 步骤5：在cBioPortal中搜索研究
print("\n5. Searching cBioPortal for breast cancer studies...")
studies = gget.cbio_search(["breast", "cancer"])
print(f"Found {len(studies)} studies")
print(studies[:5])

# 步骤6：创建癌症基因组学热图
print("\n6. Creating cancer genomics heatmap...")
if len(studies) > 0:
    # 选择相关研究
    selected_studies = studies[:2]  # 前2项研究

    gget.cbio_plot(
        selected_studies,
        target_genes,
        stratification="cancer_type",
        variation_type="mutation_occurrences",
        show=False
    )
    print("Heatmap saved to ./gget_cbio_figures/")

# 步骤7：查询COSMIC数据库（需配置）
print("\n7. Querying COSMIC database...")
# 注意：需要COSMIC账户和数据库下载
# 取消注释运行：
# for gene in target_genes[:2]:
#     cosmic_results = gget.cosmic(
#         gene,
#         cosmic_tsv_path="cosmic_cancer.tsv",
#         limit=10
#     )
#     print(f"\n{gene} mutations in COSMIC:")
#     print(cosmic_results)
print("(COSMIC query skipped - requires database download)")

# 步骤8：富集分析
print("\n8. Performing pathway enrichment...")
enrichment = gget.enrichr(target_genes, database="pathway", plot=True)
print("\nTop enriched pathways:")
print(enrichment.head(10))

print("\nCancer genomics analysis completed!")
```

---

## 单细胞表达分析

分析特定细胞类型和组织的单细胞RNA-seq数据。

```python
import gget
import scanpy as sc

print("Single-Cell Expression Analysis Workflow")
print("=" * 50)

# 注意：需要配置gget cellxgene

# 步骤1：定义目标基因和细胞类型
genes_of_interest = ["ACE2", "TMPRSS2", "CD4", "CD8A"]
tissue = "lung"
cell_types = ["type ii pneumocyte", "macrophage", "t cell"]

print(f"\nAnalyzing genes: {', '.join(genes_of_interest)}")
print(f"Tissue: {tissue}")
print(f"Cell types: {', '.join(cell_types)}")

# 步骤2：先获取元数据
print("\n1. Retrieving metadata...")
metadata = gget.cellxgene(
    gene=genes_of_interest,
    tissue=tissue,
    species="homo_sapiens",
    meta_only=True
)
print(f"Found {len(metadata)} datasets")
print(metadata.head())

# 步骤3：下载计数矩阵
print("\n2. Downloading single-cell data...")
# 注意：可能下载大量数据
adata = gget.cellxgene(
    gene=genes_of_interest,
    tissue=tissue,
    species="homo_sapiens",
    census_version="stable"
)
print(f"AnnData shape: {adata.shape}")
print(f"Genes: {adata.n_vars}")
print(f"Cells: {adata.n_obs}")

# 步骤4：使用scanpy进行基础QC和过滤
print("\n3. Performing quality control...")
sc.pp.filter_cells(adata, min_genes=200)
sc.pp.filter_genes(adata, min_cells=3)
print(f"After QC - Cells: {adata.n_obs}, Genes: {adata.n_vars}")

# 步骤5：标准化和对数转换
print("\n4. Normalizing data...")
sc.pp.normalize_total(adata, target_sum=1e4)
sc.pp.log1p(adata)

# 步骤6：计算基因表达统计
print("\n5. Calculating expression statistics...")
for gene in genes_of_interest:
    if gene in adata.var_names:
        expr = adata[:, gene].X.toarray().flatten()
        print(f"\n{gene} expression:")
        print(f"  Mean: {expr.mean():.3f}")
        print(f"  Median: {np.median(expr):.3f}")
        print(f"  % expressing: {(expr > 0).sum() / len(expr) * 100:.1f}%")

# 步骤7：从ARCHS4获取组织表达进行对比
print("\n6. Getting bulk tissue expression from ARCHS4...")
for gene in genes_of_interest:
    tissue_expr = gget.archs4(gene, which="tissue")
    lung_expr = tissue_expr[tissue_expr["tissue"] == "lung"]
    if len(lung_expr)

```markdown
genes_of_interest = ["TP53", "BRCA1", "BRCA2", "MYC", "EGFR"]

# 步骤1：获取参考信息
print("\n1. 正在获取参考信息...")
ref_info = gget.ref(species, release=release)

# 保存参考信息
with open("reference_info.json", "w") as f:
    json.dump(ref_info, f, indent=2)
print("参考信息已保存至 reference_info.json")

# 步骤2：下载特定文件
print("\n2. 正在下载参考文件...")
# GTF注释文件
gget.ref(species, which="gtf", release=release, download=True)
# cDNA序列
gget.ref(species, which="cdna", release=release, download=True)

# 步骤3：获取目标基因信息
print(f"\n3. 正在获取{len(genes_of_interest)}个基因的信息...")
gene_data = []
for gene in genes_of_interest:
    result = gget.search([gene], species=species, limit=1)
    if len(result) > 0:
        gene_data.append(result.iloc[0])

# 获取详细信息
if gene_data:
    gene_ids = [g["ensembl_id"] for g in gene_data]
    detailed_info = gget.info(gene_ids)
    detailed_info.to_csv("genes_of_interest_info.csv", index=False)
    print("基因信息已保存至 genes_of_interest_info.csv")

# 步骤4：获取序列
print("\n4. 正在检索序列...")
sequences_nt = gget.seq(gene_ids)
sequences_aa = gget.seq(gene_ids, translate=True)

with open("key_genes_nucleotide.fasta", "w") as f:
    f.write(sequences_nt)
with open("key_genes_protein.fasta", "w") as f:
    f.write(sequences_aa)

print("\n参考转录组构建完成！")
print(f"生成文件：")
print("  - reference_info.json")
print("  - genes_of_interest_info.csv")
print("  - key_genes_nucleotide.fasta")
print("  - key_genes_protein.fasta")
```

---

## 突变影响评估

分析基因突变对蛋白质结构和功能的影响。

```python
import gget
import pandas as pd

print("突变影响评估流程")
print("=" * 50)

# 定义待分析突变
mutations = [
    {"gene": "TP53", "mutation": "c.818G>A", "description": "R273H热点突变"},
    {"gene": "EGFR", "mutation": "c.2573T>G", "description": "L858R激活突变"},
]

# 步骤1：获取基因信息
print("\n1. 正在获取基因信息...")
for mut in mutations:
    results = gget.search([mut["gene"]], species="homo_sapiens", limit=1)
    if len(results) > 0:
        mut["ensembl_id"] = results["ensembl_id"].iloc[0]
        print(f"{mut['gene']}: {mut['ensembl_id']}")

# 步骤2：获取序列
print("\n2. 正在检索野生型序列...")
for mut in mutations:
    # 获取核苷酸序列
    nt_seq = gget.seq(mut["ensembl_id"])
    mut["wt_sequence"] = nt_seq

    # 获取蛋白质序列
    aa_seq = gget.seq(mut["ensembl_id"], translate=True)
    mut["wt_protein"] = aa_seq

# 步骤3：生成突变序列
print("\n3. 正在生成突变序列...")
# 创建gget突变所需数据框
mut_df = pd.DataFrame({
    "seq_ID": [m["gene"] for m in mutations],
    "mutation": [m["mutation"] for m in mutations]
})

# 处理每个突变
for mut in mutations:
    # 从FASTA提取序列
    lines = mut["wt_sequence"].split("\n")
    seq = "".join(lines[1:])

    # 创建单突变数据框
    single_mut = pd.DataFrame({
        "seq_ID": [mut["gene"]],
        "mutation": [mut["mutation"]]
    })

    # 生成突变序列
    mutated = gget.mutate([seq], mutations=single_mut)
    mut["mutated_sequence"] = mutated

print("突变序列已生成")

# 步骤4：获取现有结构信息
print("\n4. 正在获取结构信息...")
for mut in mutations:
    # 获取含PDB ID的信息
    info = gget.info([mut["ensembl_id"]], pdb=True)

    if "pdb_id" in info.columns and pd.notna(info["pdb_id"].iloc[0]):
        pdb_ids = info["pdb_id"].iloc[0].split(";")
        print(f"\n{mut['gene']} PDB结构: {', '.join(pdb_ids[:3])}")

        # 下载首个结构
        if len(pdb_ids) > 0:
            pdb_id = pdb_ids[0].strip()
            mut["pdb_id"] = pdb_id
            gget.pdb(pdb_id, save=True)
    else:
        print(f"\n{mut['gene']}: 无可用PDB结构")
        mut["pdb_id"] = None

# 步骤5：使用AlphaFold预测结构（可选）
print("\n5. 正在通过AlphaFold预测结构...")
# 注意：需配置gget alphafold且计算密集
# 取消注释运行：
# for mut in mutations:
#     print(f"预测{mut['gene']}野生型结构...")
#     wt_structure = gget.alphafold(mut["wt_protein"])
#
#     print(f"预测{mut['gene']}突变型结构...")
#     # 需先翻译突变序列
#     # mutant_structure = gget.alphafold(mutated_protein)
print("(AlphaFold预测已跳过 - 取消注释运行)")

# 步骤6：查找功能基序
print("\n6. 正在识别功能基序...")
# 注意：需配置gget elm
# 取消注释运行：
# for mut in mutations:
#     ortholog_df, regex_df = gget.elm(mut["wt_protein"])
#     print(f"\n{mut['gene']}功能基序：")
#     print(regex_df)
print("(ELM分析已跳过 - 取消注释运行)")

# 步骤7：获取疾病关联
print("\n7. 正在获取疾病关联...")
for mut in mutations:
    diseases = gget.opentargets(
        mut["ensembl_id"],
        resource="diseases",
        limit=5
    )
    print(f"\n{mut['gene']} ({mut['description']}) 疾病关联：")
    print(diseases[["disease_name", "overall_score"]])

# 步骤8：查询COSMIC突变频率
print("\n8. 正在查询COSMIC数据库...")
# 注意：需下载COSMIC数据库
# 取消注释运行：
# for mut in mutations:
#     cosmic_results = gget.cosmic(
#         mut["mutation"],
#         cosmic_tsv_path="cosmic_cancer.tsv",
#         limit=10
#     )
#     print(f"\n{mut['gene']} {mut['mutation']}在COSMIC中：")
#     print(cosmic_results)
print("(COSMIC查询已跳过 - 需下载数据库)")

print("\n突变影响评估完成！")
```

---

## 药物靶点发现

识别并验证特定疾病的潜在药物靶点。

```python
import gget
import pandas as pd

print("药物靶点发现流程")
print("=" * 50)

# 步骤1：搜索疾病相关基因
disease = "alzheimer"
print(f"\n1. 正在搜索{disease}疾病相关基因...")
genes = gget.search([disease], species="homo_sapiens", limit=50)
print(f"发现{len(genes)}个潜在基因")

# 步骤2：获取详细信息
print("\n2. 正在获取详细基因信息...")
gene_ids = genes["ensembl_id"].tolist()[:20]  # 前20个
gene_info = gget.info(gene_ids[:10])  # 限制数量避免超时

# 步骤3：从OpenTargets获取疾病关联
print("\n3. 正在获取疾病关联...")
disease_scores = []
for gene_id, gene_name in zip(gene_info["ensembl_id"], gene_info["gene_name"]):
    diseases = gget.opentargets(gene_id, resource="diseases", limit=10)

    # 筛选阿尔茨海默病
    alzheimer = diseases[diseases["disease_name"].str.contains("Alzheimer", case=False, na=False)]

    if len(alzheimer) > 0:
        disease_scores.append({
            "ensembl_id": gene_id,
            "gene_name": gene_name,
            "disease_score": alzheimer["overall_score"].max()
        })

disease_df = pd.DataFrame(disease_scores).sort_values("disease_score", ascending=False)
print("\n疾病关联性最高的基因：")
print(disease_df.head(10))

# 步骤4：获取靶点成药性信息
print("\n4. 正在评估靶点成药性...")
top_targets = disease_df.head(5)
for _, row in top_targets.iterrows():
    tractability = gget.opentargets(
        row["ensembl_id"],
        resource="tractability"
    )
    print(f"\n{row['gene_name']}成药性：")
    print(tractability)

# 步骤5：获取表达数据
print("\n5. 正在获取组织表达数据...")
for _, row in top_targets.iterrows():
    # 从OpenTargets获取脑部表达
    expression = gget.opentargets(
        row["ensembl_id"],
        resource="expression",
        filter_tissue="brain"
    )
    print(f"\n{row['gene_name']}脑部表达：")
    print(expression)

    # 从ARCHS4获取组织表达
    tissue_expr = gget.archs4(row["gene_name"], which="tissue")
    brain_expr = tissue_expr[tissue_expr["tissue"].str.contains("brain", case=False, na=False)]
    print(f"ARCHS4脑部表达：")
    print(brain_expr)

# 步骤6：检查现有药物
print("\n6. 正在检查现有药物...")
for _, row in top_targets.iterrows():
    drugs = gget.opentargets(row["ensembl_id"], resource="drugs", limit=5)
    print(f"\n{row['gene_name']}药物关联：")
    if len(drugs) > 0:
        print(drugs[["drug_name", "drug_type", "max_phase_for_all_diseases"]])
    else:
        print("未发现相关药物")

# 步骤7：获取蛋白质相互作用
print("\n7. 正在获取蛋白质相互作用...")
for _, row in top_targets.iterrows():
    interactions = gget.opentargets(
        row["ensembl_id"],
        resource="interactions",
        limit=10
    )
    print(f"\n{row['gene_name']}相互作用对象：")
    if len(interactions) > 0:
        print(interactions[["gene_b_symbol", "interaction_score"]])

# 步骤8：富集分析
print("\n8. 正在进行通路富集分析...")
gene_list = top_targets["gene_name"].tolist()
enrichment = gget.enrichr(gene_list, database="pathway", plot=True)
print("\n富集程度最高的通路：")
print(enrichment.head(10))

# 步骤9：获取结构信息
print("\n9. 正在获取结构信息...")
for _, row in top_targets.iterrows():
    info = gget.info([row["ensembl_id"]], pdb=True)

    if "pdb_id" in info.columns and pd.notna(info["pdb_id"].iloc[0]):
        pdb_ids = info["pdb_id"].iloc[0].split(";")
        print(f"\n{row['gene_name']} PDB结构: {', '.join(pdb_ids[:3])}")
    else:
        print(f"\n{row['gene_name']}: 无可用PDB结构")
        # 可使用AlphaFold预测
        print(f"  建议使用AlphaFold预测")

# 步骤10：生成靶点摘要报告
print("\n10. 正在生成靶点摘要报告...")
report = []
for _, row in top_targets.iterrows():
    report.append({
        "Gene": row["gene_name"],
        "Ensembl ID": row["ensembl_id"],
        "Disease Score": row["disease_score"],
        "Target Status": "高优先级"
    })

report_df = pd.DataFrame(report)
report_df.to_csv("drug_targets_report.csv", index=False)
print("\n靶点报告已保存至 drug_targets_report.csv")

print("\n药物靶点发现流程完成！")
```

---

## 工作流开发建议

### 错误处理
```python
import gget

def safe_gget_call(func, *args, **kwargs):
    """带错误处理的gget调用封装"""
    try:
        result = func(*args, **kwargs)
        return result
    except Exception as e:
        print(f"{func.__name__}错误: {str(e)}")
        return None

# 使用示例
result = safe_gget_call(gget.search, ["ACE2"], species="homo_sapiens")
if result is not None:
    print(result)
```

### 速率限制
```python
import time
import gget

def rate_limited_queries(gene_ids, delay=1):
    """带速率限制的多基因查询"""
    results = []
    for i, gene_id in enumerate(gene_ids):
        print(f"正在查询 {i+1}/{len(gene_ids)}: {gene_id}")
        result = gget.info([gene_id])
        results.append(result)

        if i < len(gene_ids) - 1:  # 最后一次查询后不等待
            time.sleep(delay)

    return pd.concat(results, ignore_index=True)
```

### 结果缓存
```python
import os
import pickle
import gget

def cached_gget(cache_file, func, *args, **kwargs):
    """缓存gget结果避免重复查询"""
    if os.path.exists(cache_file):
        print(f"从缓存加载: {cache_file}")
        with open(cache_file, "rb") as f:
            return pickle.load(f)

    result = func(*args, **kwargs)

    with open(cache_file, "wb") as f:
        pickle.dump(result, f)
    print(f"已缓存至: {cache_file}")

    return result

# 使用示例
result = cached_gget("ace2_info.pkl", gget.info, ["ENSG00000130234"])
```

---

这些工作流展示了如何组合多个gget模块进行综合生物信息学分析。请根据具体研究问题和数据类型进行调整。
