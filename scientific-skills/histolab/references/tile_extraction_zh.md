# 分块提取

## 概述

分块提取是从大型全切片图像中裁剪出较小、可管理区域的过程。Histolab 提供三种主要提取策略，分别适用于不同的分析需求。所有分块器共享通用参数，并提供预览和提取分块的方法。

## 通用参数

所有分块器类都接受以下参数：

```python
tile_size: tuple = (512, 512)           # 分块尺寸（宽，高），单位像素
level: int = 0                          # 提取的金字塔层级（0为最高分辨率）
check_tissue: bool = True               # 根据组织内容过滤分块
tissue_percent: float = 80.0            # 最小组织覆盖率（0-100）
pixel_overlap: int = 0                  # 相邻分块重叠像素数（仅GridTiler适用）
prefix: str = ""                        # 保存分块文件名的前缀
suffix: str = ".png"                    # 保存分块的文件扩展名
extraction_mask: BinaryMask = BiggestTissueBoxMask()  # 定义提取区域的掩膜
```

## 随机分块器（RandomTiler）

**用途：** 从组织区域提取固定数量的随机位置分块。

```python
from histolab.tiler import RandomTiler

random_tiler = RandomTiler(
    tile_size=(512, 512),
    n_tiles=100,                # 要提取的随机分块数量
    level=0,
    seed=42,                    # 确保可重现性的随机种子
    check_tissue=True,
    tissue_percent=80.0
)

# 提取分块
random_tiler.extract(slide, extraction_mask=TissueMask())
```

**关键参数：**
- `n_tiles`：要提取的随机分块数量
- `seed`：用于可重现分块选择的随机种子
- `max_iter`：寻找有效分块的最大尝试次数（默认1000）

**适用场景：**
- 切片内容的探索性分析
- 为训练数据采样多样化区域
- 快速评估组织特征
- 从多个切片创建平衡数据集

**优势：**
- 计算效率高
- 适合采样多样化组织形态
- 通过种子参数确保可重现性
- 执行速度快

**局限性：**
- 可能遗漏罕见组织模式
- 无法保证覆盖范围
- 随机分布可能无法捕捉结构化特征

## 网格分块器（GridTiler）

**用途：** 按照网格模式系统性地提取组织区域的分块。

```python
from histolab.tiler import GridTiler

grid_tiler = GridTiler(
    tile_size=(512, 512),
    level=0,
    check_tissue=True,
    tissue_percent=80.0,
    pixel_overlap=0             # 相邻分块间的重叠像素数
)

# 提取分块
grid_tiler.extract(slide)
```

**关键参数：**
- `pixel_overlap`：相邻分块间的重叠像素数
  - `pixel_overlap=0`：无重叠分块
  - `pixel_overlap=128`：每边128像素重叠
  - 可用于滑动窗口方法

**适用场景：**
- 全面覆盖切片
- 需要位置信息的空间分析
- 从分块重建图像
- 语义分割任务
- 基于区域的分析

**优势：**
- 完整覆盖组织区域
- 保留空间关系
- 可预测的分块位置
- 适合全切片分析

**局限性：**
- 大型切片计算密集
- 可能生成大量背景分块（可通过`check_tissue`缓解）
- 输出数据集较大

**网格模式：**
```
[分块1][分块2][分块3]
[分块4][分块5][分块6]
[分块7][分块8][分块9]
```

当`pixel_overlap=64`时：
```
[分块1-重叠-分块2-重叠-分块3]
[   重叠区域      重叠区域      重叠区域]
[分块4-重叠-分块5-重叠-分块6]
```

## 评分分块器（ScoreTiler）

**用途：** 基于自定义评分函数提取评分最高的分块。

```python
from histolab.tiler import ScoreTiler
from histolab.scorer import NucleiScorer

score_tiler = ScoreTiler(
    tile_size=(512, 512),
    n_tiles=50,                 # 要提取的顶级评分分块数量
    level=0,
    scorer=NucleiScorer(),      # 评分函数
    check_tissue=True
)

# 提取顶级评分分块
score_tiler.extract(slide)
```

**关键参数：**
- `n_tiles`：要提取的顶级评分分块数量
- `scorer`：评分函数（如`NucleiScorer`, `CellularityScorer`, 自定义评分器）

**适用场景：**
- 提取信息最丰富的区域
- 优先选择具有特定特征的分块（细胞核、细胞等）
- 基于质量的分块选择
- 聚焦诊断相关区域
- 训练数据筛选

**优势：**
- 聚焦信息量最大的分块
- 在保持质量的同时减小数据集规模
- 可定制不同评分器
- 适合针对性分析

**局限性：**
- 比RandomTiler慢（需对所有候选分块评分）
- 需要任务适配的评分器
- 可能遗漏低评分但相关区域

## 可用评分器

### 细胞核评分器（NucleiScorer）

基于细胞核检测和密度对分块评分。

```python
from histolab.scorer import NucleiScorer

nuclei_scorer = NucleiScorer()
```

**工作原理：**
1. 将分块转为灰度图
2. 应用阈值检测细胞核
3. 统计细胞核状结构
4. 根据细胞核密度分配分数

**最佳适用：**
- 细胞密集的组织区域
- 肿瘤检测
- 有丝分裂分析
- 高细胞含量区域

### 细胞密度评分器（CellularityScorer）

基于整体细胞含量对分块评分。

```python
from histolab.scorer import CellularityScorer

cellularity_scorer = CellularityScorer()
```

**最佳适用：**
- 区分细胞区域与间质区域
- 评估肿瘤细胞密度
- 分离致密与稀疏组织区域

### 自定义评分器

创建满足特定需求的自定义评分函数：

```python
from histolab.scorer import Scorer
import numpy as np

class ColorVarianceScorer(Scorer):
    def __call__(self, tile):
        """基于颜色方差对分块评分"""
        tile_array = np.array(tile.image)
        # 计算颜色方差
        variance = np.var(tile_array, axis=(0, 1)).sum()
        return variance

# 使用自定义评分器
variance_scorer = ColorVarianceScorer()
score_tiler = ScoreTiler(
    tile_size=(512, 512),
    n_tiles=30,
    scorer=variance_scorer
)
```

## 使用locate_tiles()预览分块

在提取前预览分块位置以验证分块器配置：

```python
# 预览随机分块位置
random_tiler.locate_tiles(
    slide=slide,
    extraction_mask=TissueMask(),
    n_tiles=20  # 要预览的分块数量（RandomTiler专用）
)
```

这将显示带有彩色矩形标记分块位置的切片缩略图。

## 提取工作流

### 基础提取

```python
from histolab.slide import Slide
from histolab.tiler import RandomTiler

# 加载切片
slide = Slide("slide.svs", processed_path="output/tiles/")

# 配置分块器
tiler = RandomTiler(
    tile_size=(512, 512),
    n_tiles=100,
    level=0,
    seed=42
)

# 提取分块（保存到processed_path）
tiler.extract(slide)
```

### 带日志记录的提取

```python
import logging

# 启用日志记录
logging.basicConfig(level=logging.INFO)

# 带进度信息的提取
tiler.extract(slide)
# 输出：INFO: 分块1/100已保存...
# 输出：INFO: 分块2/100已保存...
```

### 带报告的提取

```python
# 生成包含分块信息的CSV报告
score_tiler = ScoreTiler(
    tile_size=(512, 512),
    n_tiles=50,
    scorer=NucleiScorer()
)

# 提取并保存报告
score_tiler.extract(slide, report_path="tiles_report.csv")

# 报告包含：分块名称、坐标、评分、组织百分比
```

报告格式：
```csv
tile_name,x_coord,y_coord,level,score,tissue_percent
tile_001.png,10240,5120,0,0.89,95.2
tile_002.png,15360,7680,0,0.85,91.7
...
```

## 高级提取模式

### 多层级提取

在不同放大层级提取分块：

```python
# 高分辨率分块（层级0）
high_res_tiler = RandomTiler(tile_size=(512, 512), n_tiles=50, level=0)
high_res_tiler.extract(slide)

# 中分辨率分块（层级1）
med_res_tiler = RandomTiler(tile_size=(512, 512), n_tiles=50, level=1)
med_res_tiler.extract(slide)

# 低分辨率分块（层级2）
low_res_tiler = RandomTiler(tile_size=(512, 512), n_tiles=50, level=2)
low_res_tiler.extract(slide)
```

### 分层提取

从相同位置提取多尺度分块：

```python
# 在层级0提取随机位置
random_tiler_l0 = RandomTiler(
    tile_size=(512, 512),
    n_tiles=30,
    level=0,
    seed=42,
    prefix="level0_"
)
random_tiler_l0.extract(slide)

# 在层级1提取相同位置（使用相同种子）
random_tiler_l1 = RandomTiler(
    tile_size=(512, 512),
    n_tiles=30,
    level=1,
    seed=42,
    prefix="level1_"
)
random_tiler_l1.extract(slide)
```

### 自定义分块过滤

提取后应用额外过滤：

```python
from PIL import Image
import numpy as np
from pathlib import Path

def filter_blurry_tiles(tile_dir, threshold=100):
    """使用拉普拉斯方差过滤模糊分块"""
    for tile_path in Path(tile_dir).glob("*.png"):
        img = Image.open(tile_path)
        gray = np.array(img.convert('L'))
        laplacian_var = cv2.Laplacian(gray, cv2.CV_64F).var()

        if laplacian_var < threshold:
            tile_path.unlink()  # 移除模糊分块
            print(f"已移除模糊分块：{tile_path.name}")

# 在提取后使用
tiler.extract(slide)
filter_blurry_tiles("output/tiles/")
```

## 最佳实践

1. **提取前预览**：始终使用`locate_tiles()`验证分块位置
2. **使用合适层级**：使提取层级匹配分析分辨率需求
3. **设置组织百分比阈值**：根据染色和组织类型调整（通常70-90%）
4. **选择正确的分块器**：
   - RandomTiler用于采样和探索
   - GridTiler用于全面覆盖
   - ScoreTiler用于针对性、质量驱动的提取
5. **启用日志记录**：监控大型数据集的提取进度
6. **使用种子确保可重现性**：在RandomTiler中设置随机种子
7. **考虑存储**：GridTiler每切片可能生成数千个分块
8. **验证分块质量**：检查提取分块的伪影、模糊或聚焦问题

## 性能优化

1. **在合适层级提取**：较低层级（1,2）提取更快
2. **调整组织百分比**：更高阈值减少无效分块尝试
3. **使用BiggestTissueBoxMask**：比TissueMask处理单组织切片更快
4. **限制分块数量**：针对RandomTiler和ScoreTiler
5. **使用pixel_overlap=0**：用于非重叠GridTiler提取

## 故障排除

### 问题：未提取到分块
**解决方案：**
- 降低`tissue_percent`阈值
- 确认切片包含组织（检查缩略图）
- 确保extraction_mask捕获组织区域
- 检查分块尺寸是否适合切片分辨率

### 问题：提取到过多背景分块
**解决方案：**
- 启用`check_tissue=True`
- 提高`tissue_percent`阈值
- 使用合适掩膜（TissueMask vs BiggestTissueBoxMask）

### 问题：提取速度极慢
**解决方案：**
- 在较低金字塔层级提取（level=1或2）
- 减少RandomTiler/ScoreTiler的`n_tiles`
- 改用RandomTiler替代GridTiler进行采样
- 使用BiggestTissueBoxMask替代TissueMask

### 问题：分块重叠过多（GridTiler）
**解决方案：**
- 设置`pixel_overlap=0`实现非重叠分块
- 减小`pixel_overlap`值
