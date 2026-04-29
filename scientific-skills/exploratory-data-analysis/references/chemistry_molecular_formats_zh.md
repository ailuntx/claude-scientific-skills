# 化学与分子文件格式参考

本文档涵盖计算化学、化学信息学、分子建模及相关领域常用的文件格式。

## 结构文件格式

### .pdb - 蛋白质数据库
**描述：** 生物大分子三维结构的标准格式  
**典型数据：** 原子坐标、残基信息、二级结构、晶体结构数据  
**应用场景：** 蛋白质结构分析、分子可视化、对接研究  
**Python库：**  
- `Biopython`: `Bio.PDB`  
- `MDAnalysis`: `MDAnalysis.Universe('file.pdb')`  
- `PyMOL`: `pymol.cmd.load('file.pdb')`  
- `ProDy`: `prody.parsePDB('file.pdb')`  
**探索性数据分析方法：**  
- 结构验证（键长、键角、碰撞）  
- 二级结构分析  
- B因子分布  
- 缺失残基/原子检测  
- 拉氏图验证  
- 表面积与体积计算  

### .cif - 晶体学信息文件
**描述：** 晶体学信息的结构化数据格式  
**典型数据：** 晶胞参数、原子坐标、对称操作、实验数据  
**应用场景：** 晶体结构测定、结构生物学、材料科学  
**Python库：**  
- `gemmi`: `gemmi.cif.read_file('file.cif')`  
- `PyCifRW`: `CifFile.ReadCif('file.cif')`  
- `Biopython`: `Bio.PDB.MMCIFParser()`  
**探索性数据分析方法：**  
- 数据完整性检查  
- 分辨率与质量指标  
- 晶胞参数分析  
- 对称群验证  
- 原子位移参数  
- R因子与验证指标  

### .mol - MDL分子文件
**描述：** MDL/Accelrys开发的化学结构文件格式  
**典型数据：** 2D/3D坐标、原子类型、键级、电荷  
**应用场景：** 化学数据库存储、化学信息学、药物设计  
**Python库：**  
- `RDKit`: `Chem.MolFromMolFile('file.mol')`  
- `Open Babel`: `pybel.readfile('mol', 'file.mol')`  
- `ChemoPy`: 用于描述符计算  
**探索性数据分析方法：**  
- 分子性质计算（分子量、logP、TPSA）  
- 官能团分析  
- 环系统检测  
- 立体化学验证  
- 2D/3D坐标一致性  
- 化合价与电荷验证  

### .mol2 - Tripos Mol2
**描述：** 包含原子类型的完整3D分子结构格式  
**典型数据：** 坐标、SYBYL原子类型、键类型、电荷、子结构  
**应用场景：** 分子对接、QSAR研究、药物发现  
**Python库：**  
- `RDKit`: `Chem.MolFromMol2File('file.mol2')`  
- `Open Babel`: `pybel.readfile('mol2', 'file.mol2')`  
- `MDAnalysis`: 可解析mol2拓扑  
**探索性数据分析方法：**  
- 原子类型分布  
- 部分电荷分析  
- 键类型统计  
- 子结构识别  
- 构象分析  
- 能量最小化状态检查  

### .sdf - 结构数据文件
**描述：** 包含关联数据的多结构文件格式  
**典型数据：** 含属性/注释的多个分子结构  
**应用场景：** 化学数据库、虚拟筛选、化合物库  
**Python库：**  
- `RDKit`: `Chem.SDMolSupplier('file.sdf')`  
- `Open Babel`: `pybel.readfile('sdf', 'file.sdf')`  
- `PandasTools` (RDKit): 用于DataFrame集成  
**探索性数据分析方法：**  
- 数据集规模与多样性指标  
- 性质分布分析（分子量、logP等）  
- 结构多样性（Tanimoto相似度）  
- 缺失数据评估  
- 性质异常值检测  
- 骨架分析  

### .xyz - XYZ坐标
**描述：** 简单的笛卡尔坐标格式  
**典型数据：** 原子类型与3D坐标  
**应用场景：** 量子化学、几何优化、分子动力学  
**Python库：**  
- `ASE`: `ase.io.read('file.xyz')`  
- `Open Babel`: `pybel.readfile('xyz', 'file.xyz')`  
- `cclib`: 用于解析含xyz的量子化学输出  
**探索性数据分析方法：**  
- 几何分析（键长、键角、二面角）  
- 质心计算  
- 转动惯量  
- 分子尺寸指标  
- 坐标验证  
- 对称性检测  

### .smi / .smiles - SMILES字符串
**描述：** 化学结构的线性表示法  
**典型数据：** 分子结构的文本表示  
**应用场景：** 化学数据库、文献挖掘、数据交换  
**Python库：**  
- `RDKit`: `Chem.MolFromSmiles(smiles)`  
- `Open Babel`: 可解析SMILES  
- `DeepChem`: 用于SMILES的机器学习  
**探索性数据分析方法：**  
- SMILES语法验证  
- 基于SMILES的描述符计算  
- 指纹生成  
- 子结构搜索  
- 互变异构体枚举  
- 立体异构体处理  

### .pdbqt - AutoDock PDBQT
**描述：** 用于AutoDock对接的改进PDB格式  
**典型数据：** 对接用坐标、部分电荷、原子类型  
**应用场景：** 分子对接、虚拟筛选  
**Python库：**  
- `Meeko`: 用于PDBQT准备  
- `Open Babel`: 可读取PDBQT  
- `ProDy`: 有限支持  
**探索性数据分析方法：**  
- 电荷分布分析  
- 可旋转键识别  
- 原子类型验证  
- 坐标质量检查  
- 氢原子位置验证  
- 扭转定义分析  

### .mae - Maestro格式
**描述：** Schrödinger专有分子结构格式  
**典型数据：** Schrödinger套件的结构、属性、注释  
**应用场景：** 药物发现、Schrödinger工具分子建模  
**Python库：**  
- `schrodinger.structure`: 需安装Schrödinger  
- 基础读取的自定义解析器  
**探索性数据分析方法：**  
- 属性提取与分析  
- 结构质量指标  
- 构象分析  
- 对接分数分布  
- 配体效率指标  

### .gro - GROMACS坐标文件
**描述：** GROMACS分子动力学模拟的结构文件  
**典型数据：** 原子位置、速度、盒子向量  
**应用场景：** 分子动力学模拟、GROMACS工作流  
**Python库：**  
- `MDAnalysis`: `Universe('file.gro')`  
- `MDTraj`: `mdtraj.load_gro('file.gro')`  
- `GromacsWrapper`: 用于GROMACS集成  
**探索性数据分析方法：**  
- 系统组成分析  
- 盒子尺寸验证  
- 原子位置分布  
- 速度分布（如存在）  
- 密度计算  
- 溶剂化分析  

## 计算化学输出格式

### .log - Gaussian日志文件
**描述：** Gaussian量子化学计算输出  
**典型数据：** 能量、几何结构、频率、轨道、布居  
**应用场景：** 量子化学计算、几何优化、频率分析  
**Python库：**  
- `cclib`: `cclib.io.ccread('file.log')`  
- `GaussianRunPack`: 用于Gaussian工作流  
- 正则表达式自定义解析器  
**探索性数据分析方法：**  
- 收敛性分析  
- 能量曲线提取  
- 振动频率分析  
- 轨道能级  
- 布居分析（Mulliken、NBO）  
- 热化学数据提取  

### .out - 量子化学输出
**描述：** 各类量子化学软件的通用输出文件  
**典型数据：** 计算结果、能量、性质  
**应用场景：** 跨软件量子化学计算  
**Python库：**  
- `cclib`: 量子化学输出的通用解析器  
- `ASE`: 可读取部分输出格式  
**探索性数据分析方法：**  
- 软件特定解析  
- 收敛标准检查  
- 能量与梯度趋势  
- 基组与方法验证  
- 计算成本分析  

### .wfn / .wfx - 波函数文件
**描述：** 量子化学分析的波函数数据  
**典型数据：** 分子轨道、基组、密度矩阵  
**应用场景：** 电子密度分析、QTAIM分析  
**Python库：**  
- `Multiwfn`: 通过Python接口  
- `Horton`: 用于波函数分析  
- 特定格式的自定义解析器  
**探索性数据分析方法：**  
- 轨道布居分析  
- 电子密度分布  
- 临界点分析（QTAIM）  
- 分子轨道可视化  
- 成键分析  

### .fchk - Gaussian格式化检查点
**描述：** Gaussian的格式化检查点文件  
**典型数据：** 完整波函数数据、结果、几何结构  
**应用场景：** Gaussian计算后处理  
**Python库：**  
- `cclib`: 可解析fchk文件  
- `GaussView` Python API（如可用）  
- 自定义解析器  
**探索性数据分析方法：**  
- 波函数质量评估  
- 属性提取  
- 基组信息  
- 梯度与Hessian分析  
- 自然轨道分析  

### .cube - Gaussian Cube文件
**描述：** 三维网格上的体积数据  
**典型数据：** 电子密度、分子轨道、静电势网格数据  
**应用场景：** 体积属性可视化  
**Python库：**  
- `cclib`: `cclib.io.ccread('file.cube')`  
- `ase.io`: `ase.io.read('file.cube')`  
- `pyquante`: 用于cube文件操作  
**探索性数据分析方法：**  
- 网格维度与间距分析  
- 数值分布统计  
- 等值面值确定  
- 体积积分  
- 不同cube文件对比  

## 分子动力学格式

### .dcd - 二进制轨迹
**描述：** 二进制轨迹格式（CHARMM, NAMD）  
**典型数据：** 原子坐标时间序列  
**应用场景：** 分子动力学轨迹分析  
**Python库：**  
- `MDAnalysis`: `Universe(topology, 'traj.dcd')`  
- `MDTraj`: `mdtraj.load_dcd('traj.dcd', top='topology.pdb')`  
- `PyTraj` (Amber): 有限支持  
**探索性数据分析方法：**  
- RMSD/RMSF分析  
- 轨迹长度与帧数统计  
- 坐标范围与漂移  
- 周期性边界处理  
- 文件完整性检查  
- 时间步长验证  

### .xtc - 压缩轨迹
**描述：** GROMACS压缩轨迹格式  
**典型数据：** 分子动力学模拟的压缩坐标  
**应用场景：** 空间高效的分子动力学轨迹存储  
**Python库：**  
- `MDAnalysis`: `Universe(topology, 'traj.xtc')`  
- `MDTraj`: `mdtraj.load_xtc('traj.xtc', top='topology.pdb')`  
**探索性数据分析方法：**  
- 压缩比评估  
- 精度损失评价  
- 时间维度RMSD  
- 结构稳定性指标  
- 采样频率分析  

### .trr - GROMACS轨迹
**描述：** 全精度GROMACS轨迹  
**典型数据：** 分子动力学的坐标、速度、力  
**应用场景：** 高精度分子动力学分析  
**Python库：**  
- `MDAnalysis`: 完整支持  
- `MDTraj`: 可读取trr文件  
- `GromacsWrapper`  
**探索性数据分析方法：**  
- 全系统动力学分析  
- 能量守恒检查（含速度）  
- 力分析  
- 温度与压力验证  
- 系统平衡评估  

### .nc / .netcdf - Amber NetCDF轨迹
**描述：** 网络通用数据格式轨迹  
**典型数据：** 分子动力学的坐标、速度、力  
**应用场景：** Amber分子动力学模拟、大轨迹存储  
**Python库：**  
- `MDAnalysis`: NetCDF支持  
- `PyTraj`: 原生Amber分析  
- `netCDF4`: 底层访问  
**探索性数据分析方法：**  
- 元数据提取  
- 轨迹统计  
- 时间序列分析  
- 副本交换分析  
- 多维数据提取  

### .top - GROMACS拓扑
**描述：** GROMACS分子拓扑  
**典型数据：** 原子类型、键、角、力场参数  
**应用场景：** 分子动力学模拟设置与分析  
**Python库：**  
- `ParmEd`: `parmed.load_file('system.top')`  
- `MDAnalysis`: 可解析拓扑  
- 特定字段的自定义解析器  
**探索性数据分析方法：**  
- 力场参数验证  
- 系统组成  
- 键/角/二面角分布  
- 电荷中性检查  
- 分子类型枚举  

### .psf - 蛋白质结构文件 (CHARMM)
**描述：** CHARMM/NAMD的拓扑文件  
**典型数据：** 原子连接性、类型、电荷  
**应用场景：** CHARMM/NAMD分子动力学模拟  
**Python库：**  
- `MDAnalysis`: 原生PSF支持  
- `ParmEd`: 可读取PSF文件  
**探索性数据分析方法：**  
- 拓扑验证  
- 连接性分析  
- 电荷分布  
- 原子类型统计  
- 片段分析  

### .prmtop - Amber参数/拓扑
**描述：** Amber拓扑与参数文件  
**典型数据：** 系统拓扑、力场参数  
**应用场景：** Amber分子动力学模拟  
**Python库：**  
- `ParmEd`: `parmed.load_file('system.prmtop')`  
- `PyTraj`: 原生Amber支持  
**探索性数据分析方法：**  
- 力场完整性  
- 参数验证  
- 系统规模与组成  
- 周期性盒子信息  
- 分析用原子掩码创建  

### .inpcrd / .rst7 - Amber坐标
**描述：** Amber坐标/重启文件  
**典型数据：** 原子坐标、速度、盒子信息  
**应用场景：** Amber分子动力学起始坐标  
**Python库：**  
- `ParmEd`: 需配合prmtop使用  
- `PyTraj`: Amber坐标读取  
**探索性数据分析方法：**  
- 坐标有效性  
- 系统初始化检查  
- 盒子向量验证  
- 速度分布（如为重启文件）  
- 能量最小化状态  

## 光谱与分析数据

### .jcamp / .jdx - JCAMP-DX
**描述：** 原子分子物理数据交换联合委员会标准  
**典型数据：** 光谱数据（红外、核磁、质谱、紫外可见）  
**应用场景：** 光谱数据交换与归档  
**Python库：**  
- `jcamp`: `jcamp.jcamp_reader('file.jdx')`  
- `nmrglue`: 用于核磁JCAMP文件  
- 特定子类型的自定义解析器  
**探索性数据分析方法：**  
- 峰检测与分析  
- 基线校正评估  
- 信噪比计算  
- 光谱范围验证  
- 积分分析  
- 参考光谱对比  

### .mzML - 质谱标记语言
**描述：** 质谱数据的标准XML格式  
**典型数据：** 串联质谱、色谱图、元数据  
**应用场景：** 蛋白质组学、代谢组学、质谱工作流  
**Python库：**  
- `pymzml`: `pymzml.run.Reader('file.mzML')`  
- `pyteomics`: `pyteomics.mzml.read('file.mzML')`  
- `MSFileReader`封装器  
**探索性数据分析方法：**  
- 扫描计数与类型  
- 质谱层级分布  
- 保留时间范围  
- m/z范围与分辨率  
- 峰强度分布  
- 数据完整性  
- 质量控制指标  

### .mzXML - 质谱XML
**描述：** 质谱数据的开放XML格式  
**典型数据：** 质谱图、保留时间、峰列表  
**应用场景：** 历史质谱数据、代谢组学  
**Python

- 导数光谱计算

## 化学数据库格式

### .inchi - 国际化合物标识符
**描述：** 化学物质的文本标识符  
**典型数据：** 分层化学结构表示  
**应用场景：** 化学数据库键值、结构搜索  
**Python库：**  
- `RDKit`: `Chem.MolFromInchi(inchi)`  
- `Open Babel`: InChI转换  
**EDA分析方法：**  
- InChI有效性验证  
- 层级分析  
- 立体化学验证  
- InChI密钥生成  
- 结构往返验证  

### .cdx / .cdxml - ChemDraw交换格式
**描述：** ChemDraw绘图文件格式  
**典型数据：** 带标注的2D化学结构  
**应用场景：** 化学绘图、出版物配图  
**Python库：**  
- `RDKit`: 支持部分CDXML导入  
- `Open Babel`: 有限支持  
- `ChemDraw` Python API (商业版)  
**EDA分析方法：**  
- 结构提取  
- 标注保留  
- 样式一致性检查  
- 2D坐标验证  

### .cml - 化学标记语言
**描述：** 基于XML的化学结构格式  
**典型数据：** 化学结构、反应式、性质数据  
**应用场景：** 语义化化学数据表示  
**Python库：**  
- `RDKit`: CML支持  
- `Open Babel`: 良好支持CML  
- `lxml`: XML解析  
**EDA分析方法：**  
- XML模式验证  
- 命名空间处理  
- 性质数据提取  
- 反应路径分析  
- 元数据完整性检查  

### .rxn - MDL反应文件
**描述：** 化学反应结构文件  
**典型数据：** 反应物、产物、反应箭头  
**应用场景：** 反应数据库、合成规划  
**Python库：**  
- `RDKit`: `Chem.ReactionFromRxnFile('file.rxn')`  
- `Open Babel`: 反应支持  
**EDA分析方法：**  
- 反应配平验证  
- 原子映射分析  
- 试剂识别  
- 立体化学变化  
- 反应类型分类  

### .rdf - 反应数据文件
**描述：** 多反应文件格式  
**典型数据：** 含数据的多个反应  
**应用场景：** 反应数据库  
**Python库：**  
- `RDKit`: RDF读取功能  
- 自定义解析器  
**EDA分析方法：**  
- 反应产率统计  
- 条件分析  
- 成功率模式识别  
- 试剂频率分析  

## 计算输出与数据

### .hdf5 / .h5 - 分层数据格式
**描述：** 科学数据阵列容器  
**典型数据：** 大型数组、元数据、分层组织  
**应用场景：** 大规模数据集存储、计算结果  
**Python库：**  
- `h5py`: `h5py.File('file.h5', 'r')`  
- `pytables`: 高级HDF5接口  
- `pandas`: 支持HDF5读取  
**EDA分析方法：**  
- 数据集结构探索  
- 数组形状与数据类型分析  
- 元数据提取  
- 内存高效数据采样  
- 分块优化分析  
- 压缩率评估  

### .pkl / .pickle - Python序列化
**描述：** Python对象序列化  
**典型数据：** 任意Python对象（分子、数据框、模型）  
**应用场景：** 中间数据存储、模型持久化  
**Python库：**  
- `pickle`: 内置序列化  
- `joblib`: 大数组增强序列化  
- `dill`: 扩展序列化支持  
**EDA分析方法：**  
- 对象类型检查  
- 大小与复杂度分析  
- 版本兼容性检查  
- 安全验证（可信来源）  
- 反序列化测试  

### .npy / .npz - NumPy数组
**描述：** NumPy数组二进制格式  
**典型数据：** 数值数组（坐标、特征、矩阵）  
**应用场景：** 快速数值数据I/O  
**Python库：**  
- `numpy`: `np.load('file.npy')`  
- 大文件直接内存映射  
**EDA分析方法：**  
- 数组形状与维度  
- 数据类型与精度  
- 统计摘要（均值、标准差、范围）  
- 缺失值检测  
- 异常值识别  
- 内存占用分析  

### .mat - MATLAB数据文件
**描述：** MATLAB工作区数据  
**典型数据：** MATLAB中的数组与结构体  
**应用场景：** MATLAB-Python数据交换  
**Python库：**  
- `scipy.io`: `scipy.io.loadmat('file.mat')`  
- `h5py`: 支持v7.3 MAT文件  
**EDA分析方法：**  
- 变量提取与类型分析  
- 数组维度分析  
- 结构体字段探索  
- MATLAB版本兼容性  
- 数据类型转换验证  

### .csv - 逗号分隔值
**描述：** 文本格式表格数据  
**典型数据：** 化学性质、实验数据、描述符  
**应用场景：** 数据交换、分析、机器学习  
**Python库：**  
- `pandas`: `pd.read_csv('file.csv')`  
- `csv`: 内置模块  
- `polars`: 快速CSV读取  
**EDA分析方法：**  
- 数据类型推断  
- 缺失值模式分析  
- 统计摘要  
- 相关性分析  
- 分布可视化  
- 异常值检测  

### .json - JavaScript对象表示法
**描述：** 结构化文本数据格式  
**典型数据：** 化学性质、元数据、API响应  
**应用场景：** 数据交换、配置、Web API  
**Python库：**  
- `json`: 内置JSON支持  
- `pandas`: `pd.read_json()`  
- `ujson`: 快速JSON解析  
**EDA分析方法：**  
- 模式验证  
- 嵌套深度分析  
- 键值分布统计  
- 数据类型一致性  
- 数组长度统计  

### .parquet - Apache列式存储
**描述：** 列式存储格式  
**典型数据：** 高效存储大规模表格数据  
**应用场景：** 大数据、高效列式分析  
**Python库：**  
- `pandas`: `pd.read_parquet('file.parquet')`  
- `pyarrow`: 直接访问Parquet  
- `fastparquet`: 替代实现  
**EDA分析方法：**  
- 基于元数据的列统计  
- 分区分析  
- 压缩效率评估  
- 行组结构检查  
- 大文件快速采样  
- 模式演进追踪
