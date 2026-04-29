# PptxGenJS 教程

## 设置与基本结构

```javascript
const pptxgen = require("pptxgenjs");

let pres = new pptxgen();
pres.layout = 'LAYOUT_16x9';  // 或 'LAYOUT_16x10'、'LAYOUT_4x3'、'LAYOUT_WIDE'
pres.author = '您的姓名';
pres.title = '演示文稿标题';

let slide = pres.addSlide();
slide.addText("Hello World!", { x: 0.5, y: 0.5, fontSize: 36, color: "363636" });

pres.writeFile({ fileName: "Presentation.pptx" });
```

## 布局尺寸

幻灯片尺寸（单位：英寸）：
- `LAYOUT_16x9`: 10" × 5.625"（默认）
- `LAYOUT_16x10`: 10" × 6.25"
- `LAYOUT_4x3`: 10" × 7.5"
- `LAYOUT_WIDE`: 13.3" × 7.5"

---

## 文本与格式

```javascript
// 基础文本
slide.addText("简单文本", {
  x: 1, y: 1, w: 8, h: 2, fontSize: 24, fontFace: "Arial",
  color: "363636", bold: true, align: "center", valign: "middle"
});

// 字符间距（使用 charSpacing，不要用 letterSpacing，它会被静默忽略）
slide.addText("带间距文本", { x: 1, y: 1, w: 8, h: 1, charSpacing: 6 });

// 富文本数组
slide.addText([
  { text: "粗体 ", options: { bold: true } },
  { text: "斜体 ", options: { italic: true } }
], { x: 1, y: 3, w: 8, h: 1 });

// 多行文本（需要 breakLine: true）
slide.addText([
  { text: "第一行", options: { breakLine: true } },
  { text: "第二行", options: { breakLine: true } },
  { text: "第三行" }  // 最后一项不需要 breakLine
], { x: 0.5, y: 0.5, w: 8, h: 2 });

// 文本框边距（内部填充）
slide.addText("标题", {
  x: 0.5, y: 0.3, w: 9, h: 0.6,
  margin: 0  // 当文本需要与形状/图标在相同x位置精确对齐时使用0
});
```

**提示：** 文本框默认有内部边距。当需要文本与同一x位置的形状、线条或图标精确对齐时，设置 `margin: 0`。

---

## 列表与项目符号

```javascript
// ✅ 正确：多个项目符号
slide.addText([
  { text: "第一项", options: { bullet: true, breakLine: true } },
  { text: "第二项", options: { bullet: true, breakLine: true } },
  { text: "第三项", options: { bullet: true } }
], { x: 0.5, y: 0.5, w: 8, h: 3 });

// ❌ 错误：切勿使用Unicode项目符号
slide.addText("• 第一项", { ... });  // 会产生双重项目符号

// 子项和编号列表
{ text: "子项", options: { bullet: true, indentLevel: 1 } }
{ text: "第一项", options: { bullet: { type: "number" }, breakLine: true } }
```

---

## 形状

```javascript
slide.addShape(pres.shapes.RECTANGLE, {
  x: 0.5, y: 0.8, w: 1.5, h: 3.0,
  fill: { color: "FF0000" }, line: { color: "000000", width: 2 }
});

slide.addShape(pres.shapes.OVAL, { x: 4, y: 1, w: 2, h: 2, fill: { color: "0000FF" } });

slide.addShape(pres.shapes.LINE, {
  x: 1, y: 3, w: 5, h: 0, line: { color: "FF0000", width: 3, dashType: "dash" }
});

// 带透明度
slide.addShape(pres.shapes.RECTANGLE, {
  x: 1, y: 1, w: 3, h: 2,
  fill: { color: "0088CC", transparency: 50 }
});

// 圆角矩形（rectRadius 仅适用于 ROUNDED_RECTANGLE，不适用于 RECTANGLE）
// ⚠️ 不要与矩形强调覆盖层配对使用——它们无法覆盖圆角。请改用 RECTANGLE。
slide.addShape(pres.shapes.ROUNDED_RECTANGLE, {
  x: 1, y: 1, w: 3, h: 2,
  fill: { color: "FFFFFF" }, rectRadius: 0.1
});

// 带阴影
slide.addShape(pres.shapes.RECTANGLE, {
  x: 1, y: 1, w: 3, h: 2,
  fill: { color: "FFFFFF" },
  shadow: { type: "outer", color: "000000", blur: 6, offset: 2, angle: 135, opacity: 0.15 }
});
```

阴影选项：

| 属性 | 类型 | 范围 | 说明 |
|----------|------|-------|-------|
| `type` | string | `"outer"`, `"inner"` | |
| `color` | string | 6位十六进制（如 `"000000"`) | 不要加 `#` 前缀，不要用8位十六进制——参见常见陷阱 |
| `blur` | number | 0-100 pt | |
| `offset` | number | 0-200 pt | **必须为非负数**——负值会导致文件损坏 |
| `angle` | number | 0-359 度 | 阴影方向（135=右下，270=向上） |
| `opacity` | number | 0.0-1.0 | 用此属性设置透明度，切勿在颜色字符串中编码

slide.addShape(pres.shapes.RECTANGLE, { shadow, ... });  // ❌ 第二次调用会获取已转换的值
   slide.addShape(pres.shapes.RECTANGLE, { shadow, ... });

   const makeShadow = () => ({ type: "outer", blur: 6, offset: 2, color: "000000", opacity: 0.15 });
   slide.addShape(pres.shapes.RECTANGLE, { shadow: makeShadow(), ... });  // ✅ 每次生成新对象
   slide.addShape(pres.shapes.RECTANGLE, { shadow: makeShadow(), ... });
   ```

8. **不要将 `ROUNDED_RECTANGLE` 与强调边框一起使用** - 矩形覆盖条无法覆盖圆角。请改用 `RECTANGLE`。
   ```javascript
   // ❌ 错误：强调条无法覆盖圆角
   slide.addShape(pres.shapes.ROUNDED_RECTANGLE, { x: 1, y: 1, w: 3, h: 1.5, fill: { color: "FFFFFF" } });
   slide.addShape(pres.shapes.RECTANGLE, { x: 1, y: 1, w: 0.08, h: 1.5, fill: { color: "0891B2" } });

   // ✅ 正确：使用 RECTANGLE 实现整洁对齐
   slide.addShape(pres.shapes.RECTANGLE, { x: 1, y: 1, w: 3, h: 1.5, fill: { color: "FFFFFF" } });
   slide.addShape(pres.shapes.RECTANGLE, { x: 1, y: 1, w: 0.08, h: 1.5, fill: { color: "0891B2" } });
   ```

---

## 快速参考

- **形状**: RECTANGLE, OVAL, LINE, ROUNDED_RECTANGLE
- **图表**: BAR, LINE, PIE, DOUGHNUT, SCATTER, BUBBLE, RADAR
- **布局**: LAYOUT_16x9 (10"×5.625"), LAYOUT_16x10, LAYOUT_4x3, LAYOUT_WIDE
- **对齐方式**: "left", "center", "right"
- **图表数据标签**: "outEnd", "inEnd", "center"
