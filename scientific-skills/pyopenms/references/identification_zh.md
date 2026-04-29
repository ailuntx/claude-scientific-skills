# 肽段与蛋白质鉴定

## 概述

PyOpenMS 通过集成多种搜索引擎支持肽段/蛋白质鉴定，并提供后处理工具用于假发现率控制、蛋白质推断和注释等操作。

## 支持的搜索引擎

PyOpenMS 集成了以下搜索引擎：

- **Comet**：快速串联质谱搜索
- **Mascot**：商业搜索引擎
- **MSGFPlus**：基于谱图概率的搜索
- **XTandem**：开源搜索工具
- **OMSSA**：NCBI 搜索引擎
- **Myrimatch**：高通量搜索
- **MSFragger**：超快速搜索

## 读取鉴定数据

### idXML 格式

```python
import pyopenms as ms

# 加载鉴定结果
protein_ids = []
peptide_ids = []

ms.IdXMLFile().load("identifications.idXML", protein_ids, peptide_ids)

print(f"蛋白质鉴定数: {len(protein_ids)}")
print(f"肽段鉴定数: {len(peptide_ids)}")
```

### 访问肽段鉴定结果

```python
# 遍历肽段鉴定结果
for peptide_id in peptide_ids:
    # 谱图元数据
    print(f"保留时间: {peptide_id.getRT():.2f}")
    print(f"质荷比: {peptide_id.getMZ():.4f}")

    # 获取肽段匹配结果（按分数排序）
    hits = peptide_id.getHits()
    print(f"匹配数: {len(hits)}")

    for hit in hits:
        sequence = hit.getSequence()
        print(f"  序列: {sequence.toString()}")
        print(f"  分数: {hit.getScore()}")
        print(f"  电荷: {hit.getCharge()}")
        print(f"  质量误差 (ppm): {hit.getMetaValue('mass_error_ppm')}")

        # 获取修饰信息
        if sequence.isModified():
            for i in range(sequence.size()):
                residue = sequence.getResidue(i)
                if residue.isModified():
                    print(f"    位置 {i} 修饰: {residue.getModificationName()}")
```

### 访问蛋白质鉴定结果

```python
# 访问蛋白质层级信息
for protein_id in protein_ids:
    # 搜索参数
    search_params = protein_id.getSearchParameters()
    print(f"搜索引擎: {protein_id.getSearchEngine()}")
    print(f"数据库: {search_params.db}")

    # 蛋白质匹配结果
    hits = protein_id.getHits()
    for hit in hits:
        print(f"  登录号: {hit.getAccession()}")
        print(f"  分数: {hit.getScore()}")
        print(f"  覆盖率: {hit.getCoverage()}")
        print(f"  序列: {hit.getSequence()}")
```

## 假发现率控制 (FDR)

### FDR 过滤

应用 FDR 过滤控制假阳性：

```python
# 创建 FDR 对象
fdr = ms.FalseDiscoveryRate()

# 在 PSM 层级应用 FDR
fdr.apply(peptide_ids)

# 按 FDR 阈值过滤
fdr_threshold = 0.01  # 1% FDR
filtered_peptide_ids = []

for peptide_id in peptide_ids:
    # 保留低于 FDR 阈值的匹配
    filtered_hits = []
    for hit in peptide_id.getHits():
        if hit.getScore() <= fdr_threshold:  # 分数越低越好
            filtered_hits.append(hit)

    if filtered_hits:
        peptide_id.setHits(filtered_hits)
        filtered_peptide_ids.append(peptide_id)

print(f"通过 FDR 的肽段数: {len(filtered_peptide_ids)}")
```

### 分数转换

将分数转换为 q 值：

```python
# 应用分数转换
fdr.apply(peptide_ids)

# 访问 q 值
for peptide_id in peptide_ids:
    for hit in peptide_id.getHits():
        q_value = hit.getMetaValue("q-value")
        print(f"序列: {hit.getSequence().toString()}, q值: {q_value}")
```

## 蛋白质推断

### ID 映射器

将肽段鉴定映射到蛋白质：

```python
# 创建映射器
mapper = ms.IDMapper()

# 映射到特征
feature_map = ms.FeatureMap()
ms.FeatureXMLFile().load("features.featureXML", feature_map)

# 用鉴定结果注释特征
mapper.annotate(feature_map, peptide_ids, protein_ids)

# 检查注释特征
for feature in feature_map:
    pep_ids = feature.getPeptideIdentifications()
    if pep_ids:
        for pep_id in pep_ids:
            for hit in pep_id.getHits():
                print(f"特征 {feature.getMZ():.4f}: {hit.getSequence().toString()}")
```

### 蛋白质分组

通过共享肽段进行蛋白质分组：

```python
# 创建蛋白质推断算法
inference = ms.BasicProteinInferenceAlgorithm()

# 执行推断
inference.run(peptide_ids, protein_ids)

# 访问蛋白质组
for protein_id in protein_ids:
    hits = protein_id.getHits()
    if len(hits) > 1:
        print("蛋白质组:")
        for hit in hits:
            print(f"  {hit.getAccession()}")
```

## 肽段序列处理

### AASequence 对象

操作肽段序列：

```python
# 创建肽段序列
seq = ms.AASequence.fromString("PEPTIDE")

print(f"序列: {seq.toString()}")
print(f"单同位素质量: {seq.getMonoWeight():.4f}")
print(f"平均质量: {seq.getAverageWeight():.4f}")
print(f"长度: {seq.size()}")

# 访问单个氨基酸
for i in range(seq.size()):
    residue = seq.getResidue(i)
    print(f"位置 {i}: {residue.getOneLetterCode()}, 质量: {residue.getMonoWeight():.4f}")
```

### 修饰序列

处理翻译后修饰：

```python
# 含修饰的序列
mod_seq = ms.AASequence.fromString("PEPTIDEM(Oxidation)K")

print(f"修饰序列: {mod_seq.toString()}")
print(f"含修饰质量: {mod_seq.getMonoWeight():.4f}")

# 检查修饰状态
print(f"是否修饰: {mod_seq.isModified()}")

# 获取修饰信息
for i in range(mod_seq.size()):
    residue = mod_seq.getResidue(i)
    if residue.isModified():
        print(f"位置 {i} 的残基 {residue.getOneLetterCode()}")
        print(f"  修饰类型: {residue.getModificationName()}")
```

### 肽段酶切

模拟酶切消化：

```python
# 创建消化酶
enzyme = ms.ProteaseDigestion()
enzyme.setEnzyme("Trypsin")

# 设置漏切位点数
enzyme.setMissedCleavages(2)

# 消化蛋白质序列
protein_seq = "MKTAYIAKQRQISFVKSHFSRQLEERLGLIEVQAPILSRVGDGTQDNLSGAEKAVQVKVKALPDAQFEVVHSLAKWKRQTLGQHDFSAGEGLYTHMKALRPDEDRLSPLHSVYVDQWDWERVMGDGERQFSTLKSTVEAIWAGIKATEAAVSEEFGLAPFLPDQIHFVHSQELLSRYPDLDAKGRERAIAKDLGAVFLVGIGGKLSDGHRHDVRAPDYDDWSTPSELGHAGLNGDILVWNPVLEDAFELSSMGIRVDADTLKHQLALTGDEDRLELEWHQALLRGEMPQTIGGGIGQSRLTMLLLQLPHIGQVQAGVWPAAVRESVPSLL"

# 获取肽段
peptides = []
enzyme.digest(ms.AASequence.fromString(protein_seq), peptides)

print(f"生成 {len(peptides)} 个肽段")
for peptide in peptides[:5]:  # 显示前5个
    print(f"  {peptide.toString()}, 质量: {peptide.getMonoWeight():.2f}")
```

## 理论谱图生成

### 碎片离子计算

生成理论碎片离子：

```python
# 创建肽段
peptide = ms.AASequence.fromString("PEPTIDE")

# 生成 b/y 离子
fragments = []
ms.TheoreticalSpectrumGenerator().getSpectrum(fragments, peptide, 1, 1)

print(f"生成 {fragments.size()} 个碎片离子")

# 访问碎片
mz, intensity = fragments.get_peaks()
for m, i in zip(mz[:10], intensity[:10]):  # 显示前10个
    print(f"质荷比: {m:.4f}, 强度: {i}")
```

## 完整鉴定流程

### 端到端示例

```python
import pyopenms as ms

def identification_workflow(spectrum_file, fasta_file, output_file):
    """
    含FDR控制的完整鉴定流程

    参数:
        spectrum_file: 输入mzML文件
        fasta_file: 蛋白质数据库(FASTA)
        output_file: 输出idXML文件
    """

    # 步骤1: 加载谱图数据
    exp = ms.MSExperiment()
    ms.MzMLFile().load(spectrum_file, exp)
    print(f"加载 {exp.getNrSpectra()} 张谱图")

    # 步骤2: 配置搜索参数
    search_params = ms.SearchParameters()
    search_params.db = fasta_file
    search_params.precursor_mass_tolerance = 10.0  # ppm
    search_params.fragment_mass_tolerance = 0.5  # Da
    search_params.enzyme = "Trypsin"
    search_params.missed_cleavages = 2
    search_params.modifications = ["Oxidation (M)", "Carbamidomethyl (C)"]

    # 步骤3: 执行搜索(以Comet适配器为例)
    # 注意: 需预先安装搜索引擎
    # comet = ms.CometAdapter()
    # protein_ids, peptide_ids = comet.search(exp, search_params)

    # 本示例加载预计算结果
    protein_ids = []
    peptide_ids = []
    ms.IdXMLFile().load("raw_identifications.idXML", protein_ids, peptide_ids)

    print(f"初始肽段鉴定数: {len(peptide_ids)}")

    # 步骤4: 应用FDR过滤
    fdr = ms.FalseDiscoveryRate()
    fdr.apply(peptide_ids)

    # 按1% FDR过滤
    filtered_peptide_ids = []
    for peptide_id in peptide_ids:
        filtered_hits = []
        for hit in peptide_id.getHits():
            q_value = hit.getMetaValue("q-value")
            if q_value <= 0.01:
                filtered_hits.append(hit)

        if filtered_hits:
            peptide_id.setHits(filtered_hits)
            filtered_peptide_ids.append(peptide_id)

    print(f"FDR过滤后肽段数(1%): {len(filtered_peptide_ids)}")

    # 步骤5: 蛋白质推断
    inference = ms.BasicProteinInferenceAlgorithm()
    inference.run(filtered_peptide_ids, protein_ids)

    print(f"鉴定蛋白质数: {len(protein_ids)}")

    # 步骤6: 保存结果
    ms.IdXMLFile().store(output_file, protein_ids, filtered_peptide_ids)
    print(f"结果保存至 {output_file}")

    return protein_ids, filtered_peptide_ids

# 执行流程
protein_ids, peptide_ids = identification_workflow(
    "spectra.mzML",
    "database.fasta",
    "identifications_fdr.idXML"
)
```

## 谱图库搜索

### 库匹配

```python
# 加载谱图库
library = ms.MSPFile()
library_spectra = []
library.load("spectral_library.msp", library_spectra)

# 加载实验谱图
exp = ms.MSExperiment()
ms.MzMLFile().load("data.mzML", exp)

# 谱图比对
spectra_compare = ms.SpectraSTSimilarityScore()

for exp_spec in exp:
    if exp_spec.getMSLevel() == 2:
        best_match_score = 0
        best_match_lib = None

        for lib_spec in library_spectra:
            score = spectra_compare.operator()(exp_spec, lib_spec)
            if score > best_match_score:
                best_match_score = score
                best_match_lib = lib_spec

        if best_match_score > 0.7:  # 阈值
            print(f"找到匹配: 分数 {best_match_score:.3f}")
```

## 最佳实践

### 诱饵数据库

使用靶标-诱饵策略计算 FDR：

```python
# 生成诱饵数据库
decoy_generator = ms.DecoyGenerator()

# 加载靶标数据库
fasta_entries = []
ms.FASTAFile().load("target.fasta", fasta_entries)

# 生成诱饵序列
decoy_entries = []
for entry in fasta_entries:
    decoy_entry = decoy_generator.reverseProtein(entry)
    decoy_entries.append(decoy_entry)

# 保存合并数据库
all_entries = fasta_entries + decoy_entries
ms.FASTAFile().store("target_decoy.fasta", all_entries)
```

### 分数解释

理解不同引擎的分数类型：

```python
# 根据搜索引擎解释分数
for peptide_id in peptide_ids:
    search_engine = peptide_id.getIdentifier()

    for hit in peptide_id.getHits():
        score = hit.getScore()

        # 不同引擎的分数解释不同
        if "Comet" in search_engine:
            # Comet: E值越高越差
            print(f"E值: {score}")
        elif "Mascot" in search_engine:
            # Mascot: 分数越高越好
            print(f"离子分数: {score}")
```
