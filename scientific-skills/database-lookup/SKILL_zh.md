---
name: 数据库查询
description: 通过REST API搜索78个公共科学、生物医学、材料科学和经济数据库。涵盖物理/天文学（NASA、NIST、SDSS、SIMBAD）、地球/环境（USGS、NOAA、EPA）、化学/药物（PubChem、ChEMBL、DrugBank、FDA、KEGG、ZINC、BindingDB）、材料（Materials Project、COD）、生物学/基因组学（Reactome、UniProt、STRING、Ensembl、NCBI Gene、GEO、GTEx、PDB、AlphaFold、InterPro、BioGRID、Gene Ontology、dbSNP、gnomAD、ENCODE、Human Protein Atlas、Human Cell Atlas）、疾病/临床（COSMIC、Open Targets、ClinicalTrials.gov、OMIM、ClinVar、GDC/TCGA、cBioPortal、DisGeNET、GWAS Catalog）、监管（FDA、USPTO、SEC EDGAR）、经济/金融（FRED、World Bank、US Treasury）、人口统计（US Census、Eurostat、WHO）。用于查询化合物、基因、蛋白质、通路、变异、临床试验、专利、经济指标或任何公共数据库API查询。
metadata:
  skill-author: K-Dense Inc.
---

# 数据库查询

你可以通过REST API访问78个公共数据库。你的任务是确定哪些数据库与用户的问题相关，查询它们，并返回原始JSON结果以及你使用的数据库。

## 核心工作流程

1. **理解查询** — 用户在寻找什么？化合物？基因？通路？专利？表达数据？经济指标？这决定了要访问哪些数据库。

2. **选择数据库** — 使用下方的数据库选择指南。如有疑问，查询多个数据库——广泛撒网总比遗漏相关数据要好。

3. **读取参考文件** — 每个数据库都有一个参考文件，位于`references/`目录下，包含端点详细信息、查询格式和示例调用。在调用API之前，请阅读相关文件。

4. **进行API调用** — 请参阅下方**进行API调用**部分，了解在你的平台上使用哪个HTTP获取工具。

5. **返回结果** — 始终返回：
   - 来自每个数据库的**原始JSON**响应
   - 已查询的**数据库列表**及使用的具体端点
   - 如果查询未返回结果，请明确说明，而不是省略该信息

## 数据库选择指南

将用户的意图与正确的数据库匹配。许多查询可以通过访问多个数据库受益。

### 物理学与天文学
| 用户询问内容... | 主要数据库 | 也可考虑 |
|---|---|---|
| 近地天体、小行星 | NASA (NeoWs) | — |
| 火星车图像 | NASA (Mars Rover Photos) | — |
| 系外行星、轨道参数 | NASA Exoplanet Archive | — |
| 按名称/坐标的天体 | SIMBAD | SDSS |
| 星系/恒星光谱、测光 | SDSS | SIMBAD |
| 物理常数 | NIST | — |
| 原子光谱、谱线 | NIST (ASD) | — |

### 地球与环境科学
| 用户询问内容... | 主要数据库 | 也可考虑 |
|---|---|---|
| 地震、地震事件 | USGS Earthquakes | — |
| 水文数据、径流、地下水 | USGS Water Services | — |
| 天气（当前、预报、历史） | OpenWeatherMap | NOAA |
| 气候数据、历史气象站 | NOAA (CDO) | — |
| 空气质量、有毒物质排放 | EPA (Envirofacts) | — |

### 化学与药物
| 用户询问内容... | 主要数据库 | 也可考虑 |
|---|---|---|
| 化合物、分子 | PubChem | ChEMBL |
| 分子性质（分子量、分子式、SMILES） | PubChem | — |
| 药物同义词、CAS号 | PubChem (synonyms) | DrugBank |
| 生物活性数据、IC50、结合测定 | ChEMBL | BindingDB, PubChem |
| 药物结合亲和力（Ki、IC50、Kd） | ChEMBL, BindingDB | PubChem |
| 药物-靶点相互作用 | ChEMBL, DrugBank | BindingDB, Open Targets |
| 针对蛋白质靶点的配体（按UniProt） | BindingDB | ChEMBL |
| 从化合物结构进行靶点识别 | BindingDB (SMILES相似性) | ChEMBL |
| 药品说明书、不良事件、召回 | FDA (OpenFDA) | DailyMed |
| 药品说明书（结构化产品标签） | DailyMed | FDA (OpenFDA) |
| 药物药理学、适应症 | DrugBank | FDA |
| 化学交叉引用 | PubChem (xrefs) | ChEMBL |
| 用于筛选的商业化化合物 | ZINC | PubChem |
| 相似性/子结构搜索（可购买） | ZINC | PubChem, ChEMBL |
| 类药化合物库、构建模块 | ZINC | — |
| FDA批准的药物结构 | ZINC (fda子集) | PubChem, FDA |
| 化合物可购买性、供应商目录 | ZINC | — |

### 材料科学与晶体学
| 用户询问内容... | 主要数据库 | 也可考虑 |
|---|---|---|
| 按分子式或元素查找材料 | Materials Project | COD |
| 带隙、电子结构 | Materials Project | — |
| 晶体结构、CIF文件 | COD | Materials Project |
| 弹性/力学性能 | Materials Project | — |
| 形成能、热力学 | Materials Project | — |
| 晶胞参数、空间群 | COD | Materials Project |

### 生物学与基因组学
| 用户询问内容... | 主要数据库 | 也可考虑 |
|---|---|---|
| 生物通路 | Reactome, KEGG | — |
| 某个基因/蛋白参与的通路 | Reactome (mapping), KEGG | — |
| 酶动力学、催化活性 | BRENDA | KEGG |
| 代谢组学研究、代谢物谱 | Metabolomics Workbench | PubChem |
| m/z或精确质量查询 | Metabolomics Workbench (moverz/exactmass) | PubChem |
| 蛋白质序列、功能、注释 | UniProt | Ensembl |
| 蛋白质-蛋白质相互作用 | STRING | BioGRID |
| 基因信息、基因组位置 | NCBI Gene | Ensembl |
| 基因组序列、变异、转录本 | Ensembl | NCBI Gene |
| 基因表达数据集 | GEO (NCBI E-utilities) | — |
| 跨组织基因表达 | GTEx | Human Protein Atlas |
| 基因表达特征（CMap/L1000） | LINCS L1000 | GEO |
| 针对GEO的基因集富集 | RummaGEO | GEO |
| 蛋白质序列（NCBI） | NCBI Protein | UniProt |
| 分类学分类 | NCBI Taxonomy | — |
| SNP/变异数据（dbSNP） | dbSNP | ClinVar, gnomAD |
| 群体变异频率 | gnomAD | dbSNP |
| 测序运行元数据 | SRA | ENA, GEO |
| 核苷酸序列（欧洲档案） | ENA | SRA, NCBI Gene |
| 基因组组装、原始读段（欧洲） | ENA | SRA, Ensembl |
| 序列登录号的交叉引用 | ENA (xref) | NCBI Gene, UniProt |
| 基因组注释、轨道 | UCSC Genome Browser | Ensembl |
| 3D蛋白质结构（实验） | PDB (RCSB) | EMDB |
| 3D蛋白质结构（预测） | AlphaFold DB | PDB |
| 电子显微镜图谱、cryo-EM结构 | EMDB | PDB |
| 蛋白质家族、结构域 | InterPro | UniProt |
| 化学实体（生物） | ChEBI | PubChem |
| 蛋白质/遗传相互作用 | BioGRID | STRING |
| 基因功能注释（GO术语） | QuickGO | Gene Ontology |
| 调控元件、ChIP-seq、ATAC-seq | ENCODE | — |
| 转录因子结合谱/基序 | JASPAR | ENCODE |
| 跨组织蛋白质表达 | Human Protein Atlas | UniProt |
| 单细胞图谱项目 | Human Cell Atlas | — |
| 蛋白质组学数据集 | PRIDE | — |
| 小鼠基因数据 | MouseMine | NCBI Gene |
| 质粒库 | Addgene | — |

**生物/物种很重要。** 大多数生物学数据库涵盖多种生物。如果用户的查询涉及特定生物，请明确传递——不要假设是人类。常见模式：Ensembl在URL路径中使用`{species}`（例如`homo_sapiens`），STRING/BioGRID/QuickGO使用NCBI分类ID（`species=9606`表示人类，`10090`表示小鼠），UniProt在搜索查询中使用`organism_id:9606`，KEGG使用生物代码（`hsa`、`mmu`）。GTEx和Human Protein Atlas仅限人类。请参考每个数据库的具体参数文件。

### 疾病与临床
| 用户询问内容... | 主要数据库 | 也可考虑 |
|---|---|---|
| 癌症体细胞突变 | COSMIC | Open Targets, cBioPortal |
| 癌症基因组学（TCGA） | GDC (TCGA) | COSMIC, cBioPortal |
| 癌症研究突变、CNA、表达 | cBioPortal | GDC (TCGA), COSMIC |
| 肿瘤临床数据（生存、分期） | cBioPortal | GDC (TCGA) |
| 药物-靶点-疾病关联 | Open Targets | ChEMBL |
| 基因-疾病关联 | DisGeNET | Open Targets, Monarch |
| 孟德尔疾病-基因关系 | OMIM | NCBI Gene |
| 变异临床意义 | ClinVar (NCBI) | OMIM |
| GWAS SNP-性状关联 | GWAS Catalog | — |
| 疾病-表型-基因链接 | Monarch Initiative | HPO |
| 表型本体、HPO术语 | HPO | Monarch |
| 药物基因组学、药物-基因相互作用 | ClinPGx (PharmGKB) | DrugBank |
| 针对药物/疾病的临床试验 | ClinicalTrials.gov | FDA |
| 疾病相关表达数据 | GEO | Open Targets |

### 专利与监管
| 用户询问内容... | 主要数据库 | 也可考虑 |
|---|---|---|
| 按关键词或技术查询专利 | USPTO (PatentsView) | — |
| 按发明人或受让人查询专利 | USPTO (PatentsView) | — |
| 专利审查状态 | USPTO (PEDS) | — |
| 商标查询 | USPTO (TSDR) | — |
| SEC公司文件、10-K、10-Q | SEC EDGAR | — |

### 经济学与金融
| 用户询问内容... | 主要数据库 | 也可考虑 |
|---|---|---|
| 美国经济时间序列（GDP、CPI、利率） | FRED | BEA |
| 就业、工资、劳动统计 | BLS | FRED |
| GDP、国民账户 | BEA | FRED, World Bank |
| 国际发展指标 | World Bank | FRED |
| 利率、货币供应 | Federal Reserve | FRED |
| 欧元汇率、欧洲央行货币统计 | ECB | — |
| 美国债务、收益率曲线、财政数据 | US Treasury | FRED |
| 股票价格、外汇、加密货币 | Alpha Vantage | — |
| 跨多主题统计数据 | Data Commons | — |

### 社会科学与人口统计
| 用户询问内容... | 主要数据库 | 也可考虑 |
|---|---|---|
| 美国人口、住房、收入数据 | US Census | Data Commons |
| 欧盟统计（经济、贸易、健康） | Eurostat | World Bank |
| 全球健康指标（死亡率、疾病） | WHO GHO | World Bank |

### 跨领域查询
| 用户询问内容... | 主要数据库 | 也可考虑 |
|---|---|---|
| 关于一个化合物的所有信息 | PubChem + ChEMBL + DrugBank | BindingDB, ZINC, Reactome, FDA |
| 关于一个基因的所有信息 | NCBI Gene + UniProt + Ensembl | Reactome, STRING, COSMIC, cBioPortal, ENA |
| 关于一个变异的所有信息 | dbSNP + ClinVar + gnomAD | GWAS Catalog, COSMIC, cBioPortal |
| 药物靶点通路 | ChEMBL + Reactome | Open Targets, GEO |
| 化学发明的现有技术 | USPTO + PubChem | ChEMBL |
| 关于一种材料的所有信息 | Materials Project + COD | — |
| 美国经济概览 | FRED + BLS + BEA | Federal Reserve |

当用户查询涉及多个领域时（例如“关于阿司匹林我们知道什么”或“查找BRCA1的所有信息”），并行查询所有相关数据库。

## 常见标识符格式

不同的数据库使用不同的标识符系统。如果查询失败，可能是标识符格式错误。以下是快速参考：

| 标识符 | 格式 | 示例 | 使用者 |
|---|---|---|---|
| UniProt登录号 | `P#####` 或 `Q#####` | `P04637` (TP53) | UniProt, STRING, AlphaFold, Reactome mapping |
| Ensembl基因ID | `ENSG###########` | `ENSG00000141510` | Ensembl, Open Targets, GTEx |
| NCBI基因ID | 整数 | `7157` (TP53) | NCBI Gene, GEO, DisGeNET, HPO |
| HGNC ID | `HGNC:#####` | `HGNC:11998` | Monarch |
| PubChem CID | 整数 | `2244` (阿司匹林) | PubChem |
| ZINC ID | `ZINC` + 15位数字 | `ZINC000000000053` (阿司匹林) | ZINC |
| ENA项目 | `PRJEB` + 数字 | `PRJEB40665` | ENA |
| ENA运行 | `ERR` + 数字 | `ERR1234567` | ENA |
| ENA实验 | `ERX` + 数字 | `ERX1234567` | ENA |
| ENA样本 | `ERS` + 数字 | `ERS1234567` | ENA |
| ChEMBL ID | `CHEMBL####` | `CHEMBL25` (阿司匹林) | ChEMBL |
| Reactome稳定ID | `R-HSA-######` | `R-HSA-109581` | Reactome |
| HP术语 | `HP:#######` | `HP:0001250` (癫痫发作) | HPO (URL编码冒号为%3A) |
| MONDO疾病 | `MONDO:#######` | `MONDO:0007947` | Monarch |
| GO术语 | `GO:#######` | `GO:0008150` | QuickGO, Gene Ontology |
| dbSNP rsID | `rs########` | `rs334` | dbSNP, GWAS Catalog, gnomAD |
| GENCODE ID | `ENSG###.##` (带版本) | `ENSG00000139618.17` | GTEx (需要版本后缀) |

### 标识符解析

当数据库无法识别标识符时，使用以下工作流程进行转换：

**基因**：符号（例如“TP53”）→ 在**NCBI Gene**中查找（按符号esearch）→ 获取NCBI基因ID → 通过**Ensembl** `/xrefs/symbol/homo_sapiens/{symbol}`转换为Ensembl ID，或通过**UniProt**搜索（`gene_exact:{symbol} AND organism_id:9606`）转换为UniProt登录号。

**化合物**：名称 → **PubChem** `/compound/name/{name}/cids/JSON` → 获取CID → 通过**UniChem**或**ChEMBL**分子搜索转换为ChEMBL ID。如果名称查找失败，尝试SMILES、InChIKey或CAS号。

**变异**：rsID（例如“rs334”）可直接在**dbSNP**、**ClinVar**、**GWAS Catalog**、**gnomAD**中使用。对于基因组坐标，使用**Ensembl** VEP获取后果注释和关联的rsID。

**疾病**：名称 → **Open Targets**或**Monarch**搜索 → 获取EFO或MONDO ID → 在后续查询中使用。

## 仅支持POST的API

以下数据库需要HTTP POST，**不能使用WebFetch**（仅GET）。请通过平台上的shell工具使用`curl`：

| 数据库 | 为何需要POST | 示例 |
|---|---|---|
| Open Targets | GraphQL端点 | `curl -X POST -H "Content-Type: application/json" -d '{"query":"..."}' https://api.platform.opentargets.org/api/v4/graphql` |
| gnomAD | GraphQL端点 | `curl -X POST -H "Content-Type: application/json" -d '{"query":"..."}' https://gnomad.broadinstitute.org/api` |
| RummaGEO | 仅POST的富集 | `curl -X POST -H "Content-Type: application/json" -d '{"genes":["..."]}' https://rummageo.com/api/enrich` |
| GDC/TCGA | 复杂的筛选查询 | `curl -X POST -H "Content-Type: application/json" -d '{"filters":...}' https://api.gdc.cancer.gov/ssms` |
| SEC EDGAR | 需要User-Agent头 | `curl -H "User-Agent: YourApp you@email.com" https://efts.sec.gov/LATEST/search-index?q=...` |

## API密钥和访问限制

某些数据库需要API密钥或存在访问限制。当需要API密钥时：

1. **首先检查当前环境** — 密钥可能已作为shell环境变量导出（例如`$FRED_API_KEY`）。直接从环境中读取。
2. **回退到`.env`文件** — 如果环境变量不存在，请检查当前工作目录中的`.env`文件。
3. **如果两者都没有** — 在没有密钥的情况下继续（大多数API在较低速率限制下仍可工作），并告诉用户缺少哪个密钥以及如何获取。

### 需要API密钥的数据库（免费注册）

| 数据库 | 环境变量 | 注册URL |
|---|---|---|
| FRED | `FRED_API_KEY` | https://fred.stlouisfed.org/docs/api/api_key.html |
| BEA | `BEA_API_KEY` | https://apps.bea.gov/API/signup/ |
| BLS | `BLS_API_KEY` | https://data.bls.gov/registrationEngine/ |
| NCBI（GEO、Gene） | `NCBI_API_KEY` | https://www.ncbi.nlm.nih.gov/account/settings/ |
| OpenFDA | `OPENFDA_API_KEY` | https://open.fda.gov/apis/authentication/ |
| USPTO（PatentsView） | `PATENTSVIEW_API_KEY` | https://patentsview.org/apis/keyrequest |
| Data Commons | `DATACOMMONS_API_KEY` | Google Cloud Console |
| Materials Project | `MP_API_KEY` | https://materialsproject.org（免费账户） |
| NASA | `NASA_API_KEY` | https://api.nasa.gov（免费，提供DEMO_KEY） |
| NOAA（CDO） | `NOAA_API_KEY` | https://www.ncdc.noaa.gov/cdo-web/token |
| OpenWeatherMap | `OPENWEATHERMAP_API_KEY` | https://openweathermap.org/appid |
| OMIM | `OMIM_API_KEY` | https://omim.org/api（免费学术） |
| BioGRID | `BIOGRID_API_KEY` | https://webservice.thebiogrid.org（免费） |
| Alpha Vantage | `ALPHAVANTAGE_API_KEY` | https://www.alphavantage.co/support/#api-key |
| US Census | `CENSUS_API_KEY` | https://api.census.gov/data/key_signup.html |
| DisGeNET | `DISGENET_API_KEY` | https://www.disgenet.org（免费学术） |
| Addgene | `ADDGENE_API_KEY` | https://www.addgene.org（免费账户） |
| LINCS L1000（CLUE） | `CLUE_API_KEY` | https://clue.io（免费学术） |

这些密钥均可免费获取。API在没有密钥的情况下也能工作，但速率限制较低。始终优先尝试使用密钥——如果环境变量未设置，则在不带密钥的情况下继续，并在响应中注明速率限制可能较低。

### 需要付费或受限访问的数据库

| 数据库 | 限制 | 免费替代方案 |
|---|---|---|
| DrugBank | 需要付费API许可 | 改用**ChEMBL** + **PubChem** + **OpenFDA** |
| COSMIC | 需要免费学术注册（JWT认证） | 使用**Open Targets**获取癌症突变数据 |
| BRENDA | 需要免费注册（SOAP，非REST） | 使用**KEGG**获取酶/通路数据 |

当数据库需要付费访问或用户尚未设置的注册时：
1. **回退到可以回答相同问题的免费替代方案**
2. **告诉用户**你无法访问哪个数据库、原因以及你改用了什么
3. 如果用户特别请求某个受限数据库，请解释访问要求，以便他们进行设置

### 加载API密钥

**步骤1 — 检查当前环境。** 密钥可能已作为shell变量导出。例如，在Claude Code中，你可以使用Bash检查：`echo $FRED_API_KEY`。如果变量已设置且非空，则使用它。

**步骤2 — 检查`.env`文件。** 如果环境变量未设置，从当前工作目录读取`.env`。格式：
```
FRED_API_KEY=your_key_here
BEA_API_KEY=your_key_here
```

**步骤3 — 不带密钥继续。** 如果两个来源都没有密钥，则在不带密钥的情况下继续（大多数API在较低速率限制下仍可工作），并向用户提及此事。

## 进行API调用

使用你所在环境的HTTP获取工具调用REST端点。工具名称因平台而异：

| 平台 | HTTP获取工具 | 回退方案 |
|---|---|---|
| Claude Code | `WebFetch` | 通过Bash使用`curl` |
| Gemini CLI | `web_fetch` | 通过shell使用`curl` |
| Windsurf | `read_url_content` | 通过终端使用`curl` |
| Cursor | 无专用获取工具 | 通过`run_terminal_cmd`使用`curl` |
| Codex CLI | 无专用获取工具 | 通过`shell`使用`curl` |
| Cline | 无专用获取工具 | 通过`execute_command`使用`curl` |

如果你无法识别你的平台或获取工具失败，请通过任何可用的shell/终端工具回退到`curl`。示例：
```bash
curl -s -H "Accept: application/json" "https://api.example.com/endpoint"
```

### 请求指南

- 在支持的地方设置`Accept: application/json`头
- 对查询参数中的特殊字符进行URL编码——SMILES字符串（`/`、`#`、`=`、`@`）、带括号的化合物名称以及带冒号的本体术语（`HP:0001250` → `HP%3A0001250`）是常见的失败源。使用`curl`时，为安全起见使用`--data-urlencode`。
- **并行OK**：当查询*不同*的数据库时（例如PubChem + ChEMBL + Reactome），可以并行运行——大多数API的速率限制较为宽松。
- **对速率限制的API进行串行请求**：NCBI API（Gene、GEO、Protein、Taxonomy、dbSNP、SRA）无密钥3次/秒，有密钥10次/秒。还需注意：Ensembl（15次/秒）、BLS v1（无密钥每天25次）、SEC EDGAR（10次/秒）、NOAA（有令牌5次/秒）。
- 如果遇到速率限制错误（HTTP 429或503），稍等片刻后重试一次。

### 错误恢复

如果API返回错误或空结果：
1. **检查标识符格式** — 使用上方的常见标识符格式表。基因符号可能需要先转换为NCBI基因ID或Ensembl ID。
2. **尝试替代标识符** — 如果化合物名称在PubChem中失败，尝试SMILES、InChIKey或CID。如果基因符号失败，尝试NCBI基因ID。
3. **尝试不同的数据库** — 如果某个数据库宕机或返回空，查看选择指南中的“也可考虑”列寻找替代品。
4. **报告失败** — 告诉用户哪个数据库失败、错误信息以及你尝试了什么替代方案。

### 分页

许多API返回分页结果——如果你只读取第一页，可能会遗漏数据。常见模式：

- **偏移/限制**：`offset=0&limit=100` → 将偏移量增加限制值以获取下一页（ChEMBL、FRED、NOAA、USGS、NCBI E-utilities、ENA、GDC、FDA）
- **基于游标**：响应包含`nextPageToken`或`cursor`值——在下一个请求中传递（ClinicalTrials.gov、UniProt）
- **页码**：`page=1&per_page=50` → 递增页码（World Bank、cBioPortal、ZINC）

请查看每个数据库参考文件中的具体分页参数。如果响应包含`total`、`totalCount`或`next`，并且返回的结果数小于总数，则还有更多页面。

对于定向查询（单个基因、单个化合物），第一页通常就足够了。当用户需要全面结果时（例如“所有关于X的临床试验”或“基因Y中的所有已知变异”），请进行分页。

## 输出格式

像这样组织你的响应：

```
## 查询的数据库
- **PubChem** — /compound/name/aspirin/property/...
- **Reactome** — /search/query?query=aspirin

## 结果

### PubChem
[原始JSON响应]

### Reactome
[原始JSON响应]
```

如果结果非常大，展示最相关的部分，并注明有更多数据可用。但默认显示完整的原始JSON——用户要求如此。

## 添加新数据库

此技能旨在不断扩展。每个数据库都是`references/`中的一个独立参考文件。要添加新数据库：

1. 创建`references/<数据库名称>.md`，遵循现有文件的相同格式
2. 在上方的数据库选择指南中添加一个条目
3. 参考文件应包括：基础URL、关键端点、查询参数格式、示例调用、速率限制和响应结构

## 可用数据库

在进行任何API调用之前，请阅读相关的参考文件。

### 物理学与天文学
| 数据库 | 参考文件 | 涵盖内容 |
|---|---|---|
| NASA | `references/nasa.md` | NEO小行星、火星车、APOD |
| NASA Exoplanet Archive | `references/nasa-exoplanet-archive.md` | 系外行星、轨道参数 |
| NIST | `references/nist.md` | 物理常数、原子光谱 |
| SDSS | `references/sdss.md` | 星系/恒星光谱、测光 |
| SIMBAD | `references/simbad.md` | 天体对象目录 |

### 地球与环境科学
| 数据库 | 参考文件 | 涵盖内容 |
|---|---|---|
| USGS | `references/usgs.md` | 地震、水文数据 |
| NOAA | `references/noaa.md` | 气候、气象站数据 |
| EPA | `references/epa.md` | 空气质量、有毒物质释放 |
| OpenWeatherMap | `references/openweathermap.md` | 当前/预报天气 |

### 化学与药物
| 数据库 | 参考文件 | 涵盖内容 |
|---|---|---|
| PubChem | `references/pubchem.md` | 化合物、性质、同义词 |
| ChEMBL | `references/chembl.md` | 生物活性、药物发现 |
| DrugBank | `references/drugbank.md` | 药物数据、相互作用（付费） |
| FDA (OpenFDA) | `references/fda.md` | 药品说明书、不良事件、召回 |
| DailyMed | `references/dailymed.md` | 药品说明书（NIH/NLM） |
| KEGG | `references/kegg.md` | 通路、基因、化合物 |
| ChEBI | `references/chebi.md` | 具有生物意义的化学实体 |
| ZINC | `references/zinc.md` | 商业化化合物、虚拟筛选 |
| BindingDB | `references/bindingdb.md` | 实验测量的结合亲和力 |

### 材料科学
| 数据库 | 参考文件 | 涵盖内容 |
|---|---|---|
| Materials Project | `references/materials-project.md` | 带隙、弹性性质、晶体结构 |
| COD | `references/cod.md` | 晶体结构、CIF文件 |

### 生物学与基因组学
| 数据库 | 参考文件 | 涵盖内容 |
|---|---|---|
| Reactome | `references/reactome.md` | 生物学通路、反应 |
| BRENDA | `references/brenda.md` | 酶动力学、催化（SOAP） |
| UniProt | `references/uniprot.md` | 蛋白质序列、功能 |
| STRING | `references/string.md` | 蛋白质-蛋白质相互作用 |
| Ensembl | `references/ensembl.md` | 基因组、变异、序列 |
| NCBI Gene | `references/ncbi-gene.md` | 基因信息、链接 |
| NCBI Protein | `references/ncbi-protein.md` | 蛋白质序列、记录 |
| NCBI Taxonomy | `references/ncbi-taxonomy.md` | 分类学分类 |
| GEO (NCBI) | `references/geo.md` | 基因表达数据集 |
| GTEx | `references/gtex.md` | 跨组织基因表达 |
| PDB | `references/pdb.md` | 蛋白质3D结构 |
| AlphaFold DB | `references/alphafold.md` | 预测的蛋白质结构 |
| EMDB | `references/emdb.md` | 电子显微镜图谱 |
| InterPro | `references/interpro.md` | 蛋白质家族、结构域 |
| BioGRID | `references/biogrid.md` | 蛋白质/遗传相互作用 |
| Gene Ontology | `references/gene-ontology.md` | GO术语、基因注释 |
| QuickGO | `references/quickgo.md` | GO注释（EBI，推荐） |
| dbSNP | `references/dbsnp.md` | SNP/变异数据 |
| SRA | `references/sra.md` | 测序运行元数据 |
| gnomAD | `references/gnomad.md` | 群体变异频率（POST） |
| UCSC Genome Browser | `references/ucsc-genome.md` | 基因组注释、轨道 |
| ENCODE | `references/encode.md` | DNA元件、ChIP-seq、ATAC-seq |
| JASPAR | `references/jaspar.md` | 转录因子结合谱/基序 |
| Human Protein Atlas | `references/human-protein-atlas.md` | 跨组织蛋白质表达 |
| Human Cell Atlas | `references/hca.md` | 单细胞图谱数据 |
| LINCS L1000 | `references/lincs-l1000.md` | 基因表达特征（CMap） |
| RummaGEO | `references/rummageo.md` | GEO基因集富集（POST） |
| PRIDE | `references/pride.md` | 蛋白质组学数据存储库 |
| Metabolomics Workbench | `references/metabolomics-workbench.md` | 代谢组学研究、代谢物 |
| MouseMine | `references/mousemine.md` | 小鼠基因组信息学 |
| ENA | `references/ena.md` | 核苷酸序列、读段、组装、分类（EMBL-EBI） |
| Addgene | `references/addgene.md` | 质粒库 |

### 疾病与临床
| 数据库 | 参考文件 | 涵盖内容 |
|---|---|---|
| Open Targets | `references/opentargets.md` | 靶点-疾病关联（POST） |
| COSMIC | `references/cosmic.md` | 癌症体细胞突变 |
| ClinPGx (PharmGKB) | `references/clinpgx.md` | 药物基因组学 |
| ClinicalTrials.gov | `references/clinicaltrials.md` | 临床试验注册处 |
| OMIM | `references/omim.md` | 孟德尔疾病-基因数据 |
| ClinVar | `references/clinvar.md` | 变异临床意义 |
| GDC (TCGA) | `references/tcga-gdc.md` | 癌症基因组学、突变（POST） |
| cBioPortal | `references/cbioportal.md` | 癌症研究突变、CNA、表达、临床数据 |
| DisGeNET | `references/disgenet.md` | 基因-疾病关联 |
| GWAS Catalog | `references/gwas-catalog.md` | GWAS SNP-性状关联 |
| Monarch Initiative | `references/monarch.md` | 疾病-表型-基因链接 |
| HPO | `references/hpo.md` | 人类表型本体 |

### 专利与监管
| 数据库 | 参考文件 | 涵盖内容 |
|---|---|---|
| USPTO | `references/uspto.md` | 专利、商标 |
| SEC EDGAR | `references/sec-edgar.md` | 公司文件（需要User-Agent头） |

### 经济学与金融
| 数据库 | 参考文件 | 涵盖内容 |
|---|---|---|
| FRED | `references/fred.md` | 美国经济时间序列 |
| Federal Reserve | `references/federal-reserve.md` | 货币/金融数据 |
| BEA | `references/bea.md` | GDP、国民账户 |
| BLS | `references/bls.md` | 就业、工资、CPI |
| World Bank | `references/worldbank.md` | 发展指标 |
| ECB | `references/ecb.md` | 欧元汇率、货币统计 |
| US Treasury | `references/treasury.md` | 债务、收益率曲线、财政数据 |
| Alpha Vantage | `references/alphavantage.md` | 股票、外汇、加密货币 |
| Data Commons | `references/datacommons.md` | 统计知识图谱 |

### 社会科学与人口统计
| 数据库 | 参考文件 | 涵盖内容 |
|---|---|---|
| US Census | `references/census.md` | 人口、住房、经济调查 |
| Eurostat | `references/eurostat.md` | 欧盟统计 |
| WHO GHO | `references/who.md` | 全球健康指标 |
