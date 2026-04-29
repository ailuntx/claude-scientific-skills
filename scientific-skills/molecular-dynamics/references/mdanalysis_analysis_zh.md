# MDAnalysis 分析参考

## MDAnalysis Universe 与 AtomGroup

```python
import MDAnalysis as mda

# 加载 Universe
u = mda.Universe("topology.pdb", "trajectory.dcd")
# 或对于单个结构：
u = mda.Universe("structure.pdb")

# 关键属性
print(u.atoms.n_atoms)          # 总原子数
print(u.residues.n_residues)    # 总残基数
print(u.trajectory.n_frames)   # 帧数
print(u.trajectory.dt)         # 时间步长（单位：ps）
print(u.trajectory.totaltime)  # 总模拟时间（单位：ps）
```

## 原子选择语言

MDAnalysis 使用丰富的选择语言：

```python
# 基本选择
protein = u.select_atoms("protein")
backbone = u.select_atoms("backbone")  # CA, N, C, O
calpha = u.select_atoms("name CA")
water = u.select_atoms("resname WAT or resname HOH or resname TIP3")
ligand = u.select_atoms("resname LIG")

# 通过残基编号
region = u.select_atoms("resid 10:50")
specific = u.select_atoms("resid 45 and name CA")

# 通过邻近性
near_ligand = u.select_atoms("protein and around 5.0 resname LIG")

# 通过性质
charged = u.select_atoms("resname ARG LYS ASP GLU")
hydrophobic = u.select_atoms("resname ALA VAL LEU ILE PRO PHE TRP MET")

# 布尔组合
active_site = u.select_atoms("(resid 100 102 145 200) and protein")

# 反向选择
not_water = u.select_atoms("not (resname WAT HOH)")
```

## 常用分析模块

### RMSD 与 RMSF

```python
from MDAnalysis.analysis import rms, align

# 将轨迹对齐到第一帧
align.AlignTraj(u, u, select='backbone', in_memory=True).run()

# RMSD
R = rms.RMSD(u, u, select='backbone', groupselections=['name CA'])
R.run()
# R.results.rmsd: 形状 (n_frames, 3) = [帧, 时间, RMSD]

# RMSF（每个原子的波动）
from MDAnalysis.analysis.rms import RMSF
rmsf = RMSF(u.select_atoms('backbone')).run()
# rmsf.results.rmsf: 每个原子的 RMSF 值（单位：埃）
```

### 回转半径

```python
rg = []
for ts in u.trajectory:
    rg.append(u.select_atoms("protein").radius_of_gyration())
import numpy as np
print(f"Mean Rg: {np.mean(rg):.2f} Å")
```

### 二级结构分析

```python
from MDAnalysis.analysis.dssp import DSSP

# 每帧的DSSP二级结构分配
dssp = DSSP(u).run()
# dssp.results.dssp: 每个残基每帧的二级结构代码
# H = α-螺旋, E = β-折叠, C = 无规卷曲
```

### 氢键

```python
from MDAnalysis.analysis.hydrogenbonds import HydrogenBondAnalysis

hbonds = HydrogenBondAnalysis(
    u,
    donors_sel="protein and name N",
    acceptors_sel="protein and name O",
    d_h_cutoff=1.2,          # 供体-H距离（Å）
    d_a_cutoff=3.0,          # 供体-受体距离（Å）
    d_h_a_angle_cutoff=150   # D-H-A角度（度）
)
hbonds.run()

# 统计每帧的氢键数
import pandas as pd
df = pd.DataFrame(hbonds.results.hbonds,
                  columns=['frame', 'donor_ix', 'hydrogen_ix', 'acceptor_ix',
                           'DA_dist', 'DHA_angle'])
```

### 主成分分析 (PCA)

```python
from MDAnalysis.analysis import pca

pca_analysis = pca.PCA(u, select='backbone', align=True).run()

# 主成分方差
print(pca_analysis.results.variance[:5])  # 前5个主成分的方差百分比

# 将轨迹投影到主成分上
projected = pca_analysis.transform(u.select_atoms('backbone'), n_components=3)
# 形状: (n_frames, n_components)
```

### 自由能面 (FES)

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.stats import gaussian_kde

def plot_free_energy_surface(x, y, bins=50, T=300, xlabel="PC1", ylabel="PC2",
                              output="fes.png"):
    """
    Compute 2D free energy surface from two order parameters.
    FES = -kT * ln(P(x,y))
    """
    kB = 0.0083144621  # kJ/mol/K
    kT = kB * T

    # 二维直方图
    H, xedges, yedges = np.histogram2d(x, y, bins=bins, density=True)
    H = H.T

    # 自由能
    H_safe = np.where(H > 0, H, np.nan)
    fes = -kT * np.log(H_safe)
    fes -= np.nanmin(fes)  # 将最小值移至0

    # 绘图
    fig, ax = plt.subplots(figsize=(8, 6))
    im = ax.contourf(xedges[:-1], yedges[:-1], fes, levels=20, cmap='RdYlBu_r')
    plt.colorbar(im, ax=ax, label='Free Energy (kJ/mol)')
    ax.set_xlabel(xlabel)
    ax.set_ylabel(ylabel)
    plt.savefig(output, dpi=150, bbox
