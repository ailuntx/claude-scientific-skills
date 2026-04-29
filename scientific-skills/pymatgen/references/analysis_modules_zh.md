# Pymatgen 分析模块参考手册

本文档详细介绍了 pymatgen 在材料表征、性能预测和计算分析方面的强大功能。

## 相图与热力学

### 相图构建

```python
from pymatgen.analysis.phase_diagram import PhaseDiagram, PDPlotter
from pymatgen.entries.computed_entries import ComputedEntry

# 创建条目（组分与总能量）
entries = [
    ComputedEntry("Fe", -8.4),
    ComputedEntry("O2", -4.9),
    ComputedEntry("FeO", -6.7),
    ComputedEntry("Fe2O3", -8.3),
    ComputedEntry("Fe3O4", -9.1),
]

# 构建相图
pd = PhaseDiagram(entries)

# 获取稳定条目
stable_entries = pd.stable_entries

# 获取凸包上能量（稳定性）
entry_to_test = ComputedEntry("Fe2O3", -8.0)
energy_above_hull = pd.get_e_above_hull(entry_to_test)

# 获取分解产物
decomp = pd.get_decomposition(entry_to_test.composition)
# 返回格式: {条目1: 比例1, 条目2: 比例2, ...}

# 获取平衡反应能
rxn_energy = pd.get_equilibrium_reaction_energy(entry_to_test)

# 绘制相图
plotter = PDPlotter(pd)
plotter.show()
plotter.write_image("phase_diagram.png")
```

### 化学势图

```python
from pymatgen.analysis.phase_diagram import ChemicalPotentialDiagram

# 创建化学势图
cpd = ChemicalPotentialDiagram(entries, limits={"O": (-10, 0)})

# 获取稳定域
domains = cpd.domains
```

### 普贝图

包含 pH 和电势轴的电极化学相图。

```python
from pymatgen.analysis.pourbaix_diagram import PourbaixDiagram, PourbaixPlotter
from pymatgen.entries.computed_entries import ComputedEntry

# 创建含水溶液物种校正的条目
entries = [...]  # 包含固体和离子

# 构建普贝图
pb = PourbaixDiagram(entries)

# 获取特定 pH 和电势下的稳定条目
stable_entry = pb.get_stable_entry(pH=7, V=0)

# 绘图
plotter = PourbaixPlotter(pb)
plotter.show()
```

## 结构分析

### 结构匹配与比较

```python
from pymatgen.analysis.structure_matcher import StructureMatcher

matcher = StructureMatcher()

# 检查结构是否匹配
is_match = matcher.fit(struct1, struct2)

# 获取结构间映射关系
mapping = matcher.get_mapping(struct1, struct2)

# 分组相似结构
grouped = matcher.group_structures([struct1, struct2, struct3, ...])
```

### 埃瓦尔德求和

计算离子结构的静电能。

```python
from pymatgen.analysis.ewald import EwaldSummation

ewald = EwaldSummation(struct)
total_energy = ewald.total_energy  # 单位 eV
forces = ewald.forces  # 各格点受力
```

### 对称性分析

```python
from pymatgen.symmetry.analyzer import SpacegroupAnalyzer

sga = SpacegroupAnalyzer(struct)

# 获取空间群信息
spacegroup_symbol = sga.get_space_group_symbol()  # 例如 "Fm-3m"
spacegroup_number = sga.get_space_group_number()   # 例如 225
crystal_system = sga.get_crystal_system()           # 例如 "cubic"

# 获取对称化结构
sym_struct = sga.get_symmetrized_structure()
equivalent_sites = sym_struct.equivalent_sites

# 获取常规/原胞
conventional = sga.get_conventional_standard_structure()
primitive = sga.get_primitive_standard_structure()

# 获取对称操作
symmetry_ops = sga.get_symmetry_operations()
```

## 局域环境分析

### 配位环境

```python
from pymatgen.analysis.local_env import (
    VoronoiNN,           # 沃罗诺伊剖分
    CrystalNN,           # 晶体学方法
    MinimumDistanceNN,   # 距离截断法
    BrunnerNN_real,      # 布鲁纳方法
)

# 沃罗诺伊最近邻
voronoi = VoronoiNN()
neighbors = voronoi.get_nn_info(struct, n=0)  # 0号位点的邻位

# CrystalNN（推荐首选）
crystalnn = CrystalNN()
neighbors = crystalnn.get_nn_info(struct, n=0)

# 分析所有位点
for i, site in enumerate(struct):
    neighbors = voronoi.get_nn_info(struct, i)
    coordination_number = len(neighbors)
    print(f"位点 {i} ({site.species_string}): 配位数 = {coordination_number}")
```

### 配位几何（ChemEnv）

详细配位环境识别。

```python
from pymatgen.analysis.chemenv.coordination_environments.coordination_geometry_finder import LocalGeometryFinder
from pymatgen.analysis.chemenv.coordination_environments.chemenv_strategies import SimplestChemenvStrategy

lgf = LocalGeometryFinder()
lgf.setup_structure(struct)

# 获取位点配位环境
se = lgf.compute_structure_environments(only_indices=[0])
strategy = SimplestChemenvStrategy()
lse = strategy.get_site_coordination_environment(se[0])

print(f"配位环境: {lse}")
```

### 键价求和

```python
from pymatgen.analysis.bond_valence import BVAnalyzer

bva = BVAnalyzer()

# 计算氧化态
valences = bva.get_valences(struct)

# 获取含氧化态的结构
struct_with_oxi = bva.get_oxi_state_decorated_structure(struct)
```

## 表面与界面分析

### 表面（薄板）生成

```python
from pymatgen.core.surface import SlabGenerator, generate_all_slabs

# 生成特定米勒指数的薄板
slabgen = SlabGenerator(
    struct,
    miller_index=(1, 1, 1),
    min_slab_size=10.0,     # 最小薄板厚度 (Å)
    min_vacuum_size=10.0,   # 最小真空层厚度 (Å)
    center_slab=True
)

slabs = slabgen.get_slabs()

# 生成米勒指数范围内的所有薄板
all_slabs = generate_all_slabs(
    struct,
    max_index=2,
    min_slab_size=10.0,
    min_vacuum_size=10.0
)
```

### 乌尔夫晶形构建

```python
from pymatgen.analysis.wulff import WulffShape

# 定义表面能 (J/m²)
surface_energies = {
    (1, 0, 0): 1.0,
    (1, 1, 0): 1.1,
    (1, 1, 1): 0.9,
}

wulff = WulffShape(struct.lattice, surface_energies, symm_reduce=True)

# 获取等效半径和表面积
effective_radius = wulff.effective_radius
surface_area = wulff.surface_area
volume = wulff.volume

# 可视化
wulff.show()
```

### 吸附位点查找

```python
from pymatgen.analysis.adsorption import AdsorbateSiteFinder

asf = AdsorbateSiteFinder(slab)

# 查找吸附位点
ads_sites = asf.find_adsorption_sites()
# 返回字典: {"顶位": [...], "桥位": [...], "空穴位": [...]}

# 生成含吸附物的结构
from pymatgen.core import Molecule
adsorbate = Molecule("O", [[0, 0, 0]])

ads_structs = asf.generate_adsorption_structures(
    adsorbate,
    repeat=[2, 2, 1],  # 超胞设置以降低吸附覆盖度
)
```

### 界面构建

```python
from pymatgen.analysis.interfaces.coherent_interfaces import CoherentInterfaceBuilder

# 构建两材料间的界面
builder = CoherentInterfaceBuilder(
    substrate_structure=substrate,
    film_structure=film,
    substrate_miller=(0, 0, 1),
    film_miller=(1, 1, 1),
)

interfaces = builder.get_interfaces()
```

## 磁学分析

### 磁结构分析

```python
from pymatgen.analysis.magnetism import CollinearMagneticStructureAnalyzer

analyzer = CollinearMagneticStructureAnalyzer(struct)

# 获取磁有序类型
ordering = analyzer.ordering  # 例如 "FM" (铁磁), "AFM", "FiM"

# 获取磁空间群
mag_space_group = analyzer.get_structure_with_spin().get_space_group_info()
```

### 磁有序枚举

```python
from pymatgen.transformations.advanced_transformations import MagOrderingTransformation

# 枚举可能的磁有序构型
mag_trans = MagOrderingTransformation({"Fe": 5.0})  # 磁矩单位 μB
transformed_structures = mag_trans.apply_transformation(struct, return_ranked_list=True)
```

## 电子结构分析

### 能带结构分析

```python
from pymatgen.electronic_structure.bandstructure import BandStructureSymmLine
from pymatgen.electronic_structure.plotter import BSPlotter

# 从 VASP 计算读取能带结构
from pymatgen.io.vasp import Vasprun
vasprun = Vasprun("vasprun.xml")
bs = vasprun.get_band_structure()

# 获取带隙
band_gap = bs.get_band_gap()
# 返回格式: {'energy': 带隙值, 'direct': 是否直接带隙, 'transition': '跃迁路径'}

# 检查是否为金属
is_metal = bs.is_metal()

# 获取价带顶和导带底
vbm = bs.get_vbm()
cbm = bs.get_cbm()

# 绘制能带图
plotter = BSPlotter(bs)
plotter.show()
plotter.save_plot("band_structure.png")
```

### 态密度（DOS）

```python
from pymatgen.electronic_structure.dos import CompleteDos
from pymatgen.electronic_structure.plotter import DosPlotter

# 从 VASP 计算读取态密度
vasprun = Vasprun("vasprun.xml")
dos = vasprun.complete_dos

# 获取总态密度
total_dos = dos.densities

# 获取投影态密度
pdos = dos.get_element_dos()  # 按元素
site_dos = dos.get_site_dos(struct[0])  # 特定位点
spd_dos = dos.get_spd_dos()  # 按轨道 (s, p, d)

# 绘制态密度图
plotter = DosPlotter()
plotter.add_dos("总态密度", dos)
plotter.show()
```

### 费米面

```python
from pymatgen.electronic_structure.boltztrap2 import BoltztrapRunner

runner = BoltztrapRunner(struct, nelec=n_electrons)
runner.run()

# 获取不同温度下的输运性质
results = runner.get_results()
```

## 衍射分析

### X射线衍射（XRD）

```python
from pymatgen.analysis.diffraction.xrd import XRDCalculator

xrd = XRDCalculator()

pattern = xrd.get_pattern(struct, two_theta_range=(0, 90))

# 获取衍射峰数据
for peak in pattern.hkls:
    print(f"2θ = {peak['2theta']:.2f}°, hkl = {peak['hkl']}, 强度 = {peak['intensity']:.1f}")

# 绘制衍射图谱
pattern.plot()
```

### 中子衍射

```python
from pymatgen.analysis.diffraction.neutron import NDCalculator

nd = NDCalculator()
pattern = nd.get_pattern(struct)
```

## 弹性与力学性能

```python
from pymatgen.analysis.elasticity import ElasticTensor, Stress, Strain

# 从矩阵创建弹性张量
elastic_tensor = ElasticTensor([[...]])  # 6x6 或 3x3x3x3 矩阵

# 获取力学性能
bulk_modulus = elastic_tensor.k_voigt  # Voigt 体积模量 (GPa)
shear_modulus = elastic_tensor.g_voigt  # 剪切模量 (GPa)
youngs_modulus = elastic_tensor.y_mod  # 杨氏模量 (GPa)

# 施加应变
strain = Strain([[0.01, 0, 0], [0, 0, 0], [0, 0, 0]])
stress = elastic_tensor.calculate_stress(strain)
```

## 反应分析

### 反应计算

```python
from pymatgen.analysis.reaction_calculator import ComputedReaction

reactants = [ComputedEntry("Fe", -8.4), ComputedEntry("O2", -4.9)]
products = [ComputedEntry("Fe2O3", -8.3)]

rxn = ComputedReaction(reactants, products)

# 获取配平反应式
balanced_rxn = rxn.normalized_repr  # 例如 "2 Fe + 1.5 O2 -> Fe2O3"

# 获取反应能
energy = rxn.calculated_reaction_energy  # 单位 eV/化学式
```

### 反应路径搜索

```python
from pymatgen.analysis.path_finder import ChgcarPotential, NEBPathfinder

# 读取电荷密度
chgcar_potential = ChgcarPotential.from_file("CHGCAR")

# 寻找扩散路径
neb_path = NEBPathfinder(
    start_struct,
    end_struct,
    relax_sites=[i for i in range(len(start_struct))],
    v=chgcar_potential
)

images = neb_path.images  # NEB 插值结构
```

## 分子分析

### 键分析

```python
# 获取共价键
bonds = mol.get_covalent_bonds()

for bond in bonds:
    print(f"{bond.site1.species_string} - {bond.site2.species_string}: 键长 {bond.length:.2f} Å")
```

### 分子图

```python
from pymatgen.analysis.graphs import MoleculeGraph
from pymatgen.analysis.local_env import OpenBabelNN

# 构建分子图
mg = MoleculeGraph.with_local_env_strategy(mol, OpenBabelNN())

# 获取片段
fragments = mg.get_disconnected_fragments()

# 查找环结构
rings = mg.find_rings()
```

## 光谱分析

### X射线吸收谱（XAS）

```python
from pymatgen.analysis.xas.spectrum import XAS

# 读取 XAS 谱
xas = XAS.from_file("xas.dat")

# 归一化处理
xas.normalize()
```

## 其他分析工具

### 晶界分析

```python
from pymatgen.analysis.gb.grain import GrainBoundaryGenerator

gb_gen = GrainBoundaryGenerator(struct)
gb_structures = gb_gen.generate_grain_boundaries(
    rotation_axis=[0, 0, 1],
    rotation_angle=36.87,  # 角度制
)
```

### 原型与结构匹配

```python
from pymatgen.analysis.prototypes import AflowPrototypeMatcher

matcher = AflowPrototypeMatcher()
prototype = matcher.get_prototypes(struct)
```

## 最佳实践指南

1. **从简入手**：先使用基础分析方法
2. **结果验证**：通过多种方法交叉验证
3. **考虑对称性**：使用 `SpacegroupAnalyzer` 降低计算成本
4. **检查收敛性**：确保输入结构充分弛豫
5. **选用合适方法**：不同分析存在精度/速度权衡
6. **可视化结果**：使用内置绘图工具快速验证
7. **保存中间结果**：复杂分析可能耗时较长
