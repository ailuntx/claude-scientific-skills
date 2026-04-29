---
name: molecular-dynamics
description: 使用OpenMM和MDAnalysis运行和分析分子动力学模拟。设置蛋白质/小分子系统，定义力场，执行能量最小化和生产级MD，分析轨迹（RMSD、RMSF、接触图、自由能面）。适用于结构生物学、药物结合和生物物理学研究。
license: MIT
metadata:
    skill-author: 黄宽霖
---

# 分子动力学

## 概述

分子动力学（MD）模拟通过数值求解牛顿运动方程，在计算机上模拟分子系统随时间的演化过程。本技能涵盖两个互补工具：

- **OpenMM** (https://openmm.org/)：支持GPU加速的高性能MD引擎，提供Python API和灵活的力场支持
- **MDAnalysis** (https://mdanalysis.org/)：用于读取、写入和分析主流模拟软件MD轨迹的Python库

**安装方法：**
```bash
conda install -c conda-forge openmm mdanalysis nglview
# 或
pip install openmm mdanalysis
```

## 适用场景

在以下情况使用分子动力学：

- **蛋白质稳定性分析**：突变如何影响蛋白质动力学？
- **药物结合模拟**：表征配体的结合模式和停留时间
- **构象采样**：探索蛋白质灵活性和构象变化
- **蛋白质-蛋白质相互作用**：模拟界面动力学和结合能量
- **RMSD/RMSF分析**：量化相对于参考结构的结构波动
- **自由能估算**：计算结合自由能或构象自由能
- **膜系统模拟**：在脂质双分子层中建模蛋白质
- **固有无序蛋白研究**：分析IDR构象系综

## 核心流程：OpenMM模拟

### 1. 系统准备

```python
from openmm.app import *
from openmm import *
from openmm.unit import *
import sys

def prepare_system_from_pdb(pdb_file, forcefield_name="amber14-all.xml",
                              water_model="amber14/tip3pfb.xml"):
    """
    从PDB文件准备OpenMM系统
    
    参数：
        pdb_file: 处理过的PDB文件路径（原始PDB需用PDBFixer预处理）
        forcefield_name: 力场XML文件
        water_model: 水模型XML文件
    
    返回：
        pdb, forcefield, system, topology
    """
    # 加载PDB
    pdb = PDBFile(pdb_file)

    # 加载力场
    forcefield = ForceField(forcefield_name, water_model)

    # 添加氢原子和溶剂化
    modeller = Modeller(pdb.topology, pdb.positions)
    modeller.addHydrogens(forcefield)

    # 添加溶剂盒（10Å边距，150mM NaCl）
    modeller.addSolvent(
        forcefield,
        model='tip3p',
        padding=10*angstroms,
        ionicStrength=0.15*molar
    )

    print(f"系统：{modeller.topology.getNumAtoms()}原子, "
          f"{modeller.topology.getNumResidues()}残基")

    # 创建系统
    system = forcefield.createSystem(
        modeller.topology,
        nonbondedMethod=PME,         # 长程静电使用粒子网格Ewald法
        nonbondedCutoff=1.0*nanometer,
        constraints=HBonds,           # 约束氢键（允许2fs时间步长）
        rigidWater=True,
        ewaldErrorTolerance=0.0005
    )

    return modeller, system
```

### 2. 能量最小化

```python
from openmm.app import *
from openmm import *
from openmm.unit import *

def minimize_energy(modeller, system, output_pdb="minimized.pdb",
                     max_iterations=1000, tolerance=10.0):
    """
    能量最小化消除空间冲突
    
    参数：
        modeller: 包含拓扑和位置的Modeller对象
        system: OpenMM系统
        output_pdb: 最小化结构保存路径
        max_iterations: 最大迭代步数
        tolerance: 收敛阈值(kJ/mol/nm)
    
    返回：
        包含最小化位置的模拟对象
    """
    # 设置积分器（最小化中不影响结果）
    integrator = LangevinMiddleIntegrator(300*kelvin, 1/picosecond, 0.004*picoseconds)

    # 创建模拟
    # 优先使用GPU（CUDA/OpenCL），回退到CPU
    try:
        platform = Platform.getPlatformByName('CUDA')
        properties = {'DeviceIndex': '0', 'Precision': 'mixed'}
    except Exception:
        try:
            platform = Platform.getPlatformByName('OpenCL')
            properties = {}
        except Exception:
            platform = Platform.getPlatformByName('CPU')
            properties = {}

    simulation = Simulation(
        modeller.topology, system, integrator,
        platform, properties
    )
    simulation.context.setPositions(modeller.positions)

    # 检查初始能量
    state = simulation.context.getState(getEnergy=True)
    print(f"初始能量：{state.getPotentialEnergy()}")

    # 最小化
    simulation.minimizeEnergy(
        tolerance=tolerance*kilojoules_per_mole/nanometer,
        maxIterations=max_iterations
    )

    state = simulation.context.getState(getEnergy=True, getPositions=True)
    print(f"最小化后能量：{state.getPotentialEnergy()}")

    # 保存最小化结构
    with open(output_pdb, 'w') as f:
        PDBFile.writeFile(simulation.topology, state.getPositions(), f)

    return simulation
```

### 3. NVT平衡

```python
from openmm.app import *
from openmm import *
from openmm.unit import *

def run_nvt_equilibration(simulation, n_steps=50000, temperature=300,
                            report_interval=1000, output_prefix="nvt"):
    """
    NVT平衡：恒定粒子数、体积、温度
    使速度分布达到目标温度
    
    参数：
        simulation: OpenMM模拟对象（最小化后）
        n_steps: MD步数（50000×2fs=100ps）
        temperature: 开尔文温度
        report_interval: 数据记录间隔
        output_prefix: 轨迹和日志文件前缀
    """
    # 可选：在NVT阶段对主链添加位置约束
    
    # 设置温度
    simulation.context.setVelocitiesToTemperature(temperature*kelvin)

    # 添加记录器
    simulation.reporters = []

    # 日志文件
    simulation.reporters.append(
        StateDataReporter(
            f"{output_prefix}_log.txt",
            report_interval,
            step=True,
            potentialEnergy=True,
            kineticEnergy=True,
            temperature=True,
            volume=True,
            speed=True
        )
    )

    # DCD轨迹（紧凑二进制格式）
    simulation.reporters.append(
        DCDReporter(f"{output_prefix}_traj.dcd", report_interval)
    )

    print(f"运行NVT平衡：{n_steps}步（{n_steps*2/1000:.1f}ps）")
    simulation.step(n_steps)
    print("NVT平衡完成")

    return simulation
```

### 4. NPT平衡与生产模拟

```python
def run_npt_production(simulation, n_steps=500000, temperature=300, pressure=1.0,
                        report_interval=5000, output_prefix="npt"):
    """
    NPT生产模拟：恒定粒子数、压强、温度
    
    参数：
        n_steps: 生产步数（500000×2fs=1ns）
        temperature: 开尔文温度
        pressure: 压强(bar)
        report_interval: 记录间隔
    """
    # 添加蒙特卡洛气压控制器
    system = simulation.context.getSystem()
    system.addForce(MonteCarloBarostat(pressure*bar, temperature*kelvin, 25))
    simulation.context.reinitialize(preserveState=True)

    # 更新记录器
    simulation.reporters = []
    simulation.reporters.append(
        StateDataReporter(
            f"{output_prefix}_log.txt",
            report_interval,
            step=True,
            potentialEnergy=True,
            temperature=True,
            density=True,
            speed=True
        )
    )
    simulation.reporters.append(
        DCDReporter(f"{output_prefix}_traj.dcd", report_interval)
    )

    # 保存检查点
    simulation.reporters.append(
        CheckpointReporter(f"{output_prefix}_checkpoint.chk", 50000)
    )

    print(f"运行NPT生产模拟：{n_steps}步（{n_steps*2/1000000:.2f}ns）")
    simulation.step(n_steps)
    print("生产MD完成")
    return simulation
```

## MDAnalysis轨迹分析

### 1. 加载轨迹

```python
import MDAnalysis as mda
from MDAnalysis.analysis import rms, align, contacts
import numpy as np
import matplotlib.pyplot as plt

def load_trajectory(topology_file, trajectory_file):
    """
    用MDAnalysis加载MD轨迹
    
    参数：
        topology_file: 拓扑文件（PDB/PSF等）
        trajectory_file: 轨迹文件（DCD/XTC/TRR等）
    """
    u = mda.Universe(topology_file, trajectory_file)
    print(f"体系：{u.atoms.n_atoms}原子，{u.trajectory.n_frames}帧")
    print(f"时间范围：0到{u.trajectory.totaltime:.0f}ps")
    return u
```

### 2. RMSD分析

```python
def compute_rmsd(u, selection="backbone", reference_frame=0):
    """
    计算选定原子相对于参考帧的RMSD
    
    参数：
        u: MDAnalysis体系对象
        selection: 原子选择语句（MDAnalysis语法）
        reference_frame: 参考结构帧索引
    
    返回：
        (时间, RMSD)值的numpy数组
    """
    # 对齐轨迹以最小化RMSD
    aligner = align.AlignTraj(u, u, select=selection, in_memory=True)
    aligner.run()

    # 计算RMSD
    R = rms.RMSD(u, select=selection, ref_frame=reference_frame)
    R.run()

    rmsd_data = R.results.rmsd  # 列：帧号, 时间, RMSD
    return rmsd_data

def plot_rmsd(rmsd_data, title="RMSD随时间变化", output_file="rmsd.png"):
    """绘制RMSD随时间变化图"""
    fig, ax = plt.subplots(figsize=(10, 4))
    ax.plot(rmsd_data[:, 1] / 1000, rmsd_data[:, 2], 'b-', linewidth=0.5)
    ax.set_xlabel("时间(ns)")
    ax.set_ylabel("RMSD(Å)")
    ax.set_title(title)
    ax.axhline(rmsd_data[:, 2].mean(), color='r', linestyle='--',
               label=f'均值：{rmsd_data[:, 2].mean():.2f}Å')
    ax.legend()
    plt.tight_layout()
    plt.savefig(output_file, dpi=150)
    return fig
```

### 3. RMSF分析（残基柔性）

```python
def compute_rmsf(u, selection="backbone", start_frame=0):
    """
    计算残基级RMSF（柔性）
    
    返回：
        残基ID数组, RMSF值数组
    """
    # 选择原子
    atoms = u.select_atoms(selection)

    # 计算RMSF
    R = rms.RMSF(atoms)
    R.run(start=start_frame)

    # 按残基平均
    resids = []
    rmsf_per_res = []
    for res in u.select_atoms(selection).residues:
        res_atoms = res.atoms.intersection(atoms)
        if len(res_atoms) > 0:
            resids.append(res.resid)
            rmsf_per_res.append(R.results.rmsf[res_atoms.indices].mean())

    return np.array(resids), np.array(rmsf_per_res)
```

### 4. 蛋白质-配体接触分析

```python
def analyze_contacts(u, protein_sel="protein", ligand_sel="resname LIG",
                      radius=4.5, start_frame=0):
    """
    追踪轨迹中的蛋白质-配体接触
    
    参数：
        radius: 接触距离截断值(Å)
    """
    protein = u.select_atoms(protein_sel)
    ligand = u.select_atoms(ligand_sel)

    contact_frames = []
    for ts in u.trajectory[start_frame:]:
        # 查找配体半径内的蛋白质原子
        distances = contacts.contact_matrix(
            protein.positions, ligand.positions, radius
        )
        contact_residues = set()
        for i in range(distances.shape[0]):
            if distances[i].any():
                contact_residues.add(protein.atoms[i].resid)
        contact_frames.append(contact_residues)

    return contact_frames
```

## 力场选择指南

| 体系 | 推荐力场 | 水模型 |
|------|----------|--------|
| 标准蛋白质 | AMBER14 (`amber14-all.xml`) | TIP3P-FB |
| 蛋白质+小分子 | AMBER14 + GAFF2 | TIP3P-FB |
| 膜蛋白 | CHARMM36m | TIP3P |
| 核酸 | AMBER99-bsc1 或 AMBER14 | TIP3P |
| 无序蛋白 | ff19SB 或 CHARMM36m | TIP3P |

## 系统准备工具

### PDBFixer（原始PDB处理）

```python
from pdbfixer import PDBFixer
from openmm.app import PDBFile

def fix_pdb(input_pdb, output_pdb, ph=7.0):
    """修复常见PDB问题：缺失残基/原子、加氢、标准化"""
    fixer = PDBFixer(filename=input_pdb)
    fixer.findMissingResidues()
    fixer.findNonstandardResidues()
    fixer.replaceNonstandardResidues()
    fixer.removeHeterogens(True)    # 移除水分子/配体
    fixer.findMissingAtoms()
    fixer.addMissingAtoms()
    fixer.addMissingHydrogens(ph)

    with open(output_pdb, 'w') as f:
        PDBFile.writeFile(fixer.topology, fixer.positions, f)

    return output_pdb
```

### 小分子GAFF2参数化（通过OpenFF工具包）

```python
# 配体参数化使用OpenFF工具包或ACPYPE
# pip install openff-toolkit
from openff.toolkit import Molecule, ForceField as OFFForceField
from openff.interchange import Interchange

def parameterize_ligand(smiles, ff_name="openff-2.0.0.offxml"):
    """为小分子生成GAFF2/OpenFF参数"""
    mol = Molecule.from_smiles(smiles)
    mol.generate_conformers(n_conformers=1)

    off_ff = OFFForceField(ff_name)
    interchange = off_ff.create_interchange(mol.to_topology())
    return interchange
```

## 最佳实践

- **MD前必做能量最小化**：原始PDB存在空间冲突
- **生产模拟前充分平衡**：NVT(50-100ps) → NPT(100-500ps) → 生产模拟
- **使用GPU加速**：GPU(CUDA/OpenCL)可提速10-100倍
- **2fs时间步长+氢键约束**：标准配置；使用氢质量重分配(HMR)时可增至4fs
- **仅分析平衡后轨迹**：舍弃前20-50%作为平衡阶段
- **定期保存检查点**：防止模拟意外中断
- **周期性边界条件**：溶剂化体系必需
- **静电处理用PME**：对带电体系比截断法更精确

## 扩展资源

- **OpenMM文档**：https://openmm.org/documentation.html
- **MDAnalysis用户指南**：https://docs.md
