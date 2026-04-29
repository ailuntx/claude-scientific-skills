# gget 模块参考

所有 gget 模块的完整参数参考。

## 参考与基因信息模块

### gget ref
检索 Ensembl 参考基因组 FTP 链接及元数据。

**参数：**
| 参数 | 类型 | 描述 | 默认值 |
|-----------|------|-------------|---------|
| `species` | str | 物种（Genus_species 格式或简写：'human', 'mouse'） | 必填 |
| `-w/--which` | str | 返回文件类型：gtf, cdna, dna, cds, cdrna, pep | 全部 |
| `-r/--release` | int | Ensembl 版本号 | 最新版 |
| `-od/--out_dir` | str | 输出目录路径 | 无 |
| `-o/--out` | str | 结果 JSON 文件路径 | 无 |
| `-l/--list_species` | flag | 列出可用脊椎动物物种 | 关闭 |
| `-liv/--list_iv_species` | flag | 列出可用无脊椎动物物种 | 关闭 |
| `-ftp` | flag | 仅返回 FTP 链接 | 关闭 |
| `-d/--download` | flag | 下载文件（需 curl） | 关闭 |
| `-q/--quiet` | flag | 隐藏进度信息 | 关闭 |

**返回：** 包含 FTP 链接、Ensembl 版本号、发布日期、文件大小的 JSON

---

### gget search
在 Ensembl 中按名称或描述搜索基因。

**参数：**
| 参数 | 类型 | 描述 | 默认值 |
|-----------|------|-------------|---------|
| `searchwords` | str/list | 搜索词（不区分大小写） | 必填 |
| `-s/--species` | str | 目标物种或核心数据库名 | 必填 |
| `-r/--release` | int | Ensembl 版本号 | 最新版 |
| `-t/--id_type` | str | 返回 'gene' 或 'transcript' | 'gene' |
| `-ao/--andor` | str | 'or'（任意词）或 'and'（所有词） | 'or' |
| `-l/--limit` | int | 返回结果上限 | 无 |
| `-o/--out` | str | 输出文件路径（CSV/JSON） | 无 |

**返回：** ensembl_id, gene_name, ensembl_description, ext_ref_description, biotype, URL

---

### gget info
从 Ensembl、UniProt 和 NCBI 获取全面的基因/转录本元数据。

**参数：**
| 参数 | 类型 | 描述 | 默认值 |
|-----------|------|-------------|---------|
| `ens_ids` | str/list | Ensembl ID（支持 WormBase、Flybase） | 必填 |
| `-o/--out` | str | 输出文件路径（CSV/JSON） | 无 |
| `-n/--ncbi` | bool | 禁用 NCBI 数据检索 | 关闭 |
| `-u/--uniprot` | bool | 禁用 UniProt 数据检索 | 关闭 |
| `-pdb` | bool | 包含 PDB 标识符 | 关闭 |
| `-csv` | flag | 返回 CSV 格式（CLI） | 关闭 |
| `-q/--quiet` | flag | 隐藏进度显示 | 关闭 |

**Python 专属：**
- `save=True`：保存输出至当前目录
- `wrap_text=True`：用自动换行格式化数据框

**注意：** 同时处理超过 1000 个 ID 可能导致服务器错误

**返回：** UniProt ID, NCBI 基因 ID, 基因名, 别名, 蛋白名, 描述, 生物类型, 典型转录本

---

### gget seq
以 FASTA 格式检索核苷酸或氨基酸序列。

**参数：**
| 参数 | 类型 | 描述 | 默认值 |
|-----------|------|-------------|---------|
| `ens_ids` | str/list | Ensembl 标识符 | 必填 |
| `-o/--out` | str | 输出文件路径 | 标准输出 |
| `-t/--translate` | flag | 获取氨基酸序列 | 关闭 |
| `-iso/--isoforms` | flag | 返回所有转录变体 | 关闭 |
| `-q/--quiet` | flag | 隐藏进度信息 | 关闭 |

**数据源：** Ensembl（核苷酸），UniProt（氨基酸）

**返回：** FASTA 格式序列

---

## 序列分析与比对模块

### gget blast
对标准数据库进行 BLAST 序列比对。

**参数：**
| 参数 | 类型 | 描述 | 默认值 |
|-----------|------|-------------|---------|
| `sequence` | str | 序列或 FASTA/.txt 文件路径 | 必填 |
| `-p/--program` | str | blastn, blastp, blastx, tblastn, tblastx | 自动检测 |
| `-db/--database` | str | nt, refseq_rna, pdbnt, nr, swissprot, pdbaa, refseq_protein | nt 或 nr |
| `-l/--limit` | int | 返回结果上限 | 50 |
| `-e/--expect` | float | E 值阈值 | 10.0 |
| `-lcf/--low_comp_filt` | flag | 启用低复杂度过滤 | 关闭 |
| `-mbo/--megablast_off` | flag | 禁用 MegaBLAST（仅 blastn） | 关闭 |
| `-o/--out` | str | 输出文件路径 | 无 |
| `-q/--quiet` | flag | 隐藏进度 | 关闭 |

**返回：** 描述, 学名, 俗名, 分类号, 最高分, 总分, 查询覆盖率

---

### gget blat
使用 UCSC BLAT 定位基因组位置。

**参数：**
| 参数 | 类型 | 描述 | 默认值 |
|-----------|------|-------------|---------|
| `sequence` | str | 序列或 FASTA/.txt 文件路径 | 必填 |
| `-st/--seqtype` | str | 'DNA', 'protein', 'translated%20RNA', 'translated%20DNA' | 自动检测 |
| `-a/--assembly` | str | 目标基因组版本（hg38, mm39, taeGut2 等） | 'human'/hg38 |
| `-o/--out` | str | 输出文件路径 | 无 |
| `-csv` | flag | 返回 CSV 格式（CLI） | 关闭 |
| `-q/--quiet` | flag | 隐藏进度 | 关闭 |

**返回：** 基因组, 查询长度, 比对起止位点, 匹配数, 错配数, 比对百分比

---

### gget muscle
使用 Muscle5 进行多序列比对。

**参数：**
| 参数 | 类型 | 描述 | 默认值 |
|-----------|------|-------------|---------|
| `fasta` | str/list | 序列或 FASTA 文件路径 | 必填 |
| `-o/--out` | str | 输出文件路径 | 标准输出 |
| `-s5/--super5` | flag | 使用 Super5 算法（大型数据集更快） | 关闭 |
| `-q/--quiet` | flag | 隐藏进度 | 关闭 |

**返回：** ClustalW 格式比对结果或对齐 FASTA（.afa）

---

### gget diamond
快速本地蛋白/翻译 DNA 比对。

**参数：**
| 参数 | 类型 | 描述 | 默认值 |
|-----------|------|-------------|---------|
| `query` | str/list | 查询序列或 FASTA 文件 | 必填 |
| `--reference` | str/list | 参考序列或 FASTA 文件 | 必填 |
| `--sensitivity` | str | fast, mid-sensitive, sensitive, more-sensitive, very-sensitive, ultra-sensitive | very-sensitive |
| `--threads` | int | CPU 线程数 | 1 |
| `--diamond_binary` | str | DIAMOND 安装路径 | 自动检测 |
| `--diamond_db` | str | 保存数据库供复用 | 无 |
| `--translated` | flag | 启用核苷酸到氨基酸比对 | 关闭 |
| `-o/--out` | str | 输出文件路径 | 无 |
| `-csv` | flag | CSV 格式（CLI） | 关闭 |
| `-q/--quiet` | flag | 隐藏进度 | 关闭 |

**返回：** 相似度百分比, 序列长度, 匹配位置, 缺口开放数, E 值, 比特得分

---

## 结构与蛋白分析模块

### gget pdb
查询 RCSB 蛋白质数据库。

**参数：**
| 参数 | 类型 | 描述 | 默认值 |
|-----------|------|-------------|---------|
| `pdb_id` | str | PDB 标识符（如 '7S7U'） | 必填 |
| `-r/--resource` | str | pdb, entry, pubmed, assembly, entity types | 'pdb' |
| `-i/--identifier` | str | 组装体、实体或链 ID | 无 |
| `-o/--out` | str | 输出文件路径 | 标准输出 |

**返回：** PDB 格式（结构）或 JSON（元数据）

---

### gget alphafold
使用 AlphaFold2 预测 3D 蛋白结构。

**配置：** 需 OpenMM 及 `gget setup alphafold`（约 4GB 下载）

**参数：**
| 参数 | 类型 | 描述 | 默认值 |
|-----------|------|-------------|---------|
| `sequence` | str/list | 氨基酸序列或 FASTA 文件 | 必填 |
| `-mr/--multimer_recycles` | int | 多聚体重循环次数 | 3 |
| `-o/--out` | str | 输出文件夹路径 | 时间戳命名 |
| `-mfm/--multimer_for_monomer` | flag | 对单体应用多聚体模型 | 关闭 |
| `-r/--relax` | flag | 对最优模型进行 AMBER 松弛 | 关闭 |
| `-q/--quiet` | flag | 隐藏进度 | 关闭 |

**Python 专属：**
- `plot` (bool)：生成 3D 可视化（默认：开启）
- `show_sidechains` (bool)：包含侧链（默认：开启）

**注意：** 多序列自动触发多聚体建模

**返回：** PDB 结构文件, JSON 比对误差数据, 可选 3D 图

---

### gget elm
预测真核生物线性基序。

**配置：** 需 `gget setup elm`

**参数：**
| 参数 | 类型 | 描述 | 默认值 |
|-----------|------|-------------|---------|
| `sequence` | str | 氨基酸序列或 UniProt 编号 | 必填 |
| `-s/--sensitivity` | str | DIAMOND 比对敏感度 | very-sensitive |
| `-t/--threads` | int | 线程数 | 1 |
| `-bin/--diamond_binary` | str | DIAMOND 二进制路径 | 自动检测 |
| `-o/--out` | str | 输出目录路径 | 无 |
| `-u/--uniprot` | flag | 输入为 UniProt 编号 | 关闭 |
| `-e/--expand` | flag | 包含蛋白名、生物体、参考文献 | 关闭 |
| `-csv` | flag | CSV 格式（CLI） | 关闭 |
| `-q/--quiet` | flag | 隐藏进度 | 关闭 |

**返回：** 双输出：
1. **ortholog_df**：直系同源蛋白基序
2. **regex_df**：输入序列匹配基序

---

## 表达与疾病数据模块

### gget archs4
查询 ARCHS4 获取基因相关性或组织表达数据。

**参数：**
| 参数 | 类型 | 描述 | 默认值 |
|-----------|------|-------------|---------|
| `gene` | str | 基因符号或 Ensembl ID | 必填 |
| `-w/--which` | str | 'correlation' 或 'tissue' | 'correlation' |
| `-s/--species` | str | 'human' 或 'mouse'（仅 tissue） | 'human' |
| `-o/--out` | str | 输出文件路径 | 无 |
| `-e/--ensembl` | flag | 输入为 Ensembl ID | 关闭 |
| `-csv` | flag | CSV 格式（CLI） | 关闭 |
| `-q/--quiet` | flag | 隐藏进度 | 关闭 |

**返回：**
- **correlation**：基因符号, 皮尔逊相关系数（前 100）
- **tissue**：组织 ID, 最小/Q1/中位数/Q3/最大表达量

---

### gget cellxgene
查询 CZ CELLxGENE Discover Census 单细胞数据库。

**配置：** 需 `gget setup cellxgene`

**参数：**
| 参数 | 类型 | 描述 | 默认值 |
|-----------|------|-------------|---------|
| `--gene` (-g) | list | 基因名或 Ensembl ID（区分大小写！） | 必填 |
| `--tissue` | list | 组织类型 | 无 |
| `--cell_type` | list | 细胞类型 | 无 |
| `--species` (-s) | str | 'homo_sapiens' 或 'mus_musculus' | 'homo_sapiens' |
| `--census_version` (-cv) | str | "stable", "latest" 或日期版本 | "stable" |
| `-o/--out` | str | 输出文件路径（CLI 必填） | 必填 |
| `--ensembl` (-e) | flag | 使用 Ensembl ID | 关闭 |
| `--meta_only` (-mo) | flag | 仅返回元数据 | 关闭 |
| `-q/--quiet` | flag | 隐藏进度 | 关闭 |

**附加筛选器：** disease, development_stage, sex, assay, dataset_id, donor_id, ethnicity, suspension_type

**重要：** 基因符号区分大小写（人类用 'PAX7'，小鼠用 'Pax7'）

**返回：** 包含计数矩阵和元数据的 AnnData 对象

---

### gget enrichr
使用 Enrichr/modEnrichr 进行富集分析。

**参数：**
| 参数 | 类型 | 描述 | 默认值 |
|-----------|------|-------------|---------|
| `genes` | list | 基因符号或 Ensembl ID | 必填 |
| `-db/--database` | str | 参考数据库或简写 | 必填 |
| `-s/--species` | str | human, mouse, fly, yeast, worm, fish | 'human' |
| `-bkg_l/--background_list` | list | 背景基因集 | 无 |
| `-o/--out` | str | 输出文件路径 | 无 |
| `-ko/--kegg_out` | str | KEGG 通路图像目录 | 无 |

**Python 专属：**
- `plot` (bool)：生成图形结果

**数据库简写：**
- 'pathway' → KEGG_2021_Human
- 'transcription' → ChEA_2016
- 'ontology' → GO_Biological_Process_2021
- 'diseases_drugs' → GWAS_Catalog_2019
- 'celltypes' → PanglaoDB_Augmented_2021

**返回：** 通路/功能关联（含校正 p 值），重叠基因计数

---

### gget bgee
从 Bgee 检索直系同源和表达数据。

**参数：**
| 参数 | 类型 | 描述 | 默认值 |
|-----------|------|-------------|---------|
| `ens_id` | str/list | Ensembl 或 NCBI 基因 ID | 必填 |
| `-t/--type` | str | 'orthologs' 或 'expression' | 'orthologs' |
| `-o/--out` | str | 输出文件路径 | 无 |
| `-csv` | flag | CSV 格式（CLI） | 关闭 |
| `-q/--quiet` | flag | 隐藏进度 |

**绘图参数：**
| 参数 | 类型 | 描述 | 默认值 |
|-----------|------|-------------|---------|
| `-s/--study_ids` | 列表 | cBioPortal研究ID | 必填 |
| `-g/--genes` | 列表 | 基因名称或Ensembl ID | 必填 |
| `-st/--stratification` | 字符串 | 组织、癌症类型、详细癌症类型、研究ID、样本 | None |
| `-vt/--variation_type` | 字符串 | 突变发生数、非二元拷贝数变异、结构变异发生数、拷贝数变异发生数、变异效应 | None |
| `-f/--filter` | 字符串 | 按列值筛选（例如'study_id:msk_impact_2017'） | None |
| `-dd/--data_dir` | 字符串 | 缓存目录 | ./gget_cbio_cache |
| `-fd/--figure_dir` | 字符串 | 输出目录 | ./gget_cbio_figures |
| `-t/--title` | 字符串 | 自定义图表标题 | None |
| `-dpi` | 整数 | 分辨率 | 100 |
| `-q/--quiet` | 标志 | 隐藏进度 | False |
| `-nc/--no_confirm` | 标志 | 跳过下载确认 | False |
| `-sh/--show` | 标志 | 在窗口中显示图表 | False |

**返回：** PNG热图

---

### gget cosmic
在COSMIC数据库中检索癌症突变。

**重要：** 商业用途需支付许可费。需要COSMIC账户。

**查询参数：**
| 参数 | 类型 | 描述 | 默认值 |
|-----------|------|-------------|---------|
| `searchterm` | 字符串 | 基因名称、Ensembl ID、突变、样本ID | 必填 |
| `-ctp/--cosmic_tsv_path` | 字符串 | COSMIC TSV文件路径 | 必填 |
| `-l/--limit` | 整数 | 最大结果数 | 100 |
| `-csv` | 标志 | CSV格式（命令行） | False |

**下载参数：**
| 参数 | 类型 | 描述 | 默认值 |
|-----------|------|-------------|---------|
| `-d/--download_cosmic` | 标志 | 启用下载模式 | False |
| `-gm/--gget_mutate` | 标志 | 创建gget mutate专用版本 | False |
| `-cp/--cosmic_project` | 字符串 | 癌症、普查、细胞系、耐药性、基因组筛查、靶向筛查 | None |
| `-cv/--cosmic_version` | 字符串 | COSMIC版本 | 最新版 |
| `-gv/--grch_version` | 整数 | 人类参考基因组（37或38） | None |
| `--email` | 字符串 | COSMIC账户邮箱 | 必填 |
| `--password` | 字符串 | COSMIC账户密码 | 必填 |

**注意：** 首次使用需下载数据库

**返回：** COSMIC突变数据

---

## 附加工具

### gget mutate
生成突变核苷酸序列。

**参数：**
| 参数 | 类型 | 描述 | 默认值 |
|-----------|------|-------------|---------|
| `sequences` | 字符串/列表 | FASTA文件或序列 | 必填 |
| `-m/--mutations` | 字符串/数据框 | CSV/TSV文件或DataFrame | 必填 |
| `-mc/--mut_column` | 字符串 | 突变列名称 | 'mutation' |
| `-sic/--seq_id_column` | 字符串 | 序列ID列 | 'seq_ID' |
| `-mic/--mut_id_column` | 字符串 | 突变ID列 | None |
| `-k/--k` | 整数 | 侧翼序列长度 | 30 |
| `-o/--out` | 字符串 | 输出FASTA文件路径 | stdout |
| `-q/--quiet` | 标志 | 隐藏进度 | False |

**返回：** FASTA格式的突变序列

---

### gget gpt
使用OpenAI API生成文本。

**配置：** 需运行`gget setup gpt`并提供OpenAI API密钥

**参数：**
| 参数 | 类型 | 描述 | 默认值 |
|-----------|------|-------------|---------|
| `prompt` | 字符串 | 生成文本的输入 | 必填 |
| `api_key` | 字符串 | OpenAI API密钥 | 必填 |
| `model` | 字符串 | OpenAI模型名称 | gpt-3.5-turbo |
| `temperature` | 浮点数 | 采样温度（0-2） | 1.0 |
| `top_p` | 浮点数 | 核心采样 | 1.0 |
| `max_tokens` | 整数 | 最大生成标记数 | None |
| `frequency_penalty` | 浮点数 | 频率惩罚（0-2） | 0 |
| `presence_penalty` | 浮点数 | 存在惩罚（0-2） | 0 |

**重要：** 免费层限用3个月。请设置账单限额。

**返回：** 生成的文本字符串

---

### gget setup
安装/下载模块依赖项。

**参数：**
| 参数 | 类型 | 描述 | 默认值 |
|-----------|------|-------------|---------|
| `module` | 字符串 | 模块名称 | 必填 |
| `-o/--out` | 字符串 | 输出文件夹（仅elm模块） | 包安装目录 |
| `-q/--quiet` | 标志 | 隐藏进度 | False |

**需配置的模块：**
- `alphafold` - 下载约4GB模型参数
- `cellxgene` - 安装cellxgene-census
- `elm` - 下载本地ELM数据库
- `gpt` - 配置OpenAI集成

**返回：** 无（安装依赖项）
