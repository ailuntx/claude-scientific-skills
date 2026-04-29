# 编辑演示文稿

## 基于模板的工作流

使用现有演示文稿作为模板时：

1. **分析现有幻灯片**：
   ```bash
   python scripts/thumbnail.py template.pptx
   python -m markitdown template.pptx
   ```
   查看 `thumbnails.jpg` 了解布局，检查 markitdown 输出查看占位符文本。

2. **规划幻灯片映射**：为每个内容区块选择模板幻灯片。

   ⚠️ **使用多样化布局**——单调的演示是常见失败模式。避免默认使用基础标题+项目符号幻灯片。主动寻找：
   - 多列布局（双列、三列）
   - 图文组合
   - 全出血图片配文字覆盖
   - 引用或标注幻灯片
   - 章节分隔页
   - 数据/数字标注
   - 图标网格或图标+文本行

   **避免：** 每页重复使用相同文字密集型布局。

   根据内容类型匹配布局风格（例如：要点→项目符号页，团队信息→多列布局，客户评价→引用页）。

3. **解包**：`python scripts/office/unpack.py template.pptx unpacked/`

4. **构建演示文稿**（自行操作，勿用子代理）：
   - 删除多余幻灯片（从 `<p:sldIdLst>` 移除）
   - 复制需重用的幻灯片（使用 `add_slide.py`）
   - 在 `<p:sldIdLst>` 中调整顺序
   - **所有结构调整必须在步骤5前完成**

5. **编辑内容**：更新每个 `slide{N}.xml` 中的文本。
   **此处可使用子代理**——幻灯片是独立XML文件，子代理可并行编辑。

6. **清理**：`python scripts/clean.py unpacked/`

7. **打包**：`python scripts/office/pack.py unpacked/ output.pptx --original template.pptx`

---

## 脚本说明

| 脚本 | 用途 |
|--------|---------|
| `unpack.py` | 解压并格式化PPTX |
| `add_slide.py` | 复制幻灯片或从布局创建 |
| `clean.py` | 清除孤立文件 |
| `pack.py` | 带验证的重新打包 |
| `thumbnail.py` | 创建幻灯片视觉网格 |

### unpack.py

```bash
python scripts/office/unpack.py input.pptx unpacked/
```

解压PPTX，格式化XML，转义智能引号。

### add_slide.py

```bash
python scripts/add_slide.py unpacked/ slide2.xml      # 复制幻灯片
python scripts/add_slide.py unpacked/ slideLayout2.xml # 从布局创建
```

输出需插入 `<p:sldIdLst>` 指定位置的 `<p:sldId>`。

### clean.py

```bash
python scripts/clean.py unpacked/
```

清除未在 `<p:sldIdLst>` 引用的幻灯片、未关联媒体及孤立关系文件。

### pack.py

```bash
python scripts/office/pack.py unpacked/ output.pptx --original input.pptx
```

验证修复、压缩XML、重新编码智能引号。

### thumbnail.py

```bash
python scripts/thumbnail.py input.pptx [output_prefix] [--cols N]
```

创建带幻灯片文件名标签的 `thumbnails.jpg`。默认3列，每网格最多12张。

**仅用于模板分析**（选择布局）。视觉质量检查请用 `soffice` + `pdftoppm` 生成高清单页图——参见SKILL.md。

---

## 幻灯片操作

幻灯片顺序位于 `ppt/presentation.xml` → `<p:sldIdLst>`。

**调整顺序**：重排 `<p:sldId>` 元素。

**删除**：移除 `<p:sldId>` 后执行 `clean.py`。

**新增**：使用 `add_slide.py`。切勿手动复制幻灯片文件——该脚本处理备注引用、Content_Types.xml 和手动复制会遗漏的关系ID。

---

## 内容编辑

**子代理：** 如可用，在此步骤使用（完成步骤4后）。每张幻灯片是独立XML文件，子代理可并行编辑。给子代理的指令需包含：
- 待编辑的幻灯片文件路径
- **"所有修改必须使用编辑工具"**
- 下方格式规则及常见陷阱

每张幻灯片操作：
1. 读取XML
2. 识别所有占位内容（文本/图片/图表/图标/标注）
3. 用最终内容替换每个占位符

**使用编辑工具而非sed或Python脚本**。编辑工具强制明确替换目标，可靠性更高。

### 格式规则

- **所有标题/子标题/行内标签加粗**：在 `<a:rPr>` 设置 `b="1"`。包括：
  - 幻灯片主标题
  - 页内分区标题
  - 行首标签（如"状态："、"描述："）
- **禁用Unicode项目符号（•）**：使用 `<a:buChar>` 或 `<a:buAutoNum>` 规范列表格式
- **项目符号一致性**：从布局继承符号。仅指定 `<a:buChar>` 或 `<a:buNone>`

---

## 常见陷阱

### 模板适配

当源内容少于模板元素时：
- **完全移除多余元素**（图片/形状/文本框），勿仅清除文本
- 清除文本后检查残留视觉元素
- 执行视觉质量检查以发现数量不匹配

替换文本长度变化时：
- **缩短内容**：通常安全
- **延长内容**：可能溢出或异常换行
- 文本修改后执行视觉质量检查
- 考虑截断或拆分内容以适应模板设计约束

**模板槽位≠源内容项**：若模板有4个团队成员位但源数据仅3人，删除第4成员的全部组块（图片+文本框），而非仅清除文本。

### 多项目内容

源内容含多项时（编号列表/多区块），为每项创建独立 `<a:p>` 元素——**切勿拼接为单一字符串**。

**❌ 错误**——所有项目合并为单段落：
```xml
<a:p>
  <a:r><a:rPr .../><a:t>步骤1：执行第一步。步骤2：执行第二步。</a:t></a:r>
</a:p>
```

**✅ 正确**——带加粗标题的独立段落：
```xml
<a:p>
  <a:pPr algn="l"><a:lnSpc><a:spcPts val="3919"/></a:lnSpc></a:pPr>
  <a:r><a:rPr lang="en-US" sz="2799" b="1" .../><a:t>步骤1</a:t></a:r>
</a:p>
<a:p>
  <a:pPr algn="l"><a:lnSpc><a:spcPts val="3919"/></a:lnSpc></a:pPr>
  <a:r><a:rPr lang="en-US" sz="2799" .../><a:t>执行第一步。</a:t></a:r>
</a:p>
<a:p>
  <a:pPr algn="l"><a:lnSpc><a:spcPts val="3919"/></a:lnSpc></a:pPr>
  <a:r><a:rPr lang="en-US" sz="2799" b="1" .../><a:t>步骤2</a:t></a:r>
</a:p>
<!-- 延续此模式 -->
```

从原段落复制 `<a:pPr>` 保留行距。标题使用 `b="1"`。

### 智能引号

解包/打包过程自动处理。但编辑工具会将智能引号转为ASCII。

**新增含引号文本时使用XML实体：**

```xml
<a:t>该 &#x201C;协议&#x201D;</a:t>
```

| 字符 | 名称 | Unicode | XML实体 |
|-----------|------|---------|------------|
| `“` | 左双引号 | U+201C | `&#x201C;` |
| `”` | 右双引号 | U+201D | `&#x201D;` |
| `‘` | 左单引号 | U+2018 | `&#x2018;` |
| `’` | 右单引号 | U+2019 | `&#x2019;` |

### 其他

- **空格处理**：含首尾空格的 `<a:t>` 使用 `xml:space="preserve"`
- **XML解析**：使用 `defusedxml.minidom`，禁用 `xml.etree.ElementTree`（破坏命名空间）
