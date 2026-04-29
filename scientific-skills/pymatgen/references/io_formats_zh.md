# Pymatgen I/O 与文件格式参考

本文档详细记录了 pymatgen 在 100 多种文件格式中读写结构和计算数据的广泛输入/输出能力。

## 通用 I/O 理念

Pymatgen 通过 `from_file()` 和 `to()` 方法提供统一的文件操作接口，支持基于文件扩展名的自动格式检测。

### 读取文件

```python
from pymatgen.core import Structure, Molecule

# 自动格式检测
struct = Structure.from_file("POSCAR")
struct = Structure.from_file("structure.cif")
mol = Molecule.from_file("molecule.xyz")

# 显式指定格式
struct = Structure.from_file("file.txt", fmt="cif")
```

### 写入文件

```python
# 写入文件（根据扩展名推断格式）
struct.to(filename="output.cif")
struct.to(filename="POSCAR")
struct.to(filename="structure.xyz")

# 获取字符串表示而不写入文件
cif_string = struct.to(fmt="cif")
poscar_string = struct.to(fmt="poscar")
```

## 结构文件格式

### CIF（晶体学信息文件）
晶体学数据的标准格式。

```python
from pymatgen.io.cif import CifParser, CifWriter

# 读取
parser = CifParser("structure.cif")
structure = parser.get_structures()[0]  # 返回结构列表

# 写入
writer = CifWriter(struct)
writer.write_file("output.cif")

# 或使用便捷方法
struct = Structure.from_file("structure.cif")
struct.to(filename="output.cif")
```

**关键特性：**
- 支持对称性信息
- 可包含多个结构
- 保留空间群和对称操作
- 处理部分占位

### POSCAR/CONTCAR（VASP）
VASP 的结构格式。

```python
from pymatgen.io.vasp import Poscar

# 读取
poscar = Poscar.from_file("POSCAR")
structure = poscar.structure

# 写入
poscar = Poscar(struct)
poscar.write_file("POSCAR")

# 或使用便捷方法
struct = Structure.from_file("POSCAR")
struct.to(filename="POSCAR")
```

**关键特性：**
- 支持选择性动力学
- 可包含速度（XDATCAR 格式）
- 保留晶格和坐标精度

### XYZ
简单的分子坐标格式。

```python
# 用于分子
mol = Molecule.from_file("molecule.xyz")
mol.to(filename="output.xyz")

# 用于结构（笛卡尔坐标）
struct.to(filename="structure.xyz")
```

### PDB（蛋白质数据库）
生物分子的常用格式。

```python
mol = Molecule.from_file("protein.pdb")
mol.to(filename="output.pdb")
```

### JSON/YAML
通过字典进行序列化。

```python
import json
import yaml

# JSON
with open("structure.json", "w") as f:
    json.dump(struct.as_dict(), f)

with open("structure.json", "r") as f:
    struct = Structure.from_dict(json.load(f))

# YAML
with open("structure.yaml", "w") as f:
    yaml.dump(struct.as_dict(), f)

with open("structure.yaml", "r") as f:
    struct = Structure.from_dict(yaml.safe_load(f))
```

## 电子结构代码 I/O

### VASP

pymatgen 中最全面的集成。

#### 输入文件

```python
from pymatgen.io.vasp.inputs import Incar, Poscar, Potcar, Kpoints, VaspInput

# INCAR（计算参数）
incar = Incar.from_file("INCAR")
incar = Incar({"ENCUT": 520, "ISMEAR": 0, "SIGMA": 0.05})
incar.write_file("INCAR")

# KPOINTS（k点网格）
from pymatgen.io.vasp.inputs import Kpoints
kpoints = Kpoints.automatic(20)  # 20x20x20 Gamma中心网格
kpoints = Kpoints.automatic_density(struct, 1000)  # 按密度生成
kpoints.write_file("KPOINTS")

# POTCAR（赝势）
potcar = Potcar(["Fe_pv", "O"])  # 指定泛函变体

# 完整输入集
vasp_input = VaspInput(incar, kpoints, poscar, potcar)
vasp_input.write_input("./vasp_calc")
```

#### 输出文件

```python
from pymatgen.io.vasp.outputs import Vasprun, Outcar, Oszicar, Eigenval

# vasprun.xml（综合输出）
vasprun = Vasprun("vasprun.xml")
final_structure = vasprun.final_structure
energy = vasprun.final_energy
band_structure = vasprun.get_band_structure()
dos = vasprun.complete_dos

# OUTCAR
outcar = Outcar("OUTCAR")
magnetization = outcar.total_mag
elastic_tensor = outcar.elastic_tensor

# OSZICAR（收敛信息）
oszicar = Oszicar("OSZICAR")
```

#### 输入集

Pymatgen 提供预配置的输入集用于常见计算：

```python
from pymatgen.io.vasp.sets import (
    MPRelaxSet,      # Materials Project 弛豫
    MPStaticSet,     # 静态计算
    MPNonSCFSet,     # 非自洽（能带结构）
    MPSOCSet,        # 自旋轨道耦合
    MPHSERelaxSet,   # HSE06 杂化泛函
)

# 创建输入集
relax = MPRelaxSet(struct)
relax.write_input("./relax_calc")

# 自定义参数
static = MPStaticSet(struct, user_incar_settings={"ENCUT": 600})
static.write_input("./static_calc")
```

### Gaussian

量子化学软件包集成。

```python
from pymatgen.io.gaussian import GaussianInput, GaussianOutput

# 输入
gin = GaussianInput(
    mol,
    charge=0,
    spin_multiplicity=1,
    functional="B3LYP",
    basis_set="6-31G(d)",
    route_parameters={"Opt": None, "Freq": None}
)
gin.write_file("input.gjf")

# 输出
gout = GaussianOutput("output.log")
final_mol = gout.final_structure
energy = gout.final_energy
frequencies = gout.frequencies
```

### LAMMPS

经典分子动力学。

```python
from pymatgen.io.lammps.data import LammpsData
from pymatgen.io.lammps.inputs import LammpsInputFile

# 结构转 LAMMPS 数据文件
lammps_data = LammpsData.from_structure(struct)
lammps_data.write_file("data.lammps")

# LAMMPS 输入脚本
lammps_input = LammpsInputFile.from_file("in.lammps")
```

### Quantum ESPRESSO

```python
from pymatgen.io.pwscf import PWInput, PWOutput

# 输入
pwin = PWInput(
    struct,
    control={"calculation": "scf"},
    system={"ecutwfc": 50, "ecutrho": 400},
    electrons={"conv_thr": 1e-8}
)
pwin.write_file("pw.in")

# 输出
pwout = PWOutput("pw.out")
final_structure = pwout.final_structure
energy = pwout.final_energy
```

### ABINIT

```python
from pymatgen.io.abinit import AbinitInput

abin = AbinitInput(struct, pseudos)
abin.set_vars(ecut=10, nband=10)
abin.write("abinit.in")
```

### CP2K

```python
from pymatgen.io.cp2k.inputs import Cp2kInput
from pymatgen.io.cp2k.outputs import Cp2kOutput

# 输入
cp2k_input = Cp2kInput.from_file("cp2k.inp")

# 输出
cp2k_output = Cp2kOutput("cp2k.out")
```

### FEFF（XAS/XANES）

```python
from pymatgen.io.feff import FeffInput

feff_input = FeffInput(struct, absorbing_atom="Fe")
feff_input.write_file("feff.inp")
```

### LMTO（斯图加特 TB-LMTO-ASA）

```python
from pymatgen.io.lmto import LMTOCtrl

ctrl = LMTOCtrl.from_file("CTRL")
ctrl.structure
```

### Q-Chem

```python
from pymatgen.io.qchem.inputs import QCInput
from pymatgen.io.qchem.outputs import QCOutput

# 输入
qc_input = QCInput(
    mol,
    rem={"method": "B3LYP", "basis": "6-31G*", "job_type": "opt"}
)
qc_input.write_file("mol.qin")

# 输出
qc_output = QCOutput("mol.qout")
```

### Exciting

```python
from pymatgen.io.exciting import ExcitingInput

exc_input = ExcitingInput(struct)
exc_input.write_file("input.xml")
```

### ATAT（合金理论自动化工具包）

```python
from pymatgen.io.atat import Mcsqs

mcsqs = Mcsqs(struct)
mcsqs.write_input(".")
```

## 专用格式

### Phonopy

```python
from pymatgen.io.phonopy import get_phonopy_structure, get_pmg_structure

# 转换为 phonopy 结构
phonopy_struct = get_phonopy_structure(struct)

# 从 phonopy 转换
struct = get_pmg_structure(phonopy_struct)
```

### ASE（原子模拟环境）

```python
from pymatgen.io.ase import AseAtomsAdaptor

adaptor = AseAtomsAdaptor()

# Pymatgen 转 ASE
atoms = adaptor.get_atoms(struct)

# ASE 转 Pymatgen
struct = adaptor.get_structure(atoms)
```

### Zeo++（多孔材料）

```python
from pymatgen.io.zeopp import get_voronoi_nodes, get_high_accuracy_voronoi_nodes

# 分析孔结构
vor_nodes = get_voronoi_nodes(struct)
```

### BabelMolAdaptor（OpenBabel）

```python
from pymatgen.io.babel import BabelMolAdaptor

adaptor = BabelMolAdaptor(mol)

# 转换为不同格式
pdb_str = adaptor.pdbstring
sdf_str = adaptor.write_file("mol.sdf", file_format="sdf")

# 生成 3D 坐标
adaptor.add_hydrogen()
adaptor.make3d()
```

## 转换与变形 I/O

### TransformedStructure

记录转换历史的结构。

```python
from pymatgen.alchemy.materials import TransformedStructure
from pymatgen.transformations.standard_transformations import (
    SupercellTransformation,
    SubstitutionTransformation
)

# 创建转换结构
ts = TransformedStructure(struct, [])
ts.append_transformation(SupercellTransformation([[2,0,0],[0,2,0],[0,0,2]]))
ts.append_transformation(SubstitutionTransformation({"Fe": "Mn"}))

# 带历史写入
ts.write_vasp_input("./calc_dir")

# 从 SNL（结构笔记本语言）读取
ts = TransformedStructure.from_snl(snl)
```

## 批量操作

### CifTransmuter

处理多个 CIF 文件。

```python
from pymatgen.alchemy.transmuters import CifTransmuter

transmuter = CifTransmuter.from_filenames(
    ["structure1.cif", "structure2.cif"],
    [SupercellTransformation([[2,0,0],[0,2,0],[0,0,2]])]
)

# 写入所有结构
transmuter.write_vasp_input("./batch_calc")
```

### PoscarTransmuter

类似处理 POSCAR 文件。

```python
from pymatgen.alchemy.transmuters import PoscarTransmuter

transmuter = PoscarTransmuter.from_filenames(
    ["POSCAR1", "POSCAR2"],
    [transformation1, transformation2]
)
```

## 最佳实践

1. **自动格式检测**：尽可能使用 `from_file()` 和 `to()` 方法
2. **错误处理**：始终在 try-except 块中封装文件 I/O
3. **格式特定解析器**：使用专用解析器（如 `Vasprun`）进行详细输出分析
4. **输入集**：优先使用预配置输入集而非手动指定参数
5. **序列化**：使用 JSON/YAML 进行长期存储和版本控制
6. **批量处理**：使用转换器对多个结构应用变换

## 支持格式摘要

### 结构格式：
CIF, POSCAR/CONTCAR, XYZ, PDB, XSF, PWMAT, Res, CSSR, JSON, YAML

### 电子结构代码：
VASP, Gaussian, LAMMPS, Quantum ESPRESSO, ABINIT, CP2K, FEFF, Q-Chem, LMTO, Exciting, NWChem, AIMS, 晶体学数据格式

### 分子格式：
XYZ, PDB, MOL, SDF, PQR, 通过 OpenBabel（支持多种附加格式）

### 专用格式：
Phonopy, ASE, Zeo++, Lobster, BoltzTraP
