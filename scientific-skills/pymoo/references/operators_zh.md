# Pymoo 遗传算子参考指南

pymoo 遗传算子的全面参考手册。

## 采样算子

采样算子在优化开始时初始化种群。

### 随机采样
**目的：** 生成随机初始解  
**类型：**  
- `FloatRandomSampling`：连续变量  
- `BinaryRandomSampling`：二进制变量  
- `IntegerRandomSampling`：整型变量  
- `PermutationRandomSampling`：基于排列的问题  

**用法：**  
```python
from pymoo.operators.sampling.rnd import FloatRandomSampling
sampling = FloatRandomSampling()
```

### 拉丁超立方采样 (LHS)
**目的：** 空间填充初始种群  
**优势：** 比随机采样具有更好的搜索空间覆盖  
**类型：**  
- `LHS`：标准拉丁超立方采样  

**用法：**  
```python
from pymoo.operators.sampling.lhs import LHS
sampling = LHS()
```

### 自定义采样
通过 Population 对象或 NumPy 数组提供初始种群

## 选择算子

选择算子为繁殖选择父代。

### 锦标赛选择
**目的：** 通过锦标赛竞争选择父代  
**机制：** 随机选择 k 个个体，择优选取  
**参数：**  
- `pressure`：锦标赛规模（默认：2）  
- `func_comp`：比较函数  

**用法：**  
```python
from pymoo.operators.selection.tournament import TournamentSelection
selection = TournamentSelection(pressure=2)
```

### 随机选择
**目的：** 均匀随机选择父代  
**适用场景：** 基线算法或侧重探索的算法  

**用法：**  
```python
from pymoo.operators.selection.rnd import RandomSelection
selection = RandomSelection()
```

## 交叉算子

交叉算子重组父代解以创建后代。

### 连续变量交叉

#### 模拟二进制交叉 (SBX)
**目的：** 连续优化的主要交叉算子  
**机制：** 模拟二进制编码变量的单点交叉  
**参数：**  
- `prob`：交叉概率（默认：0.9）  
- `eta`：分布指数（默认：15）  
  - 高 eta → 后代更接近父代  
  - 低 eta → 探索性更强  

**用法：**  
```python
from pymoo.operators.crossover.sbx import SBX
crossover = SBX(prob=0.9, eta=15)
```

**字符串简写：** `"real_sbx"`

#### 差分进化交叉
**目的：** 差分进化专用重组  
**变体：**  
- `DE/rand/1/bin`  
- `DE/best/1/bin`  
- `DE/current-to-best/1/bin`  

**参数：**  
- `CR`：交叉率  
- `F`：缩放因子  

### 二进制变量交叉

#### 单点交叉
**目的：** 单点切割交换  
**用法：**  
```python
from pymoo.operators.crossover.pntx import SinglePointCrossover
crossover = SinglePointCrossover()
```

#### 两点交叉
**目的：** 两点间切割交换  
**用法：**  
```python
from pymoo.operators.crossover.pntx import TwoPointCrossover
crossover = TwoPointCrossover()
```

#### K点交叉
**目的：** 多点切割  
**参数：**  
- `n_points`：交叉点数量  

#### 均匀交叉
**目的：** 每个基因独立继承自任一父代  
**参数：**  
- `prob`：基因交换概率（默认：0.5）  

**用法：**  
```python
from pymoo.operators.crossover.ux import UniformCrossover
crossover = UniformCrossover(prob=0.5)
```

#### 半均匀交叉 (HUX)
**目的：** 精确交换半数差异基因  
**优势：** 保持遗传多样性  

### 排列交叉

#### 顺序交叉 (OX)
**目的：** 保留父代的相对顺序  
**适用场景：** 旅行商问题、调度问题  

**用法：**  
```python
from pymoo.operators.crossover.ox import OrderCrossover
crossover = OrderCrossover()
```

#### 边重组交叉 (ERX)
**目的：** 保留父代的边连接信息  
**适用场景：** 边连通性重要的路径问题  

#### 部分映射交叉 (PMX)
**目的：** 交换片段同时保持排列有效性  

## 变异算子

变异算子引入多样性以维持种群变异。

### 连续变量变异

#### 多项式变异 (PM)
**目的：** 连续优化的主要变异算子  
**机制：** 多项式概率分布  
**参数：**  
- `prob`：单变量变异概率  
- `eta`：分布指数（默认：20）  
  - 高 eta → 微扰动  
  - 低 eta → 强扰动  

**用法：**  
```python
from pymoo.operators.mutation.pm import PM
mutation = PM(prob=None, eta=20)  # prob=None 表示 1/n_var
```

**字符串简写：** `"real_pm"`

**概率指导：**  
- `None` 或 `1/n_var`：标准推荐  
- 更高值增强探索性  
- 更低值增强开发性  

### 二进制变量变异

#### 位翻转变异
**目的：** 按概率翻转比特位  
**参数：**  
- `prob`：单比特翻转概率  

**用法：**  
```python
from pymoo.operators.mutation.bitflip import BitflipMutation
mutation = BitflipMutation(prob=0.05)
```

### 整型变量变异

#### 整型多项式变异
**目的：** 适配整型的多项式变异  
**确保：** 变异后生成有效整数值  

### 排列变异

#### 倒位变异
**目的：** 反转排列片段  
**适用场景：** 保持顺序结构  

**用法：**  
```python
from pymoo.operators.mutation.inversion import InversionMutation
mutation = InversionMutation()
```

#### 扰乱变异
**目的：** 随机打乱片段顺序  

### 自定义变异
通过继承 `Mutation` 类定义自定义变异  

## 修复算子

修复算子修正约束违反或确保解的可行性。

### 舍入修复
**目的：** 四舍五入到最近有效值  
**适用场景：** 带边界约束的整型/离散变量  

### 反弹修复
**目的：** 将越界值反射回可行域  
**适用场景：** 带边界约束的连续问题  

### 投影修复
**目的：** 将不可行解投影到可行域  
**适用场景：** 线性约束  

### 自定义修复
**目的：** 领域特定约束处理  
**实现：** 继承 `Repair` 类  

**示例：**  
```python
from pymoo.core.repair import Repair

class MyRepair(Repair):
    def _do(self, problem, X, **kwargs):
        # 修改X以满足约束
        # 返回修复后的X
        return X
```

## 算子配置指南

### 参数调优

**交叉概率：**  
- 高值（0.8-0.95）：多数问题标准设置  
- 低值：侧重变异操作  

**变异概率：**  
- `1/n_var`：标准推荐  
- 更高值：增强探索性，收敛变慢  
- 更低值：加速收敛，可能早熟  

**分布指数 (eta)：**  
- 交叉 eta (15-30)：高值适合局部搜索  
- 变异 eta (20-50)：高值侧重开发  

### 问题特定选择

**连续问题：**  
- 交叉：SBX  
- 变异：多项式变异  
- 选择：锦标赛  

**二进制问题：**  
- 交叉：两点或均匀交叉  
- 变异：位翻转  
- 选择：锦标赛  

**排列问题：**  
- 交叉：顺序交叉 (OX)  
- 变异：倒位或扰乱变异  
- 选择：锦标赛  

**混合变量问题：**  
- 按变量类型选用对应算子  
- 确保算子兼容性  

### 基于字符串的配置

pymoo 支持便捷的字符串算子配置：

```python
from pymoo.algorithms.soo.nonconvex.ga import GA

algorithm = GA(
    pop_size=100,
    sampling="real_random",
    crossover="real_sbx",
    mutation="real_pm"
)
```

**可用字符串：**  
- 采样：`"real_random"`, `"real_lhs"`, `"bin_random"`, `"perm_random"`  
- 交叉：`"real_sbx"`, `"real_de"`, `"int_sbx"`, `"bin_ux"`, `"bin_hux"`  
- 变异：`"real_pm"`, `"int_pm"`, `"bin_bitflip"`, `"perm_inv"`  

## 算子组合示例

### 标准连续遗传算法：
```python
from pymoo.operators.sampling.rnd import FloatRandomSampling
from pymoo.operators.crossover.sbx import SBX
from pymoo.operators.mutation.pm import PM
from pymoo.operators.selection.tournament import TournamentSelection

sampling = FloatRandomSampling()
crossover = SBX(prob=0.9, eta=15)
mutation = PM(eta=20)
selection = TournamentSelection()
```

### 二进制遗传算法：
```python
from pymoo.operators.sampling.rnd import BinaryRandomSampling
from pymoo.operators.crossover.pntx import TwoPointCrossover
from pymoo.operators.mutation.bitflip import BitflipMutation

sampling = BinaryRandomSampling()
crossover = TwoPointCrossover()
mutation = BitflipMutation(prob=0.05)
```

### 排列遗传算法 (TSP)：
```python
from pymoo.operators.sampling.rnd import PermutationRandomSampling
from pymoo.operators.crossover.ox import OrderCrossover
from pymoo.operators.mutation.inversion import InversionMutation

sampling = PermutationRandomSampling()
crossover = OrderCrossover()
mutation = InversionMutation()
```
