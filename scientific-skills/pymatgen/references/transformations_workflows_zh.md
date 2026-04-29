# Pymatgen 转换框架与常见工作流

本文档介绍 pymatgen 的转换框架，并提供材料科学常见工作流的实践方案。

## 转换框架

转换框架提供系统化修改结构的方法，同时记录修改历史。

### 标准转换

位于 `pymatgen.transformations.standard_transformations`。

#### 超胞转换

使用任意缩放矩阵创建超胞。

```python
from pymatgen.transformations.standard_transformations import SupercellTransformation

# 简单2x2x2超胞
trans = SupercellTransformation([[2,0,0], [0,2,0], [0,0,2]])
new_struct = trans.apply_transformation(struct)

# 非正交超胞
trans = SupercellTransformation([[2,1,0], [0,2,0], [0,0,2]])
new_struct = trans.apply_transformation(struct)
```

#### 元素替换转换

替换结构中的元素。

```python
from pymatgen.transformations.standard_transformations import SubstitutionTransformation

# 将所有Fe替换为Mn
trans = SubstitutionTransformation({"Fe": "Mn"})
new_struct = trans.apply_transformation(struct)

# 部分替换（50% Fe -> Mn）
trans = SubstitutionTransformation({"Fe": {"Mn": 0.5, "Fe": 0.5}})
new_struct = trans.apply_transformation(struct)
```

#### 元素移除转换

从结构中移除特定元素。

```python
from pymatgen.transformations.standard_transformations import RemoveSpeciesTransformation

trans = RemoveSpeciesTransformation(["H"])  # 移除所有氢
new_struct = trans.apply_transformation(struct)
```

#### 无序结构有序化转换

将具有部分占位的无序结构有序化。

```python
from pymatgen.transformations.standard_transformations import OrderDisorderedStructureTransformation

trans = OrderDisorderedStructureTransformation()
new_struct = trans.apply_transformation(disordered_struct)
```

#### 原胞转换

转换为原胞。

```python
from pymatgen.transformations.standard_transformations import PrimitiveCellTransformation

trans = PrimitiveCellTransformation()
primitive_struct = trans.apply_transformation(struct)
```

#### 常规晶胞转换

转换为常规晶胞。

```python
from pymatgen.transformations.standard_transformations import ConventionalCellTransformation

trans = ConventionalCellTransformation()
conventional_struct = trans.apply_transformation(struct)
```

#### 旋转转换

旋转结构。

```python
from pymatgen.transformations.standard_transformations import RotationTransformation

# 按轴和角度旋转
trans = RotationTransformation([0, 0, 1], 45)  # 绕z轴旋转45°
new_struct = trans.apply_transformation(struct)
```

#### 弛豫结构缩放转换

缩放晶格以匹配弛豫结构。

```python
from pymatgen.transformations.standard_transformations import ScaleToRelaxedTransformation

trans = ScaleToRelaxedTransformation(relaxed_struct)
scaled_struct = trans.apply_transformation(unrelaxed_struct)
```

### 高级转换

位于 `pymatgen.transformations.advanced_transformations`。

#### 结构枚举转换

从无序结构中枚举所有对称性不同的有序结构。

```python
from pymatgen.transformations.advanced_transformations import EnumerateStructureTransformation

# 枚举每个晶胞最多8个原子的结构
trans = EnumerateStructureTransformation(max_cell_size=8)
structures = trans.apply_transformation(struct, return_ranked_list=True)

# 返回排序结构列表
for s in structures[:5]:  # 前5个结构
    print(f"能量: {s['energy']}, 结构: {s['structure']}")
```

#### 磁序转换

枚举磁序结构。

```python
from pymatgen.transformations.advanced_transformations import MagOrderingTransformation

# 指定每个元素的磁矩
trans = MagOrderingTransformation({"Fe": 5.0, "Ni": 2.0})
mag_structures = trans.apply_transformation(struct, return_ranked_list=True)
```

#### 掺杂转换

系统化掺杂结构。

```python
from pymatgen.transformations.advanced_transformations import DopingTransformation

# 用Mn替换12.5%的Fe位点
trans = DopingTransformation("Mn", min_length=10)
doped_structs = trans.apply_transformation(struct, return_ranked_list=True)
```

#### 电荷平衡转换

通过氧化态调控平衡结构电荷。

```python
from pymatgen.transformations.advanced_transformations import ChargeBalanceTransformation

trans = ChargeBalanceTransformation("Li")
charged_struct = trans.apply_transformation(struct)
```

#### 表面薄片转换

生成表面薄片。

```python
from pymatgen.transformations.advanced_transformations import SlabTransformation

trans = SlabTransformation(
    miller_index=[1, 0, 0],
    min_slab_size=10,
    min_vacuum_size=10,
    shift=0,
    lll_reduce=True
)
slab = trans.apply_transformation(struct)
```

### 链式转换

```python
from pymatgen.alchemy.materials import TransformedStructure

# 创建记录历史的转换结构
ts = TransformedStructure(struct, [])

# 应用多个转换
ts.append_transformation(SupercellTransformation([[2,0,0],[0,2,0],[0,0,2]]))
ts.append_transformation(SubstitutionTransformation({"Fe": "Mn"}))
ts.append_transformation(PrimitiveCellTransformation())

# 获取最终结构
final_struct = ts.final_structure

# 查看转换历史
print(ts.history)
```

## 常见工作流

### 工作流1：高通量结构生成

为筛选研究生成多个结构。

```python
from pymatgen.core import Structure
from pymatgen.transformations.standard_transformations import (
    SubstitutionTransformation,
    SupercellTransformation
)
from pymatgen.io.vasp.sets import MPRelaxSet

# 初始结构
base_struct = Structure.from_file("POSCAR")

# 定义掺杂元素
dopants = ["Mn", "Co", "Ni", "Cu"]
structures = {}

for dopant in dopants:
    # 创建掺杂结构
    trans = SubstitutionTransformation({"Fe": dopant})
    new_struct = trans.apply_transformation(base_struct)

    # 生成VASP输入
    vasp_input = MPRelaxSet(new_struct)
    vasp_input.write_input(f"./calcs/Fe_{dopant}")

    structures[dopant] = new_struct

print(f"已生成 {len(structures)} 个结构")
```

### 工作流2：相图构建

基于Materials Project数据构建并分析相图。

```python
from mp_api.client import MPRester
from pymatgen.analysis.phase_diagram import PhaseDiagram, PDPlotter
from pymatgen.core import Composition

# 从Materials Project获取数据
with MPRester() as mpr:
    entries = mpr.get_entries_in_chemsys("Li-Fe-O")

# 构建相图
pd = PhaseDiagram(entries)

# 分析特定成分
comp = Composition("LiFeO2")
e_above_hull = pd.get_e_above_hull(entries[0])

# 获取分解产物
decomp = pd.get_decomposition(comp)
print(f"分解产物: {decomp}")

# 可视化
plotter = PDPlotter(pd)
plotter.show()
```

### 工作流3：表面能计算

通过薄片计算表面能。

```python
from pymatgen.core.surface import SlabGenerator, generate_all_slabs
from pymatgen.io.vasp.sets import MPStaticSet, MPRelaxSet
from pymatgen.core import Structure

# 读取体相结构
bulk = Structure.from_file("bulk_POSCAR")

# 获取体相能量（来自先前计算）
from pymatgen.io.vasp import Vasprun
bulk_vasprun = Vasprun("bulk/vasprun.xml")
bulk_energy_per_atom = bulk_vasprun.final_energy / len(bulk)

# 生成薄片
miller_indices = [(1,0,0), (1,1,0), (1,1,1)]
surface_energies = {}

for miller in miller_indices:
    slabgen = SlabGenerator(
        bulk,
        miller_index=miller,
        min_slab_size=10,
        min_vacuum_size=15,
        center_slab=True
    )

    slab = slabgen.get_slabs()[0]

    # 为薄片写入VASP输入
    relax = MPRelaxSet(slab)
    relax.write_input(f"./slab_{miller[0]}{miller[1]}{miller[2]}")

    # 计算完成后计算表面能：
    # slab_vasprun = Vasprun(f"slab_{miller[0]}{miller[1]}{miller[2]}/vasprun.xml")
    # slab_energy = slab_vasprun.final_energy
    # n_atoms = len(slab)
    # area = slab.surface_area  # 单位：Ų
    #
    # # 表面能 (J/m²)
    # surf_energy = (slab_energy - n_atoms * bulk_energy_per_atom) / (2 * area)
    # surf_energy *= 16.021766  # 将eV/Ų转换为J/m²
    # surface_energies[miller] = surf_energy

print(f"已为 {len(miller_indices)} 个表面设置计算")
```

### 工作流4：能带结构计算

完整的能带结构计算流程。

```python
from pymatgen.core import Structure
from pymatgen.io.vasp.sets import MPRelaxSet, MPStaticSet, MPNonSCFSet
from pymatgen.symmetry.bandstructure import HighSymmKpath

# 步骤1：结构弛豫
struct = Structure.from_file("initial_POSCAR")
relax = MPRelaxSet(struct)
relax.write_input("./1_relax")

# 弛豫后读取结构
relaxed_struct = Structure.from_file("1_relax/CONTCAR")

# 步骤2：静态计算
static = MPStaticSet(relaxed_struct)
static.write_input("./2_static")

# 步骤3：能带结构（非自洽）
kpath = HighSymmKpath(relaxed_struct)
nscf = MPNonSCFSet(relaxed_struct, mode="line")  # 能带模式
nscf.write_input("./3_bandstructure")

# 计算完成后分析
from pymatgen.io.vasp import Vasprun
from pymatgen.electronic_structure.plotter import BSPlotter

vasprun = Vasprun("3_bandstructure/vasprun.xml")
bs = vasprun.get_band_structure(line_mode=True)

print(f"带隙: {bs.get_band_gap()}")

plotter = BSPlotter(bs)
plotter.save_plot("band_structure.png")
```

### 工作流5：分子动力学设置

设置并分析分子动力学模拟。

```python
from pymatgen.core import Structure
from pymatgen.io.vasp.sets import MVLRelaxSet
from pymatgen.io.vasp.inputs import Incar

# 读取结构
struct = Structure.from_file("POSCAR")

# 为MD创建2x2x2超胞
from pymatgen.transformations.standard_transformations import SupercellTransformation
trans = SupercellTransformation([[2,0,0],[0,2,0],[0,0,2]])
supercell = trans.apply_transformation(struct)

# 设置VASP输入
md_input = MVLRelaxSet(supercell)

# 修改MD的INCAR参数
incar = md_input.incar
incar.update({
    "IBRION": 0,      # 分子动力学
    "NSW": 1000,      # 步数
    "POTIM": 2,       # 时间步长 (fs)
    "TEBEG": 300,     # 初始温度 (K)
    "TEEND": 300,     # 终止温度 (K)
    "SMASS": 0,       # NVT系综
    "MDALGO": 2,      # Nose-Hoover热浴
})

md_input.incar = incar
md_input.write_input("./md_calc")
```

### 工作流6：扩散分析

从AIMD轨迹分析离子扩散。

```python
from pymatgen.io.vasp import Xdatcar
from pymatgen.analysis.diffusion.analyzer import DiffusionAnalyzer

# 从XDATCAR读取轨迹
xdatcar = Xdatcar("XDATCAR")
structures = xdatcar.structures

# 分析特定元素扩散（如Li）
analyzer = DiffusionAnalyzer.from_structures(
    structures,
    specie="Li",
    temperature=300,  # K
    time_step=2,      # fs
    step_skip=10      # 跳过初始平衡阶段
)

# 获取扩散系数
diffusivity = analyzer.diffusivity  # cm²/s
conductivity = analyzer.conductivity  # mS/cm

# 获取均方位移
msd = analyzer.msd

# 绘制均方位移图
analyzer.plot_msd()

print(f"扩散系数: {diffusivity:.2e} cm²/s")
print(f"电导率: {conductivity:.2e} mS/cm")
```

### 工作流7：结构预测与枚举

预测并枚举可能的结构。

```python
from pymatgen.core import Structure, Lattice
from pymatgen.transformations.advanced_transformations import (
    EnumerateStructureTransformation,
    SubstitutionTransformation
)

# 从已知结构类型开始（如岩盐结构）
lattice = Lattice.cubic(4.2)
struct = Structure.from_spacegroup("Fm-3m", lattice, ["Li", "O"], [[0,0,0], [0.5,0.5,0.5]])

# 创建无序结构
from pymatgen.core import Species
species_on_site = {Species("Li"): 0.5, Species("Na"): 0.5}
struct[0] = species_on_site  # Li位点混合占位

# 枚举所有有序结构
trans = EnumerateStructureTransformation(max_cell_size=4)
ordered_structs = trans.apply_transformation(struct, return_ranked_list=True)

print(f"发现 {len(ordered_structs)} 个不同的有序结构")

# 写入所有结构
for i, s_dict in enumerate(ordered_structs[:10]):  # 前10个
    s_dict['structure'].to(filename=f"ordered_struct_{i}.cif")
```

### 工作流8：弹性常数计算

使用应力-应变法计算弹性常数。

```python
from pymatgen.core import Structure
from pymatgen.transformations.standard_transformations import DeformStructureTransformation
from pymatgen.io.vasp.sets import MPStaticSet

# 读取平衡结构
struct = Structure.from_file("relaxed_POSCAR")

# 生成变形结构
strains = [0.00, 0.01, 0.02, -0.01, -0.02]  # 施加的应变
deformation_sets = []

for strain in strains:
    # 在不同方向施加应变
    trans = DeformStructureTransformation([[1+strain, 0, 0], [0, 1, 0], [0, 0, 1]])
    deformed = trans.apply_transformation(struct)

    # 设置VASP计算
    static = MPStaticSet(deformed)
    static.write_input(f"./strain_{strain:.2f}")

# 计算完成后拟合应力-应变曲线获取弹性常数
# from pymatgen.analysis.elasticity import ElasticTensor
# ... (从OUTCAR收集应力张量)
# elastic_tensor = ElasticTensor.from_stress_list(stress_list)
```

### 工作流9：吸附能计算

计算表面吸附能。

```python
from pymatgen.core import Structure, Molecule
from pymatgen.core.surface import SlabGenerator
from pymatgen.analysis.adsorption import AdsorbateSiteFinder
from pymatgen.io.vasp.sets import MPRelaxSet

# 生成薄片
bulk = Structure.from_file("bulk_POSCAR")
sl

"""筛选潜在的电池正极材料"""
    criteria = {
        "has_li": "Li" in material.composition.elements,
        "stable": material.energy_above_hull < 0.05,
        "good_voltage": 2.5 < material.formation_energy_per_atom < 4.5,
        "electronically_conductive": material.band_gap < 0.5
    }
    return all(criteria.values()), criteria

# 查询 Materials Project
with MPRester() as mpr:
    # 获取潜在材料
    materials = mpr.materials.summary.search(
        elements=["Li"],
        energy_above_hull=(0, 0.05),
    )

    results = []
    for mat in materials:
        passes, criteria = screen_material(mat)
        if passes:
            results.append({
                "material_id": mat.material_id,
                "formula": mat.formula_pretty,
                "energy_above_hull": mat.energy_above_hull,
                "band_gap": mat.band_gap,
            })

    # 保存结果
    df = pd.DataFrame(results)
    df.to_csv("screened_materials.csv", index=False)

    print(f"发现 {len(results)} 种有前景的材料")
```

## 工作流最佳实践

1. **模块化设计**：将工作流拆分为离散步骤
2. **错误处理**：检查文件存在性和计算收敛性
3. **文档记录**：使用 `TransformedStructure` 跟踪结构变换历史
4. **版本控制**：将输入参数和脚本存储在 git 中
5. **自动化**：使用工作流管理器（Fireworks、AiiDA）处理复杂流程
6. **数据管理**：在清晰的目录结构中组织计算
7. **验证**：在继续之前始终验证中间结果

## 与工作流工具的集成

Pymatgen 集成了多种工作流管理系统：

- **Atomate**：预构建的 VASP 工作流
- **Fireworks**：工作流执行引擎
- **AiiDA**：溯源跟踪和工作流管理
- **Custodian**：错误修正和作业监控

这些工具为生产计算提供了强大的自动化支持。
