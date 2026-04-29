# 使用 Bio.PDB 进行结构生物信息学分析

## 概述

Bio.PDB 提供了处理来自 PDB 和 mmCIF 文件的大分子三维结构的工具。该模块采用 SMCRA（结构/模型/链/残基/原子）分层架构来表示蛋白质结构。

## SMCRA 架构

Bio.PDB 模块按层次组织结构：

```
Structure
  └── Model       (NMR结构包含多个模型)
      └── Chain   (例如链A、B、C)
          └── Residue  (氨基酸、核苷酸、杂原子)
              └── Atom (单个原子)
```

## 解析结构文件

### PDB 格式

```python
from Bio.PDB import PDBParser

# 创建解析器
parser = PDBParser(QUIET=True)  # QUIET=True 抑制警告信息

# 解析结构
structure = parser.get_structure("1crn", "1crn.pdb")

# 访问基本信息
print(f"结构ID: {structure.id}")
print(f"模型数量: {len(structure)}")
```

### mmCIF 格式

mmCIF 格式更现代且能更好处理大型结构：

```python
from Bio.PDB import MMCIFParser

# 创建解析器
parser = MMCIFParser(QUIET=True)

# 解析结构
structure = parser.get_structure("1crn", "1crn.cif")
```

### 从 PDB 下载

```python
from Bio.PDB import PDBList

# 创建PDB列表对象
pdbl = PDBList()

# 下载PDB文件
pdbl.retrieve_pdb_file("1CRN", file_format="pdb", pdir="structures/")

# 下载mmCIF文件
pdbl.retrieve_pdb_file("1CRN", file_format="mmCif", pdir="structures/")

# 下载过时结构
pdbl.retrieve_pdb_file("1CRN", obsolete=True, pdir="structures/")
```

## 导航结构层次

### 访问模型

```python
# 获取第一个模型
model = structure[0]

# 遍历所有模型
for model in structure:
    print(f"模型 {model.id}")
```

### 访问链

```python
# 获取特定链
chain = model["A"]

# 遍历所有链
for chain in model:
    print(f"链 {chain.id}")
```

### 访问残基

```python
# 遍历链中的残基
for residue in chain:
    print(f"残基: {residue.resname} {residue.id[1]}")

# 通过ID获取特定残基
# 残基ID是元组: (hetfield, 序列ID, 插入码)
residue = chain[(" ", 10, " ")]  # 第10位的标准氨基酸
```

### 访问原子

```python
# 遍历残基中的原子
for atom in residue:
    print(f"原子: {atom.name}, 坐标: {atom.coord}")

# 获取特定原子
ca_atom = residue["CA"]  # α碳原子
print(f"CA坐标: {ca_atom.coord}")
```

### 替代访问模式

```python
# 通过层次直接访问
atom = structure[0]["A"][10]["CA"]

# 获取所有原子
atoms = list(structure.get_atoms())
print(f"总原子数: {len(atoms)}")

# 获取所有残基
residues = list(structure.get_residues())

# 获取所有链
chains = list(structure.get_chains())
```

## 处理原子坐标

### 访问坐标

```python
# 获取原子坐标
coord = atom.coord
print(f"X: {coord[0]}, Y: {coord[1]}, Z: {coord[2]}")

# 获取B因子（温度因子）
b_factor = atom.bfactor

# 获取占有率
occupancy = atom.occupancy

# 获取元素
element = atom.element
```

### 计算距离

```python
from Bio.PDB import Vector

# 计算两个原子间距离
atom1 = residue1["CA"]
atom2 = residue2["CA"]

distance = atom1 - atom2  # 返回单位为埃的距离
print(f"距离: {distance:.2f} Å")
```

### 计算角度

```python
from Bio.PDB.vectors import calc_angle

# 计算三个原子间的角度
angle = calc_angle(
    atom1.get_vector(),
    atom2.get_vector(),
    atom3.get_vector()
)
print(f"角度: {angle:.2f} 弧度")
```

### 计算二面角

```python
from Bio.PDB.vectors import calc_dihedral

# 计算四个原子间的二面角
dihedral = calc_dihedral(
    atom1.get_vector(),
    atom2.get_vector(),
    atom3.get_vector(),
    atom4.get_vector()
)
print(f"二面角: {dihedral:.2f} 弧度")
```

## 结构分析

### 二级结构 (DSSP)

DSSP 为蛋白质结构分配二级结构：

```python
from Bio.PDB import DSSP, PDBParser

# 解析结构
parser = PDBParser()
structure = parser.get_structure("1crn", "1crn.pdb")

# 运行DSSP (需安装DSSP可执行文件)
model = structure[0]
dssp = DSSP(model, "1crn.pdb")

# 访问结果
for residue_key in dssp:
    dssp_data = dssp[residue_key]
    residue_id = residue_key[1]
    ss = dssp_data[2]  # 二级结构代码
    phi = dssp_data[4]  # Phi角
    psi = dssp_data[5]  # Psi角
    print(f"残基 {residue_id}: {ss}, φ={phi:.1f}°, ψ={psi:.1f}°")
```

二级结构代码：
- `H` - α螺旋
- `B` - β桥
- `E` - 折叠股
- `G` - 3-10螺旋
- `I` - π螺旋
- `T` - 转角
- `S` - 弯曲
- `-` - 无规卷曲/环区

### 溶剂可及性 (DSSP)

```python
# 获取相对溶剂可及性
for residue_key in dssp:
    acc = dssp[residue_key][3]  # 相对可及性
    print(f"残基 {residue_key[1]}: {acc:.2f} 相对可及性")
```

### 邻近搜索

高效查找附近原子：

```python
from Bio.PDB import NeighborSearch

# 获取所有原子
atoms = list(structure.get_atoms())

# 创建邻近搜索对象
ns = NeighborSearch(atoms)

# 查找半径范围内的原子
center_atom = structure[0]["A"][10]["CA"]
nearby_atoms = ns.search(center_atom.coord, 5.0)  # 5 Å半径
print(f"在5 Å范围内找到 {len(nearby_atoms)} 个原子")

# 查找半径范围内的残基
nearby_residues = ns.search(center_atom.coord, 5.0, level="R")

# 查找半径范围内的链
nearby_chains = ns.search(center_atom.coord, 10.0, level="C")
```

### 接触图

```python
def calculate_contact_map(chain, distance_threshold=8.0):
    """计算残基-残基接触图"""
    residues = list(chain.get_residues())
    n = len(residues)
    contact_map = [[0] * n for _ in range(n)]

    for i, res1 in enumerate(residues):
        for j, res2 in enumerate(residues):
            if i < j:
                # 获取CA原子
                if res1.has_id("CA") and res2.has_id("CA"):
                    dist = res1["CA"] - res2["CA"]
                    if dist < distance_threshold:
                        contact_map[i][j] = 1
                        contact_map[j][i] = 1

    return contact_map
```

## 结构质量评估

### 拉氏图数据

```python
from Bio.PDB import Polypeptide

def get_phi_psi(structure):
    """提取拉氏图的phi和psi角度"""
    phi_psi = []

    for model in structure:
        for chain in model:
            polypeptides = Polypeptide.PPBuilder().build_peptides(chain)
            for poly in polypeptides:
                angles = poly.get_phi_psi_list()
                for residue, (phi, psi) in zip(poly, angles):
                    if phi and psi:  # 跳过None值
                        phi_psi.append((residue.resname, phi, psi))

    return phi_psi
```

### 检查缺失原子

```python
def check_missing_atoms(structure):
    """识别缺失原子的残基"""
    missing = []

    for residue in structure.get_residues():
        if residue.id[0] == " ":  # 标准氨基酸
            resname = residue.resname

            # 预期的主链原子
            expected = ["N", "CA", "C", "O"]

            for atom_name in expected:
                if not residue.has_id(atom_name):
                    missing.append((residue.full_id, atom_name))

    return missing
```

## 结构操作

### 选择特定原子

```python
from Bio.PDB import Select

class CASelect(Select):
    """仅选择CA原子"""
    def accept_atom(self, atom):
        return atom.name == "CA"

class ChainASelect(Select):
    """仅选择A链"""
    def accept_chain(self, chain):
        return chain.id == "A"

# 配合PDBIO使用
from Bio.PDB import PDBIO

io = PDBIO()
io.set_structure(structure)
io.save("ca_only.pdb", CASelect())
io.save("chain_a.pdb", ChainASelect())
```

### 变换结构

```python
import numpy as np

# 旋转结构
from Bio.PDB.vectors import rotaxis

# 定义旋转轴和角度
axis = Vector(1, 0, 0)  # X轴
angle = np.pi / 4  # 45度

# 创建旋转矩阵
rotation = rotaxis(angle, axis)

# 对所有原子应用旋转
for atom in structure.get_atoms():
    atom.transform(rotation, Vector(0, 0, 0))
```

### 结构叠合

```python
from Bio.PDB import Superimposer, PDBParser

# 解析两个结构
parser = PDBParser()
structure1 = parser.get_structure("ref", "reference.pdb")
structure2 = parser.get_structure("mov", "mobile.pdb")

# 获取两个结构的CA原子
ref_atoms = [atom for atom in structure1.get_atoms() if atom.name == "CA"]
mov_atoms = [atom for atom in structure2.get_atoms() if atom.name == "CA"]

# 叠合
super_imposer = Superimposer()
super_imposer.set_atoms(ref_atoms, mov_atoms)

# 应用变换
super_imposer.apply(structure2.get_atoms())

# 获取RMSD
rmsd = super_imposer.rms
print(f"RMSD: {rmsd:.2f} Å")

# 保存叠合后的结构
from Bio.PDB import PDBIO
io = PDBIO()
io.set_structure(structure2)
io.save("superimposed.pdb")
```

## 保存结构文件

### 保存 PDB 文件

```python
from Bio.PDB import PDBIO

io = PDBIO()
io.set_structure(structure)
io.save("output.pdb")
```

### 保存 mmCIF 文件

```python
from Bio.PDB import MMCIFIO

io = MMCIFIO()
io.set_structure(structure)
io.save("output.cif")
```

## 从结构提取序列

### 提取序列

```python
from Bio.PDB import Polypeptide

# 从结构获取多肽链
ppb = Polypeptide.PPBuilder()

for model in structure:
    for chain in model:
        for pp in ppb.build_peptides(chain):
            sequence = pp.get_sequence()
            print(f"链 {chain.id}: {sequence}")
```

### 映射到 FASTA

```python
from Bio import SeqIO
from Bio.SeqRecord import SeqRecord

# 提取序列并创建FASTA
records = []
ppb = Polypeptide.PPBuilder()

for model in structure:
    for chain in model:
        for pp in ppb.build_peptides(chain):
            seq_record = SeqRecord(
                pp.get_sequence(),
                id=f"{structure.id}_{chain.id}",
                description=f"链 {chain.id}"
            )
            records.append(seq_record)

SeqIO.write(records, "structure_sequences.fasta", "fasta")
```

## 最佳实践

1. **使用 mmCIF** 处理大型结构和现代数据
2. **设置 QUIET=True** 抑制解析器警告
3. **分析前检查缺失原子**
4. **使用 NeighborSearch** 进行高效空间查询
5. **通过 DSSP 或拉氏分析验证结构质量**
6. **正确处理多模型**（NMR结构）
7. **注意杂原子** - 它们具有不同的残基ID
8. **使用 Select 类** 进行目标结构输出
9. **本地缓存下载的结构**
10. **考虑替代构象** - 某些残基具有多个位置

## 常见用例

### 计算结构间 RMSD

```python
from Bio.PDB import PDBParser, Superimposer

parser = PDBParser()
structure1 = parser.get_structure("s1", "structure1.pdb")
structure2 = parser.get_structure("s2", "structure2.pdb")

# 获取CA原子
atoms1 = [atom for atom in structure1[0]["A"].get_atoms() if atom.name == "CA"]
atoms2 = [atom for atom in structure2[0]["A"].get_atoms() if atom.name == "CA"]

# 确保原子数相同
min_len = min(len(atoms1), len(atoms2))
atoms1 = atoms1[:min_len]
atoms2 = atoms2[:min_len]

# 计算RMSD
sup = Superimposer()
sup.set_atoms(atoms1, atoms2)
print(f"RMSD: {sup.rms:.3f} Å")
```

### 查找结合位点残基

```python
def find_binding_site(structure, ligand_chain, ligand_res_id, distance=5.0):
    """查找配体附近的残基"""
    from Bio.PDB import NeighborSearch

    # 获取配体原子
    ligand = structure[0][ligand_chain][ligand_res_id]
    ligand_atoms = list(ligand.get_atoms())

    # 获取所有蛋白质原子
    protein_atoms = []
    for chain in structure[0]:
        if chain.id != ligand_chain:
            for residue in chain:
                if residue.id[0] == " ":  # 标准残基
                    protein_atoms.extend(residue.get_atoms())

    # 查找附近原子
    ns = NeighborSearch(protein_atoms)
    binding_site = set()

    for ligand_atom in ligand_atoms:
        nearby = ns.search(ligand_atom.coord, distance, level="R")
        binding_site.update(nearby)

    return list(binding_site)
```

### 计算质心

```python
import numpy as np

def center_of_mass(entity):
    """计算结构实体的质心"""
    masses = []
    coords = []

    # 原子质量（简化版）
    mass_dict = {"C": 12.0, "N": 14.0, "O": 16.0, "S": 32.0}

    for atom in entity.get_atoms():
        mass = mass_dict.get(atom.element, 12.0)
        masses.append(mass)
        coords.append(atom.coord)

    masses = np.array(masses)
    coords = np.array(coords)

    com = np.sum(coords * masses[:, np.newaxis], axis=0) / np.sum(masses)
    return com
```
