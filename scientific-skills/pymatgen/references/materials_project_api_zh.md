# Materials Project API 参考

本文档介绍如何通过 pymatgen 的 API 集成访问和使用 Materials Project 数据库。

## 概述

Materials Project 是一个综合性的计算材料属性数据库，包含数十万种无机晶体和分子的数据。该 API 通过 `MPRester` 客户端提供对此数据的程序化访问。

## 安装与设置

Materials Project API 客户端现已独立打包：

```bash
pip install mp-api
```

### 获取 API 密钥

1. 访问 https://next-gen.materialsproject.org/
2. 创建账户或登录
3. 进入仪表盘/设置页面
4. 生成 API 密钥
5. 存储为环境变量：

```bash
export MP_API_KEY="your_api_key_here"
```

或添加到 shell 配置文件（~/.bashrc, ~/.zshrc 等）

## 基础用法

### 初始化

```python
from mp_api.client import MPRester

# 使用环境变量（推荐）
with MPRester() as mpr:
    # 执行查询
    pass

# 或显式传递 API 密钥
with MPRester("your_api_key_here") as mpr:
    # 执行查询
    pass
```

**重要提示**：始终使用 `with` 上下文管理器确保会话正确关闭。

## 查询材料数据

### 按化学式搜索

```python
with MPRester() as mpr:
    # 获取所有匹配化学式的材料
    materials = mpr.materials.summary.search(formula="Fe2O3")

    for mat in materials:
        print(f"材料 ID: {mat.material_id}")
        print(f"化学式: {mat.formula_pretty}")
        print(f"凸包能量: {mat.energy_above_hull} eV/atom")
        print(f"带隙: {mat.band_gap} eV")
        print()
```

### 按材料 ID 搜索

```python
with MPRester() as mpr:
    # 获取特定材料
    material = mpr.materials.summary.search(material_ids=["mp-149"])[0]

    print(f"化学式: {material.formula_pretty}")
    print(f"空间群: {material.symmetry.symbol}")
    print(f"密度: {material.density} g/cm³")
```

### 按化学体系搜索

```python
with MPRester() as mpr:
    # 获取 Fe-O 体系所有材料
    materials = mpr.materials.summary.search(chemsys="Fe-O")

    # 获取三元体系材料
    materials = mpr.materials.summary.search(chemsys="Li-Fe-O")
```

### 按元素搜索

```python
with MPRester() as mpr:
    # 含 Fe 和 O 的材料
    materials = mpr.materials.summary.search(elements=["Fe", "O"])

    # 仅含 Fe 和 O 的材料（排除其他元素）
    materials = mpr.materials.summary.search(
        elements=["Fe", "O"],
        exclude_elements=True
    )
```

## 获取结构数据

### 通过材料 ID 获取结构

```python
with MPRester() as mpr:
    # 获取单结构
    structure = mpr.get_structure_by_material_id("mp-149")

    # 获取多结构
    structures = mpr.get_structures(["mp-149", "mp-510", "mp-19017"])
```

### 获取化学式所有结构

```python
with MPRester() as mpr:
    # 获取所有 Fe2O3 结构
    materials = mpr.materials.summary.search(formula="Fe2O3")

    for mat in materials:
        structure = mpr.get_structure_by_material_id(mat.material_id)
        print(f"{mat.material_id}: {structure.get_space_group_info()}")
```

## 高级查询

### 属性过滤

```python
with MPRester() as mpr:
    # 特定属性范围的材料
    materials = mpr.materials.summary.search(
        chemsys="Li-Fe-O",
        energy_above_hull=(0, 0.05),  # 稳定或近稳定
        band_gap=(1.0, 3.0),           # 半导体特性
    )

    # 磁性材料
    materials = mpr.materials.summary.search(
        elements=["Fe"],
        is_magnetic=True
    )

    # 仅金属材料
    materials = mpr.materials.summary.search(
        chemsys="Fe-Ni",
        is_metal=True
    )
```

### 排序与限制

```python
with MPRester() as mpr:
    # 获取最稳定材料
    materials = mpr.materials.summary.search(
        chemsys="Li-Fe-O",
        sort_fields=["energy_above_hull"],
        num_chunks=1,
        chunk_size=10  # 限制 10 条结果
    )
```

## 电子结构数据

### 能带结构

```python
with MPRester() as mpr:
    # 获取能带结构
    bs = mpr.get_bandstructure_by_material_id("mp-149")

    # 分析能带结构
    if bs:
        print(f"带隙: {bs.get_band_gap()}")
        print(f"是否为金属: {bs.is_metal()}")
        print(f"直接带隙: {bs.get_band_gap()['direct']}")

        # 绘图
        from pymatgen.electronic_structure.plotter import BSPlotter
        plotter = BSPlotter(bs)
        plotter.show()
```

### 态密度

```python
with MPRester() as mpr:
    # 获取态密度
    dos = mpr.get_dos_by_material_id("mp-149")

    if dos:
        # 从态密度获取带隙
        gap = dos.get_gap()
        print(f"态密度带隙: {gap} eV")

        # 绘图
        from pymatgen.electronic_structure.plotter import DosPlotter
        plotter = DosPlotter()
        plotter.add_dos("总态密度", dos)
        plotter.show()
```

### 费米面

```python
with MPRester() as mpr:
    # 获取费米面电子结构数据
    bs = mpr.get_bandstructure_by_material_id("mp-149", line_mode=False)
```

## 热力学数据

### 相图构建

```python
from pymatgen.analysis.phase_diagram import PhaseDiagram, PDPlotter

with MPRester() as mpr:
    # 获取相图条目
    entries = mpr.get_entries_in_chemsys("Li-Fe-O")

    # 构建相图
    pd = PhaseDiagram(entries)

    # 绘图
    plotter = PDPlotter(pd)
    plotter.show()
```

### 普贝图

```python
from pymatgen.analysis.pourbaix_diagram import PourbaixDiagram, PourbaixPlotter

with MPRester() as mpr:
    # 获取普贝图条目
    entries = mpr.get_pourbaix_entries(["Fe"])

    # 构建普贝图
    pb = PourbaixDiagram(entries)

    # 绘图
    plotter = PourbaixPlotter(pb)
    plotter.show()
```

### 形成能

```python
with MPRester() as mpr:
    materials = mpr.materials.summary.search(material_ids=["mp-149"])

    for mat in materials:
        print(f"形成能: {mat.formation_energy_per_atom} eV/atom")
        print(f"凸包能量: {mat.energy_above_hull} eV/atom")
```

## 弹性和力学性能

```python
with MPRester() as mpr:
    # 搜索含弹性数据的材料
    materials = mpr.materials.elasticity.search(
        chemsys="Fe-O",
        bulk_modulus_vrh=(100, 300)  # GPa
    )

    for mat in materials:
        print(f"{mat.material_id}: K = {mat.bulk_modulus_vrh} GPa")
```

## 介电性能

```python
with MPRester() as mpr:
    # 获取介电数据
    materials = mpr.materials.dielectric.search(
        material_ids=["mp-149"]
    )

    for mat in materials:
        print(f"介电常数: {mat.e_electronic}")
        print(f"折射率: {mat.n}")
```

## 压电性能

```python
with MPRester() as mpr:
    # 获取压电材料
    materials = mpr.materials.piezoelectric.search(
        piezoelectric_modulus=(1, 100)
    )
```

## 表面性能

```python
with MPRester() as mpr:
    # 获取表面数据
    surfaces = mpr.materials.surface_properties.search(
        material_ids=["mp-149"]
    )
```

## 分子数据（分子材料）

```python
with MPRester() as mpr:
    # 搜索分子
    molecules = mpr.molecules.summary.search(
        formula="H2O"
    )

    for mol in molecules:
        print(f"分子 ID: {mol.molecule_id}")
        print(f"化学式: {mol.formula_pretty}")
```

## 批量数据下载

### 下载材料完整数据

```python
with MPRester() as mpr:
    # 获取综合数据
    materials = mpr.materials.summary.search(
        material_ids=["mp-149"],
        fields=[
            "material_id",
            "formula_pretty",
            "structure",
            "energy_above_hull",
            "band_gap",
            "density",
            "symmetry",
            "elasticity",
            "magnetic_ordering"
        ]
    )
```

## 数据来源与计算详情

```python
with MPRester() as mpr:
    # 获取计算详情
    materials = mpr.materials.summary.search(
        material_ids=["mp-149"],
        fields=["material_id", "origins"]
    )

    for mat in materials:
        print(f"数据来源: {mat.origins}")
```

## 使用计算条目

### 用于热力学分析的 ComputedEntry

```python
with MPRester() as mpr:
    # 获取条目（含能量和成分）
    entries = mpr.get_entries_in_chemsys("Li-Fe-O")

    # 条目可直接用于相图分析
    from pymatgen.analysis.phase_diagram import PhaseDiagram
    pd = PhaseDiagram(entries)

    # 检查稳定性
    for entry in entries[:5]:
        e_above_hull = pd.get_e_above_hull(entry)
        print(f"{entry.composition.reduced_formula}: {e_above_hull:.3f} eV/atom")
```

## 速率限制与最佳实践

### 速率限制

Materials Project API 设有速率限制以确保公平使用：
- 注意请求频率
- 尽可能使用批量查询
- 重复分析时本地缓存结果

### 高效查询

```python
# 错误：多次独立查询
with MPRester() as mpr:
    for mp_id in ["mp-149", "mp-510", "mp-19017"]:
        struct = mpr.get_structure_by_material_id(mp_id)  # 3 次 API 调用

# 正确：单次批量查询
with MPRester() as mpr:
    structs = mpr.get_structures(["mp-149", "mp-510", "mp-19017"])  # 1 次 API 调用
```

### 结果缓存

```python
import json

# 保存结果供后续使用
with MPRester() as mpr:
    materials = mpr.materials.summary.search(chemsys="Li-Fe-O")

    # 保存到文件
    with open("li_fe_o_materials.json", "w") as f:
        json.dump([mat.dict() for mat in materials], f)

# 加载缓存结果
with open("li_fe_o_materials.json", "r") as f:
    cached_data = json.load(f)
```

## 错误处理

```python
from mp_api.client.core.client import MPRestError

try:
    with MPRester() as mpr:
        materials = mpr.materials.summary.search(material_ids=["invalid-id"])
except MPRestError as e:
    print(f"API 错误: {e}")
except Exception as e:
    print(f"意外错误: {e}")
```

## 常见用例

### 寻找稳定化合物

```python
with MPRester() as mpr:
    # 获取化学体系中所有稳定化合物
    materials = mpr.materials.summary.search(
        chemsys="Li-Fe-O",
        energy_above_hull=(0, 0.001)  # 位于凸包上
    )

    print(f"找到 {len(materials)} 种稳定化合物")
    for mat in materials:
        print(f"  {mat.formula_pretty} ({mat.material_id})")
```

### 电池材料筛选

```python
with MPRester() as mpr:
    # 筛选潜在正极材料
    materials = mpr.materials.summary.search(
        elements=["Li"],  # 必须含 Li
        energy_above_hull=(0, 0.05),  # 近稳定
        band_gap=(0, 0.5),  # 金属性或小带隙
    )

    print(f"找到 {len(materials)} 种潜在正极材料")
```

### 寻找特定晶体结构材料

```python
with MPRester() as mpr:
    # 按空间群搜索材料
    materials = mpr.materials.summary.search(
        chemsys="Fe-O",
        spacegroup_number=167  # R-3c（刚玉结构）
    )
```

## 与其他 Pymatgen 功能集成

所有从 Materials Project 获取的数据均可直接用于 pymatgen 的分析工具：

```python
with MPRester() as mpr:
    # 获取结构
    struct = mpr.get_structure_by_material_id("mp-149")

    # 使用 pymatgen 分析
    from pymatgen.symmetry.analyzer import SpacegroupAnalyzer
    sga = SpacegroupAnalyzer(struct)

    # 生成表面
    from pymatgen.core.surface import SlabGenerator
    slabgen = SlabGenerator(struct, (1,0,0), 10, 10)
    slabs = slabgen.get_slabs()

    # 相图分析
    entries = mpr.get_entries_in_chemsys(struct.composition.chemical_system)
    from pymatgen.analysis.phase_diagram import PhaseDiagram
    pd = PhaseDiagram(entries)
```

## 附加资源

- **API 文档**: https://docs.materialsproject.org/
- **Materials Project 官网**: https://next-gen.materialsproject.org/
- **GitHub**: https://github.com/materialsproject/api
- **论坛**: https://matsci.org/

## 最佳实践总结

1. **始终使用上下文管理器**: 使用 `with MPRester() as mpr:`
2. **API 密钥存为环境变量**: 切勿硬编码 API 密钥
3. **批量查询**: 尽可能批量请求数据
4. **缓存结果**: 本地保存常用数据
5. **错误处理**: 用 try-except 包裹 API 调用
6. **精确查询**: 使用过滤器限制结果并减少数据传输
7. **检查数据可用性**: 并非所有属性对所有材料都可用
