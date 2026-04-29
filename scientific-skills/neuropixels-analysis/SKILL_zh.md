---
name: neuropixels-analysis
description: Neuropixels 神经记录分析。加载 SpikeGLX/OpenEphys 数据、预处理、运动校正、Kilosort4 峰值分选、质量指标、Allen/IBL 筛选、AI 辅助视觉分析，适用于 Neuropixels 1.0/2.0 细胞外电生理。当处理神经记录、峰值分选、细胞外电生理，或用户提及 Neuropixels、SpikeGLX、Open Ephys、Kilosort、质量指标或单元筛选时使用。
license: MIT 许可证
metadata:
    skill-author: K-Dense Inc.
---

# Neuropixels 数据分析

## 概述

基于 SpikeInterface、Allen Institute 和国际脑实验室 (IBL) 的最新最佳实践，用于分析 Neuropixels 高密度神经记录的综合工具包。支持从原始数据到可发表级别的筛选单元的完整工作流程。

## 何时使用本技能

在以下情况下应使用本技能：
- 处理 Neuropixels 记录（.ap.bin、.lf.bin、.meta 文件）
- 从 SpikeGLX、Open Ephys 或 NWB 格式加载数据
- 预处理神经记录（滤波、CAR、坏通道检测）
- 检测并校正记录中的运动/漂移
- 运行峰值分选（Kilosort4、SpykingCircus2、Mountainsort5）
- 计算质量指标（SNR、ISI 违反率、存在率）
- 使用 Allen/IBL 标准筛选单元
- 创建神经数据的可视化
- 导出结果到 Phy 或 NWB

## 支持的硬件与格式

| 探针 | 电极数 | 通道数 | 备注 |
|-------|-----------|----------|-------|
| Neuropixels 1.0 | 960 | 384 | 需要相位偏移校正 |
| Neuropixels 2.0（单柄） | 1280 | 384 | 更密集的几何布局 |
| Neuropixels 2.0（四柄） | 5120 | 384 | 多区域记录 |

| 格式 | 扩展名 | 读取器 |
|--------|-----------|--------|
| SpikeGLX | `.ap.bin`, `.lf.bin`, `.meta` | `si.read_spikeglx()` |
| Open Ephys | `.continuous`, `.oebin` | `si.read_openephys()` |
| NWB | `.nwb` | `si.read_nwb()` |

## 快速开始

### 基本导入与设置

```python
import spikeinterface.full as si
import neuropixels_analysis as npa

# 配置并行处理
job_kwargs = dict(n_jobs=-1, chunk_duration='1s', progress_bar=True)
```

### 加载数据

```python
# SpikeGLX（最常见）
recording = si.read_spikeglx('/path/to/data', stream_id='imec0.ap')

# Open Ephys（许多实验室常用）
recording = si.read_openephys('/path/to/Record_Node_101/')

# 检查可用流
streams, ids = si.get_neo_streams('spikeglx', '/path/to/data')
print(streams)  # ['imec0.ap', 'imec0.lf', 'nidq']

# 使用数据子集进行测试
recording = recording.frame_slice(0, int(60 * recording.get_sampling_frequency()))
```

### 完整流水线（单命令）

```python
# 运行完整分析流水线
results = npa.run_pipeline(
    recording,
    output_dir='output/',
    sorter='kilosort4',
    curation_method='allen',
)

# 访问结果
sorting = results['sorting']
metrics = results['metrics']
labels = results['labels']
```

## 标准分析工作流程

### 1. 预处理

```python
# 推荐的预处理链
rec = si.highpass_filter(recording, freq_min=400)
rec = si.phase_shift(rec)  # Neuropixels 1.0 必需
bad_ids, _ = si.detect_bad_channels(rec)
rec = rec.remove_channels(bad_ids)
rec = si.common_reference(rec, operator='median')

# 或使用我们的封装
rec = npa.preprocess(recording)
```

### 2. 检查并校正漂移

```python
# 检查漂移（始终进行！）
motion_info = npa.estimate_motion(rec, preset='kilosort_like')
npa.plot_drift(rec, motion_info, output='drift_map.png')

# 必要时应用校正
if motion_info['motion'].max() > 10:  # 微米
    rec = npa.correct_motion(rec, preset='nonrigid_accurate')
```

### 3. 峰值分选

```python
# Kilosort4（推荐，需要 GPU）
sorting = si.run_sorter('kilosort4', rec, folder='ks4_output')

# CPU 替代方案
sorting = si.run_sorter('tridesclous2', rec, folder='tdc2_output')
sorting = si.run_sorter('spykingcircus2', rec, folder='sc2_output')
sorting = si.run_sorter('mountainsort5', rec, folder='ms5_output')

# 检查已安装的分选器
print(si.installed_sorters())
```

### 4. 后处理

```python
# 创建分析器并计算所有扩展
analyzer = si.create_sorting_analyzer(sorting, rec, sparse=True)

analyzer.compute('random_spikes', max_spikes_per_unit=500)
analyzer.compute('waveforms', ms_before=1.0, ms_after=2.0)
analyzer.compute('templates', operators=['average', 'std'])
analyzer.compute('spike_amplitudes')
analyzer.compute('correlograms', window_ms=50.0, bin_ms=1.0)
analyzer.compute('unit_locations', method='monopolar_triangulation')
analyzer.compute('quality_metrics')

metrics = analyzer.get_extension('quality_metrics').get_data()
```

### 5. 筛选

```python
# Allen Institute 标准（保守）
good_units = metrics.query("""
    presence_ratio > 0.9 and
    isi_violations_ratio < 0.5 and
    amplitude_cutoff < 0.1
""").index.tolist()

# 或使用自动筛选
labels = npa.curate(metrics, method='allen')  # 'allen'、'ibl'、'strict'
```

### 6. AI 辅助筛选（用于不确定的单元）

在使用本技能与 Claude Code 时，Claude 可以直接分析波形图并提供专家筛选决策。对于程序化 API 访问：

```python
from anthropic import Anthropic

# 设置 API 客户端
client = Anthropic()

# 视觉分析不确定的单元
uncertain = metrics.query('snr > 3 and snr < 8').index.tolist()

for unit_id in uncertain:
    result = npa.analyze_unit_visually(analyzer, unit_id, api_client=client)
    print(f"单元 {unit_id}: {result['classification']}")
    print(f"  推理: {result['reasoning'][:100]}...")
```

**Claude Code 集成**：在 Claude Code 中运行时，让 Claude 直接检查波形/相关图——无需设置 API。

### 7. 生成分析报告

```python
# 生成包含可视化的综合 HTML 报告
report_dir = npa.generate_analysis_report(results, 'output/')
# 打开 report.html，包含汇总统计、图表和单元表格

# 在控制台打印格式化汇总
npa.print_analysis_summary(results)
```

### 8. 导出结果

```python
# 导出到 Phy 进行人工审核
si.export_to_phy(analyzer, output_folder='phy_export/',
                 compute_pc_features=True, compute_amplitudes=True)

# 导出到 NWB
from spikeinterface.exporters import export_to_nwb
export_to_nwb(rec, sorting, 'output.nwb')

# 保存质量指标
metrics.to_csv('quality_metrics.csv')
```

## 常见陷阱与最佳实践

1. **在峰值分选前始终检查漂移**——超过 10μm 的漂移会显著影响质量
2. **对 Neuropixels 1.0 探针使用 phase_shift**（2.0 不需要）
3. **保存预处理数据**以避免重新计算——使用 `rec.save(folder='preprocessed/')`
4. **对 Kilosort4 使用 GPU**——比 CPU 替代方案快 10-50 倍
5. **人工审查不确定的单元**——自动筛选只是一个起点
6. **将指标与 AI 结合**——明确情况用指标，边缘情况用 AI
7. **记录你的阈值**——不同分析可能需要不同标准
8. **对关键实验导出到 Phy**——人工监督很有价值

## 需要调整的关键参数

### 预处理
- `freq_min`：高通截止频率（通常 300-400 Hz）
- `detect_threshold`：坏通道检测灵敏度

### 运动校正
- `preset`：'kilosort_like'（快速）或 'nonrigid_accurate'（适用于严重漂移）

### 峰值分选（Kilosort4）
- `batch_size`：每批样本数（默认 30000）
- `nblocks`：漂移块数（长记录增加）
- `Th_learned`：检测阈值（越低检测到更多峰值）

### 质量指标
- `snr_threshold`：信噪比截止（通常 3-5）
- `isi_violations_ratio`：不应期违反率（0.01-0.5）
- `presence_ratio`：记录覆盖度（0.5-0.95）

## 附带资源

### scripts/preprocess_recording.py
自动化预处理脚本：
```bash
python scripts/preprocess_recording.py /path/to/data --output preprocessed/
```

### scripts/run_sorting.py
运行峰值分选：
```bash
python scripts/run_sorting.py preprocessed/ --sorter kilosort4 --output sorting/
```

### scripts/compute_metrics.py
计算质量指标并应用筛选：
```bash
python scripts/compute_metrics.py sorting/ preprocessed/ --output metrics/ --curation allen
```

### scripts/export_to_phy.py
导出到 Phy 进行人工筛选：
```bash
python scripts/export_to_phy.py metrics/analyzer --output phy_export/
```

### assets/analysis_template.py
完整分析模板。复制并自定义：
```bash
cp assets/analysis_template.py my_analysis.py
# 编辑参数并运行
python my_analysis.py
```

### references/standard_workflow.md
详细的逐步工作流程，包含每个阶段的说明。

### references/api_reference.md
按模块组织的快速函数参考。

### references/plotting_guide.md
用于出版级图表的综合可视化指南。

## 详细参考指南

| 主题 | 参考 |
|-------|-----------|
| 完整工作流程 | [references/standard_workflow.md](references/standard_workflow.md) |
| API 参考 | [references/api_reference.md](references/api_reference.md) |
| 绘图指南 | [references/plotting_guide.md](references/plotting_guide.md) |
| 预处理 | [references/PREPROCESSING.md](references/PREPROCESSING.md) |
| 峰值分选 | [references/SPIKE_SORTING.md](references/SPIKE_SORTING.md) |
| 运动校正 | [references/MOTION_CORRECTION.md](references/MOTION_CORRECTION.md) |
| 质量指标 | [references/QUALITY_METRICS.md](references/QUALITY_METRICS.md) |
| 自动筛选 | [references/AUTOMATED_CURATION.md](references/AUTOMATED_CURATION.md) |
| AI 辅助筛选 | [references/AI_CURATION.md](references/AI_CURATION.md) |
| 波形分析 | [references/ANALYSIS.md](references/ANALYSIS.md) |

## 安装

```bash
# 核心包
pip install spikeinterface[full] probeinterface neo

# 峰值分选器
pip install kilosort          # Kilosort4（需要 GPU）
pip install spykingcircus     # SpykingCircus2（CPU）
pip install mountainsort5     # Mountainsort5（CPU）

# 我们的工具包
pip install neuropixels-analysis

# 可选：AI 筛选
pip install anthropic

# 可选：IBL 工具
pip install ibl-neuropixel ibllib
```

## 项目结构

```
project/
├── raw_data/
│   └── recording_g0/
│       └── recording_g0_imec0/
│           ├── recording_g0_t0.imec0.ap.bin
│           └── recording_g0_t0.imec0.ap.meta
├── preprocessed/           # 保存的预处理记录
├── motion/                 # 运动估计结果
├── sorting_output/         # 峰值分选输出
├── analyzer/               # SortingAnalyzer（波形、指标）
├── phy_export/             # 用于人工筛选
├── ai_curation/            # AI 分析报告
└── results/
    ├── quality_metrics.csv
    ├── curation_labels.json
    └── output.nwb
```

## 其他资源

- **SpikeInterface 文档**：https://spikeinterface.readthedocs.io/
- **Neuropixels 教程**：https://spikeinterface.readthedocs.io/en/stable/how_to/analyze_neuropixels.html
- **Kilosort4 GitHub**：https://github.com/MouseLand/Kilosort
- **IBL Neuropixel 工具**：https://github.com/int-brain-lab/ibl-neuropixel
- **Allen Institute ecephys**：https://github.com/AllenInstitute/ecephys_spike_sorting
- **Bombcell（自动 QC）**：https://github.com/Julie-Fabre/bombcell
- **SpikeAgent（AI 筛选）**：https://github.com/SpikeAgent/SpikeAgent
