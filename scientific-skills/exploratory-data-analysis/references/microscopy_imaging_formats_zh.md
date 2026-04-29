# 显微镜与成像文件格式参考

本参考涵盖显微镜、医学影像、遥感及科学图像分析中使用的文件格式。

## 显微镜专用格式

### .tif / .tiff - 标签图像文件格式
**描述：** 支持多页面和元数据的灵活图像格式  
**典型数据：** 显微镜图像、Z轴堆栈、时间序列、多通道  
**应用场景：** 荧光显微镜、共聚焦成像、生物成像  
**Python库：**  
- `tifffile`：`tifffile.imread('file.tif')` - 显微镜TIFF支持  
- `PIL/Pillow`：`Image.open('file.tif')` - 基础TIFF  
- `scikit-image`：`io.imread('file.tif')`  
- `AICSImageIO`：多格式显微镜读取器  
**EDA方法：**  
- 图像尺寸与位深度  
- 多页面/Z轴堆栈分析  
- 元数据提取（OME-TIFF）  
- 通道分析与强度分布  
- 时间动态分析（延时摄影）  
- 像素尺寸与空间校准  
- 各通道直方图分析  
- 动态范围利用率  

### .nd2 - 尼康NIS-Elements  
**描述：** 尼康显微镜专有格式  
**典型数据：** 多维显微镜数据（XYZCT）  
**应用场景：** 尼康显微镜数据、共聚焦、宽场  
**Python库：**  
- `nd2reader`：`ND2Reader('file.nd2')`  
- `pims`：`pims.ND2_Reader('file.nd2')`  
- `AICSImageIO`：通用读取器  
**EDA方法：**  
- 实验元数据提取  
- 通道配置分析  
- 延时摄影帧分析  
- Z轴堆栈深度与间距  
- XY载物台位置  
- 激光器设置与功率  
- 像素合并信息  
- 采集时间戳  

### .lif - 徕卡图像格式  
**描述：** 徕卡显微镜专有格式  
**典型数据：** 多实验多维图像  
**应用场景：** 徕卡共聚焦与宽场数据  
**Python库：**  
- `readlif`：`readlif.LifFile('file.lif')`  
- `AICSImageIO`：LIF支持  
- `python-bioformats`：通过Bio-Formats  
**EDA方法：**  
- 多实验检测  
- 图像序列枚举  
- 各实验元数据  
- 通道与时间点结构  
- 物理尺寸提取  
- 物镜与探测器信息  
- 扫描设置分析  

### .czi - 蔡司图像格式  
**描述：** 蔡司显微镜格式  
**典型数据：** 含丰富元数据的多维显微镜数据  
**应用场景：** 蔡司共聚焦、光片、宽场  
**Python库：**  
- `czifile`：`czifile.CziFile('file.czi')`  
- `AICSImageIO`：CZI支持  
- `pylibCZIrw`：官方蔡司库  
**EDA方法：**  
- 场景与位置分析  
- 马赛克瓦片结构  
- 通道波长信息  
- 采集模式检测  
- 缩放与校准  
- 仪器配置  
- ROI定义  

### .oib / .oif - 奥林巴斯图像格式  
**描述：** 奥林巴斯显微镜格式  
**典型数据：** 共聚焦与多光子成像  
**应用场景：** 奥林巴斯FluoView数据  
**Python库：**  
- `AICSImageIO`：OIB/OIF支持  
- `python-bioformats`：通过Bio-Formats  
**EDA方法：**  
- 目录结构验证（OIF）  
- 元数据文件解析  
- 通道配置  
- 扫描参数  
- 物镜与滤光片信息  
- PMT设置  

### .vsi - 奥林巴斯VSI  
**描述：** 奥林巴斯玻片扫描格式  
**典型数据：** 全玻片成像、大型马赛克  
**应用场景：** 虚拟显微镜、病理学  
**Python库：**  
- `openslide-python`：`openslide.OpenSlide('file.vsi')`  
- `AICSImageIO`：VSI支持  
**EDA方法：**  
- 金字塔层级分析  
- 瓦片结构与重叠  
- 宏观与标签图像  
- 放大倍率级别  
- 全玻片统计  
- 区域检测  

### .ims - Imaris格式  
**描述：** Bitplane Imaris基于HDF5的格式  
**典型数据：** 大型3D/4D显微镜数据集  
**应用场景：** 3D渲染、延时分析  
**Python库：**  
- `h5py`：直接HDF5访问  
- `imaris_ims_file_reader`：专用读取器  
**EDA方法：**  
- 分辨率级别分析  
- 时间点结构  
- 通道组织  
- 数据集层次结构  
- 缩略图生成  
- 内存映射访问策略  
- 分块优化  

### .lsm - 蔡司LSM  
**描述：** 传统蔡司共聚焦格式  
**典型数据：** 共聚焦激光扫描显微镜  
**应用场景：** 旧版蔡司共聚焦数据  
**Python库：**  
- `tifffile`：LSM支持（基于TIFF）  
- `python-bioformats`：LSM读取  
**EDA方法：**  
- 类似TIFF含LSM特定元数据  
- 扫描速度与分辨率  
- 激光谱线与功率  
- 探测器增益与偏移  
- LUT信息  

### .stk - MetaMorph堆栈  
**描述：** MetaMorph图像堆栈格式  
**典型数据：** 延时或Z轴堆栈序列  
**应用场景：** MetaMorph软件输出  
**Python库：**  
- `tifffile`：STK基于TIFF  
- `python-bioformats`：STK支持  
**EDA方法：**  
- 堆栈维度  
- 平面元数据  
- 时间信息  
- 载物台位置  
- UIC标签解析  

### .dv - DeltaVision  
**描述：** Applied Precision DeltaVision格式  
**典型数据：** 去卷积显微镜  
**应用场景：** DeltaVision显微镜数据  
**Python库：**  
- `mrc`：可读取DV（与MRC相关）  
- `AICSImageIO`：DV支持  
**EDA方法：**  
- 波段信息（通道）  
- 扩展头分析  
- 镜头与放大倍率  
- 去卷积状态  
- 各切片时间戳  

### .mrc - 医学研究委员会  
**描述：** 电子显微镜格式  
**典型数据：** EM图像、冷冻电镜、断层扫描  
**应用场景：** 结构生物学、电子显微镜  
**Python库：**  
- `mrcfile`：`mrcfile.open('file.mrc')`  
- `EMAN2`：EM专用工具  
**EDA方法：**  
- 体积维度  
- 体素尺寸与单位  
- 原点与图谱统计  
- 对称性信息  
- 扩展头分析  
- 密度统计  
- 头文件一致性验证  

### .dm3 / .dm4 - Gatan数字显微图像  
**描述：** Gatan TEM/STEM格式  
**典型数据：** 透射电子显微镜  
**应用场景：** TEM成像与分析  
**Python库：**  
- `hyperspy`：`hs.load('file.dm3')`  
- `ncempy`：`ncempy.io.dm.dmReader('file.dm3')`  
**EDA方法：**  
- 显微镜参数  
- 能量色散谱数据  
- 衍射图案  
- 校准信息  
- 标签结构分析  
- 图像序列处理  

### .eer - 电子事件表示  
**描述：** 直接电子探测器格式  
**典型数据：** 探测器电子计数数据  
**应用场景：** 冷冻电镜数据采集  
**Python库：**  
- `mrcfile`：部分EER支持  
- 厂商专用工具（Gatan, TFS）  
**EDA方法：**  
- 事件计数统计  
- 帧率与剂量  
- 探测器配置  
- 运动校正评估  
- 增益参考验证  

### .ser - TIA序列  
**描述：** FEI/TFS TIA格式  
**典型数据：** EM图像序列  
**应用场景：** FEI/Thermo Fisher EM数据  
**Python库：**  
- `hyperspy`：SER支持  
- `ncempy`：TIA读取器  
**EDA方法：**  
- 序列结构  
- 校准数据  
- 采集元数据  
- 时间戳  
- 多维数据组织  

## 医学与生物成像

### .dcm - DICOM  
**描述：** 医学数字成像与通信  
**典型数据：** 含患者/研究元数据的医学图像  
**应用场景：** 临床影像、放射学、CT、MRI、PET  
**Python库：**  
- `pydicom`：`pydicom.dcmread('file.dcm')`  
- `SimpleITK`：`sitk.ReadImage('file.dcm')`  
- `nibabel`：有限DICOM支持  
**EDA方法：**  
- 患者元数据提取（匿名化检查）  
- 模态特定分析  
- 序列与研究组织  
- 切片厚度与间距  
- 窗宽/窗位设置  
- 亨氏单位（CT）  
- 图像方向与位置  
- 多帧分析  

### .nii / .nii.gz - NIfTI  
**描述：** 神经影像信息技术倡议  
**典型数据：** 脑成像、fMRI、结构MRI  
**应用场景：** 神经影像研究、脑分析  
**Python库：**  
- `nibabel`：`nibabel.load('file.nii')`  
- `nilearn`：神经影像机器学习  
- `SimpleITK`：NIfTI支持  
**EDA方法：**  
- 体积维度与体素尺寸  
- 仿射变换矩阵  
- 时间序列分析（fMRI）  
- 强度分布  
- 脑提取质量  
- 配准评估  
- 方向验证  
- 头文件信息一致性  

### .mnc - MINC格式  
**描述：** 医学图像NetCDF  
**典型数据：** 医学成像（NIfTI前身）  
**应用场景：** 传统神经影像数据  
**Python库：**  
- `pyminc`：MINC专用工具  
- `nibabel`：MINC支持  
**EDA方法：**  
- 类似NIfTI  
- NetCDF结构探索  
- 维度排序  
- 元数据提取  

### .nrrd - 近原始栅格数据  
**描述：** 含独立头文件的医学影像格式  
**典型数据：** 医学图像、科研影像  
**应用场景：** 3D Slicer、基于ITK的应用  
**Python库：**  
- `pynrrd`：`nrrd.read('file.nrrd')`  
- `SimpleITK`：NRRD支持  
**EDA方法：**  
- 头字段分析  
- 编码格式  
- 维度与间距  
- 方向矩阵  
- 压缩评估  
- 字节序处理  

### .mha / .mhd - MetaImage  
**描述：** MetaImage格式（ITK）  
**典型数据：** 医学/科学3D图像  
**应用场景：** ITK/SimpleITK应用  
**Python库：**  
- `SimpleITK`：原生MHA/MHD支持  
- `itk`：直接ITK集成  
**EDA方法：**  
- 头文件-数据文件配对（MHD）  
- 变换矩阵  
- 元素间距  
- 压缩格式  
- 数据类型与维度  

### .hdr / .img - Analyze格式  
**描述：** 传统医学影像格式  
**典型数据：** 脑成像（NIfTI前身）  
**应用场景：** 旧版神经影像数据集  
**Python库：**  
- `nibabel`：Analyze支持  
- 建议转换为NIfTI  
**EDA方法：**  
- 头文件-图像配对验证  
- 字节序问题  
- 转换为现代格式  
- 元数据限制  

## 科学图像格式

### .png - 便携式网络图形  
**描述：** 无损压缩图像格式  
**典型数据：** 2D图像、截图、处理数据  
**应用场景：** 出版物图表、无损存储  
**Python库：**  
- `PIL/Pillow`：`Image.open('file.png')`  
- `scikit-image`：`io.imread('file.png')`  
- `imageio`：`imageio.imread('file.png')`  
**EDA方法：**  
- 位深度分析（8位、16位）  
- 色彩模式（灰度、RGB、调色板）  
- 元数据（PNG数据块）  
- 透明度处理  
- 压缩效率  
- 直方图分析  

### .jpg / .jpeg - 联合图像专家组  
**描述：** 有损压缩图像格式  
**典型数据：** 自然图像、照片  
**应用场景：** 可视化、网络图形（非原始数据）  
**Python库：**  
- `PIL/Pillow`：标准JPEG支持  
- `scikit-image`：JPEG读取  
**EDA方法：**  
- 压缩伪影检测  
- 质量因子估算  
- 色彩空间（RGB、灰度）  
- EXIF元数据  
- 量化表分析  
- 注意：不适用于定量分析  

### .bmp - 位图图像  
**描述：** 未压缩栅格图像  
**典型数据：** 简单图像、截图  
**应用场景：** 兼容性、简单存储  
**Python库：**  
- `PIL/Pillow`：BMP支持  
- `scikit-image`：BMP读取  
**EDA方法：**  
- 色彩深度  
- 调色板分析（索引模式）  
- 文件大小效率  
- 像素格式验证  

### .gif - 图形交换格式  
**描述：** 支持动画的图像格式  
**典型数据：** 动画图像、简单图形  
**应用场景：** 动画、延时可视化  
**Python库：**  
- `PIL/Pillow`：GIF支持  
- `imageio`：更佳GIF动画支持  
**EDA方法：**  
- 帧数与时序  
- 调色板限制（256色）  
- 循环次数  
- 帧处理方法  
- 透明度处理  

### .svg - 可缩放矢量图形  
**描述：** 基于XML的矢量图形  
**典型数据：** 矢量绘图、图表、示意图  
**应用场景：** 出版物级图表、绘图  
**Python库：**  
- `svgpathtools`：路径操作  
- `cairosvg`：栅格化  
- `lxml`：XML解析  
**EDA方法：**  
- 元素结构分析  
- 样式信息  
- 视框与尺寸  
- 路径复杂度  
- 文本元素提取  
- 图层组织  

### .eps - 封装式PostScript  
**描述：** 矢量图形格式  
**典型数据：** 出版物图表  
**应用场景：** 传统出版物图形  
**Python库：**  
- `PIL/Pillow`：基础EPS栅格化  
- 通过子进程调用`ghostscript`  
**EDA方法：**  
- 边界框信息  
- 预览图验证  
- 字体嵌入  
- 转换为现代格式  

### .pdf (图像)  
**描述：** 含图像的便携文档格式  
**典型数据：** 出版物图表、多页文档  
**应用场景：** 出版物、数据展示  
**Python库：**  
- `PyMuPDF/fitz`：`fitz.open('file.pdf')`  
- `pdf2image`：栅格化  
- `pdfplumber`：文本与布局提取

- OME-XML 验证
- 物理维度提取
- 通道命名与波长
- 平面位置（Z, C, T）
- 仪器元数据
- Bio-Formats 兼容性

### .ome.zarr - OME-ZARR
**描述：** 基于 ZARR 的 OME-NGFF 规范  
**典型数据：** 新一代生物成像文件格式  
**应用场景：** 云原生成像、大型数据集  
**Python 库：**  
- `ome-zarr-py`：官方实现  
- `zarr`：底层数组存储  
**EDA 分析方向：**  
- 多尺度分辨率层级  
- OME-NGFF 规范元数据合规性  
- 坐标变换  
- 标签与 ROI 处理  
- 云存储优化  
- 数据块访问模式  

### .klb - Keller Lab Block
**描述：** 面向大型数据的快速显微成像格式  
**典型数据：** 光片显微镜、延时成像  
**应用场景：** 高通量成像  
**Python 库：**  
- `pyklb`：KLB 读写支持  
**EDA 分析方向：**  
- 压缩效率  
- 块状结构  
- 多分辨率支持  
- 读取性能基准测试  
- 元数据提取  

### .vsi - 全玻片成像
**描述：** 虚拟玻片格式（多厂商支持）  
**典型数据：** 病理玻片、大型拼图  
**应用场景：** 数字病理学  
**Python 库：**  
- `openslide-python`：多格式 WSI 支持  
- `tiffslide`：纯 Python 替代方案  
**EDA 分析方向：**  
- 金字塔层级数量  
- 下采样因子  
- 关联图像（宏观图、标签图）  
- 切片尺寸与重叠区域  
- MPP（每像素微米值）  
- 背景检测  
- 组织分割  

### .ndpi - 滨松 NanoZoomer
**描述：** 滨松玻片扫描仪格式  
**典型数据：** 全玻片病理图像  
**应用场景：** 数字病理学工作流  
**Python 库：**  
- `openslide-python`：NDPI 支持  
**EDA 分析方向：**  
- 多分辨率金字塔  
- 镜头与物镜信息  
- 扫描区域与放大倍率  
- 焦平面信息  
- 组织检测  

### .svs - Aperio ScanScope
**描述：** Aperio 全玻片格式  
**典型数据：** 数字病理学玻片  
**应用场景：** 病理图像分析  
**Python 库：**  
- `openslide-python`：SVS 支持  
**EDA 分析方向：**  
- 金字塔结构  
- MPP 校准  
- 标签图与宏观图  
- 压缩质量  
- 缩略图生成  

### .scn - 徕卡 SCN
**描述：** 徕卡玻片扫描仪格式  
**典型数据：** 全玻片成像  
**应用场景：** 数字病理学  
**Python 库：**  
- `openslide-python`：SCN 支持  
**EDA 分析方向：**  
- 切片结构分析  
- 采集组织方式  
- 元数据提取  
- 放大倍率层级
