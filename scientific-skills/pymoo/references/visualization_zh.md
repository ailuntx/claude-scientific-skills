# Pymoo 可视化参考指南

pymoo 可视化功能的全面参考手册。

## 概述

Pymoo 提供八种可视化类型用于分析多目标优化结果。所有图表均基于 matplotlib 封装，并接受标准 matplotlib 关键字参数进行自定义。

## 核心可视化类型

### 1. 散点图
**用途：** 可视化二维、三维或更高维度的目标空间  
**最佳场景：** 帕累托前沿、解分布、算法比较  

**用法：**
```python
from pymoo.visualization.scatter import Scatter

# 二维散点图
plot = Scatter()
plot.add(result.F, color="red", label="算法A")
plot.add(ref_pareto_front, color="black", alpha=0.3, label="真实PF")
plot.show()

# 三维散点图
plot = Scatter(title="三维帕累托前沿")
plot.add(result.F)
plot.show()
```

**参数：**
- `title`: 图表标题
- `figsize`: 图形尺寸元组 (宽, 高)
- `legend`: 显示图例 (默认: True)
- `labels`: 坐标轴标签列表

**添加方法参数：**
- `color`: 颜色设定
- `alpha`: 透明度 (0-1)
- `s`: 标记尺寸
- `marker`: 标记样式
- `label`: 图例标签

**高维投影：**
目标数>3时自动创建散点图矩阵

### 2. 平行坐标图 (PCP)
**用途：** 跨多目标比较多个解  
**最佳场景：** 多目标问题、算法性能比较  

**机制：** 每个垂直轴代表一个目标，线条连接每个解的目标值  

**用法：**
```python
from pymoo.visualization.pcp import PCP

plot = PCP()
plot.add(result.F, color="blue", alpha=0.5)
plot.add(reference_set, color="red", alpha=0.8)
plot.show()
```

**参数：**
- `title`: 图表标题
- `figsize`: 图形尺寸
- `labels`: 目标标签
- `bounds`: 各目标归一化边界 (最小值, 最大值)
- `normalize_each_axis`: 按轴归一化至[0,1] (默认: True)

**最佳实践：**
- 不同目标尺度需归一化
- 使用透明度处理重叠线条
- 限制解的数量保证清晰度 (<1000)

### 3. 热力图
**用途：** 展示解密度与分布模式  
**最佳场景：** 理解解聚类、识别分布间隙  

**用法：**
```python
from pymoo.visualization.heatmap import Heatmap

plot = Heatmap(title="解密度分布")
plot.add(result.F)
plot.show()
```

**参数：**
- `bins`: 每维度分箱数 (默认: 20)
- `cmap`: 色彩映射名称 (如 "viridis", "plasma", "hot")
- `norm`: 归一化方法

**解读：**
- 亮色区域：高解密度
- 暗色区域：少量或无解
- 揭示分布均匀性

### 4. 花瓣图
**用途：** 多目标的径向表示  
**最佳场景：** 跨目标比较单个解  

**结构：** 每个"花瓣"代表一个目标，长度表示目标值  

**用法：**
```python
from pymoo.visualization.petal import Petal

plot = Petal(title="解对比", bounds=[min_vals, max_vals])
plot.add(result.F[0], color="blue", label="解1")
plot.add(result.F[1], color="red", label="解2")
plot.show()
```

**参数：**
- `bounds`: 各目标归一化边界 [最小值, 最大值]
- `labels`: 目标名称
- `reverse`: 反转特定目标（用于最小化显示）

**适用场景：**
- 少量解间的决策
- 向利益相关者展示权衡关系

### 5. 雷达图
**用途：** 多准则性能剖面  
**最佳场景：** 比较解的特征  

**类似图表：** 花瓣图（但采用顶点连接形式）  

**用法：**
```python
from pymoo.visualization.radar import Radar

plot = Radar(bounds=[min_vals, max_vals])
plot.add(solution_A, label="设计A")
plot.add(solution_B, label="设计B")
plot.show()
```

### 6. Radviz
**用途：** 降维可视化  
**最佳场景：** 高维数据探索、模式识别  

**机制：** 将高维点投影到二维圆上，维度锚点位于圆周  

**用法：**
```python
from pymoo.visualization.radviz import Radviz

plot = Radviz(title="高维解空间")
plot.add(result.F, color="blue", s=30)
plot.show()
```

**参数：**
- `endpoint_style`: 锚点可视化样式
- `labels`: 维度标签

**解读：**
- 靠近锚点：该维度值高
- 中心点：各维度均衡
- 聚类：相似解

### 7. 星坐标图
**用途：** 替代性高维可视化  
**最佳场景：** 比较多维数据集  

**机制：** 每个维度作为从原点出发的轴，基于值绘制点  

**用法：**
```python
from pymoo.visualization.star_coordinate import StarCoordinate

plot = StarCoordinate()
plot.add(result.F)
plot.show()
```

**参数：**
- `axis_style`: 坐标轴样式
- `axis_extension`: 超出最大值的轴延伸
- `labels`: 维度标签

### 8. 视频/动画
**用途：** 展示优化过程随时间变化  
**最佳场景：** 理解收敛行为、演示展示  

**用法：**
```python
from pymoo.visualization.video import Video

# 从算法历史创建动画
anim = Video(result.algorithm)
anim.save("optimization_progress.mp4")
```

**要求：**
- 算法需存储历史记录 (在minimize中使用`save_history=True`)
- 视频导出需安装ffmpeg

**自定义：**
- 帧率
- 每帧图表类型
- 叠加信息（代数、超体积等）

## 高级功能

### 多数据集叠加

所有图表类型支持添加多个数据集：

```python
plot = Scatter(title="算法比较")
plot.add(nsga2_result.F, color="red", alpha=0.5, label="NSGA-II")
plot.add(nsga3_result.F, color="blue", alpha=0.5, label="NSGA-III")
plot.add(true_pareto_front, color="black", linewidth=2, label="真实PF")
plot.show()
```

### 自定义样式

直接传递 matplotlib 参数：

```python
plot = Scatter(
    title="我的结果",
    figsize=(10, 8),
    tight_layout=True
)
plot.add(
    result.F,
    color="red",
    marker="o",
    s=50,
    alpha=0.7,
    edgecolors="black",
    linewidth=0.5
)
```

### 归一化处理

将目标归一化至[0,1]进行公平比较：

```python
plot = PCP(normalize_each_axis=True, bounds=[min_bounds, max_bounds])
```

### 保存文件

保存图表而非显示：

```python
plot = Scatter()
plot.add(result.F)
plot.save("my_plot.png", dpi=300)
```

## 可视化选择指南

**根据问题类型选择：**

| 问题类型 | 主图表 | 辅助图表 |
|----------|--------|----------|
| 2目标 | 散点图 | 热力图 |
| 3目标 | 3D散点图 | 平行坐标图 |
| 多目标(4-10) | 平行坐标图 | Radviz |
| 多目标(>10) | Radviz | 星坐标图 |
| 解比较 | 花瓣图/雷达图 | 平行坐标图 |
| 算法收敛 | 视频 | 散点图(最终) |
| 分布分析 | 热力图 | 散点图 |

**组合策略：**
- 散点图 + 热力图：整体分布 + 密度
- PCP + 花瓣图：种群概览 + 个体解
- 散点图 + 视频：最终结果 + 收敛过程

## 常用可视化工作流

### 1. 算法比较
```python
from pymoo.visualization.scatter import Scatter

plot = Scatter(title="ZDT1算法比较")
plot.add(ga_result.F, color="blue", label="GA", alpha=0.6)
plot.add(nsga2_result.F, color="red", label="NSGA-II", alpha=0.6)
plot.add(zdt1.pareto_front(), color="black", label="真实PF")
plot.show()
```

### 2. 多目标分析
```python
from pymoo.visualization.pcp import PCP

plot = PCP(
    title="5目标DTLZ2结果",
    labels=["f1", "f2", "f3", "f4", "f5"],
    normalize_each_axis=True
)
plot.add(result.F, alpha=0.3)
plot.show()
```

### 3. 决策支持
```python
from pymoo.visualization.petal import Petal

# 比较前3个解
candidates = result.F[:3]

plot = Petal(
    title="前3候选解",
    bounds=[result.F.min(axis=0), result.F.max(axis=0)],
    labels=["成本", "重量", "效率", "安全性"]
)
for i, sol in enumerate(candidates):
    plot.add(sol, label=f"解{i+1}")
plot.show()
```

### 4. 收敛可视化
```python
from pymoo.optimize import minimize

# 启用历史记录
result = minimize(
    problem,
    algorithm,
    ('n_gen', 200),
    seed=1,
    save_history=True,
    verbose=False
)

# 创建收敛图
from pymoo.visualization.scatter import Scatter

plot = Scatter(title="代际收敛过程")
for gen in [0, 50, 100, 150, 200]:
    F = result.history[gen].opt.get("F")
    plot.add(F, alpha=0.5, label=f"第{gen}代")
plot.show()
```

## 技巧与最佳实践

1. **合理使用透明度：** 重叠点使用 `alpha=0.3-0.7`
2. **目标归一化：** 不同尺度需归一化
3. **清晰标注：** 始终提供有意义的标签和图例
4. **限制数据点：** >10000点？采样或使用热力图
5. **配色方案：** 使用色盲友好调色板
6. **保存高清图：** 发表用图使用 `dpi=300`
7. **交互探索：** 考虑使用 plotly 交互图表
8. **组合视图：** 多角度展示进行全面分析
