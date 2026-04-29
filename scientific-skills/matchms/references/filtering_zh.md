# matchms 过滤函数参考手册

本文档全面介绍了 matchms 中用于处理质谱数据的过滤函数。

## 元数据处理过滤器

### 化合物与化学信息

**add_compound_name(spectrum)**
- 将化合物名称添加至正确的元数据字段
- 标准化化合物名称存储位置

**clean_compound_name(spectrum)**
- 清除化合物名称中常见冗余信息
- 修正格式不一致问题

**derive_adduct_from_name(spectrum)**
- 从化合物名称提取加合物信息
- 将加合物标记移至正确元数据字段

**derive_formula_from_name(spectrum)**
- 检测化合物名称中的化学式
- 将化学式转移至适当元数据字段

**derive_annotation_from_compound_name(spectrum)**
- 通过化合物名称从PubChem获取SMILES/InChI
- 自动注释化学结构

### 化学结构转换

**derive_inchi_from_smiles(spectrum)**
- 从SMILES字符串生成InChI
- 需rdkit库支持

**derive_inchikey_from_inchi(spectrum)**
- 通过InChI计算InChIKey
- 27字符哈希标识符

**derive_smiles_from_inchi(spectrum)**
- 从InChI表示法创建SMILES
- 需rdkit库支持

**repair_inchi_inchikey_smiles(spectrum)**
- 修正错位的化学标识符
- 修复元数据字段混淆问题

**repair_not_matching_annotation(spectrum)**
- 确保SMILES、InChI和InChIKey一致性
- 验证化学结构注释匹配性

**add_fingerprint(spectrum, fingerprint_type="daylight", nbits=2048, radius=2)**
- 生成用于相似度计算的分子指纹
- 指纹类型："daylight", "morgan1", "morgan2", "morgan3"
- 与FingerprintSimilarity评分配合使用

### 质量与电荷信息

**add_precursor_mz(spectrum)**
- 标准化前体m/z值
- 统一前体质量元数据

**add_parent_mass(spectrum, estimate_from_adduct=True)**
- 根据前体m/z和加合物计算中性母体质量
- 无法直接获取时可基于加合物估算

**correct_charge(spectrum)**
- 使电荷值与离子模式对齐
- 确保电荷符号匹配电离模式

**make_charge_int(spectrum)**
- 将电荷转换为整数格式
- 标准化电荷表示形式

**clean_adduct(spectrum)**
- 标准化加合物标记
- 修正常见加合物格式问题

**interpret_pepmass(spectrum)**
- 解析pepmass字段为分量值
- 从合并字段提取前体m/z和强度

### 离子模式与验证

**derive_ionmode(spectrum)**
- 根据加合物信息确定离子模式
- 从加合物类型推断正/负模式

**require_correct_ionmode(spectrum, ion_mode)**
- 按指定离子模式过滤谱图
- 模式不匹配时返回None
- 用法：`spectrum = require_correct_ionmode(spectrum, "positive")`

**require_precursor_mz(spectrum, minimum_accepted_mz=0.0)**
- 验证前体m/z存在性及数值
- 缺失或低于阈值时返回None

**require_precursor_below_mz(spectrum, maximum_accepted_mz=1000.0)**
- 强制执行前体m/z上限
- 超过阈值时返回None

### 保留信息

**add_retention_time(spectrum)**
- 将保留时间统一为浮点值
- 标准化RT元数据字段

**add_retention_index(spectrum)**
- 在标准化字段存储保留指数
- 统一RI元数据

### 数据统一化

**harmonize_undefined_inchi(spectrum, undefined="", aliases=None)**
- 标准化未定义/空InChI条目
- 用统一值替换各种"未知"表示形式

**harmonize_undefined_inchikey(spectrum, undefined="", aliases=None)**
- 标准化未定义/空InChIKey条目
- 统一缺失数据表示形式

**harmonize_undefined_smiles(spectrum, undefined="", aliases=None)**
- 标准化未定义/空SMILES条目
- 统一处理缺失结构数据

### 修复与质量函数

**repair_adduct_based_on_smiles(spectrum, mass_tolerance=0.1)**
- 使用SMILES和质量匹配修正加合物
- 验证加合物与计算质量匹配性

**repair_parent_mass_is_mol_wt(spectrum, mass_tolerance=0.1)**
- 将分子量转换为单同位素质量
- 修复常见元数据混淆问题

**repair_precursor_is_parent_mass(spectrum)**
- 修正前体/母体质量值互换错误
- 修复字段分配错误

**repair_smiles_of_salts(spectrum, mass_tolerance=0.1)**
- 去除盐组分以匹配母体质量
- 提取相关分子片段

**require_parent_mass_match_smiles(spectrum, mass_tolerance=0.1)**
- 验证母体质量与SMILES计算质量匹配性
- 质量超出容差范围时返回None

**require_valid_annotation(spectrum)**
- 确保完整一致的化学注释
- 验证SMILES、InChI和InChIKey的存在性与一致性

## 峰处理过滤器

### 标准化与选择

**normalize_intensities(spectrum)**
- 将峰强度缩放至单位高度（最大值=1.0）
- 相似度计算的关键预处理步骤

**select_by_intensity(spectrum, intensity_from=0.0, intensity_to=1.0)**
- 保留指定绝对强度范围内的峰
- 按原始强度值过滤

**select_by_relative_intensity(spectrum, intensity_from=0.0, intensity_to=1.0)**
- 保留相对强度范围内的峰
- 按最大强度比例过滤

**select_by_mz(spectrum, mz_from=0.0, mz_to=1000.0)**
- 按m/z值范围过滤峰
- 移除指定m/z窗口外的峰

### 峰缩减与过滤

**reduce_to_number_of_peaks(spectrum, n_max=None, ratio_desired=None)**
- 超过最大值时移除最低强度峰
- 可指定绝对数量或比例
- 用法：`spectrum = reduce_to_number_of_peaks(spectrum, n_max=100)`

**remove_peaks_around_precursor_mz(spectrum, mz_tolerance=17)**
- 消除前体m/z容差范围内的峰
- 移除前体及同位素峰
- 基于碎片相似度计算的常用预处理

**remove_peaks_outside_top_k(spectrum, k=10, ratio_desired=None)**
- 仅保留k个最高强度峰附近的峰
- 聚焦于最具信息量的信号

**require_minimum_number_of_peaks(spectrum, n_required=10)**
- 丢弃峰数量不足的谱图
- 质量控制过滤器
- 峰数低于阈值时返回None

**require_minimum_number_of_high_peaks(spectrum, n_required=5, intensity_threshold=0.05)**
- 移除缺乏高强度峰的谱图
- 确保数据质量
- 阈值以上峰数不足时返回None

### 丢失计算

**add_losses(spectrum, loss_mz_from=5.0, loss_mz_to=200.0)**
- 从前体质量推导中性丢失
- 计算丢失量 = 前体m/z - 碎片m/z
- 为NeutralLossesCosine评分添加丢失信息

## 流程函数

**default_filters(spectrum)**
- 顺序应用九个核心元数据过滤器：
  1. make_charge_int
  2. add_precursor_mz
  3. add_retention_time
  4. add_retention_index
  5. derive_adduct_from_name
  6. derive_formula_from_name
  7. clean_compound_name
  8. harmonize_undefined_smiles
  9. harmonize_undefined_inchi
- 元数据统一化的推荐起点

**SpectrumProcessor(filters)**
- 编排多过滤器流程
- 接收过滤器函数列表
- 示例：
```python
from matchms import SpectrumProcessor
processor = SpectrumProcessor([
    default_filters,
    normalize_intensities,
    lambda s: select_by_relative_intensity(s, intensity_from=0.01)
])
processed = processor(spectrum)
```

## 常用过滤器组合

### 标准预处理流程
```python
from matchms.filtering import (default_filters, normalize_intensities,
                               select_by_relative_intensity,
                               require_minimum_number_of_peaks)

spectrum = default_filters(spectrum)
spectrum = normalize_intensities(spectrum)
spectrum = select_by_relative_intensity(spectrum, intensity_from=0.01)
spectrum = require_minimum_number_of_peaks(spectrum, n_required=5)
```

### 质量控制流程
```python
from matchms.filtering import (require_precursor_mz, require_minimum_number_of_peaks,
                               require_minimum_number_of_high_peaks)

spectrum = require_precursor_mz(spectrum, minimum_accepted_mz=50.0)
if spectrum is None:
    # 谱图未通过质量控制
    pass
spectrum = require_minimum_number_of_peaks(spectrum, n_required=10)
spectrum = require_minimum_number_of_high_peaks(spectrum, n_required=5)
```

### 化学注释流程
```python
from matchms.filtering import (derive_inchi_from_smiles, derive_inchikey_from_inchi,
                               add_fingerprint, require_valid_annotation)

spectrum = derive_inchi_from_smiles(spectrum)
spectrum = derive_inchikey_from_inchi(spectrum)
spectrum = add_fingerprint(spectrum, fingerprint_type="morgan2", nbits=2048)
spectrum = require_valid_annotation(spectrum)
```

### 峰清理流程
```python
from matchms.filtering import (normalize_intensities, remove_peaks_around_precursor_mz,
                               select_by_relative_intensity, reduce_to_number_of_peaks)

spectrum = normalize_intensities(spectrum)
spectrum = remove_peaks_around_precursor_mz(spectrum, mz_tolerance=17)
spectrum = select_by_relative_intensity(spectrum, intensity_from=0.01)
spectrum = reduce_to_number_of_peaks(spectrum, n_max=200)
```

## 过滤器使用说明

1. **顺序重要性**：按逻辑顺序应用过滤器（如先标准化再选择相对强度）
2. **返回None机制**：无效谱图时许多过滤器返回None；继续处理前需检查None
3. **不可变性**：过滤器通常返回修改后的副本；需将结果重新赋值给变量
4. **流程效率**：使用SpectrumProcessor实现多谱图一致性处理
5. **文档参考**：详细参数请参阅 matchms.readthedocs.io/en/latest/api/matchms.filtering.html
