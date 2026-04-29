# 常用DICOM标签参考

本文档按类别整理了常用的DICOM标签列表。在pydicom中可通过属性表示法（如`ds.PatientName`）或标签元组表示法（如`ds[0x0010, 0x0010]`）访问标签。

## 患者信息标签

| 标签 | 名称 | 类型 | 描述 |
|-----|------|------|-------------|
| (0010,0010) | PatientName | PN | 患者全名 |
| (0010,0020) | PatientID | LO | 患者主标识符 |
| (0010,0030) | PatientBirthDate | DA | 出生日期（YYYYMMDD） |
| (0010,0032) | PatientBirthTime | TM | 出生时间（HHMMSS） |
| (0010,0040) | PatientSex | CS | 患者性别（M，F，O） |
| (0010,1010) | PatientAge | AS | 患者年龄（格式：nnnD/W/M/Y） |
| (0010,1020) | PatientSize | DS | 患者身高（米） |
| (0010,1030) | PatientWeight | DS | 患者体重（千克） |
| (0010,1040) | PatientAddress | LO | 患者邮寄地址 |
| (0010,2160) | EthnicGroup | SH | 患者族群 |
| (0010,4000) | PatientComments | LT | 患者相关附加注释 |

## 检查信息标签

| 标签 | 名称 | 类型 | 描述 |
|-----|------|------|-------------|
| (0020,000D) | StudyInstanceUID | UI | 检查唯一标识符 |
| (0008,0020) | StudyDate | DA | 检查开始日期（YYYYMMDD） |
| (0008,0030) | StudyTime | TM | 检查开始时间（HHMMSS） |
| (0008,1030) | StudyDescription | LO | 检查描述 |
| (0020,0010) | StudyID | SH | 用户或站点定义的检查标识符 |
| (0008,0050) | AccessionNumber | SH | RIS生成的检查标识符 |
| (0008,0090) | ReferringPhysicianName | PN | 转诊医师姓名 |
| (0008,1060) | NameOfPhysiciansReadingStudy | PN | 阅片医师姓名 |
| (0008,1080) | AdmittingDiagnosesDescription | LO | 入院诊断描述 |

## 序列信息标签

| 标签 | 名称 | 类型 | 描述 |
|-----|------|------|-------------|
| (0020,000E) | SeriesInstanceUID | UI | 序列唯一标识符 |
| (0020,0011) | SeriesNumber | IS | 序列数字标识符 |
| (0008,103E) | SeriesDescription | LO | 序列描述 |
| (0008,0060) | Modality | CS | 设备类型（CT、MR、US等） |
| (0008,0021) | SeriesDate | DA | 序列开始日期（YYYYMMDD） |
| (0008,0031) | SeriesTime | TM | 序列开始时间（HHMMSS） |
| (0018,0015) | BodyPartExamined | CS | 检查身体部位 |
| (0018,5100) | PatientPosition | CS | 患者体位（HFS、FFS等） |
| (0020,0060) | Laterality | CS | 成对身体部位偏侧性（R、L） |

## 图像信息标签

| 标签 | 名称 | 类型 | 描述 |
|-----|------|------|-------------|
| (0008,0018) | SOPInstanceUID | UI | 实例唯一标识符 |
| (0020,0013) | InstanceNumber | IS | 图像标识编号 |
| (0008,0008) | ImageType | CS | 图像识别特征 |
| (0008,0023) | ContentDate | DA | 内容创建日期（YYYYMMDD） |
| (0008,0033) | ContentTime | TM | 内容创建时间（HHMMSS） |
| (0020,0032) | ImagePositionPatient | DS | 图像位置（x, y, z，单位：毫米） |
| (0020,0037) | ImageOrientationPatient | DS | 图像行列方向余弦 |
| (0020,1041) | SliceLocation | DS | 图像平面相对位置 |
| (0018,0050) | SliceThickness | DS | 层厚（毫米） |
| (0018,0088) | SpacingBetweenSlices | DS | 层间距（毫米） |

## 像素数据标签

| 标签 | 名称 | 类型 | 描述 |
|-----|------|------|-------------|
| (7FE0,0010) | PixelData | OB/OW | 图像实际像素数据 |
| (0028,0010) | Rows | US | 图像行数 |
| (0028,0011) | Columns | US | 图像列数 |
| (0028,0100) | BitsAllocated | US | 每个像素样本分配的比特数 |
| (0028,0101) | BitsStored | US | 每个像素样本存储的比特数 |
| (0028,0102) | HighBit | US | 像素样本最高有效位 |
| (0028,0103) | PixelRepresentation | US | 0=无符号，1=有符号 |
| (0028,0002) | SamplesPerPixel | US | 每像素样本数（1或3） |
| (0028,0004) | PhotometricInterpretation | CS | 色彩空间（MONOCHROME2、RGB等） |
| (0028,0006) | PlanarConfiguration | US | 彩色像素数据排列方式 |
| (0028,0030) | PixelSpacing | DS | 物理间距[行, 列]（毫米） |
| (0028,0008) | NumberOfFrames | IS | 多帧图像的帧数 |
| (0028,0034) | PixelAspectRatio | IS | 像素纵横比（垂直/水平） |

## 窗宽窗位标签

| 标签 | 名称 | 类型 | 描述 |
|-----|------|------|-------------|
| (0028,1050) | WindowCenter | DS | 显示窗位 |
| (0028,1051) | WindowWidth | DS | 显示窗宽 |
| (0028,1052) | RescaleIntercept | DS | 输出公式中的b（输出 = m*SV + b） |
| (0028,1053) | RescaleSlope | DS | 输出公式中的m（输出 = m*SV + b） |
| (0028,1054) | RescaleType | LO | 重缩放类型（HU等） |
| (0028,1055) | WindowCenterWidthExplanation | LO | 窗宽窗位说明 |
| (0028,3010) | VOILUTSequence | SQ | 窗值转换序列描述 |

## CT专用标签

| 标签 | 名称 | 类型 | 描述 |
|-----|------|------|-------------|
| (0018,0060) | KVP | DS | 峰值千伏电压 |
| (0018,1030) | ProtocolName | LO | 扫描协议名称 |
| (0018,1100) | ReconstructionDiameter | DS | 重建圆直径 |
| (0018,1110) | DistanceSourceToDetector | DS | 源到探测器距离（毫米） |
| (0018,1111) | DistanceSourceToPatient | DS | 源到患者距离（毫米） |
| (0018,1120) | GantryDetectorTilt | DS | 机架倾斜角度（度） |
| (0018,1130) | TableHeight | DS | 检查床高度（毫米） |
| (0018,1150) | ExposureTime | IS | 曝光时间（毫秒） |
| (0018,1151) | XRayTubeCurrent | IS | X射线管电流（毫安） |
| (0018,1152) | Exposure | IS | 曝光量（毫安秒） |
| (0018,1160) | FilterType | SH | X射线滤过材料 |
| (0018,1210) | ConvolutionKernel | SH | 重建算法 |

## MR专用标签

| 标签 | 名称 | 类型 | 描述 |
|-----|------|------|-------------|
| (0018,0080) | RepetitionTime | DS | 重复时间TR（毫秒） |
| (0018,0081) | EchoTime | DS | 回波时间TE（毫秒） |
| (0018,0082) | InversionTime | DS | 反转时间TI（毫秒） |
| (0018,0083) | NumberOfAverages | DS | 数据平均次数 |
| (0018,0084) | ImagingFrequency | DS | 成像频率（兆赫兹） |
| (0018,0085) | ImagedNucleus | SH | 成像原子核（1H等） |
| (0018,0086) | EchoNumbers | IS | 回波编号 |
| (0018,0087) | MagneticFieldStrength | DS | 磁场强度（特斯拉） |
| (0018,0088) | SpacingBetweenSlices | DS | 层间距（毫米） |
| (0018,0089) | NumberOfPhaseEncodingSteps | IS | 相位编码步数 |
| (0018,0091) | EchoTrainLength | IS | 回波链长度 |
| (0018,0093) | PercentSampling | DS | 采集矩阵采样百分比 |
| (0018,0094) | PercentPhaseFieldOfView | DS | 相位与频率视野比例 |
| (0018,1030) | ProtocolName | LO | 扫描协议名称 |
| (0018,1314) | FlipAngle | DS | 翻转角度（度） |

## 文件元信息标签

| 标签 | 名称 | 类型 | 描述 |
|-----|------|------|-------------|
| (0002,0000) | FileMetaInformationGroupLength | UL | 文件元信息长度 |
| (0002,0001) | FileMetaInformationVersion | OB | 文件元信息版本 |
| (0002,0002) | MediaStorageSOPClassUID | UI | SOP类UID |
| (0002,0003) | MediaStorageSOPInstanceUID | UI | SOP实例UID |
| (0002,0010) | TransferSyntaxUID | UI | 传输语法UID |
| (0002,0012) | ImplementationClassUID | UI | 实现类UID |
| (0002,0013) | ImplementationVersionName | SH | 实现版本名称 |

## 设备信息标签

| 标签 | 名称 | 类型 | 描述 |
|-----|------|------|-------------|
| (0008,0070) | Manufacturer | LO | 设备制造商 |
| (0008,0080) | InstitutionName | LO | 机构名称 |
| (0008,0081) | InstitutionAddress | ST | 机构地址 |
| (0008,1010) | StationName | SH | 设备站点名称 |
| (0008,1040) | InstitutionalDepartmentName | LO | 科室名称 |
| (0008,1050) | PerformingPhysicianName | PN | 操作医师姓名 |
| (0008,1070) | OperatorsName | PN | 操作员姓名 |
| (0008,1090) | ManufacturerModelName | LO | 设备型号名称 |
| (0018,1000) | DeviceSerialNumber | LO | 设备序列号 |
| (0018,1020) | SoftwareVersions | LO | 软件版本 |

## 时间信息标签

| 标签 | 名称 | 类型 | 描述 |
|-----|------|------|-------------|
| (0008,0012) | InstanceCreationDate | DA | 实例创建日期 |
| (0008,0013) | InstanceCreationTime | TM | 实例创建时间 |
| (0008,0022) | AcquisitionDate | DA | 采集开始日期 |
| (0008,0032) | AcquisitionTime | TM | 采集开始时间 |
| (0008,002A) | AcquisitionDateTime | DT | 采集日期时间 |

## DICOM值表示类型（VR）

DICOM中常用的值表示类型：

- **AE**: 应用实体（最多16字符）
- **AS**: 年龄字符串（nnnD/W/M/Y）
- **CS**: 代码字符串（最多16字符）
- **DA**: 日期（YYYYMMDD）
- **DS**: 十进制字符串
- **DT**: 日期时间（YYYYMMDDHHMMSS.FFFFFF&ZZXX）
- **IS**: 整数字符串
- **LO**: 长字符串（最多64字符）
- **LT**: 长文本（最多10240字符）
- **PN**: 人员姓名
- **SH**: 短字符串（最多16字符）
- **SQ**: 项目序列
- **ST**: 短文本（最多1024字符）
- **TM**: 时间（HHMMSS.FFFFFF）
- **UI**: 唯一标识符（UID）
- **UL**: 无符号长整型（4字节）
- **US**: 无符号短整型（2字节）
- **OB**: 其他字节字符串
- **OW**: 其他字字符串

## 使用示例

### 通过名称访问标签
```python
patient_name = ds.PatientName
study_date = ds.StudyDate
modality = ds.Modality
```

### 通过编号访问标签
```python
patient_name = ds[0x0010, 0x0010].value
study_date = ds[0x0008, 0x0020].value
modality = ds[0x0008, 0x0060].value
```

### 检查标签是否存在
```python
if hasattr(ds, 'PatientName'):
    print(ds.PatientName)

# 或使用'in'运算符
if (0x0010, 0x0010) in ds:
    print(ds[0x0010, 0x0010].value)
```

### 带默认值的安全访问
```python
patient_name = getattr(ds, 'PatientName', 'Unknown')
study_desc = ds.get('StudyDescription', 'No description')
```

## 参考资料

- DICOM标准：https://www.dicomstandard.org/
- DICOM标签浏览器：https://dicom.innolitics.com/ciods
- Pydicom文档：https://pydicom.github.io/pydicom/
