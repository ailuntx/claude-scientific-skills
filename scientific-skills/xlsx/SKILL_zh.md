---
name: xlsx
description: "当电子表格文件是主要输入或输出时使用此技能。这意味着用户需要：打开、读取、编辑或修复现有的.xlsx、.xlsm、.csv或.tsv文件（例如添加列、计算公式、格式化、图表制作、清理混乱数据）；从头创建新电子表格或从其他数据源生成；或在表格文件格式之间转换。尤其当用户提及文件名或路径（即使随意如“我下载的xlsx文件”）并希望对其操作或从中生成内容时触发。同样适用于将混乱的表格数据文件（格式错误行、错位表头、垃圾数据）清理或重组为规范电子表格。交付物必须是电子表格文件。当主要交付物是Word文档、HTML报告、独立Python脚本、数据库管道或Google Sheets API集成时，即使涉及表格数据也勿触发。"
license: 专有许可。完整条款见LICENSE.txt
---

# 输出要求

## 所有Excel文件

### 专业字体
- 除非用户另有指示，所有交付物使用统一专业字体（如Arial、Times New Roman）

### 零公式错误
- 每个Excel模型交付时必须无公式错误（#REF!、#DIV/0!、#VALUE!、#N/A、#NAME?）

### 保留现有模板（更新模板时）
- 修改文件时需严格遵循现有格式、样式和规范
- 不得对已有固定模式的文件强加标准化格式
- 现有模板规范始终优先于本指南

## 财务模型

### 颜色编码标准
除非用户或现有模板另有说明

#### 行业标准颜色规范
- **蓝色文本（RGB: 0,0,255）**：硬编码输入值及用户需调整的场景数值
- **黑色文本（RGB: 0,0,0）**：所有公式和计算
- **绿色文本（RGB: 0,128,0）**：同一工作簿内跨工作表链接
- **红色文本（RGB: 255,0,0）**：指向其他文件的外部链接
- **黄色背景（RGB: 255,255,0）**：需关注的关键假设或待更新单元格

### 数字格式标准

#### 强制格式规则
- **年份**：格式化为文本字符串（如"2024"而非"2,024"）
- **货币**：使用$#,##0格式；表头必须标注单位（如"收入（$百万）"）
- **零值**：通过数字格式使所有零显示为"-"，含百分比（如"$#,##0;($#,##0);-"）
- **百分比**：默认0.0%格式（保留一位小数）
- **倍数**：估值倍数格式化为0.0x（如EV/EBITDA、P/E）
- **负数**：使用括号(123)而非减号-123

### 公式构建规则

#### 假设项放置
- 所有假设（增长率、利润率、倍数等）置于独立假设单元格
- 公式中必须引用单元格而非硬编码数值
- 示例：使用=B5*(1+$B$6)而非=B5*1.05

#### 公式错误预防
- 验证所有单元格引用正确
- 检查范围偏移错误
- 确保所有预测期公式一致
- 边界值测试（零值、负数）
- 确认无意外循环引用

#### 硬编码数据来源标注
- 在单元格批注或右侧标注（若在表格末尾）。格式："来源：[系统/文档]，[日期]，[具体引用]，[URL（若适用）]"
- 示例：
  - "来源：公司10-K年报，2024财年，第45页，收入注释，[SEC EDGAR链接]"
  - "来源：公司10-Q季报，2025年Q2，附件99.1，[SEC EDGAR链接]"
  - "来源：彭博终端，2025/8/15，AAPL US Equity"
  - "来源：FactSet，2025/8/20，一致预期屏幕"

# XLSX文件创建、编辑与分析

## 概述

用户可能要求创建、编辑或分析.xlsx文件内容。针对不同任务需采用不同工具和工作流。

## 重要要求

**公式重算需LibreOffice**：可通过`scripts/recalc.py`脚本调用LibreOffice重算公式值。首次运行时脚本会自动配置LibreOffice，包括在Unix套接字受限的沙盒环境中（由`scripts/office/soffice.py`处理）

## 数据读取与分析

### 使用pandas分析数据
进行数据分析、可视化和基础操作时，使用强大的**pandas**库：

```python
import pandas as pd

# 读取Excel
df = pd.read_excel('file.xlsx')  # 默认读取首工作表
all_sheets = pd.read_excel('file.xlsx', sheet_name=None)  # 以字典形式读取所有工作表

# 分析
df.head()      # 数据预览
df.info()      # 列信息
df.describe()  # 统计摘要

# 写入Excel
df.to_excel('output.xlsx', index=False)
```

## Excel文件工作流

## 关键原则：使用公式而非硬编码值

**始终使用Excel公式而非在Python中计算后硬编码**。这确保电子表格保持动态可更新性。

### ❌ 错误做法 - 硬编码计算值
```python
# 错误：在Python计算后硬编码结果
total = df['Sales'].sum()
sheet['B10'] = total  # 硬编码5000

# 错误：在Python计算增长率
growth = (df.iloc[-1]['Revenue'] - df.iloc[0]['Revenue']) / df.iloc[0]['Revenue']
sheet['C5'] = growth  # 硬编码0.15

# 错误：在Python计算平均值
avg = sum(values) / len(values)
sheet['D20'] = avg  # 硬编码42.5
```

### ✅ 正确做法 - 使用Excel公式
```python
# 正确：让Excel计算总和
sheet['B10'] = '=SUM(B2:B9)'

# 正确：使用Excel公式计算增长率
sheet['C5'] = '=(C4-C2)/C2'

# 正确：使用Excel函数计算平均值
sheet['D20'] = '=AVERAGE(D2:D19)'
```

此原则适用于所有计算（总计、百分比、比率、差值等）。当源数据变更时，电子表格应能重新计算。

## 通用工作流
1. **选择工具**：数据分析用pandas，公式/格式用openpyxl
2. **创建/加载**：新建工作簿或加载现有文件
3. **修改**：增删数据、公式及格式
4. **保存**：写入文件
5. **公式重算（使用公式时必做）**：执行scripts/recalc.py脚本
   ```bash
   python scripts/recalc.py output.xlsx
   ```
6. **验证并修复错误**：
   - 脚本返回含错误详情的JSON
   - 若`status`为`errors_found`，检查`error_summary`中的具体错误类型及位置
   - 修复后重新执行重算
   - 常见需修复错误：
     - `#REF!`：无效单元格引用
     - `#DIV/0!`：除数为零
     - `#VALUE!`：公式数据类型错误
     - `#NAME?`：无法识别的公式名称

### 创建新Excel文件

```python
# 使用openpyxl处理公式和格式
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment

wb = Workbook()
sheet = wb.active

# 添加数据
sheet['A1'] = 'Hello'
sheet['B1'] = 'World'
sheet.append(['行数据1', '行数据2'])

# 添加公式
sheet['B2'] = '=SUM(A1:A10)'

# 格式设置
sheet['A1'].font = Font(bold=True, color='FF0000')
sheet['A1'].fill = PatternFill('solid', start_color='FFFF00')
sheet['A1'].alignment = Alignment(horizontal='center')

# 列宽调整
sheet.column_dimensions['A'].width = 20

wb.save('output.xlsx')
```

### 编辑现有Excel文件

```python
# 使用openpyxl保留公式和格式
from openpyxl import load_workbook

# 加载现有文件
wb = load_workbook('existing.xlsx')
sheet = wb.active  # 或通过wb['工作表名']指定

# 多工作表处理
for sheet_name in wb.sheetnames:
    sheet = wb[sheet_name]
    print(f"工作表: {sheet_name}")

# 修改单元格
sheet['A1'] = '新值'
sheet.insert_rows(2)  # 在第2行插入
sheet.delete_cols(3)  # 删除第3列

# 添加新工作表
new_sheet = wb.create_sheet('新建工作表')
new_sheet['A1'] = '数据'

wb.save('modified.xlsx')
```

## 公式重算

openpyxl创建或修改的Excel文件包含公式字符串而非计算值。使用`scripts/recalc.py`脚本重算公式：

```bash
python scripts/recalc.py <excel文件> [超时秒数]
```

示例：
```bash
python scripts/recalc.py output.xlsx 30
```

脚本功能：
- 首次运行时自动配置LibreOffice宏
- 重算所有工作表的全部公式
- 扫描所有单元格的Excel错误（#REF!、#DIV/0!等）
- 返回含错误位置和计数的详细JSON
- 兼容Linux和macOS

## 公式验证清单

确保公式正确的快速检查项：

### 基础验证
- [ ] **测试2-3个样本引用**：构建完整模型前验证取值正确性
- [ ] **列映射确认**：确保Excel列匹配（如第64列应为BL而非BK）
- [ ] **行偏移注意**：Excel行号从1开始（DataFrame第5行=Excel第6行）

### 常见陷阱
- [ ] **NaN处理**：用`pd.notna()`检查空值
- [ ] **超右端列**：财年数据常在50+列
- [ ] **多重匹配**：搜索所有出现位置而非首个
- [ ] **除零错误**：公式中使用"/"前检查分母（#DIV/0!）
- [ ] **错误引用**：验证所有单元格引用指向正确位置（#REF!）
- [ ] **跨表引用**：链接工作表时使用正确格式（Sheet1!A1）

### 公式测试策略
- [ ] **小范围测试**：大范围应用前先在2-3个单元格测试公式
- [ ] **验证依赖项**：检查公式引用的所有单元格是否存在
- [ ] **边界值测试**：包含零值、负值及极大值

### 解析scripts/recalc.py输出
脚本返回含错误详情的JSON：
```json
{
  "status": "success",           // 或"errors_found"
  "total_errors": 0,              // 错误总数
  "total_formulas": 42,           // 文件公式总数
  "error_summary": {              // 仅当存在错误时出现
    "#REF!": {
      "count": 2,
      "locations": ["Sheet1!B5", "Sheet1!C10"]
    }
  }
}
```

## 最佳实践

### 库选择
- **pandas**：适用于数据分析、批量操作及简单数据导出
- **openpyxl**：适用于复杂格式、公式及Excel特性

### openpyxl使用要点
- 单元格索引从1开始（行=1, 列=1对应A1单元格）
- 读取计算值用`data_only=True`：`load_workbook('file.xlsx', data_only=True)`
- **警告**：以`data_only=True`打开后保存会永久丢失公式
- 大文件处理：读取用`read_only=True`，写入用`write_only=True`
- 公式会被保留但不计算 - 需用scripts/recalc.py更新值

### pandas使用要点
- 指定数据类型避免推断错误：`pd.read_excel('file.xlsx', dtype={'id': str})`
- 大文件读取特定列：`pd.read_excel('file.xlsx', usecols=['A', 'C', 'E'])`
- 正确处理日期：`pd.read_excel('file.xlsx', parse_dates=['日期列'])`

## 代码风格指南
**重要**：生成Excel操作Python代码时：
- 编写简洁代码，避免冗余注释
- 避免冗长变量名和重复操作
- 避免非必要打印语句

**Excel文件本身**：
- 为复杂公式或关键假设添加单元格批注
- 硬编码值需标注数据来源
- 关键计算和模型章节添加说明
