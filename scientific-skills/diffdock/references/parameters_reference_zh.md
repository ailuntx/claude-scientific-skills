# DiffDock 配置参数参考手册

本文档提供所有 DiffDock 配置参数和命令行选项的完整说明。

## 模型与检查点设置

### 模型路径
- **`--model_dir`**：包含评分模型检查点的目录
  - 默认值：`./workdir/v1.1/score_model`
  - DiffDock-L 模型（当前默认）

- **`--confidence_model_dir`**：包含置信度模型检查点的目录
  - 默认值：`./workdir/v1.1/confidence_model`

- **`--ckpt`**：评分模型检查点文件名
  - 默认值：`best_ema_inference_epoch_model.pt`

- **`--confidence_ckpt`**：置信度模型检查点文件名
  - 默认值：`best_model_epoch75.pt`

### 模型版本标志
- **`--old_score_model`**：使用原始 DiffDock 模型替代 DiffDock-L
  - 默认值：`false`（使用 DiffDock-L）

- **`--old_filtering_model`**：使用传统置信度过滤方法
  - 默认值：`true`

## 输入/输出选项

### 输入规范
- **`--protein_path`**：蛋白质 PDB 文件路径
  - 示例：`--protein_path protein.pdb`
  - 可替代 `--protein_sequence`

- **`--protein_sequence`**：用于 ESMFold 折叠的氨基酸序列
  - 自动从序列生成蛋白质结构
  - 可替代 `--protein_path`

- **`--ligand`**：配体规范（SMILES 字符串或文件路径）
  - SMILES 字符串：`--ligand "COc(cc1)ccc1C#N"`
  - 文件路径：`--ligand ligand.sdf` 或 `.mol2`

- **`--protein_ligand_csv`**：用于批量处理的 CSV 文件
  - 必需列：`complex_name`, `protein_path`, `ligand_description`, `protein_sequence`
  - 示例：`--protein_ligand_csv data/protein_ligand_example.csv`

### 输出控制
- **`--out_dir`**：预测结果输出目录
  - 示例：`--out_dir results/user_predictions/`

- **`--save_visualisation`**：导出预测分子为 SDF 文件
  - 启用结果可视化

## 推理参数

### 扩散步骤
- **`--inference_steps`**：计划的推理迭代次数
  - 默认值：`20`
  - 更高值可能提升精度但增加运行时间

- **`--actual_steps`**：实际执行的扩散步骤数
  - 默认值：`19`

- **`--no_final_step_noise`**：在最终扩散步骤省略噪声
  - 默认值：`true`

### 采样设置
- **`--samples_per_complex`**：每个复合物生成的样本数
  - 默认值：`10`
  - 更多样本提供更好覆盖但增加计算量

- **`--sigma_schedule`**：噪声调度类型
  - 默认值：`expbeta`（指数-beta）

- **`--initial_noise_std_proportion`**：初始噪声标准差缩放比例
  - 默认值：`1.46`

### 温度参数

#### 采样温度（控制预测多样性）
- **`--temp_sampling_tr`**：平移采样温度
  - 默认值：`1.17`

- **`--temp_sampling_rot`**：旋转采样温度
  - 默认值：`2.06`

- **`--temp_sampling_tor`**：扭转采样温度
  - 默认值：`7.04`

#### Psi 角度温度
- **`--temp_psi_tr`**：平移 psi 温度
  - 默认值：`0.73`

- **`--temp_psi_rot`**：旋转 psi 温度
  - 默认值：`0.90`

- **`--temp_psi_tor`**：扭转 psi 温度
  - 默认值：`0.59`

#### Sigma 数据温度
- **`--temp_sigma_data_tr`**：平移数据分布缩放
  - 默认值：`0.93`

- **`--temp_sigma_data_rot`**：旋转数据分布缩放
  - 默认值：`0.75`

- **`--temp_sigma_data_tor`**：扭转数据分布缩放
  - 默认值：`0.69`

## 处理选项

### 性能
- **`--batch_size`**：处理批次大小
  - 默认值：`10`
  - 更大值提升吞吐量但需更多内存

- **`--tqdm`**：启用进度条可视化
  - 适用于监控长时间运行任务

### 蛋白质结构
- **`--chain_cutoff`**：最大处理蛋白质链数
  - 示例：`--chain_cutoff 10`
  - 适用于大型多链复合物

- **`--esm_embeddings_path`**：预计算 ESM2 蛋白质嵌入路径
  - 通过复用嵌入加速推理
  - 可选优化项

### 数据集选项
- **`--split`**：使用的数据集划分（train/test/val）
  - 用于标准基准评估

## 高级标志

### 调试与测试
- **`--no_model`**：禁用模型推理（调试用）
  - 默认值：`false`

- **`--no_random`**：禁用随机化
  - 默认值：`false`
  - 适用于可复现性测试

### 替代采样
- **`--ode`**：使用 ODE 求解器替代 SDE
  - 默认值：`false`
  - 替代采样方法

- **`--different_schedules`**：为各组件使用不同噪声调度
  - 默认值：`false`

### 错误处理
- **`--limit_failures`**：停止前允许的最大失败次数
  - 默认值：`5`

## 配置文件

所有参数可通过 YAML 配置文件（通常为 `default_inference_args.yaml`）指定或通过命令行覆盖：

```bash
python -m inference --config default_inference_args.yaml --samples_per_complex 20
```

命令行参数优先级高于配置文件值。
