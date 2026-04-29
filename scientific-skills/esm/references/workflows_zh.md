# ESM工作流与示例

## 概述

本文档提供使用ESM3和ESM C的完整端到端工作流示例。每个工作流包含设置、执行和分析代码。

## 工作流1：基于思维链的新型GFP设计

利用ESM3的多模态生成能力设计新型荧光蛋白。

### 目标

通过序列、结构和功能的思维链推理，生成具有特定特性的绿色荧光蛋白(GFP)。

### 完整实现

```python
from esm.models.esm3 import ESM3
from esm.sdk.api import ESMProtein, GenerationConfig, FunctionAnnotation
import matplotlib.pyplot as plt

# 初始化
model = ESM3.from_pretrained("esm3-sm-open-v1").to("cuda")

# 步骤1：定义目标属性
print("步骤1：定义目标GFP属性...")

# 创建具有所需功能的蛋白质
target_length = 238  # 典型GFP长度
protein = ESMProtein(
    sequence="_" * target_length,
    function_annotations=[
        FunctionAnnotation(
            label="green_fluorescent_protein",
            start=65,
            end=75  # 发色团区域
        )
    ]
)

# 步骤2：通过功能条件生成初始序列
print("步骤2：生成初始序列...")

config = GenerationConfig(
    track="sequence",
    num_steps=target_length // 3,  # 渐进式生成
    temperature=0.7  # 中等多样性
)
protein = model.generate(protein, config)
print(f"生成序列: {protein.sequence[:50]}...")

# 步骤3：预测结构
print("步骤3：预测结构...")

config = GenerationConfig(
    track="structure",
    num_steps=target_length // 2
)
protein = model.generate(protein, config)
print(f"结构预测完成，坐标形状: {protein.coordinates.shape}")

# 步骤4：基于结构优化序列
print("步骤4：基于结构优化序列...")

# 标记待优化区域（如表面残基）
sequence_list = list(protein.sequence)
# 保留发色团区域，优化其他区域
for i in range(0, 65):
    if i % 3 == 0:  # 每三个位置优化一次
        sequence_list[i] = '_'
for i in range(75, target_length):
    if i % 3 == 0:
        sequence_list[i] = '_'

protein.sequence = ''.join(sequence_list)

config = GenerationConfig(
    track="sequence",
    num_steps=50,
    temperature=0.5  # 优化时使用较低温度
)
protein = model.generate(protein, config)

# 步骤5：最终验证
print("步骤5：最终验证...")

# 预测最终结构
config = GenerationConfig(track="structure", num_steps=30)
protein = model.generate(protein, config)

# 保存结果
with open("novel_gfp.pdb", "w") as f:
    f.write(protein.to_pdb())

with open("novel_gfp_sequence.txt", "w") as f:
    f.write(f">Novel_GFP\n{protein.sequence}\n")

print(f"\n最终GFP序列:\n{protein.sequence}")
print(f"\n功能注释: {protein.function_annotations}")
print(f"结构已保存至: novel_gfp.pdb")
```

### 验证步骤

```python
# 分析设计的GFP
def analyze_gfp(protein):
    """分析生成的GFP属性"""

    # 检查发色团区域（应在Ser65-Tyr66-Gly67附近）
    chromophore_region = protein.sequence[64:68]
    print(f"发色团区域: {chromophore_region}")

    # 检查桶状结构（GFP具有β-桶）
    # 如有二级结构则分析
    if protein.secondary_structure:
        beta_content = protein.secondary_structure.count('E') / len(protein.sequence)
        print(f"β折叠含量: {beta_content:.2%}")

    # 检查与已知GFPs的序列相似性
    # （实际应用中需BLAST或比对工具）

    return {
        'length': len(protein.sequence),
        'chromophore': chromophore_region,
        'coordinates_available': protein.coordinates is not None
    }

analysis = analyze_gfp(protein)
print(f"\n分析结果: {analysis}")
```

## 工作流2：蛋白质变体库生成

通过定向诱变生成并分析蛋白质变体库。

### 目标

在保持结构完整性的前提下，通过靶向诱变创建母本蛋白质的变体。

### 完整实现

```python
from esm.models.esm3 import ESM3
from esm.sdk.api import ESMProtein, GenerationConfig
import numpy as np
from sklearn.cluster import KMeans

# 初始化
model = ESM3.from_pretrained("esm3-sm-open-v1").to("cuda")

# 母本蛋白质
parent_sequence = "MPRTKEINDAGLIVHSPQWFYKARNDTESLGKIVHEFPM"
parent_protein = ESMProtein(sequence=parent_sequence)

# 定义突变参数
num_variants = 50
positions_to_mutate = 5  # 每个变体的突变位点数

# 步骤1：生成变体库
print("生成变体库...")

variants = []
for i in range(num_variants):
    # 创建带随机掩码位置的序列
    seq_list = list(parent_sequence)

    # 随机选择突变位点
    mutation_positions = np.random.choice(
        len(seq_list),
        size=positions_to_mutate,
        replace=False
    )

    for pos in mutation_positions:
        seq_list[pos] = '_'

    # 生成变体
    variant_protein = ESMProtein(sequence=''.join(seq_list))

    config = GenerationConfig(
        track="sequence",
        num_steps=positions_to_mutate * 2,
        temperature=0.8  # 高多样性
    )

    variant = model.generate(variant_protein, config)
    variants.append(variant.sequence)

    if (i + 1) % 10 == 0:
        print(f"已生成 {i + 1}/{num_variants} 个变体")

print(f"\n共生成 {len(variants)} 个变体")

# 步骤2：预测变体结构
print("\n预测变体结构...")

variant_proteins_with_structure = []
for i, seq in enumerate(variants):
    protein = ESMProtein(sequence=seq)

    config = GenerationConfig(
        track="structure",
        num_steps=len(seq) // 2
    )

    protein_with_structure = model.generate(protein, config)
    variant_proteins_with_structure.append(protein_with_structure)

    if (i + 1) % 10 == 0:
        print(f"已完成 {i + 1}/{len(variants)} 个变体的结构预测")

# 步骤3：分析变体多样性
print("\n分析变体多样性...")

# 计算与母本的汉明距离
def hamming_distance(seq1, seq2):
    """计算序列间的汉明距离"""
    return sum(c1 != c2 for c1, c2 in zip(seq1, seq2))

distances = [hamming_distance(parent_sequence, var) for var in variants]
print(f"平均每个变体的突变数: {np.mean(distances):.1f}")
print(f"突变范围: {min(distances)}-{max(distances)}")

# 步骤4：获取聚类用嵌入向量
print("\n生成聚类用嵌入向量...")

from esm.models.esmc import ESMC

embedding_model = ESMC.from_pretrained("esmc-300m").to("cuda")

def get_embedding(sequence):
    """获取序列的平均池化嵌入向量"""
    protein = ESMProtein(sequence=sequence)
    tensor = embedding_model.encode(protein)
    emb = embedding_model.forward(tensor)
    return emb.mean(dim=1).cpu().detach().numpy().flatten()

variant_embeddings = np.array([get_embedding(seq) for seq in variants])

# 步骤5：变体聚类
print("变体聚类...")

n_clusters = 5
kmeans = KMeans(n_clusters=n_clusters, random_state=42)
cluster_labels = kmeans.fit_predict(variant_embeddings)

# 聚类分析
print("\n聚类分析:")
for i in range(n_clusters):
    cluster_variants = [var for var, label in zip(variants, cluster_labels) if label == i]
    cluster_distances = [hamming_distance(parent_sequence, var) for var in cluster_variants]

    print(f"\n聚类 {i}:")
    print(f"  数量: {len(cluster_variants)}")
    print(f"  与母本平均距离: {np.mean(cluster_distances):.1f}")
    print(f"  代表序列: {cluster_variants[0][:40]}...")

# 步骤6：选择多样性代表序列
print("\n选择多样性代表序列...")

representatives = []
for i in range(n_clusters):
    # 获取聚类中心
    cluster_indices = np.where(cluster_labels == i)[0]
    cluster_embs = variant_embeddings[cluster_indices]

    # 寻找最接近中心的样本
    centroid = cluster_embs.mean(axis=0)
    distances_to_centroid = np.linalg.norm(cluster_embs - centroid, axis=1)
    rep_idx = cluster_indices[np.argmin(distances_to_centroid)]

    representatives.append(variants[rep_idx])

# 保存结果
print("\n保存结果...")

with open("variant_library.fasta", "w") as f:
    f.write(f">Parent\n{parent_sequence}\n\n")
    for i, var in enumerate(variants):
        f.write(f">Variant_{i+1}_Cluster_{cluster_labels[i]}\n{var}\n")

with open("representative_variants.fasta", "w") as f:
    for i, rep in enumerate(representatives):
        f.write(f">Representative_Cluster_{i}\n{rep}\n")

print("变体库已保存至: variant_library.fasta")
print("代表序列已保存至: representative_variants.fasta")
```

## 工作流3：基于结构的序列优化

在保持折叠结构的前提下优化蛋白质序列以提升稳定性。

### 目标

给定蛋白质结构，设计能维持折叠但具有改进特性的序列。

### 完整实现

```python
from esm.models.esm3 import ESM3
from esm.sdk.api import ESMProtein, GenerationConfig
import numpy as np

# 初始化
model = ESM3.from_pretrained("esm3-sm-open-v1").to("cuda")

# 加载目标结构（如从PDB文件）
target_protein = ESMProtein.from_pdb("target_structure.pdb")
original_sequence = target_protein.sequence

print(f"原始序列: {original_sequence}")
print(f"已加载结构: {target_protein.coordinates.shape}")

# 步骤1：生成多个序列设计方案
print("\n生成优化序列...")

num_designs = 20
optimized_sequences = []

for i in range(num_designs):
    # 从结构开始，移除序列
    design_protein = ESMProtein(
        coordinates=target_protein.coordinates.copy(),
        secondary_structure=target_protein.secondary_structure
    )

    # 为该结构生成序列
    config = GenerationConfig(
        track="sequence",
        num_steps=len(original_sequence),
        temperature=0.7,
        condition_on_coordinates_only=True
    )

    designed = model.generate(design_protein, config)
    optimized_sequences.append(designed.sequence)

    if (i + 1) % 5 == 0:
        print(f"已生成 {i + 1}/{num_designs} 个设计方案")

# 步骤2：验证结构兼容性
print("\n验证结构兼容性...")

validated_designs = []

for seq in optimized_sequences:
    # 预测设计序列的结构
    test_protein = ESMProtein(sequence=seq)

    config = GenerationConfig(
        track="structure",
        num_steps=len(seq) // 2
    )

    predicted = model.generate(test_protein, config)

    # 计算RMSD（简化版 - 实际需对齐）
    # 此处仅检查结构预测是否成功
    if predicted.coordinates is not None:
        validated_designs.append(seq)

print(f"已验证 {len(validated_designs)}/{num_designs} 个设计")

# 步骤3：分析序列特性
print("\n分析序列特性...")

def calculate_properties(sequence):
    """计算基本序列特性"""
    # 疏水性（简化）
    hydrophobic = "AILMFWYV"
    hydrophobic_fraction = sum(1 for aa in sequence if aa in hydrophobic) / len(sequence)

    # 电荷
    positive = "KR"
    negative = "DE"
    net_charge = sum(1 for aa in sequence if aa in positive) - sum(1 for aa in sequence if aa in negative)

    # 芳香族含量
    aromatic = "FWY"
    aromatic_fraction = sum(1 for aa in sequence if aa in aromatic) / len(sequence)

    return {
        'hydrophobic_fraction': hydrophobic_fraction,
        'net_charge': net_charge,
        'aromatic_fraction': aromatic_fraction
    }

# 与原始序列比较
original_props = calculate_properties(original_sequence)
print(f"\n原始特性:")
print(f"  疏水性: {original_props['hydrophobic_fraction']:.2%}")
print(f"  净电荷: {original_props['net_charge']:+d}")
print(f"  芳香族: {original_props['aromatic_fraction']:.2%}")

# 分析设计方案
design_properties = [calculate_properties(seq) for seq in validated_designs]

avg_hydrophobic = np.mean([p['hydrophobic_fraction'] for p in design_properties])
avg_charge = np.mean([p['net_charge'] for p in design_properties])
avg_aromatic = np.mean([p['aromatic_fraction'] for p in design_properties])

print(f"\n设计序列（平均值）:")
print(f"  疏水性: {avg_hydrophobic:.2%}")
print(f"  净电荷: {avg_charge:+.1f}")
print(f"  芳香族: {avg_aromatic:.2%}")

# 步骤4：设计方案排序
print("\n设计方案排序...")

def score_design(sequence, original_props):
    """基于期望特性评分设计方案"""
    props = calculate_properties(sequence)

    # 偏好更高疏水性（提升稳定性）
    hydrophobic_score = props['hydrophobic_fraction']

    # 偏好与原始电荷相近
    charge_score = 1.0 / (1.0 + abs(props['net_charge'] - original_props['net_charge']))

    # 综合评分
    return hydrophobic_score * 0.6 + charge_score * 0.4

scores = [(seq, score_design(seq, original_props)) for seq in validated_designs]
scores.sort(key=lambda x: x[1], reverse=True)

print("\n前5名设计方案:")
for i, (seq, score) in enumerate(scores[:5]):
    print(f"\n{i+1}. 评分: {score:.3f}")
    print(f"   序列: {seq[:40]}...")

# 步骤5：保存结果
print("\n保存结果...")

with open("optimized_sequences.fasta", "w") as f:
    f.write(f">Original\n{original_sequence}\n\n")

    for i, (seq, score) in enumerate(scores):
        props = calculate_properties(seq)
        f.write(f">Design_{i+1}_Score_{score:.3f}\n")
        f.write(f"# 疏水性: {props['hydrophobic_fraction']:.2%}, ")
        f.write(f"电荷: {props['net_charge']:+d}, ")
        f.write(f"芳香族: {props['aromatic_fraction']:.2%}\n")
        f.write(f"{seq}\n\n")

print("结果已保存至: optimized_sequences.fasta")
```

## 工作流4：功能预测流程

使用ESM3和ESM C构建蛋白质功能预测流程。

### 目标

结合生成式(ESM3)和嵌入式(ESM C)方法构建蛋白质功能预测流程。

### 完整实现

```python
from esm.models.esm3 import ESM3
from esm.models.esmc import ESMC
from esm.sdk.api import ESMProtein, GenerationConfig
import numpy as np
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import cross_val_score

# 初始化模型
esm3_model = ESM3.from_pretrained("esm3-sm-open-v1").to("cuda")
esmc_model = ESMC.from_pretrained("esmc-600m").to("cuda")

# 示例：预测蛋白质是否为酶
# （实际应用中需带标签的训练集）

def predict_function_generative(sequence):
    """使用ESM3生成式方法预测功能"""

    protein = ESMProtein(sequence=sequence)

    # 生成功能注释
    config = GenerationConfig(
        track="

```markdown
def predict_function_embedding(sequence, function_classifier):
    """使用ESM C嵌入和分类器预测蛋白质功能"""

    # 获取嵌入向量
    protein = ESMProtein(sequence=sequence)
    tensor = esmc_model.encode(protein)
    embedding = esmc_model.forward(tensor)

    # 均值池化
    embedding_pooled = embedding.mean(dim=1).cpu().detach().numpy()

    # 使用分类器预测
    prediction = function_classifier.predict(embedding_pooled)
    probability = function_classifier.predict_proba(embedding_pooled)

    return prediction[0], probability[0]

# 测试序列的示例工作流
test_sequences = {
    "kinase": "MPRTKEINDAGLIVHSPQWFYKARNDTESLGKIVHEF",
    "protease": "AGLIVHSPQWFYKARNDTESLGKIVHEFPMCDEGH",
    "transporter": "KTEFLNDGRPMLIVHSPQWFYKARNDTESLGKIVH"
}

print("正在预测功能...\n")

for name, sequence in test_sequences.items():
    print(f"{name.upper()}:")
    print(f"序列: {sequence[:30]}...")

    # 方法1：生成式预测
    functions = predict_function_generative(sequence)
    print(f"  生成式预测结果: {functions}")

    # 方法2：基于嵌入的方法需要训练好的分类器
    # (此示例中跳过，需要训练数据)

    print()

## 工作流程5：基于嵌入的聚类与分析

使用ESM C嵌入对大型蛋白质数据集进行聚类分析。

### 完整实现

```python
from esm.models.esmc import ESMC
from esm.sdk.api import ESMProtein
import numpy as np
from sklearn.cluster import DBSCAN
from sklearn.decomposition import PCA
from sklearn.manifold import TSNE
import matplotlib.pyplot as plt

# 初始化
model = ESMC.from_pretrained("esmc-600m").to("cuda")

# 加载蛋白质数据集（示例）
sequences = [
    # 实际应用中从FASTA文件或数据库加载
    "MPRTKEINDAGLIVHSPQWFYK",
    "AGLIVHSPQWFYKARNDTESL",
    # ... 更多序列
]

print(f"已加载 {len(sequences)} 条序列")

# 步骤1：生成嵌入向量
print("生成嵌入向量中...")

embeddings = []
for i, seq in enumerate(sequences):
    protein = ESMProtein(sequence=seq)
    tensor = model.encode(protein)
    emb = model.forward(tensor)

    # 均值池化
    emb_pooled = emb.mean(dim=1).cpu().detach().numpy().flatten()
    embeddings.append(emb_pooled)

    if (i + 1) % 100 == 0:
        print(f"已处理 {i + 1}/{len(sequences)}")

embeddings = np.array(embeddings)
print(f"嵌入向量维度: {embeddings.shape}")

# 步骤2：降维可视化
print("\n降维处理中...")

# PCA初步降维
pca = PCA(n_components=50)
embeddings_pca = pca.fit_transform(embeddings)
print(f"PCA解释方差: {pca.explained_variance_ratio_[:10].sum():.2%}")

# t-SNE可视化
tsne = TSNE(n_components=2, random_state=42)
embeddings_2d = tsne.fit_transform(embeddings_pca)

# 步骤3：聚类分析
print("\n聚类中...")

# DBSCAN密度聚类
clustering = DBSCAN(eps=0.5, min_samples=5)
cluster_labels = clustering.fit_predict(embeddings)

n_clusters = len(set(cluster_labels)) - (1 if -1 in cluster_labels else 0)
n_noise = list(cluster_labels).count(-1)

print(f"聚类数量: {n_clusters}")
print(f"噪声点数量: {n_noise}")

# 步骤4：可视化
print("\n生成可视化图表...")

plt.figure(figsize=(12, 8))
scatter = plt.scatter(
    embeddings_2d[:, 0],
    embeddings_2d[:, 1],
    c=cluster_labels,
    cmap='viridis',
    alpha=0.6
)
plt.colorbar(scatter)
plt.title("蛋白质序列聚类（ESM C嵌入）")
plt.xlabel("t-SNE 1")
plt.ylabel("t-SNE 2")
plt.savefig("protein_clusters.png", dpi=300, bbox_inches='tight')
print("可视化结果已保存至: protein_clusters.png")

# 步骤5：聚类分析
print("\n聚类分析:")

for cluster_id in range(n_clusters):
    cluster_indices = np.where(cluster_labels == cluster_id)[0]
    cluster_seqs = [sequences[i] for i in cluster_indices]

    print(f"\n聚类 {cluster_id}:")
    print(f"  大小: {len(cluster_seqs)}")
    print(f"  平均长度: {np.mean([len(s) for s in cluster_seqs]):.1f}")
    print(f"  示例: {cluster_seqs[0][:40]}...")

# 保存聚类分配结果
with open("cluster_assignments.txt", "w") as f:
    for i, (seq, label) in enumerate(zip(sequences, cluster_labels)):
        f.write(f"Sequence_{i}\tCluster_{label}\t{seq}\n")

print("\n聚类分配结果已保存至: cluster_assignments.txt")
```

## 附加工作流建议

### 大型数据集内存管理

```python
def process_large_dataset(sequences, batch_size=32):
    """分批处理大型数据集以管理内存"""
    import gc
    import torch

    results = []

    for i in range(0, len(sequences), batch_size):
        batch = sequences[i:i + batch_size]

        # 处理批次
        batch_results = [process_sequence(seq) for seq in batch]
        results.extend(batch_results)

        # 清理内存
        torch.cuda.empty_cache()
        gc.collect()

        if (i + batch_size) % 100 == 0:
            print(f"已处理 {min(i + batch_size, len(sequences))}/{len(sequences)}")

    return results
```

### 并行处理

```python
from concurrent.futures import ThreadPoolExecutor
import asyncio

def parallel_workflow(sequences, n_workers=4):
    """并行处理序列"""

    with ThreadPoolExecutor(max_workers=n_workers) as executor:
        results = list(executor.map(process_sequence, sequences))

    return results
```

这些工作流提供了常见ESM应用的完整示例。请根据具体需求调整，并通过生物学实验验证结果。
```
