**重要提示：必须按顺序完成以下步骤，切勿跳过步骤直接编写代码。**

如需填写PDF表单，请先检查PDF是否包含可填写表单域。从当前文件目录运行脚本：
`python scripts/check_fillable_fields <file.pdf>`，根据结果跳转至"可填写表单域"或"非可填写表单域"章节并遵循对应说明。

# 可填写表单域
若PDF包含可填写表单域：
- 从当前文件目录运行脚本：`python scripts/extract_form_field_info.py <input.pdf> <field_info.json>`。该脚本将生成包含字段列表的JSON文件，格式如下：
```
[
  {
    "field_id": (字段唯一ID),
    "page": (页码，从1开始),
    "rect": ([左, 下, 右, 上] 边界框，采用PDF坐标系，y=0表示页面底部),
    "type": ("text", "checkbox", "radio_group" 或 "choice"),
  },
  // 复选框包含"checked_value"和"unchecked_value"属性：
  {
    "field_id": (字段唯一ID),
    "page": (页码，从1开始),
    "type": "checkbox",
    "checked_value": (设置此值将勾选复选框),
    "unchecked_value": (设置此值将取消勾选),
  },
  // 单选按钮组包含"radio_options"列表：
  {
    "field_id": (字段唯一ID),
    "page": (页码，从1开始),
    "type": "radio_group",
    "radio_options": [
      {
        "value": (设置此值将选中该选项),
        "rect": (该单选按钮的边界框)
      },
      // 其他单选选项
    ]
  },
  // 多选字段包含"choice_options"列表：
  {
    "field_id": (字段唯一ID),
    "page": (页码，从1开始),
    "type": "choice",
    "choice_options": [
      {
        "value": (设置此值将选中该选项),
        "text": (选项显示文本)
      },
      // 其他选项
    ],
  }
]
```
- 通过脚本将PDF转换为PNG图像（每页一张图），从当前文件目录运行：
`python scripts/convert_pdf_to_images.py <file.pdf> <output_directory>`
随后分析图像以确定各表单域用途（需将边界框PDF坐标转换为图像坐标）。
- 创建`field_values.json`文件，按格式填入各字段值：
```
[
  {
    "field_id": "last_name", // 必须与extract_form_field_info.py中的field_id匹配
    "description": "用户姓氏",
    "page": 1, // 必须与field_info.json中的"page"值匹配
    "value": "Simpson"
  },
  {
    "field_id": "Checkbox12",
    "description": "用户年满18岁需勾选",
    "page": 1,
    "value": "/On" // 复选框使用"checked_value"值勾选；单选组使用"radio_options"中的"value"值
  },
  // 更多字段
]
```
- 从当前文件目录运行`fill_fillable_fields.py`脚本生成填充后的PDF：
`python scripts/fill_fillable_fields.py <input pdf> <field_values.json> <output pdf>`
该脚本将验证字段ID和值有效性；若报错请修正对应字段后重试。

# 非可填写表单域
若PDF无可填写表单域，需添加文本标注。优先尝试从PDF结构提取坐标（更精确），必要时采用视觉估算。

## 步骤1：优先尝试结构提取

运行脚本提取文本标签、线条和复选框的精确PDF坐标：
`python scripts/extract_form_structure.py <input.pdf> form_structure.json`

生成的JSON文件包含：
- **labels**：所有带精确坐标的文本元素（PDF坐标点单位：x0, top, x1, bottom）
- **lines**：定义行边界的水平线
- **checkboxes**：作为复选框的小型矩形（含中心坐标）
- **row_boundaries**：根据水平线计算的行边界位置

**结果检查**：若`form_structure.json`包含有效标签（与表单域对应的文本元素），采用**方法A：基于结构的坐标**；若PDF为扫描件/图像且标签稀少，采用**方法B：视觉估算**。

---

## 方法A：基于结构的坐标（首选）

当`extract_form_structure.py`检测到PDF文本标签时使用。

### A.1：结构分析

解析form_structure.json并识别：
1. **标签组**：构成单个标签的相邻文本（如"姓"+"名"）
2. **行结构**：top值相近的标签位于同行
3. **字段列**：输入区域起始于标签结束位置（x0 = label.x1 + 间隙）
4. **复选框**：直接使用结构中的复选框坐标

**坐标系**：PDF坐标系，y=0位于页面顶部，y值向下递增。

### A.2：检查缺失元素

结构提取可能遗漏部分表单元素，常见情况：
- **圆形复选框**：仅方形矩形被识别为复选框
- **复杂图形**：装饰元素或非标准控件
- **褪色/浅色元素**：可能未被提取

若PDF图像中存在未提取的表单域，需对特定字段采用**视觉分析**（见下方"混合方法"）。

### A.3：创建含PDF坐标的fields.json

根据提取结构计算各字段坐标：

**文本字段：**
- 输入区x0 = 标签x1 + 5（标签后小间隙）
- 输入区x1 = 下一标签x0 或 行边界
- 输入区top = 同标签top
- 输入区bottom = 下方行边界线 或 标签bottom + 行高

**复选框：**
- 直接使用form_structure.json中的复选框矩形坐标
- entry_bounding_box = [checkbox.x0, checkbox.top, checkbox.x1, checkbox.bottom]

使用`pdf_width`和`pdf_height`创建fields.json（标记PDF坐标系）：
```json
{
  "pages": [
    {"page_number": 1, "pdf_width": 612, "pdf_height": 792}
  ],
  "form_fields": [
    {
      "page_number": 1,
      "description": "姓氏输入区",
      "field_label": "姓氏",
      "label_bounding_box": [43, 63, 87, 73],
      "entry_bounding_box": [92, 63, 260, 79],
      "entry_text": {"text": "Smith", "font_size": 10}
    },
    {
      "page_number": 1,
      "description": "美国公民勾选框",
      "field_label": "是",
      "label_bounding_box": [260, 200, 280, 210],
      "entry_bounding_box": [285, 197, 292, 205],
      "entry_text": {"text": "X"}
    }
  ]
}
```

**关键**：直接使用form_structure.json中的`pdf_width`/`pdf_height`和坐标值。

### A.4：边界框验证

填充前检查边界框错误：
`python scripts/check_bounding_boxes.py fields.json`

该脚本检测边界框交叉及输入框过小问题，修复报错后再填充。

---

## 方法B：视觉估算（备选）

当PDF为扫描件且结构提取无有效标签时使用（如所有文本显示为"(cid:X)"模式）。

### B.1：PDF转图像

`python scripts/convert_pdf_to_images.py <input.pdf> <images_dir/>`

### B.2：初步字段识别

检查每页图像识别表单区域并获取字段**粗略位置**：
- 表单标签及其大致位置
- 输入区域（文本输入线/框/空白区）
- 复选框及其大致位置

为每个字段记录近似像素坐标（无需精确）。

### B.3：缩放精确定位（精度关键）

裁剪每个字段的预估区域进行坐标精修：

**使用ImageMagick创建缩放裁剪：**
```bash
magick <page_image> -crop <width>x<height>+<x>+<y> +repage <crop_output.png>
```

参数说明：
- `<x>, <y>` = 裁剪区域左上角（使用预估坐标减填充值）
- `<width>, <height>` = 裁剪尺寸（字段区域每边加约50px填充）

**示例**：精修"姓名"字段（预估位置100,150）：
```bash
magick images_dir/page_1.png -crop 300x80+50+120 +repage crops/name_field.png
```

（注：若`magick`命令不可用，可尝试`convert`命令）

**检查裁剪图像**确定精确坐标：
1. 定位输入区起始像素（标签后）
2. 定位输入区结束位置（下一字段或边缘前）
3. 确定输入线/框的上下边界

**将裁剪坐标转换回原图坐标：**
- 原图x = 裁剪x + 裁剪偏移x
- 原图y = 裁剪y + 裁剪偏移y

示例：若裁剪起始(50,120)，输入框在裁剪图中位于(52,18)：
- 输入区x0 = 52 + 50 = 102
- 输入区top = 18 + 120 = 138

**为每个字段重复此过程**，邻近字段尽量合并裁剪。

### B.4：创建含精确坐标的fields.json

使用`image_width`和`image_height`创建fields.json（标记图像坐标系）：
```json
{
  "pages": [
    {"page_number": 1, "image_width": 1700, "image_height": 2200}
  ],
  "form_fields": [
    {
      "page_number": 1,
      "description": "姓氏输入区",
      "field_label": "姓氏",
      "label_bounding_box": [120, 175, 242, 198],
      "entry_bounding_box": [255, 175, 720, 218],
      "entry_text": {"text": "Smith", "font_size": 10}
    }
  ]
}
```

**关键**：使用`image_width`/`image_height`及精修后的像素坐标。

### B.5：边界框验证

填充前检查边界框错误：
`python scripts/check_bounding_boxes.py fields.json`

该脚本检测边界框交叉及输入框过小问题，修复报错后再填充。

---

## 混合方法：结构+视觉

当结构提取覆盖多数字段但遗漏部分元素（如圆形复选框、特殊控件）时使用。

1. 对结构提取的字段采用**方法A**
2. **转换PDF为图像**分析缺失字段
3. 对缺失字段采用**方法B的缩放精修**
4. **坐标整合**：结构字段使用`pdf_width`/`pdf_height`；视觉字段需转换图像坐标为PDF坐标：
   - pdf_x = image_x × (pdf_width / image_width)
   - pdf_y = image_y × (pdf_height / image_height)
5. 在fields.json中**统一使用PDF坐标系**并标注`pdf_width`/`pdf_height`

---

## 步骤2：填充前验证

**填充前务必验证边界框：**
`python scripts/check_bounding_boxes.py fields.json`

该脚本检查：
- 边界框交叉（导致文字重叠）
- 输入框小于指定字号

修复报错后再继续。

## 步骤3：表单填充

填充脚本自动检测坐标系并处理转换：
`python scripts/fill_pdf_form_with_annotations.py <input.pdf> fields.json <output.pdf>`

## 步骤4：结果验证

将填充后的PDF转为图像检查文本位置：
`python scripts/convert_pdf_to_images.py <output.pdf> <verify_images/>`

若文本错位：
- **方法A**：检查是否使用form_structure.json的PDF坐标及`pdf_width`/`pdf_height`
- **方法B**：检查图像尺寸是否匹配且坐标精确
- **混合方法**：确保视觉字段的坐标转换正确
