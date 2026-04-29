# DiffDock 工作流程与示例

本文档提供常见 DiffDock 任务的实用工作流程和使用示例。

## 安装与设置

### Conda 安装（推荐）

```bash
# 克隆仓库
git clone https://github.com/gcorso/DiffDock.git
cd DiffDock

# 创建 conda 环境
conda env create --file environment.yml
conda activate diffdock
```

### Docker 安装

```bash
# 拉取 Docker 镜像
docker pull rbgcsail/diffdock

# 运行带 GPU 支持的容器
docker run -it --gpus all --entrypoint /bin/bash rbgcsail/diffdock

# 容器内激活环境
micromamba activate diffdock
```

### 首次运行
首次执行会预计算 SO(2) 和 SO(3) 查找表，耗时数分钟。后续运行将立即启动。

## 工作流程 1：单蛋白-配体对接

### 使用 PDB 文件和 SMILES 字符串

```bash
python -m inference \
  --config default_inference_args.yaml \
  --protein_path examples/protein.pdb \
  --ligand "COc1ccc(C(=O)Nc2ccccc2)cc1" \
  --out_dir results/single_docking/
```

**输出结构**：
```
results/single_docking/
├── index_0_rank_1.sdf       # 排名第一的预测
├── index_0_rank_2.sdf       # 排名第二的预测
├── ...
├── index_0_rank_10.sdf      # 第10个预测 (当 samples_per_complex=10)
└── confidence_scores.txt    # 所有预测的置信度分数
```

### 使用配体结构文件

```bash
python -m inference \
  --config default_inference_args.yaml \
  --protein_path protein.pdb \
  --ligand ligand.sdf \
  --out_dir results/ligand_file/
```

**支持的配体格式**：SDF、MOL2 或 RDKit 可读取的任何格式

## 工作流程 2：蛋白序列到结构对接

### 使用 ESMFold 进行蛋白折叠

```bash
python -m inference \
  --config default_inference_args.yaml \
  --protein_sequence "MSKGEELFTGVVPILVELDGDVNGHKFSVSGEGEGDATYGKLTLKFICTTGKLPVPWPTLVTTFSYGVQCFSRYPDHMKQHDFFKSAMPEGYVQERTIFFKDDGNYKTRAEVKFEGDTLVNRIELKGIDFKEDGNILGHKLEYNYNSHNVYIMADKQKNGIKVNFKIRHNIEDGSVQLADHYQQNTPIGDGPVLLPDNHYLSTQSALSKDPNEKRDHMVLLEFVTAAGITHGMDELYK" \
  --ligand "CC(C)Cc1ccc(cc1)C(C)C(=O)O" \
  --out_dir results/sequence_docking/
```

**适用场景**：
- PDB 中无蛋白结构
- 模拟突变或变体
- 新蛋白设计验证

**注意**：ESMFold 折叠会增加计算时间（30秒-5分钟，取决于序列长度）

## 工作流程 3：批量处理多个复合物

### 准备 CSV 文件

创建包含必需列的 `complexes.csv`：

```csv
complex_name,protein_path,ligand_description,protein_sequence
complex1,proteins/protein1.pdb,CC(=O)Oc1ccccc1C(=O)O,
complex2,,COc1ccc(C#N)cc1,MSKGEELFTGVVPILVELDGDVNGHKF...
complex3,proteins/protein3.pdb,ligands/ligand3.sdf,
```

**列描述**：
- `complex_name`：复合物唯一标识符
- `protein_path`：PDB 文件路径（使用序列时留空）
- `ligand_description`：SMILES 字符串或配体文件路径
- `protein_sequence`：氨基酸序列（使用 PDB 时留空）

### 运行批量对接

```bash
python -m inference \
  --config default_inference_args.yaml \
  --protein_ligand_csv complexes.csv \
  --out_dir results/batch_predictions/ \
  --batch_size 10
```

**输出结构**：
```
results/batch_predictions/
├── complex1/
│   ├── rank_1.sdf
│   ├── rank_2.sdf
│   └── ...
├── complex2/
│   ├── rank_1.sdf
│   └── ...
└── complex3/
    └── ...
```

## 工作流程 4：高通量虚拟筛选

### 设置大型配体库筛选

```python
# generate_screening_csv.py
import pandas as pd

# 加载配体库
ligands = pd.read_csv("ligand_library.csv")  # 包含 SMILES

# 创建 DiffDock 输入
screening_data = {
    "complex_name": [f"screen_{i}" for i in range(len(ligands))],
    "protein_path": ["target_protein.pdb"] * len(ligands),
    "ligand_description": ligands["smiles"].tolist(),
    "protein_sequence": [""] * len(ligands)
}

df = pd.DataFrame(screening_data)
df.to_csv("screening_input.csv", index=False)
```

### 运行筛选

```bash
# 预计算 ESM 嵌入以加速筛选
python datasets/esm_embedding_preparation.py \
  --protein_ligand_csv screening_input.csv \
  --out_file protein_embeddings.pt

# 使用预计算嵌入运行对接
python -m inference \
  --config default_inference_args.yaml \
  --protein_ligand_csv screening_input.csv \
  --esm_embeddings_path protein_embeddings.pt \
  --out_dir results/virtual_screening/ \
  --batch_size 32
```

### 后处理：提取高置信度结果

```python
# analyze_screening_results.py
import os
import pandas as pd

results = []
results_dir = "results/virtual_screening/"

for complex_dir in os.listdir(results_dir):
    confidence_file = os.path.join(results_dir, complex_dir, "confidence_scores.txt")
    if os.path.exists(confidence_file):
        with open(confidence_file) as f:
            scores = [float(line.strip()) for line in f]
            top_score = max(scores)
            results.append({"complex": complex_dir, "top_confidence": top_score})

# 按置信度排序
df = pd.DataFrame(results)
df_sorted = df.sort_values("top_confidence", ascending=False)

# 获取前100个结果
top_hits = df_sorted.head(100)
top_hits.to_csv("top_hits.csv", index=False)
```

## 工作流程 5：考虑蛋白柔性的集成对接

### 准备蛋白构象集合

```python
# 对于已知柔性蛋白，使用多构象
# 示例：使用 MD 快照或晶体结构

# create_ensemble_csv.py
import pandas as pd

conformations = [
    "protein_conf1.pdb",
    "protein_conf2.pdb",
    "protein_conf3.pdb",
    "protein_conf4.pdb"
]

ligand = "CC(C)Cc1ccc(cc1)C(C)C(=O)O"

data = {
    "complex_name": [f"ensemble_{i}" for i in range(len(conformations))],
    "protein_path": conformations,
    "ligand_description": [ligand] * len(conformations),
    "protein_sequence": [""] * len(conformations)
}

pd.DataFrame(data).to_csv("ensemble_input.csv", index=False)
```

### 运行集成对接

```bash
python -m inference \
  --config default_inference_args.yaml \
  --protein_ligand_csv ensemble_input.csv \
  --out_dir results/ensemble_docking/ \
  --samples_per_complex 20  # 每个构象生成更多样本
```

## 工作流程 6：与下游分析工具集成

### 示例：DiffDock + GNINA 重打分

```bash
# 1. 运行 DiffDock
python -m inference \
  --config default_inference_args.yaml \
  --protein_path protein.pdb \
  --ligand "CC(=O)OC1=CC=CC=C1C(=O)O" \
  --out_dir results/diffdock_poses/ \
  --save_visualisation

# 2. 使用 GNINA 重打分
for pose in results/diffdock_poses/*.sdf; do
    gnina -r protein.pdb -l "$pose" --score_only -o "${pose%.sdf}_gnina.sdf"
done
```

### 示例：DiffDock + OpenMM 能量最小化

```python
# minimize_poses.py
from openmm import app, LangevinIntegrator, Platform
from openmm.app import ForceField, Modeller, PDBFile
from rdkit import Chem
import os

# 加载蛋白
protein = PDBFile('protein.pdb')
forcefield = ForceField('amber14-all.xml', 'amber14/tip3pfb.xml')

# 处理每个 DiffDock 构象
pose_dir = 'results/diffdock_poses/'
for pose_file in os.listdir(pose_dir):
    if pose_file.endswith('.sdf'):
        # 加载配体
        mol = Chem.SDMolSupplier(os.path.join(pose_dir, pose_file))[0]

        # 合并蛋白+配体
        modeller = Modeller(protein.topology, protein.positions)
        # ... 将配体加入 modeller ...

        # 创建系统并最小化
        system = forcefield.createSystem(modeller.topology)
        integrator = LangevinIntegrator(300, 1.0, 0.002)
        simulation = app.Simulation(modeller.topology, system, integrator)
        simulation.minimizeEnergy(maxIterations=1000)

        # 保存最小化结构
        positions = simulation.context.getState(getPositions=True).getPositions()
        PDBFile.writeFile(simulation.topology, positions,
                         open(f"minimized_{pose_file}.pdb", 'w'))
```

## 工作流程 7：使用图形界面

### 启动 Web 界面

```bash
python app/main.py
```

### 访问界面
在浏览器中访问 `http://localhost:7860`

### 功能特性
- 上传蛋白 PDB 或输入序列
- 输入配体 SMILES 或上传结构
- 通过 GUI 调整推理参数
- 交互式可视化结果
- 直接下载预测结果

### 在线替代方案
无需本地安装，使用 Hugging Face Spaces 演示：
- URL: https://huggingface.co/spaces/reginabarzilaygroup/DiffDock-Web

## 高级配置

### 自定义推理设置

创建自定义 YAML 配置：

```yaml
# custom_inference.yaml
# 模型设置
model_dir: ./workdir/v1.1/score_model
confidence_model_dir: ./workdir/v1.1/confidence_model

# 采样参数
samples_per_complex: 20  # 增加样本以提升覆盖率
inference_steps: 25      # 增加步数以提升精度

# 温度调整 (提高值增加多样性)
temp_sampling_tr: 1.3
temp_sampling_rot: 2.2
temp_sampling_tor: 7.5

# 输出
save_visualisation: true
```

使用自定义配置：

```bash
python -m inference \
  --config custom_inference.yaml \
  --protein_path protein.pdb \
  --ligand "CC(=O)OC1=CC=CC=C1C(=O)O" \
  --out_dir results/custom_config/
```

## 常见问题排查

### 问题：内存不足错误

**解决方案**：减小批处理大小
```bash
python -m inference ... --batch_size 2
```

### 问题：运行缓慢

**解决方案**：确保使用 GPU
```python
import torch
print(torch.cuda.is_available())  # 应返回 True
```

### 问题：大分子配体预测效果差

**解决方案**：增加采样多样性
```bash
python -m inference ... --samples_per_complex 40 --temp_sampling_tor 9.0
```

### 问题：蛋白含多链

**解决方案**：限制链数或分离结合位点
```bash
python -m inference ... --chain_cutoff 4
```

或预处理 PDB 仅保留相关链。

## 最佳实践总结

1. **从简单开始**：批量处理前先测试单复合物
2. **必需 GPU**：使用 GPU 保证合理性能
3. **多样本生成**：生成 10-40 个样本以获得稳健预测
4. **结果验证**：使用分子可视化和辅助评分工具
5. **关注置信度**：置信分数仅用于初步排序，非最终决策
6. **参数迭代**：针对特定体系调整温度/步数
7. **预计算嵌入**：重复使用相同蛋白时
8. **工具组合**：与打分函数和能量最小化工具集成
