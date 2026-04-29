# 预处理流水线与变换

## 概述

PathML 提供基于可组合变换的模块化预处理架构，这些变换被组织成流水线。变换是修改图像、创建掩膜或提取特征的独立操作。流水线将变换串联起来，为计算病理学创建可复现、可扩展的预处理工作流。

## 流水线架构

### 流水线类

`Pipeline` 类组合了连续应用的变换序列：

```python
from pathml.preprocessing import Pipeline, Transform1, Transform2

# 创建流水线
pipeline = Pipeline([
    Transform1(param1=value1),
    Transform2(param2=value2),
    # ... 更多变换
])

# 在单张玻片上运行
pipeline.run(slide_data)

# 在数据集上运行
pipeline.run(dataset, distributed=True, n_workers=8)
```

**核心特性：**
- 变换的顺序执行
- 自动处理切片和掩膜
- 支持 Dask 分布式处理
- 通过可序列化配置实现可复现工作流

### 变换基类

所有变换继承自 `Transform` 基类并实现：
- `apply()` - 核心变换逻辑
- `input_type` - 预期输入类型（切片、掩膜等）
- `output_type` - 输出类型

## 变换类别

PathML 提供六大类变换：

1. **图像修改** - 模糊、重缩放、直方图均衡化
2. **掩膜创建** - 组织检测、细胞核检测、阈值处理
3. **掩膜修改** - 掩膜的形态学操作
4. **染色处理** - H&E 染色归一化与分离
5. **质量控制** - 伪影检测、空白区域标记
6. **专用处理** - 多参数成像、细胞分割

## 图像修改变换

### 模糊操作

应用不同模糊核进行降噪：

**中值模糊：**
```python
from pathml.preprocessing import MedianBlur

# 应用中值滤波
transform = MedianBlur(kernel_size=5)
```
- 有效去除椒盐噪声
- 比高斯模糊更好地保留边缘

**高斯模糊：**
```python
from pathml.preprocessing import GaussianBlur

# 应用高斯模糊
transform = GaussianBlur(kernel_size=5, sigma=1.0)
```
- 平滑降噪
- 可调节 sigma 控制模糊强度

**方框模糊：**
```python
from pathml.preprocessing import BoxBlur

# 应用方框滤波
transform = BoxBlur(kernel_size=5)
```
- 最快的模糊操作
- 核内均匀平均

### 强度调整

**强度重缩放：**
```python
from pathml.preprocessing import RescaleIntensity

# 将强度重缩放到 [0, 255]
transform = RescaleIntensity(
    in_range=(0, 1.0),
    out_range=(0, 255)
)
```

**直方图均衡化：**
```python
from pathml.preprocessing import HistogramEqualization

# 全局直方图均衡化
transform = HistogramEqualization()
```
- 增强全局对比度
- 扩展强度分布范围

**自适应直方图均衡化 (CLAHE)：**
```python
from pathml.preprocessing import AdaptiveHistogramEqualization

# 对比度受限自适应直方图均衡化
transform = AdaptiveHistogramEqualization(
    clip_limit=0.03,
    tile_grid_size=(8, 8)
)
```
- 增强局部对比度
- 通过 clip_limit 防止过度放大
- 更适合局部对比度变化的图像

### 超像素处理

**超像素插值：**
```python
from pathml.preprocessing import SuperpixelInterpolation

# 使用 SLIC 分割超像素
transform = SuperpixelInterpolation(
    n_segments=100,
    compactness=10.0
)
```
- 将图像分割为感知上有意义的区域
- 适用于特征提取和分割

## 掩膜创建变换

### H&E 组织与细胞核检测

**H&E 组织检测：**
```python
from pathml.preprocessing import TissueDetectionHE

# 在 H&E 玻片中检测组织区域
transform = TissueDetectionHE(
    use_saturation=True,  # 使用 HSV 饱和度通道
    threshold=10,  # 强度阈值
    min_region_size=500  # 最小组织区域尺寸（像素）
)
```
- 创建二值组织掩膜
- 过滤小区域和伪影
- 将掩膜存储在 `tile.masks['tissue']`

**H&E 细胞核检测：**
```python
from pathml.preprocessing import NucleusDetectionHE

# 在 H&E 图像中检测细胞核
transform = NucleusDetectionHE(
    stain='hematoxylin',  # 使用苏木精通道
    threshold=0.3,
    min_nucleus_size=10
)
```
- 分离苏木精染色
- 通过阈值创建细胞核掩膜
- 将掩膜存储在 `tile.masks['nucleus']`

### 二值阈值处理

**二值阈值：**
```python
from pathml.preprocessing import BinaryThreshold

# 使用 Otsu 方法进行阈值处理
transform = BinaryThreshold(
    method='otsu',  # 'otsu' 或手动阈值
    invert=False
)

# 或指定手动阈值
transform = BinaryThreshold(threshold=128)
```

### 前景检测

**前景检测：**
```python
from pathml.preprocessing import ForegroundDetection

# 检测前景区域
transform = ForegroundDetection(
    threshold=0.5,
    min_region_size=1000,  # 最小尺寸（像素）
    use_saturation=True
)
```

## 掩膜修改变换

应用形态学操作清理掩膜：

**形态学开运算：**
```python
from pathml.preprocessing import MorphOpen

# 移除小物体和噪声
transform = MorphOpen(
    kernel_size=5,
    mask_name='tissue'  # 指定要修改的掩膜
)
```
- 先腐蚀后膨胀
- 去除小物体和噪声

**形态学闭运算：**
```python
from pathml.preprocessing import MorphClose

# 填充小孔洞
transform = MorphClose(
    kernel_size=5,
    mask_name='tissue'
)
```
- 先膨胀后腐蚀
- 填充掩膜中的小孔洞

## 染色归一化

### H&E 染色归一化

归一化不同玻片的 H&E 染色，解决染色程序和扫描仪差异：

```python
from pathml.preprocessing import StainNormalizationHE

# 归一化到参考玻片
transform = StainNormalizationHE(
    target='normalize',  # 'normalize', 'hematoxylin' 或 'eosin'
    stain_estimation_method='macenko',  # 'macenko' 或 'vahadane'
    tissue_mask_name=None  # 可选组织掩膜用于更准确估计
)
```

**目标模式：**
- `'normalize'` - 将两种染色归一化到参考
- `'hematoxylin'` - 仅提取苏木精通道
- `'eosin'` - 仅提取伊红通道

**染色估计方法：**
- `'macenko'` - Macenko 等人 2009 方法（更快、更稳定）
- `'vahadane'` - Vahadane 等人 2016 方法（更精确、更慢）

**高级参数：**
```python
transform = StainNormalizationHE(
    target='normalize',
    stain_estimation_method='macenko',
    target_od=None,  # 参考光密度矩阵（可选）
    target_concentrations=None,  # 目标染色浓度（可选）
    regularizer=0.1,  # vahadane 方法的正则化参数
    background_intensity=240  # 背景强度水平
)
```

**工作流程：**
1. 将 RGB 转换为光密度 (OD)
2. 估计染色矩阵 (H&E 向量)
3. 分解为染色浓度
4. 归一化到参考染色分布
5. 重建归一化 RGB 图像

**使用组织掩膜的示例：**
```python
from pathml.preprocessing import Pipeline, TissueDetectionHE, StainNormalizationHE

pipeline = Pipeline([
    TissueDetectionHE(),  # 先创建组织掩膜
    StainNormalizationHE(
        target='normalize',
        stain_estimation_method='macenko',
        tissue_mask_name='tissue'  # 使用组织掩膜提高估计精度
    )
])
```

## 质量控制变换

### 伪影检测

**H&E 切片伪影标记：**
```python
from pathml.preprocessing import LabelArtifactTileHE

# 标记含伪影的切片
transform = LabelArtifactTileHE(
    pen_threshold=0.5,  # 笔迹检测阈值
    bubble_threshold=0.5  # 气泡检测阈值
)
```
- 检测笔迹、气泡等伪影
- 标记受影响的切片以便过滤

**空白区域标记：**
```python
from pathml.preprocessing import LabelWhiteSpaceHE

# 标记过度空白区域
transform = LabelWhiteSpaceHE(
    threshold=0.9,  # 空白像素占比
    mask_name='white_space'
)
```
- 识别主要含背景的切片
- 适用于过滤无信息切片

## 多参数成像变换

### 细胞分割

**多参数成像分割：**
```python
from pathml.preprocessing import SegmentMIF

# 使用 Mesmer 深度学习模型分割细胞
transform = SegmentMIF(
    nuclear_channel='DAPI',  # 细胞核标记通道名
    cytoplasm_channel='CD45',  # 细胞质标记通道名
    model='mesmer',  # 深度学习分割模型
    image_resolution=0.5,  # 每像素微米数
    compartment='whole-cell'  # 'nuclear', 'cytoplasm' 或 'whole-cell'
)
```
- 使用 DeepCell Mesmer 模型进行细胞分割
- 需指定细胞核和细胞质通道
- 生成实例分割掩膜

**远程多参数成像分割：**
```python
from pathml.preprocessing import SegmentMIFRemote

# 使用 DeepCell API 远程推理
transform = SegmentMIFRemote(
    nuclear_channel='DAPI',
    cytoplasm_channel='CD45',
    model='mesmer',
    api_url='https://deepcell.org/api'
)
```
- 功能与 SegmentMIF 相同但使用远程 API
- 无需本地 GPU
- 适用于批量处理

### 标记物定量

**多参数成像定量：**
```python
from pathml.preprocessing import QuantifyMIF

# 按细胞量化标记物表达
transform = QuantifyMIF(
    segmentation_mask_name='cell_segmentation',
    markers=['CD3', 'CD4', 'CD8', 'CD20', 'CD45'],
    output_format='anndata'  # 或 'dataframe'
)
```
- 提取每个分割细胞的平均标记强度
- 计算形态特征（面积、周长等）
- 输出 AnnData 对象用于下游单细胞分析

### CODEX/Vectra 专用

**CODEX 多轮合并：**
```python
from pathml.preprocessing import CollapseRunsCODEX

# 整合多轮 CODEX 数据
transform = CollapseRunsCODEX(
    z_slice=2,  # 选择特定 z 层
    run_order=[0, 1, 2]  # 采集轮次顺序
)
```
- 合并多轮 CODEX 采集的通道
- 从 z 栈中选择焦平面

**Vectra 多轮合并：**
```python
from pathml.preprocessing import CollapseRunsVectra

# 处理 Vectra 多重免疫荧光数据
transform = CollapseRunsVectra(
    wavelengths=[520, 570, 620, 670, 780]  # 发射波长
)
```

## 构建综合流水线

### 基础 H&E 预处理流水线

```python
from pathml.preprocessing import (
    Pipeline,
    TissueDetectionHE,
    StainNormalizationHE,
    NucleusDetectionHE,
    MedianBlur,
    LabelWhiteSpaceHE
)

pipeline = Pipeline([
    # 1. 质量控制
    LabelWhiteSpaceHE(threshold=0.9),

    # 2. 降噪
    MedianBlur(kernel_size=3),

    # 3. 组织检测
    TissueDetectionHE(min_region_size=500),

    # 4. 染色归一化
    StainNormalizationHE(
        target='normalize',
        stain_estimation_method='macenko',
        tissue_mask_name='tissue'
    ),

    # 5. 细胞核检测
    NucleusDetectionHE(threshold=0.3)
])
```

### CODEX 多参数流水线

```python
from pathml.preprocessing import (
    Pipeline,
    CollapseRunsCODEX,
    SegmentMIF,
    QuantifyMIF
)

codex_pipeline = Pipeline([
    # 1. 整合多轮数据
    CollapseRunsCODEX(z_slice=2),

    # 2. 细胞分割
    SegmentMIF(
        nuclear_channel='DAPI',
        cytoplasm_channel='CD45',
        model='mesmer',
        image_resolution=0.377
    ),

    # 3. 标记物定量
    QuantifyMIF(
        segmentation_mask_name='cell_segmentation',
        markers=['CD3', 'CD4', 'CD8', 'CD20', 'PD1', 'PDL1'],
        output_format='anndata'
    )
])
```

### 含质量控制的高级流水线

```python
from pathml.preprocessing import (
    Pipeline,
    LabelWhiteSpaceHE,
    LabelArtifactTileHE,
    TissueDetectionHE,
    MorphOpen,
    MorphClose,
    StainNormalizationHE,
    AdaptiveHistogramEqualization
)

advanced_pipeline = Pipeline([
    # 阶段1: 质量控制
    LabelWhiteSpaceHE(threshold=0.85),
    LabelArtifactTileHE(pen_threshold=0.5, bubble_threshold=0.5),

    # 阶段2: 组织检测
    TissueDetectionHE(threshold=10, min_region_size=1000),
    MorphOpen(kernel_size=5, mask_name='tissue'),
    MorphClose(kernel_size=7, mask_name='tissue'),

    # 阶段3: 染色归一化
    StainNormalizationHE(
        target='normalize',
        stain_estimation_method='vahadane',
        tissue_mask_name='tissue'
    ),

    # 阶段4: 对比度增强
    AdaptiveHistogramEqualization(clip_limit=0.03, tile_grid_size=(8, 8))
])
```

## 运行流水线

### 单张玻片处理

```python
from pathml.core import SlideData

# 加载玻片
wsi = SlideData.from_slide("slide.svs")

# 生成切片
wsi.generate_tiles(level=1, tile_size=256, stride=256)

# 运行流水线
pipeline.run(wsi)

# 访问处理后的数据
for tile in wsi.tiles:
    normalized_image = tile.image
    tissue_mask = tile.masks.get('tissue')
    nucleus_mask = tile.masks.get('nucleus')
```

### 分布式批处理

```python
from pathml.core import SlideDataset
from dask.distributed import Client
import glob

# 启动 Dask 客户端
client = Client(n_workers=8, threads_per_worker=2, memory_limit='4GB')

# 创建数据集
slide_paths = glob.glob("data/*.svs")
dataset = SlideDataset(
    slide_paths,
    tile_size=512,
    stride=512,
    level=1
)

# 并行运行流水线
dataset.run(
    pipeline,
    distributed=True,
    client=client
)

# 保存结果
dataset.to_hdf5("processed_dataset.h5")

client.close()
```

### 条件化流水线执行

仅对满足特定条件的切片执行变换：

```python
# 处理前过滤切片
wsi.generate_tiles(level=1, tile_size=256)

# 仅对组织切片运行流水线
for tile in wsi.tiles:
    if tile.masks.get('tissue') is not None:
        pipeline.run(tile)
```

## 性能优化

### 内存管理

```python
# 分批处理大型数据集
batch_size = 100
for i in range(0, len(slide_paths), batch_size):
    batch_paths = slide_paths[i:i+batch_size]
    batch_dataset = SlideDataset(batch_paths)
    batch_dataset.run(p

# 应用自定义操作
        processed = self.custom_operation(image, self.param1, self.param2)

        # 更新切片
        tile.image = processed

        return tile

    def custom_operation(self, image, param1, param2):
        # 实现自定义逻辑
        return processed_image

# 在流程中使用
pipeline = Pipeline([
    CustomTransform(param1=10, param2=0.5),
    # ... 其他转换操作
])
```

## 最佳实践

1. **合理排序转换操作：**
   - 质量控制优先（LabelWhiteSpace, LabelArtifact）
   - 早期进行降噪处理（Blur）
   - 染色归一化前先进行组织检测
   - 颜色相关操作前完成染色归一化

2. **使用组织掩模进行染色归一化：**
   - 通过排除背景提高准确性
   - 先执行`TissueDetectionHE()`再执行`StainNormalizationHE(tissue_mask_name='tissue')`

3. **应用形态学操作清理掩模：**
   - `MorphOpen`去除细小假阳性区域
   - `MorphClose`填充微小间隙

4. **利用分布式处理应对大型数据集：**
   - 使用Dask进行并行计算
   - 根据可用资源配置工作节点

5. **保存中间结果：**
   - 将处理数据存储至HDF5以便复用
   - 避免重新计算资源密集型转换操作

6. **在样本图像上验证预处理：**
   - 可视化中间步骤
   - 批量处理前在代表性样本上调整参数

7. **处理边缘情况：**
   - 下游操作前检查空掩模
   - 执行高成本计算前验证切片质量

## 常见问题与解决方案

**问题：染色归一化产生伪影**
- 使用组织掩模排除背景
- 尝试不同染色估计方法（macenko vs. vahadane）
- 确认光学密度参数与图像匹配

**问题：流程执行时内存不足**
- 减少Dask工作节点数量
- 缩小切片尺寸
- 在较低金字塔层级处理图像
- 在Dask客户端启用memory_limit参数

**问题：组织检测遗漏区域**
- 调整阈值参数
- 使用饱和度通道：`use_saturation=True`
- 减小min_region_size以捕获小组织片段

**问题：细胞核检测不准确**
- 验证染色分离质量（可视化苏木素通道）
- 调整阈值参数
- 细胞核检测前先进行染色归一化

## 附加资源

- **PathML预处理API：** https://pathml.readthedocs.io/en/latest/api_preprocessing_reference.html
- **染色归一化方法：**
  - Macenko等 2009: "A method for normalizing histology slides for quantitative analysis"
  - Vahadane等 2016: "Structure-Preserving Color Normalization and Sparse Stain Separation"
- **DeepCell Mesmer：** https://www.deepcell.org/（细胞分割模型）
