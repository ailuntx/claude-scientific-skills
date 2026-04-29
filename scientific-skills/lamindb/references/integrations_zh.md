# LaminDB 集成方案

本文档涵盖 LaminDB 与工作流管理器、MLOps 平台、可视化工具及其他第三方系统的集成方案。

## 概述

LaminDB 支持跨数据存储、计算工作流、机器学习平台和可视化工具的广泛集成，可无缝融入现有数据科学与生物信息学流程。

## 数据存储集成

### 本地文件系统

```python
import lamindb as ln

# 使用本地存储初始化
lamin init --storage ./mydata

# 保存文件到本地存储
artifact = ln.Artifact("data.csv", key="local/data.csv").save()

# 从本地存储加载
data = artifact.load()
```

### AWS S3

```python
# 使用 S3 存储初始化
lamin init --storage s3://my-bucket/path \
  --db postgresql://user:pwd@host:port/db

# 文件自动同步至 S3
artifact = ln.Artifact("data.csv", key="experiments/data.csv").save()

# 透明化 S3 访问
data = artifact.load()  # 若未缓存则从 S3 下载
```

### S3 兼容服务

支持 MinIO、Cloudflare R2 等 S3 兼容端点：

```python
# 使用自定义 S3 端点初始化
lamin init --storage 's3://bucket?endpoint_url=http://minio.example.com:9000'

# 配置凭证
export AWS_ACCESS_KEY_ID=minioadmin
export AWS_SECRET_ACCESS_KEY=minioadmin
```

### Google 云存储

```python
# 安装 GCP 扩展
pip install 'lamindb[gcp]'

# 使用 GCS 初始化
lamin init --storage gs://my-bucket/path \
  --db postgresql://user:pwd@host:port/db

# 文件同步至 GCS
artifact = ln.Artifact("data.csv", key="experiments/data.csv").save()
```

### HTTP/HTTPS (只读)

```python
# 无需复制直接访问远程文件
artifact = ln.Artifact(
    "https://example.com/data.csv",
    key="remote/data.csv"
).save()

# 流式读取远程内容
with artifact.open() as f:
    data = f.read()
```

### HuggingFace 数据集

```python
# 访问 HuggingFace 数据集
from datasets import load_dataset

dataset = load_dataset("squad", split="train")

# 注册为 LaminDB 文件
artifact = ln.Artifact.from_dataframe(
    dataset.to_pandas(),
    key="hf/squad_train.parquet",
    description="来自 HuggingFace 的 SQuAD 训练数据"
).save()
```

## 工作流管理器集成

### Nextflow

追踪 Nextflow 流水线执行与输出：

```python
# 在 Nextflow 流程脚本中
import lamindb as ln

# 初始化追踪
ln.track()

# Nextflow 流程逻辑
input_artifact = ln.Artifact.get(key="${input_key}")
data = input_artifact.load()

# 处理数据
result = process_data(data)

# 保存输出
output_artifact = ln.Artifact.from_dataframe(
    result,
    key="${output_key}"
).save()

ln.finish()
```

**Nextflow 配置示例：**
```nextflow
process ANALYZE {
    input:
    val input_key

    output:
    path "result.csv"

    script:
    """
    #!/usr/bin/env python
    import lamindb as ln
    ln.track()
    artifact = ln.Artifact.get(key="${input_key}")
    # 处理并保存
    ln.finish()
    """
}
```

### Snakemake

集成 LaminDB 至 Snakemake 工作流：

```python
# 在 Snakemake 规则中
rule process_data:
    input:
        "data/input.csv"
    output:
        "data/output.csv"
    run:
        import lamindb as ln

        ln.track()

        # 加载输入文件
        artifact = ln.Artifact.get(key="inputs/data.csv")
        data = artifact.load()

        # 处理
        result = analyze(data)

        # 保存输出
        result.to_csv(output[0])
        ln.Artifact(output[0], key="outputs/result.csv").save()

        ln.finish()
```

### Redun

追踪 Redun 任务执行：

```python
from redun import task
import lamindb as ln

@task()
@ln.tracked()
def process_dataset(input_key: str, output_key: str):
    """使用 LaminDB 追踪的 Redun 任务"""
    # 加载输入
    artifact = ln.Artifact.get(key=input_key)
    data = artifact.load()

    # 处理
    result = transform(data)

    # 保存输出
    ln.Artifact.from_dataframe(result, key=output_key).save()

    return output_key

# Redun 自动与 LaminDB 协同追踪谱系
```

## MLOps 平台集成

### Weights & Biases (W&B)

结合 W&B 实验追踪与 LaminDB 数据管理：

```python
import wandb
import lamindb as ln

# 初始化双平台
wandb.init(project="my-project", name="experiment-1")
ln.track(params={"learning_rate": 0.01, "batch_size": 32})

# 加载训练数据
train_artifact = ln.Artifact.get(key="datasets/train.parquet")
train_data = train_artifact.load()

# 训练模型
model = train_model(train_data)

# 记录至 W&B
wandb.log({"accuracy": 0.95, "loss": 0.05})

# 保存模型至 LaminDB
import joblib
joblib.dump(model, "model.pkl")
model_artifact = ln.Artifact(
    "model.pkl",
    key="models/experiment-1.pkl",
    description=f"来自 W&B 运行 {wandb.run.id} 的模型"
).save()

# 关联 W&B 运行 ID
model_artifact.features.add_values({"wandb_run_id": wandb.run.id})

ln.finish()
wandb.finish()
```

### MLflow

集成 MLflow 模型追踪与 LaminDB：

```python
import mlflow
import lamindb as ln

# 启动运行
mlflow.start_run()
ln.track()

# 向双平台记录参数
params = {"max_depth": 5, "n_estimators": 100}
mlflow.log_params(params)
ln.context.params = params

# 从 LaminDB 加载数据
data_artifact = ln.Artifact.get(key="datasets/features.parquet")
X = data_artifact.load()

# 训练并记录模型
model = train_model(X)
mlflow.sklearn.log_model(model, "model")

# 保存至 LaminDB
import joblib
joblib.dump(model, "model.pkl")
model_artifact = ln.Artifact(
    "model.pkl",
    key=f"models/{mlflow.active_run().info.run_id}.pkl"
).save()

mlflow.end_run()
ln.finish()
```

### HuggingFace Transformers

使用 LaminDB 追踪模型微调：

```python
from transformers import Trainer, TrainingArguments
import lamindb as ln

ln.track(params={"model": "bert-base", "epochs": 3})

# 加载训练数据
train_artifact = ln.Artifact.get(key="datasets/train_tokenized.parquet")
train_dataset = train_artifact.load()

# 配置训练器
training_args = TrainingArguments(
    output_dir="./results",
    num_train_epochs=3,
)

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=train_dataset,
)

# 训练
trainer.train()

# 保存模型至 LaminDB
trainer.save_model("./model")
model_artifact = ln.Artifact(
    "./model",
    key="models/bert_finetuned",
    description="在自定义数据集上微调的 BERT 模型"
).save()

ln.finish()
```

### scVI-tools

使用 scVI 与 LaminDB 进行单细胞分析：

```python
import scvi
import lamindb as ln

ln.track()

# 加载数据
adata_artifact = ln.Artifact.get(key="scrna/raw_counts.h5ad")
adata = adata_artifact.load()

# 配置 scVI
scvi.model.SCVI.setup_anndata(adata, layer="counts")

# 训练模型
model = scvi.model.SCVI(adata)
model.train()

# 保存潜在表示
adata.obsm["X_scvi"] = model.get_latent_representation()

# 保存结果
result_artifact = ln.Artifact.from_anndata(
    adata,
    key="scrna/scvi_latent.h5ad",
    description="scVI 潜在表示"
).save()

ln.finish()
```

## 数组存储集成

### TileDB-SOMA

支持 cellxgene 的可扩展数组存储：

```python
import tiledbsoma as soma
import lamindb as ln

# 创建 SOMA 实验
uri = "tiledb://my-namespace/experiment"

with soma.Experiment.create(uri) as exp:
    # 添加测量值
    exp.add_new_collection("RNA")

    # 在 LaminDB 中注册
    artifact = ln.Artifact(
        uri,
        key="cellxgene/experiment.soma",
        description="TileDB-SOMA 实验"
    ).save()

# 使用 SOMA 查询
with soma.Experiment.open(uri) as exp:
    obs = exp.obs.read().to_pandas()
```

### DuckDB

使用 DuckDB 查询文件：

```python
import duckdb
import lamindb as ln

# 获取文件
artifact = ln.Artifact.get(key="datasets/large_data.parquet")

# 使用 DuckDB 查询（无需加载完整文件）
path = artifact.cache()
result = duckdb.query(f"""
    SELECT cell_type, COUNT(*) as count
    FROM read_parquet('{path}')
    GROUP BY cell_type
    ORDER BY count DESC
""").to_df()

# 保存查询结果
result_artifact = ln.Artifact.from_dataframe(
    result,
    key="analysis/cell_type_counts.parquet"
).save()
```

## 可视化集成

### Vitessce

创建交互式可视化：

```python
from vitessce import VitessceConfig
import lamindb as ln

# 加载空间数据
artifact = ln.Artifact.get(key="spatial/visium_slide.h5ad")
adata = artifact.load()

# 创建 Vitessce 配置
vc = VitessceConfig.from_object(adata)

# 保存配置
import json
config_file = "vitessce_config.json"
with open(config_file, "w") as f:
    json.dump(vc.to_dict(), f)

# 注册配置
config_artifact = ln.Artifact(
    config_file,
    key="visualizations/spatial_config.json",
    description="Vitessce 可视化配置"
).save()
```

## 模式模块集成

### Bionty (生物本体)

```python
import bionty as bt

# 导入生物本体
bt.CellType.import_source()
bt.Gene.import_source(organism="human")

# 用于数据管理
cell_types = bt.CellType.from_values(adata.obs.cell_type)
```

### 湿实验室

追踪湿实验室实验：

```python
# 安装湿实验室模块
pip install 'lamindb[wetlab]'

# 使用湿实验室注册表
import lamindb_wetlab as wetlab

# 追踪实验、样本、协议
experiment = wetlab.Experiment(name="RNA-seq 批次1").save()
```

### 临床数据 (OMOP)

```python
# 安装临床模块
pip install 'lamindb[clinical]'

# 使用 OMOP 通用数据模型
import lamindb_clinical as clinical

# 追踪临床数据
patient = clinical.Patient(patient_id="P001").save()
```

## Git 集成

### 与 Git 仓库同步

```python
# 配置 git 同步
export LAMINDB_SYNC_GIT_REPO=https://github.com/user/repo.git

# 或通过编程方式
ln.settings.sync_git_repo = "https://github.com/user/repo.git"

# 设置开发目录
lamin settings set dev-dir .

# 脚本通过 git commit 追踪
ln.track()  # 自动捕获 git commit 哈希
# ... 您的代码 ...
ln.finish()

# 查看 git 信息
transform = ln.Transform.get(name="analysis.py")
transform.source_code  # 显示 git commit 时的代码
transform.hash        # Git commit 哈希
```

## 企业级集成

### Benchling

与 Benchling 注册表同步（需团队/企业版）：

```python
# 配置 Benchling 连接（联系 LaminDB 团队）
# 同步 Benchling 注册表的模式与数据

# 访问同步的 Benchling 数据
# 详情请咨询企业支持
```

## 自定义集成模式

### REST API 集成

```python
import requests
import lamindb as ln

ln.track()

# 从 API 获取数据
response = requests.get("https://api.example.com/data")
data = response.json()

# 转换为 DataFrame
import pandas as pd
df = pd.DataFrame(data)

# 保存至 LaminDB
artifact = ln.Artifact.from_dataframe(
    df,
    key="api/fetched_data.parquet",
    description="从外部 API 获取的数据"
).save()

artifact.features.add_values({"api_url": response.url})

ln.finish()
```

### 数据库集成

```python
import sqlalchemy as sa
import lamindb as ln

ln.track()

# 连接外部数据库
engine = sa.create_engine("postgresql://user:pwd@host:port/db")

# 查询数据
query = "SELECT * FROM experiments WHERE date > '2025-01-01'"
df = pd.read_sql(query, engine)

# 保存至 LaminDB
artifact = ln.Artifact.from_dataframe(
    df,
    key="external_db/experiments_2025.parquet",
    description="来自外部数据库的实验数据"
).save()

ln.finish()
```

## Croissant 元数据

使用 Croissant 元数据格式导出数据集：

```python
# 创建含丰富元数据的文件
artifact = ln.Artifact.from_dataframe(
    df,
    key="datasets/published_data.parquet",
    description="含 Croissant 元数据的已发布数据集"
).save()

# 导出 Croissant 元数据（需额外配置）
# 支持数据集发现与互操作性
```

## 集成最佳实践

1. **一致性追踪**：在所有集成工作流中使用 `ln.track()`
2. **关联 ID**：将外部系统 ID（W&B 运行 ID、MLflow 实验 ID）存储为特征值
3. **集中化管理**：将 LaminDB 作为数据文件的单一可信源
4. **参数同步**：向 LaminDB 和 ML 平台同时记录参数
5. **协同版本控制**：保持代码（git）、数据（LaminDB）、实验（ML 平台）同步
6. **策略性缓存**：为云存储配置合适的缓存位置
7. **使用特征集**：将 Bionty 的本体术语关联至文件
8. **文档化集成**：添加描述解释集成背景
9. **渐进式测试**：先用小数据集验证集成
10. **监控谱系**：使用 `view_lineage()` 确保集成追踪有效

## 常见问题排查

**问题：未找到 S3 凭证**
```bash
export AWS_ACCESS_KEY_ID=your_key
export AWS_SECRET_ACCESS_KEY=your_secret
export AWS_DEFAULT_REGION=us-east-1
```

**问题：GCS 认证失败**
```bash
gcloud auth application-default login
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/credentials.json
```

**问题：Git 同步失效**
```bash
# 确保 git 仓库已设置
lamin settings get sync-git-repo

# 确保位于 git 仓库内
git status

# 追踪前提交变更
git add .
git commit -m "更新分析"
ln.track()
```

**问题：MLflow 文件未同步**
```python
# 显式保存至双系统
mlflow.log_artifact("model.pkl")
ln.Artifact("model.pkl", key="models/model.pkl").save()
```
