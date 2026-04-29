# Pymatgen 核心类参考

本文档记录了 `pymatgen.core` 中的基础类，这些类构成了材料分析的基石。

## 架构原则

Pymatgen 遵循面向对象设计，其中元素、位点和结构均表示为对象。该框架在晶体表示中强调周期性边界条件，同时保持对分子系统的灵活性。

**单位约定**：pymatgen 中所有单位通常默认为原子单位：
- 长度：埃（Å）
- 能量：电子伏特（eV）
- 角度：度

## 元素与周期表

### Element 类
表示周期表元素及其综合属性。

**创建方法：**
```python
from pymatgen.core import Element

# 通过符号创建
si = Element("Si")
# 通过原子序数创建
si = Element.from_Z(14)
# 通过名称创建
si = Element.from_name("silicon")
```

**关键属性：**
- `atomic_mass`：原子质量（amu）
- `atomic_radius`：原子半径（埃）
- `electronegativity`：鲍林电负性
- `ionization_energy`：第一电离能（eV）
- `common_oxidation_states`：常见氧化态列表
- `is_metal`、`is_halogen`、`is_noble_gas` 等：布尔属性
- `X`：元素符号（字符串形式）

### Species 类
扩展 Element 类以表示带电离子和特定氧化态。

```python
from pymatgen.core import Species

# 创建 Fe2+ 离子
fe2 = Species("Fe", 2)
# 或显式指定符号
fe2 = Species("Fe", +2)
```

### DummySpecies 类
用于特殊结构表示（如空位）的占位原子。

```python
from pymatgen.core import DummySpecies

vacancy = DummySpecies("X")
```

## Composition 类

表示化学式和组成，支持化学分析和操作。

### 创建
```python
from pymatgen.core import Composition

# 通过字符串化学式
comp = Composition("Fe2O3")
# 通过字典
comp = Composition({"Fe": 2, "O": 3})
# 通过重量字典
comp = Composition.from_weight_dict({"Fe": 111.69, "O": 48.00})
```

### 关键方法
- `get_reduced_formula_and_factor()`：返回约简化学式及乘数因子
- `oxi_state_guesses()`：尝试确定氧化态
- `replace(replacements_dict)`：替换元素
- `add_charges_from_oxi_state_guesses()`：推断并添加氧化态
- `is_element`：检查是否为单质

### 关键属性
- `weight`：分子量
- `reduced_formula`：约简化学式
- `hill_formula`：希尔表示法（C、H优先，其余按字母顺序）
- `num_atoms`：原子总数
- `chemical_system`：按字母排序的元素（如"Fe-O"）
- `element_composition`：元素与数量的字典

## Lattice 类

定义晶体结构的晶胞几何。

### 创建
```python
from pymatgen.core import Lattice

# 通过晶格参数
lattice = Lattice.from_parameters(a=3.84, b=3.84, c=3.84,
                                  alpha=120, beta=90, gamma=60)

# 通过矩阵（行向量为晶格向量）
lattice = Lattice([[3.84, 0, 0],
                   [0, 3.84, 0],
                   [0, 0, 3.84]])

# 立方晶格
lattice = Lattice.cubic(3.84)
# 六方晶格
lattice = Lattice.hexagonal(a=2.95, c=4.68)
```

### 关键方法
- `get_niggli_reduced_lattice()`：返回 Niggli 约简晶格
- `get_distance_and_image(frac_coords1, frac_coords2)`：考虑周期性边界条件的分数坐标间距
- `get_all_distances(frac_coords1, frac_coords2)`：包含周期性镜像的所有间距

### 关键属性
- `volume`：晶胞体积（Å³）
- `abc`：晶格参数元组 (a, b, c)
- `angles`：晶格角度元组 (alpha, beta, gamma)
- `matrix`：晶格向量 3x3 矩阵
- `reciprocal_lattice`：倒易晶格对象
- `is_orthogonal`：晶格向量是否正交

## Sites 类

### Site 类
表示非周期性系统中的原子位置。

```python
from pymatgen.core import Site

site = Site("Si", [0.0, 0.0, 0.0])  # 物种及笛卡尔坐标
```

### PeriodicSite 类
表示周期性晶格中具有分数坐标的原子位置。

```python
from pymatgen.core import PeriodicSite

site = PeriodicSite("Si", [0.5, 0.5, 0.5], lattice)  # 物种、分数坐标、晶格
```

**关键方法：**
- `distance(other_site)`：到另一位点的距离
- `is_periodic_image(other_site)`：检查是否为周期性镜像

**关键属性：**
- `species`：位点上的物种或元素
- `coords`：笛卡尔坐标
- `frac_coords`：分数坐标（仅 PeriodicSite）
- `x`、`y`、`z`：笛卡尔坐标分量

## Structure 类

表示周期性位点集合构成的晶体结构。`Structure` 可变，`IStructure` 不可变。

### 创建
```python
from pymatgen.core import Structure, Lattice

# 从头创建
coords = [[0, 0, 0], [0.75, 0.5, 0.75]]
lattice = Lattice.from_parameters(a=3.84, b=3.84, c=3.84,
                                  alpha=120, beta=90, gamma=60)
struct = Structure(lattice, ["Si", "Si"], coords)

# 从文件读取（自动检测格式）
struct = Structure.from_file("POSCAR")
struct = Structure.from_file("structure.cif")

# 通过空间群创建
struct = Structure.from_spacegroup("Fm-3m", Lattice.cubic(3.5),
                                   ["Si"], [[0, 0, 0]])
```

### 文件输入/输出
```python
# 写入文件（根据扩展名推断格式）
struct.to(filename="output.cif")
struct.to(filename="POSCAR")
struct.to(filename="structure.xyz")

# 获取字符串表示
cif_string = struct.to(fmt="cif")
poscar_string = struct.to(fmt="poscar")
```

### 关键方法

**结构修改：**
- `append(species, coords)`：添加位点
- `insert(i, species, coords)`：在索引处插入位点
- `remove_sites(indices)`：按索引移除位点
- `replace(i, species)`：替换索引处物种
- `apply_strain(strain)`：施加应变
- `perturb(distance)`：随机扰动原子位置
- `make_supercell(scaling_matrix)`：创建超胞
- `get_primitive_structure()`：获取原胞

**分析：**
- `get_distance(i, j)`：位点 i 和 j 的间距
- `get_neighbors(site, r)`：获取半径 r 内的近邻
- `get_all_neighbors(r)`：获取所有位点的近邻
- `get_space_group_info()`：获取空间群信息
- `matches(other_struct)`：检查结构匹配性

**插值：**
- `interpolate(end_structure, nimages)`：结构间插值

### 关键属性
- `lattice`：晶格对象
- `species`：各位点物种列表
- `sites`：PeriodicSite 对象列表
- `num_sites`：位点数量
- `volume`：结构体积
- `density`：密度（g/cm³）
- `composition`：Composition 对象
- `formula`：化学式
- `distance_matrix`：位点间距矩阵

## Molecule 类

表示非周期性原子集合。`Molecule` 可变，`IMolecule` 不可变。

### 创建
```python
from pymatgen.core import Molecule

# 从头创建
coords = [[0.00, 0.00, 0.00],
          [0.00, 0.00, 1.08]]
mol
