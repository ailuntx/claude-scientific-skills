---
name: pymatgen
description: 材料科学工具包。支持晶体结构（CIF、POSCAR）、相图、能带结构、态密度、Materials Project集成、格式转换，用于计算材料科学。
license: MIT license
metadata:
    skill-author: K-Dense Inc.
---

# Pymatgen - Python 材料基因组学

## 概述

Pymatgen 是一个全面的材料分析 Python 库，为 Materials Project 提供核心支持。可创建、分析和操作晶体结构与分子，计算相图与热力学性质，分析电子结构（能带结构、态密度），生成表面与界面，并访问 Materials Project 的计算材料数据库。支持 100 多种来自不同计算代码的文件格式。

## 使用场景

本技能适用于：
- 材料科学中的晶体结构或分子系统研究
- 结构文件格式转换（CIF、POSCAR、XYZ 等）
- 分析对称性、空间群或配位环境
- 计算相图或评估热力学稳定性
- 分析电子结构数据（带隙、态密度、能带结构）
- 生成表面、薄片或研究界面
- 通过编程访问 Materials Project 数据库
- 设置高通量计算工作流
- 分析扩散、磁性或力学性质
- 使用 VASP、Gaussian、Quantum ESPRESSO 等计算代码

## 快速入门指南

### 安装

```bash
# 核心 pymatgen
uv pip install pymatgen

# 包含 Materials Project API 访问
uv pip install pymatgen mp-api

# 扩展功能的可选依赖
uv pip install pymatgen[analysis]  # 额外分析工具
uv pip install pymatgen[vis]       # 可视化工具
```

### 基础结构操作

```python
from pymatgen.core import Structure, Lattice

# 从文件读取结构（自动格式检测）
struct = Structure.from_file("POSCAR")

# 从零创建结构
lattice = Lattice.cubic(3.84)
struct = Structure(lattice, ["Si", "Si"], [[0,0,0], [0.25,0.25,0.25]])

# 写入不同格式
struct.to(filename="structure.cif")

# 基础性质
print(f"化学式: {struct.composition.reduced_formula}")
print(f"空间群: {struct.get_space_group_info()}")
print(f"密度: {struct.density:.2f} g/cm³")
```

### Materials Project 集成

```bash
# 设置 API 密钥
export MP_API_KEY="your_api_key_here"
```

```python
from mp_api.client import MPRester

with MPRester() as mpr:
    # 通过材料 ID 获取结构
    struct = mpr.get_structure_by_material_id("mp-149")

    # 材料搜索
    materials = mpr.materials.summary.search(
        formula="Fe2O3",
        energy_above_hull=(0, 0.05)
    )
```

## 核心功能

### 1. 结构创建与操作

使用多种方法创建结构并进行变换。

**从文件读取:**
```python
# 自动格式检测
struct = Structure.from_file("structure.cif")
struct = Structure.from_file("POSCAR")
mol = Molecule.from_file("molecule.xyz")
```

**从零创建:**
```python
from pymatgen.core import Structure, Lattice

# 使用晶格参数
lattice = Lattice.from_parameters(a=3.84, b=3.84, c=3.84,
                                  alpha=120, beta=90, gamma=60)
coords = [[0, 0, 0], [0.75, 0.5, 0.75]]
struct = Structure(lattice, ["Si", "Si"], coords)

# 通过空间群创建
struct = Structure.from_spacegroup(
    "Fm-3m",
    Lattice.cubic(3.5),
    ["Si"],
    [[0, 0, 0]]
)
```

**结构变换:**
```python
from pymatgen.transformations.standard_transformations import (
    SupercellTransformation,
    SubstitutionTransformation,
    PrimitiveCellTransformation
)

# 创建超胞
trans = SupercellTransformation([[2,0,0],[0,2,0],[0,0,2]])
supercell = trans.apply_transformation(struct)

# 元素替换
trans = SubstitutionTransformation({"Fe": "Mn"})
new_struct = trans.apply_transformation(struct)

# 获取原胞
trans = PrimitiveCellTransformation()
primitive = trans.apply_transformation(struct)
```

**参考:** 完整文档见 `references/core_classes.md`（Structure、Lattice、Molecule 及相关类）。

### 2. 文件格式转换

在 100+ 文件格式间自动检测并转换。

**便捷方法:**
```python
# 读取任意格式
struct = Structure.from_file("input_file")

# 写入任意格式
struct.to(filename="output.cif")
struct.to(filename="POSCAR")
struct.to(filename="output.xyz")
```

**转换脚本:**
```bash
# 单文件转换
python scripts/structure_converter.py POSCAR structure.cif

# 批量转换
python scripts/structure_converter.py *.cif --output-dir ./poscar_files --format poscar
```

**参考:** 所有支持格式及代码集成的详细文档见 `references/io_formats.md`。

### 3. 结构与对称性分析

分析结构的对称性、配位等性质。

**对称性分析:**
```python
from pymatgen.symmetry.analyzer import SpacegroupAnalyzer

sga = SpacegroupAnalyzer(struct)

# 获取空间群信息
print(f"空间群: {sga.get_space_group_symbol()}")
print(f"编号: {sga.get_space_group_number()}")
print(f"晶系: {sga.get_crystal_system()}")

# 获取常规/原胞
conventional = sga.get_conventional_standard_structure()
primitive = sga.get_primitive_standard_structure()
```

**配位环境:**
```python
from pymatgen.analysis.local_env import CrystalNN

cnn = CrystalNN()
neighbors = cnn.get_nn_info(struct, n=0)  # 0 号位点的近邻

print(f"配位数: {len(neighbors)}")
for neighbor in neighbors:
    site = struct[neighbor['site_index']]
    print(f"  {site.species_string} 距离 {neighbor['weight']:.3f} Å")
```

**分析脚本:**
```bash
# 综合分析
python scripts/structure_analyzer.py POSCAR --symmetry --neighbors

# 导出结果
python scripts/structure_analyzer.py structure.cif --symmetry --export json
```

**参考:** 所有分析功能的详细文档见 `references/analysis_modules.md`。

### 4. 相图与热力学

构建相图并分析热力学稳定性。

**相图构建:**
```python
from mp_api.client import MPRester
from pymatgen.analysis.phase_diagram import PhaseDiagram, PDPlotter

# 从 Materials Project 获取条目
with MPRester() as mpr:
    entries = mpr.get_entries_in_chemsys("Li-Fe-O")

# 构建相图
pd = PhaseDiagram(entries)

# 检查稳定性
from pymatgen.core import Composition
comp = Composition("LiFeO2")

# 查找对应成分的条目
for entry in entries:
    if entry.composition.reduced_formula == comp.reduced_formula:
        e_above_hull = pd.get_e_above_hull(entry)
        print(f"凸包上能量: {e_above_hull:.4f} eV/atom")

        if e_above_hull > 0.001:
            # 获取分解路径
            decomp = pd.get_decomposition(comp)
            print("分解产物:", decomp)

# 绘图
plotter = PDPlotter(pd)
plotter.show()
```

**相图脚本:**
```bash
# 生成相图
python scripts/phase_diagram_generator.py Li-Fe-O --output li_fe_o.png

# 分析特定成分
python scripts/phase_diagram_generator.py Li-Fe-O --analyze "LiFeO2" --show
```

**参考:** 详细示例见 `references/analysis_modules.md`（相图部分）和 `references/transformations_workflows.md`（工作流 2）。

### 5. 电子结构分析

分析能带结构、态密度及电子性质。

**能带结构:**
```python
from pymatgen.io.vasp import Vasprun
from pymatgen.electronic_structure.plotter import BSPlotter

# 从 VASP 计算结果读取
vasprun = Vasprun("vasprun.xml")
bs = vasprun.get_band_structure()

# 分析
band_gap = bs.get_band_gap()
print(f"带隙: {band_gap['energy']:.3f} eV")
print(f"直接带隙: {band_gap['direct']}")
print(f"是否为金属: {bs.is_metal()}")

# 绘图
plotter = BSPlotter(bs)
plotter.save_plot("band_structure.png")
```

**态密度:**
```python
from pymatgen.electronic_structure.plotter import DosPlotter

dos = vasprun.complete_dos

# 获取元素投影态密度
element_dos = dos.get_element_dos()
for element, element_dos_obj in element_dos.items():
    print(f"{element}: {element_dos_obj.get_gap():.3f} eV")

# 绘图
plotter = DosPlotter()
plotter.add_dos("总态密度", dos)
plotter.show()
```

**参考:** 见 `references/analysis_modules.md`（电子结构部分）和 `references/io_formats.md`（VASP 部分）。

### 6. 表面与界面分析

生成薄片、分析表面及研究界面。

**薄片生成:**
```python
from pymatgen.core.surface import SlabGenerator

# 为特定米勒指数生成薄片
slabgen = SlabGenerator(
    struct,
    miller_index=(1, 1, 1),
    min_slab_size=10.0,      # Å
    min_vacuum_size=10.0,    # Å
    center_slab=True
)

slabs = slabgen.get_slabs()

# 写入薄片
for i, slab in enumerate(slabs):
    slab.to(filename=f"slab_{i}.cif")
```

**Wulff 形状构建:**
```python
from pymatgen.analysis.wulff import WulffShape

# 定义表面能
surface_energies = {
    (1, 0, 0): 1.0,
    (1, 1, 0): 1.1,
    (1, 1, 1): 0.9,
}

wulff = WulffShape(struct.lattice, surface_energies)
print(f"表面积: {wulff.surface_area:.2f} Ų")
print(f"体积: {wulff.volume:.2f} ų")

wulff.show()
```

**吸附位点查找:**
```python
from pymatgen.analysis.adsorption import AdsorbateSiteFinder
from pymatgen.core import Molecule

asf = AdsorbateSiteFinder(slab)

# 查找位点
ads_sites = asf.find_adsorption_sites()
print(f"顶部位点: {len(ads_sites['ontop'])}")
print(f"桥位点: {len(ads_sites['bridge'])}")
print(f"空穴位点: {len(ads_sites['hollow'])}")

# 添加吸附物
adsorbate = Molecule("O", [[0, 0, 0]])
ads_struct = asf.add_adsorbate(adsorbate, ads_sites["ontop"][0])
```

**参考:** 见 `references/analysis_modules.md`（表面与界面部分）和 `references/transformations_workflows.md`（工作流 3 和 9）。

### 7. Materials Project 数据库访问

通过编程访问 Materials Project 数据库。

**设置:**
1. 从 https://next-gen.materialsproject.org/ 获取 API 密钥
2. 设置环境变量: `export MP_API_KEY="your_key_here"`

**搜索与检索:**
```python
from mp_api.client import MPRester

with MPRester() as mpr:
    # 按化学式搜索
    materials = mpr.materials.summary.search(formula="Fe2O3")

    # 按化学体系搜索
    materials = mpr.materials.summary.search(chemsys="Li-Fe-O")

    # 按性质筛选
    materials = mpr.materials.summary.search(
        chemsys="Li-Fe-O",
        energy_above_hull=(0, 0.05),  # 稳定/亚稳态
        band_gap=(1.0, 3.0)            # 半导体
    )

    # 获取结构
    struct = mpr.get_structure_by_material_id("mp-149")

    # 获取能带结构
    bs = mpr.get_bandstructure_by_material_id("mp-149")

    # 获取相图条目
    entries = mpr.get_entries_in_chemsys("Li-Fe-O")
```

**参考:** 完整 API 文档及示例见 `references/materials_project_api.md`。

### 8. 计算工作流设置

为不同电子结构代码设置计算任务。

**VASP 输入生成:**
```python
from pymatgen.io.vasp.sets import MPRelaxSet, MPStaticSet, MPNonSCFSet

# 弛豫计算
relax = MPRelaxSet(struct)
relax.write_input("./relax_calc")

# 静态计算
static = MPStaticSet(struct)
static.write_input("./static_calc")

# 能带结构（非自洽）
nscf = MPNonSCFSet(struct, mode="line")
nscf.write_input("./bandstructure_calc")

# 自定义参数
custom = MPRelaxSet(struct, user_incar_settings={"ENCUT": 600})
custom.write_input("./custom_calc")
```

**其他代码:**
```python
# Gaussian
from pymatgen.io.gaussian import GaussianInput

gin = GaussianInput(
    mol,
    functional="B3LYP",
    basis_set="6-31G(d)",
    route_parameters={"Opt": None}
)
gin.write_file("input.gjf")

# Quantum ESPRESSO
from pymatgen.io.pwscf import PWInput

pwin = PWInput(struct, control={"calculation": "scf"})
pwin.write_file("pw.in")
```

**参考:** 见 `references/io_formats.md`（电子结构代码 I/O 部分）和 `references/transformations_workflows.md` 中的工作流示例。

### 9. 高级分析

**衍射图谱:**
```python
from pymatgen.analysis.diffraction.xrd import XRDCalculator

xrd = XRDCalculator()
pattern = xrd.get_pattern(struct)

# 获取衍射峰
for peak in pattern.hkls:
    print(f"2θ = {peak['2theta']:.2f}°, hkl = {peak['hkl']}")

pattern.plot()
```

**弹性性质:**
```python
from pymatgen.analysis.elasticity import ElasticTensor

# 从弹性张量矩阵构建
elastic_tensor = ElasticTensor.from_voigt(matrix)

print(f"体模量: {elastic_tensor.k_voigt:.1f} GPa")
print(f"剪切模量: {elastic_tensor.g_voigt:.1f} GPa")
print(f"杨氏模量: {elastic_tensor.y_mod:.1f} GPa")
```

**磁序分析:**
```python
from pymatgen.transformations.advanced_transformations import MagOrderingTransformation

# 枚举磁序结构
trans = MagOrderingTransformation({"Fe": 5.0})
mag_structs = trans.apply_transformation(struct, return_ranked_list=True)

# 获取最低能量磁结构
lowest_energy_struct = mag_structs[0]['structure']
```

**参考:** 完整分析模块文档见 `references/analysis_modules.md`。

## 内置资源

### 脚本 (`scripts/`)

常用任务的 Python 可执行脚本：

- **`structure_converter.py`**: 结构文件格式转换
  - 支持批量转换与自动格式检测
  - 用法: `python scripts/structure_converter.py POSCAR structure.cif`

- **`structure_analyzer.py`**: 综合结构分析
  - 对称性、配位、晶格参数、距离矩阵
  - 用法: `python scripts/structure_analyzer.py structure.cif --symmetry --neighbors`

- **`phase_diagram_generator.py`**: 从 Materials Project 生成相图
  - 稳定性分析与热力学性质
  - 用法: `python scripts/phase_diagram_generator.py Li-Fe-O --analyze "LiFeO2"`

- **`materials_project_api.md`**：完整的 Materials Project API 指南
- **`transformations_workflows.md`**：转换框架和常见工作流程

当需要特定模块或工作流程的详细信息时，请加载参考文档。

## 常见工作流程

### 高通量结构生成

```python
from pymatgen.transformations.standard_transformations import SubstitutionTransformation
from pymatgen.io.vasp.sets import MPRelaxSet

# 生成掺杂结构
base_struct = Structure.from_file("POSCAR")
dopants = ["Mn", "Co", "Ni", "Cu"]

for dopant in dopants:
    trans = SubstitutionTransformation({"Fe": dopant})
    doped_struct = trans.apply_transformation(base_struct)

    # 生成 VASP 输入文件
    vasp_input = MPRelaxSet(doped_struct)
    vasp_input.write_input(f"./calcs/Fe_{dopant}")
```

### 能带结构计算工作流程

```python
# 1. 弛豫
relax = MPRelaxSet(struct)
relax.write_input("./1_relax")

# 2. 静态计算（弛豫后）
relaxed = Structure.from_file("1_relax/CONTCAR")
static = MPStaticSet(relaxed)
static.write_input("./2_static")

# 3. 能带结构（非自洽）
nscf = MPNonSCFSet(relaxed, mode="line")
nscf.write_input("./3_bandstructure")

# 4. 分析
from pymatgen.io.vasp import Vasprun
vasprun = Vasprun("3_bandstructure/vasprun.xml")
bs = vasprun.get_band_structure()
bs.get_band_gap()
```

### 表面能计算

```python
# 1. 获取体相能量
bulk_vasprun = Vasprun("bulk/vasprun.xml")
bulk_E_per_atom = bulk_vasprun.final_energy / len(bulk)

# 2. 生成并计算表面
slabgen = SlabGenerator(bulk, (1,1,1), 10, 15)
slab = slabgen.get_slabs()[0]

MPRelaxSet(slab).write_input("./slab_calc")

# 3. 计算表面能（计算后）
slab_vasprun = Vasprun("slab_calc/vasprun.xml")
E_surf = (slab_vasprun.final_energy - len(slab) * bulk_E_per_atom) / (2 * slab.surface_area)
E_surf *= 16.021766  # 将 eV/Å² 转换为 J/m²
```

**更多工作流程**：请参阅 `references/transformations_workflows.md` 获取 10 个详细的工作流程示例。

## 最佳实践

### 结构处理

1. **使用自动格式检测**：`Structure.from_file()` 支持多数格式
2. **优先使用不可变结构**：结构不应更改时使用 `IStructure`
3. **检查对称性**：使用 `SpacegroupAnalyzer` 还原为原胞
4. **验证结构**：检查原子重叠或不合理键长

### 文件输入/输出

1. **使用便捷方法**：优先使用 `from_file()` 和 `to()`
2. **显式指定格式**：当自动检测失败时
3. **异常处理**：在 try-except 块中封装文件操作
4. **使用序列化**：通过 `as_dict()`/`from_dict()` 实现版本安全存储

### Materials Project API

1. **使用上下文管理器**：始终使用 `with MPRester() as mpr:`
2. **批量查询**：一次性请求多个条目
3. **缓存结果**：本地保存常用数据
4. **高效筛选**：使用属性过滤器减少数据传输

### 计算工作流程

1. **使用输入集**：优先选择 `MPRelaxSet`, `MPStaticSet` 而非手动 INCAR
2. **检查收敛性**：始终验证计算是否收敛
3. **跟踪转换过程**：使用 `TransformedStructure` 记录来源
4. **组织计算**：采用清晰的目录结构

### 性能优化

1. **降低对称性**：尽可能使用原胞
2. **限制近邻搜索**：指定合理截断半径
3. **选用合适方法**：不同分析工具存在速度/精度权衡
4. **并行化处理**：多数操作可并行执行

## 单位与约定

Pymatgen 统一使用原子单位：
- **长度**：埃（Å）
- **能量**：电子伏特（eV）
- **角度**：度（°）
- **磁矩**：玻尔磁子（μB）
- **时间**：飞秒（fs）

需要时使用 `pymatgen.core.units` 进行单位转换。

## 与其他工具的集成

Pymatgen 无缝集成以下工具：
- **ASE**（原子模拟环境）
- **Phonopy**（声子计算）
- **BoltzTraP**（输运性质）
- **Atomate/Fireworks**（工作流管理）
- **AiiDA**（溯源追踪）
- **Zeo++**（孔洞分析）
- **OpenBabel**（分子转换）

## 故障排除

**导入错误**：安装缺失依赖项
```bash
uv pip install pymatgen[analysis,vis]
```

**未找到 API 密钥**：设置 MP_API_KEY 环境变量
```bash
export MP_API_KEY="your_key_here"
```

**结构读取失败**：检查文件格式和语法
```python
# 尝试显式指定格式
struct = Structure.from_file("file.txt", fmt="cif")
```

**对称性分析失败**：结构可能存在数值精度问题
```python
# 增加容差
from pymatgen.symmetry.analyzer import SpacegroupAnalyzer
sga = SpacegroupAnalyzer(struct, symprec=0.1)
```

## 其他资源

- **文档**：https://pymatgen.org/
- **Materials Project**：https://materialsproject.org/
- **GitHub**：https://github.com/materialsproject/pymatgen
- **论坛**：https://matsci.org/
- **示例 Notebook**：https://matgenb.materialsvirtuallab.org/

## 版本说明

本技能专为 pymatgen 2024.x 及更高版本设计。对于 Materials Project API，请使用 `mp-api` 包（与旧版 `pymatgen.ext.matproj` 分离）。

要求：
- Python 3.10 或更高版本
- pymatgen >= 2023.x
- mp-api（用于访问 Materials Project）
