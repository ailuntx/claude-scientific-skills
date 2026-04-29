# TDC 数据集综合目录

本文档提供治疗数据共享库（Therapeutics Data Commons）中所有可用数据集的综合目录，按任务类别组织。

## 单实例预测数据集

### ADME（吸收、分布、代谢、排泄）

**吸收：**
- `Caco2_Wang` - Caco-2细胞渗透性（906种化合物）
- `Caco2_AstraZeneca` - 阿斯利康提供的Caco-2渗透性（700种化合物）
- `HIA_Hou` - 人体肠道吸收（578种化合物）
- `Pgp_Broccatelli` - P-糖蛋白抑制（1,212种化合物）
- `Bioavailability_Ma` - 口服生物利用度（640种化合物）
- `F20_edrug3d` - 口服生物利用度F≥20%（1,017种化合物）
- `F30_edrug3d` - 口服生物利用度F≥30%（1,017种化合物）

**分布：**
- `BBB_Martins` - 血脑屏障穿透（1,975种化合物）
- `PPBR_AZ` - 血浆蛋白结合率（1,797种化合物）
- `VDss_Lombardo` - 稳态分布容积（1,130种化合物）

**代谢：**
- `CYP2C19_Veith` - CYP2C19抑制（12,665种化合物）
- `CYP2D6_Veith` - CYP2D6抑制（13,130种化合物）
- `CYP3A4_Veith` - CYP3A4抑制（12,328种化合物）
- `CYP1A2_Veith` - CYP1A2抑制（12,579种化合物）
- `CYP2C9_Veith` - CYP2C9抑制（12,092种化合物）
- `CYP2C9_Substrate_CarbonMangels` - CYP2C9底物（666种化合物）
- `CYP2D6_Substrate_CarbonMangels` - CYP2D6底物（664种化合物）
- `CYP3A4_Substrate_CarbonMangels` - CYP3A4底物（667种化合物）

**排泄：**
- `Half_Life_Obach` - 半衰期（667种化合物）
- `Clearance_Hepatocyte_AZ` - 肝细胞清除率（1,020种化合物）
- `Clearance_Microsome_AZ` - 微粒体清除率（1,102种化合物）

**溶解性与亲脂性：**
- `Solubility_AqSolDB` - 水溶性（9,982种化合物）
- `Lipophilicity_AstraZeneca` - 亲脂性（logD）（4,200种化合物）
- `HydrationFreeEnergy_FreeSolv` - 水合自由能（642种化合物）

### 毒性

**器官毒性：**
- `hERG` - hERG通道抑制/心脏毒性（648种化合物）
- `hERG_Karim` - hERG阻断剂扩展数据集（13,445种化合物）
- `DILI` - 药物性肝损伤（475种化合物）
- `Skin_Reaction` - 皮肤反应（404种化合物）
- `Carcinogens_Lagunin` - 致癌性（278种化合物）
- `Respiratory_Toxicity` - 呼吸系统毒性（278种化合物）

**一般毒性：**
- `AMES` - 艾姆斯致突变性（7,255种化合物）
- `LD50_Zhu` - 急性毒性LD50（7,385种化合物）
- `ClinTox` - 临床试验毒性（1,478种化合物）
- `SkinSensitization` - 皮肤致敏性（278种化合物）
- `EyeCorrosion` - 眼腐蚀性（278种化合物）
- `EyeIrritation` - 眼刺激性（278种化合物）

**环境毒性：**
- `Tox21-AhR` - 核受体信号传导（8,169种化合物）
- `Tox21-AR` - 雄激素受体（9,362种化合物）
- `Tox21-AR-LBD` - 雄激素受体配体结合（8,343种化合物）
- `Tox21-ARE` - 抗氧化反应元件（6,475种化合物）
- `Tox21-aromatase` - 芳香酶抑制（6,733种化合物）
- `Tox21-ATAD5` - DNA损伤（8,163种化合物）
- `Tox21-ER` - 雌激素受体（7,257种化合物）
- `Tox21-ER-LBD` - 雌激素受体配体结合（8,163种化合物）
- `Tox21-HSE` - 热休克反应（8,162种化合物）
- `Tox21-MMP` - 线粒体膜电位（7,394种化合物）
- `Tox21-p53` - p53通路（8,163种化合物）
- `Tox21-PPAR-gamma` - PPARγ激活（7,396种化合物）

### HTS（高通量筛选）

**SARS-CoV-2：**
- `SARSCoV2_Vitro_Touret` - 体外抗病毒活性（1,484种化合物）
- `SARSCoV2_3CLPro_Diamond` - 3CL蛋白酶抑制（879种化合物）
- `SARSCoV2_Vitro_AlabdulKareem` - 体外筛选（5,953种化合物）

**其他靶点：**
- `Orexin1_Receptor_Butkiewicz` - 食欲素受体筛选（4,675种化合物）
- `M1_Receptor_Agonist_Butkiewicz` - M1受体激动剂（1,700种化合物）
- `M1_Receptor_Antagonist_Butkiewicz` - M1受体拮抗剂（1,700种化合物）
- `HIV_Butkiewicz` - HIV抑制（40,000+种化合物）
- `ToxCast` - 环境化学品筛选（8,597种化合物）

### QM（量子力学）

- `QM7` - 量子力学性质（7,160个分子）
- `QM8` - 电子光谱与激发态（21,786个分子）
- `QM9` - 几何/能量/电子/热力学性质（133,885个分子）

### 产率

- `Buchwald-Hartwig` - 反应产率预测（3,955个反应）
- `USPTO_Yields` - USPTO产率预测（853,879个反应）

### 表位

- `IEDBpep-DiseaseBinder` - 疾病相关表位结合（6,080个肽段）
- `IEDBpep-NonBinder` - 非结合肽段（24,320个肽段）

### 开发

- `Manufacturing` - 生产成功率预测
- `Formulation` - 制剂稳定性预测

### CRISPROutcome

- `CRISPROutcome_Doench` - 基因编辑效率预测（5,310个引导RNA）

## 多实例预测数据集

### DTI（药物-靶点相互作用）

**结合亲和力：**
- `BindingDB_Kd` - 解离常数（52,284对，10,665种药物，1,413种蛋白）
- `BindingDB_IC50` - 半最大抑制浓度（991,486对，549,205种药物，5,078种蛋白）
- `BindingDB_Ki` - 抑制常数（375,032对，174,662种药物，3,070种蛋白）

**激酶结合：**
- `DAVIS` - Davis激酶结合数据集（30,056对，68种药物，442种蛋白）
- `KIBA` - KIBA激酶结合数据集（118,254对，2,111种药物，229种蛋白）

**二元相互作用：**
- `BindingDB_Patent` - 专利衍生的DTI（8,503对）
- `BindingDB_Approval` - FDA批准药物的DTI（1,649对）

### DDI（药物-药物相互作用）

- `DrugBank` - 药物相互作用（191,808对，1,706种药物）
- `TWOSIDES` - 基于副作用的DDI（4,649,441对，645种药物）

### PPI（蛋白质-蛋白质相互作用）

- `HuRI` - 人类参考蛋白质相互作用组（52,569个相互作用）
- `STRING` - 蛋白质功能关联（19,247个相互作用）

### GDA（基因-疾病关联）

- `DisGeNET` - 基因-疾病关联（81,746对）
- `PrimeKG_GDA` - PrimeKG知识图谱的基因-疾病关联

### DrugRes（药物反应/耐药性）

- `GDSC1` - 癌症药物敏感性基因组学v1（178,000对）
- `GDSC2` - 癌症药物敏感性基因组学v2（125,000对）

### DrugSyn（药物协同作用）

- `DrugComb` - 药物组合协同效应（345,502种组合）
- `DrugCombDB` - 药物组合数据库（448,555种组合）
- `OncoPolyPharmacology` - 肿瘤药物组合（22,737种组合）

### PeptideMHC

- `MHC1_NetMHCpan` - MHC I类结合（184,983对）
- `MHC2_NetMHCIIpan` - MHC II类结合（134,281对）

### AntibodyAff（抗体亲和力）

- `Protein_SAbDab` - 抗体-抗原亲和力（1,500+对）

### MTI（miRNA-靶点相互作用）

- `miRTarBase` - 实验验证的miRNA-靶点相互作用（380,639对）

### 催化剂

- `USPTO_Catalyst` - 反应催化剂预测（11,000+个反应）

### 临床试验结果

- `TrialOutcome_WuXi` - 临床试验结果预测（3,769项试验）

## 生成数据集

### MolGen（分子生成）

- `ChEMBL_V29` - ChEMBL类药分子（1,941,410个分子）
- `ZINC` - ZINC数据库子集（100,000+个分子）
- `GuacaMol` - 目标导向基准分子
- `Moses` - 分子集基准（1,936,962个分子）

### RetroSyn（逆合成）

- `USPTO` - USPTO专利逆合成（1,939,253个反应）
- `USPTO-50K` - 精选USPTO子集（50,000个反应）

### PairMolGen（配对分子生成）

- `Prodrug` - 前药到药物的转化（1,000+对）
- `Metabolite` - 药物到代谢物的转化

## 使用 retrieve_dataset_names

通过编程方式获取特定任务的所有可用数据集：

```python
from tdc.utils import retrieve_dataset_names

# 获取特定任务的所有数据集
adme_datasets = retrieve_dataset_names('ADME')
tox_datasets = retrieve_dataset_names('Tox')
dti_datasets = retrieve_dataset_names('DTI')
hts_datasets = retrieve_dataset_names('HTS')
```

## 数据集统计

直接访问数据集统计信息：

```python
from tdc.single_pred import ADME
data = ADME(name='Caco2_Wang')

# 打印基础统计信息
data.print_stats()

# 获取标签分布
data.label_distribution()
```

## 加载数据集

所有数据集遵循相同的加载模式：

```python
from tdc.<problem_type> import <TaskType>
data = <TaskType>(name='<DatasetName>')

# 获取完整数据集
df = data.get_data(format='df')  # 或 'dict'、'DeepPurpose' 等格式

# 获取训练/验证/测试集划分
split = data.get_split(method='scaffold', seed=1, frac=[0.7, 0.1, 0.2])
```

## 注意事项

- 数据集大小和统计信息为近似值，可能更新
- TDC定期新增数据集
- 部分数据集可能需要额外依赖项
- 最新数据集列表请访问TDC官网：https://tdcommons.ai/overview/
