---
name: docx
description: "当用户需要创建、读取、编辑或操作Word文档（.docx文件）时使用此技能。触发条件包括：提及'Word文档'、'word文档'、'.docx'，或要求生成带格式的专业文档（如含目录、标题、页码、信头）。同样适用于从.docx文件提取/重组内容、插入/替换文档图片、执行查找替换、处理修订批注，或将内容转换为精美Word文档。若用户要求生成'报告'、'备忘录'、'信函'、'模板'等Word/.docx交付物，使用此技能。不适用于PDF、电子表格、Google文档或与文档生成无关的通用编程任务。"
license: 专有许可。完整条款见LICENSE.txt
---

# DOCX文档创建、编辑与分析

## 概述

.docx文件本质上是包含XML文件的ZIP压缩包。

## 快速参考

| 任务               | 方法                     |
|--------------------|--------------------------|
| 读取/分析内容      | `pandoc` 或解包查看原始XML |
| 创建新文档         | 使用 `docx-js` - 见下文"创建新文档" |
| 编辑现有文档       | 解包 → 编辑XML → 重新打包 - 见下文"编辑现有文档" |

### 转换.doc为.docx

旧版`.doc`文件必须先转换才能编辑：
```bash
python scripts/office/soffice.py --headless --convert-to docx document.doc
```

### 读取内容
```bash
# 带修订标记的文本提取
pandoc --track-changes=all document.docx -o output.md

# 原始XML访问
python scripts/office/unpack.py document.docx unpacked/
```

### 转换为图片
```bash
python scripts/office/soffice.py --headless --convert-to pdf document.docx
pdftoppm -jpeg -r 150 document.pdf page
```

### 接受修订标记
生成已接受所有修订的纯净文档（需LibreOffice）：
```bash
python scripts/accept_changes.py input.docx output.docx
```

---

## 创建新文档

使用JavaScript生成.docx文件后需验证。安装：`npm install -g docx`

### 初始化
```javascript
const { Document, Packer, Paragraph, TextRun, Table, TableRow, TableCell, ImageRun,
        Header, Footer, AlignmentType, PageOrientation, LevelFormat, ExternalHyperlink,
        InternalHyperlink, Bookmark, FootnoteReferenceRun, PositionalTab,
        PositionalTabAlignment, PositionalTabRelativeTo, PositionalTabLeader,
        TabStopType, TabStopPosition, Column, SectionType,
        TableOfContents, HeadingLevel, BorderStyle, WidthType, ShadingType,
        VerticalAlign, PageNumber, PageBreak } = require('docx');

const doc = new Document({ sections: [{ children: [/* 内容 */] }] });
Packer.toBuffer(doc).then(buffer => fs.writeFileSync("doc.docx", buffer));
```

### 验证
文件创建后必须验证。若验证失败，需解包修复XML后重新打包：
```bash
python scripts/office/validate.py doc.docx
```

### 页面尺寸
```javascript
// 关键：docx-js默认A4而非US Letter
// 始终显式设置页面尺寸保证一致性
sections: [{
  properties: {
    page: {
      size: {
        width: 12240,   // 8.5英寸（DXA单位）
        height: 15840   // 11英寸（DXA单位）
      },
      margin: { top: 1440, right: 1440, bottom: 1440, left: 1440 } // 1英寸页边距
    }
  },
  children: [/* 内容 */]
}]
```

**常用页面尺寸（DXA单位，1440 DXA = 1英寸）：**

| 纸张类型 | 宽度 | 高度 | 内容宽度（1英寸页边距） |
|----------|------|------|------------------------|
| US Letter | 12,240 | 15,840 | 9,360 |
| A4（默认） | 11,906 | 16,838 | 9,026 |

**横向布局：** docx-js内部会交换宽高值，因此传入纵向尺寸由其处理交换：
```javascript
size: {
  width: 12240,   // 将短边设为width
  height: 15840,  // 将长边设为height
  orientation: PageOrientation.LANDSCAPE  // docx-js在XML中交换宽高
},
// 内容宽度 = 15840 - 左边距 - 右边距（使用长边计算）
```

### 样式（覆盖内置标题）

使用通用支持的Arial字体。标题保持黑色确保可读性：
```javascript
const doc = new Document({
  styles: {
    default: { document: { run: { font: "Arial", size: 24 } } }, // 默认12pt
    paragraphStyles: [
      // 重要：使用精确ID覆盖内置样式
      { id: "Heading1", name: "标题1", basedOn: "正文", next: "正文", quickFormat: true,
        run: { size: 32, bold: true, font: "Arial" },
        paragraph: { spacing: { before: 240, after: 240 }, outlineLevel: 0 } }, // outlineLevel为目录必需
      { id: "Heading2", name: "标题2", basedOn: "正文", next: "正文", quickFormat: true,
        run: { size: 28, bold: true, font: "Arial" },
        paragraph: { spacing: { before: 180, after: 180 }, outlineLevel: 1 } },
    ]
  },
  sections: [{
    children: [
      new Paragraph({ heading: HeadingLevel.HEADING_1, children: [new TextRun("标题")] }),
    ]
  }]
});
```

### 列表（禁止使用Unicode符号）
```javascript
// ❌ 错误 - 禁止手动插入符号
new Paragraph({ children: [new TextRun("• 项目")] })  // 错误
new Paragraph({ children: [new TextRun("\u2022 项目")] })  // 错误

// ✅ 正确 - 使用LevelFormat.BULLET配置
const doc = new Document({
  numbering: {
    config: [
      { reference: "bullets",
        levels: [{ level: 0, format: LevelFormat.BULLET, text: "•", alignment: AlignmentType.LEFT,
          style: { paragraph: { indent: { left: 720, hanging: 360 } } } }] },
      { reference: "numbers",
        levels: [{ level: 0, format: LevelFormat.DECIMAL, text: "%1.", alignment: AlignmentType.LEFT,
          style: { paragraph: { indent: { left: 720, hanging: 360 } } } }] },
    ]
  },
  sections: [{
    children: [
      new Paragraph({ numbering: { reference: "bullets", level: 0 },
        children: [new TextRun("项目符号")] }),
      new Paragraph({ numbering: { reference: "numbers", level: 0 },
        children: [new TextRun("编号项目")] }),
    ]
  }]
});

// ⚠️ 相同reference=连续编号(1,2,3→4,5,6)
// 不同reference=重新编号(1,2,3→1,2,3)
```

### 表格

**关键：表格需双重宽度定义** - 表格设置`columnWidths`且每个单元格设置`width`。缺少任一设置会导致跨平台渲染异常。

```javascript
// 关键：始终设置表格宽度保证渲染一致性
// 关键：使用ShadingType.CLEAR避免黑色背景
const border = { style: BorderStyle.SINGLE, size: 1, color: "CCCCCC" };
const borders = { top: border, bottom: border, left: border, right: border };

new Table({
  width: { size: 9360, type: WidthType.DXA }, // 始终用DXA（百分比在Google Docs会失效）
  columnWidths: [4680, 4680], // 总和须等于表格宽度（DXA: 1440=1英寸）
  rows: [
    new TableRow({
      children: [
        new TableCell({
          borders,
          width: { size: 4680, type: WidthType.DXA }, // 每个单元格也需设置
          shading: { fill: "D5E8F0", type: ShadingType.CLEAR }, // 必须用CLEAR而非SOLID
          margins: { top: 80, bottom: 80, left: 120, right: 120 }, // 单元格内边距（不增加宽度）
          children: [new Paragraph({ children: [new TextRun("单元格")] })]
        })
      ]
    })
  ]
})
```

**表格宽度计算：**

始终使用`WidthType.DXA` — `WidthType.PERCENTAGE`在Google Docs中会失效。

```javascript
// 表格宽度 = columnWidths总和 = 内容宽度
// US Letter带1英寸页边距：12240 - 2880 = 9360 DXA
width: { size: 9360, type: WidthType.DXA },
columnWidths: [7000, 2360]  // 总和须等于表格宽度
```

**宽度规则：**
- **始终使用`WidthType.DXA`** — 禁用`WidthType.PERCENTAGE`（与Google Docs不兼容）
- 表格宽度必须等于`columnWidths`总和
- 单元格`width`必须匹配对应`columnWidth`
- 单元格`margins`是内边距 — 减少内容区域，不增加单元格宽度
- 全宽表格：使用内容宽度（页面宽度减去左右页边距）

### 图片
```javascript
// 关键：type参数必需
new Paragraph({
  children: [new ImageRun({
    type: "png", // 必需值：png/jpg/jpeg/gif/bmp/svg
    data: fs.readFileSync("image.png"),
    transformation: { width: 200, height: 150 },
    altText: { title: "标题", description: "描述", name: "名称" } // 三项均必需
  })]
})
```

### 分页符
```javascript
// 关键：PageBreak必须包含在Paragraph内
new Paragraph({ children: [new PageBreak()] })

// 或使用pageBreakBefore属性
new Paragraph({ pageBreakBefore: true, children: [new TextRun("新页面")] })
```

### 超链接
```javascript
// 外部链接
new Paragraph({
  children: [new ExternalHyperlink({
    children: [new TextRun({ text: "点击此处", style: "超链接" })],
    link: "https://example.com",
  })]
})

// 内部链接（书签+引用）
// 1. 在目标位置创建书签
new Paragraph({ heading: HeadingLevel.HEADING_1, children: [
  new Bookmark({ id: "chapter1", children: [new TextRun("第一章")] }),
]})
// 2. 链接至该书签
new Paragraph({ children: [new InternalHyperlink({
  children: [new TextRun({ text: "参见第一章", style: "超链接" })],
  anchor: "chapter1",
})]})
```

### 脚注
```javascript
const doc = new Document({
  footnotes: {
    1: { children: [new Paragraph("来源：2024年度报告")] },
    2: { children: [new Paragraph("方法详见附录")] },
  },
  sections: [{
    children: [new Paragraph({
      children: [
        new TextRun("收入增长15%"),
        new FootnoteReferenceRun(1),
        new TextRun("（基于调整后指标）"),
        new FootnoteReferenceRun(2),
      ],
    })]
  }]
});
```

### 制表位
```javascript
// 同段落右对齐（如标题与日期并列）
new Paragraph({
  children: [
    new TextRun("公司名称"),
    new TextRun("\t2025年1月"),
  ],
  tabStops: [{ type: TabStopType.RIGHT, position: TabStopPosition.MAX }],
})

// 点引导线（如目录样式）
new Paragraph({
  children: [
    new TextRun("引言"),
    new TextRun({ children: [
      new PositionalTab({
        alignment: PositionalTabAlignment.RIGHT,
        relativeTo: PositionalTabRelativeTo.MARGIN,
        leader: PositionalTabLeader.DOT,
      }),
      "3",
    ]}),
  ],
})
```

### 多栏布局
```javascript
// 等宽分栏
sections: [{
  properties: {
    column: {
      count: 2,          // 栏数
      space: 720,        // 栏间距（720=0.5英寸）
      equalWidth: true,
      separate: true,    // 显示垂直分隔线
    },
  },
  children: [/* 内容自动跨栏流动 */]
}]

// 自定义栏宽（需关闭equalWidth）
sections: [{
  properties: {
    column: {
      equalWidth: false,
      children: [
        new Column({ width: 5400, space: 720 }),
        new Column({ width: 3240 }),
      ],
    },
  },
  children: [/* 内容 */]
}]
```

使用`type: SectionType.NEXT_COLUMN`创建新节强制分栏。

### 目录
```javascript
// 关键：标题必须使用HeadingLevel - 禁用自定义样式
new TableOfContents("目录", { hyperlink: true, headingStyleRange: "1-3" })
```

### 页眉/页脚
```javascript
sections: [{
  properties: {
    page: { margin: { top: 1440, right: 1440, bottom: 1440, left: 1440 } } // 1440=1英寸
  },
  headers: {
    default: new Header({ children: [new Paragraph({ children: [new TextRun("页眉")] })] })
  },
  footers: {
    default: new Footer({ children: [new Paragraph({
      children: [new TextRun("第"), new TextRun({ children: [PageNumber.CURRENT] }), new TextRun("页")]
    })] })
  },
  children: [/* 正文内容 */]
}]
```

### docx-js关键规则

- **显式设置页面尺寸** - 默认A4；美国文档使用US Letter（12240×15840 DXA）
- **横向布局传入纵向尺寸** - docx-js内部交换宽高；短边设为`width`，长边设为`height`，并设置`orientation: PageOrientation.LANDSCAPE`
- **禁用`\n`换行** - 使用独立Paragraph元素
- **禁用Unicode项目符号** - 使用`LevelFormat.BULLET`配置
- **PageBreak必须包含在Paragraph内** - 独立存在会产生无效XML
- **ImageRun必需`type`参数** - 必须指定png/jpg等类型
- **表格宽度始终使用DXA** - 禁用`WidthType.PERCENTAGE`（Google Docs不兼容）
- **表格需双重宽度定义** - `columnWidths`数组与单元格`width`必须匹配
- **表格宽度 = columnWidths总和** - DXA单位需精确累加
- **始终添加单元格内边距** - 使用`margins: { top:80, bottom:80, left:120, right:120 }`保证可读性
- **使用`ShadingType.CLEAR`** - 禁用SOLID填充表格
- **禁用表格作为分隔线** - 单元格有最小高度会显示为空白框（含页眉/页脚）；改用Paragraph的边框属性：`border: { bottom: { style: BorderStyle.SINGLE, size:6, color:"2E75B6", space:1 } }`。双栏页脚请使用制表位（见"制表位"章节），禁用表格
- **目录仅支持HeadingLevel** - 标题段落禁用自定义样式
- **覆盖内置样式** - 使用精确ID："Heading1"、"Heading2"等
- **必须包含`outlineLevel`** - 目录必需（H1=0, H2=1, 以此类推）

---

## 编辑现有文档

**严格按顺序执行三步操作。**

### 步骤1：解包
```bash
python scripts/office/unpack.py document.docx unpacked/
```
解压XML、格式化输出、合并相邻文本流，并将智能引号转为XML实体（`&#x201C;`等）确保编辑后保留。使用`--merge-runs false`跳过文本流合并。

### 步骤2：编辑XML

在`unpacked/word/`目录编辑文件。XML模式参考下文。

**将“Claude”作为作者**用于修订和批注，除非用户明确要求使用其他名称。

**直接使用编辑工具进行字符串替换，不要编写Python脚本。** 脚本会引入不必要的复杂性。编辑工具能清晰展示被替换的内容。

**关键：新内容使用智能引号。** 添加含撇号或引号的文本时，使用XML实体生成智能引号：
```xml
<!-- 专业排版使用以下实体 -->
<w:t>Here&#x2019;s a quote: &#x201C;Hello&#x201D;</w:t>
```
| 实体 | 字符 |
|--------|-----------|
| `&#x2018;` | ‘ (左单引号) |
| `&#x2019;` | ’ (右单引号/撇号) |
| `&#x201C;` | “ (左双引号) |
| `&#x201D;` | ” (右双引号) |

**添加批注：** 使用 `comment.py` 处理多个XML文件的模板内容（文本必须预先转义为XML）：
```bash
python scripts/comment.py unpacked/ 0 "含 &amp; 和 &#x2019; 的批注文本"
python scripts/comment.py unpacked/ 1 "回复文本" --parent 0  # 回复批注0
python scripts/comment.py unpacked/ 0 "文本" --author "自定义作者"  # 自定义作者名
```
然后在document.xml中添加标记（参见XML参考中的批注部分）。

### 步骤3：打包
```bash
python scripts/office/pack.py unpacked/ output.docx --original document.docx
```
自动修复并验证，压缩XML后生成DOCX。使用 `--validate false` 跳过验证。

**自动修复将处理：**
- `durableId` ≥ 0x7FFFFFFF（重新生成有效ID）
- 含空白字符的`<w:t>`缺失 `xml:space="preserve"`

**自动修复不处理：**
- 格式错误的XML、无效元素嵌套、缺失关联关系、模式违规

### 常见陷阱

- **替换整个`<w:r>`元素**：添加修订时，将整个`<w:r>...</w:r>`块替换为并列的`<w:del>...<w:ins>...`。切勿在运行块内注入修订标签。
- **保留`<w:rPr>`格式**：将原始运行块的`<w:rPr>`复制到修订运行块中，以保持粗体、字号等格式。

---

## XML参考

### 模式合规性

- **`<w:pPr>`中的元素顺序**：`<w:pStyle>`、`<w:numPr>`、`<w:spacing>`、`<w:ind>`、`<w:jc>`，最后是`<w:rPr>`
- **空白字符**：含首尾空格的`<w:t>`需添加 `xml:space="preserve"`
- **RSID**：必须为8位十六进制（如 `00AB1234`）

### 修订跟踪

**插入：**
```xml
<w:ins w:id="1" w:author="Claude" w:date="2025-01-01T00:00:00Z">
  <w:r><w:t>插入文本</w:t></w:r>
</w:ins>
```

**删除：**
```xml
<w:del w:id="2" w:author="Claude" w:date="2025-01-01T00:00:00Z">
  <w:r><w:delText>删除文本</w:delText></w:r>
</w:del>
```

**在`<w:del>`内**：用`<w:delText>`替代`<w:t>`，用`<w:delInstrText>`替代`<w:instrText>`。

**最小化编辑** - 仅标记变更部分：
```xml
<!-- 将"30天"改为"60天" -->
<w:r><w:t>期限为</w:t></w:r>
<w:del w:id="1" w:author="Claude" w:date="...">
  <w:r><w:delText>30</w:delText></w:r>
</w:del>
<w:ins w:id="2" w:author="Claude" w:date="...">
  <w:r><w:t>60</w:t></w:r>
</w:ins>
<w:r><w:t>天</w:t></w:r>
```

**删除整个段落/列表项** - 清空段落内容时，需同时标记段落符号为删除状态以合并后续段落。在`<w:pPr><w:rPr>`内添加`<w:del/>`：
```xml
<w:p>
  <w:pPr>
    <w:numPr>...</w:numPr>  <!-- 列表编号（如存在） -->
    <w:rPr>
      <w:del w:id="1" w:author="Claude" w:date="2025-01-01T00:00:00Z"/>
    </w:rPr>
  </w:pPr>
  <w:del w:id="2" w:author="Claude" w:date="2025-01-01T00:00:00Z">
    <w:r><w:delText>被删除的完整段落内容...</w:delText></w:r>
  </w:del>
</w:p>
```
若`<w:pPr><w:rPr>`中缺少`<w:del/>`，接受修订后将残留空段落/列表项。

**拒绝其他作者的插入** - 在其插入内容内嵌套删除标记：
```xml
<w:ins w:author="Jane" w:id="5">
  <w:del w:author="Claude" w:id="10">
    <w:r><w:delText>其插入文本</w:delText></w:r>
  </w:del>
</w:ins>
```

**恢复其他作者的删除** - 在其删除内容后添加插入（不修改原删除标记）：
```xml
<w:del w:author="Jane" w:id="5">
  <w:r><w:delText>被删文本</w:delText></w:r>
</w:del>
<w:ins w:author="Claude" w:id="10">
  <w:r><w:t>被删文本</w:t></w:r>
</w:ins>
```

### 批注

运行`comment.py`后（见步骤2），在document.xml中添加标记。回复时使用`--parent`参数并将标记嵌套在父批注内。

**关键：`<w:commentRangeStart>`和`<w:commentRangeEnd>`必须是`<w:r>`的同级元素，绝不能在`<w:r>`内部。**

```xml
<!-- 批注标记必须是w:p的直接子元素，绝不能在w:r内部 -->
<w:commentRangeStart w:id="0"/>
<w:del w:id="1" w:author="Claude" w:date="2025-01-01T00:00:00Z">
  <w:r><w:delText>已删除</w:delText></w:r>
</w:del>
<w:r><w:t>更多文本</w:t></w:r>
<w:commentRangeEnd w:id="0"/>
<w:r><w:rPr><w:rStyle w:val="CommentReference"/></w:rPr><w:commentReference w:id="0"/></w:r>

<!-- 批注0内嵌套回复1 -->
<w:commentRangeStart w:id="0"/>
  <w:commentRangeStart w:id="1"/>
  <w:r><w:t>文本</w:t></w:r>
  <w:commentRangeEnd w:id="1"/>
<w:commentRangeEnd w:id="0"/>
<w:r><w:rPr><w:rStyle w:val="CommentReference"/></w:rPr><w:commentReference w:id="0"/></w:r>
<w:r><w:rPr><w:rStyle w:val="CommentReference"/></w:rPr><w:commentReference w:id="1"/></w:r>
```

### 图片

1. 添加图片文件至 `word/media/`
2. 在 `word/_rels/document.xml.rels` 中添加关联：
```xml
<Relationship Id="rId5" Type=".../image" Target="media/image1.png"/>
```
3. 在 `[Content_Types].xml` 中添加内容类型：
```xml
<Default Extension="png" ContentType="image/png"/>
```
4. 在document.xml中引用：
```xml
<w:drawing>
  <wp:inline>
    <wp:extent cx="914400" cy="914400"/>  <!-- EMU单位：914400 = 1英寸 -->
    <a:graphic>
      <a:graphicData uri=".../picture">
        <pic:pic>
          <pic:blipFill><a:blip r:embed="rId5"/></pic:blipFill>
        </pic:pic>
      </a:graphicData>
    </a:graphic>
  </wp:inline>
</w:drawing>
```

---

## 依赖项

- **pandoc**：文本提取
- **docx**：`npm install -g docx`（新建文档）
- **LibreOffice**：PDF转换（通过`scripts/office/soffice.py`在沙盒环境中自动配置）
- **Poppler**：`pdftoppm`（用于图片处理）
