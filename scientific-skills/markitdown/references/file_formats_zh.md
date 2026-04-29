# 文件格式支持

本文档详细介绍了 MarkItDown 支持的各种文件格式。

## 文档格式

### PDF (.pdf)

**功能**：
- 文本提取
- 表格检测
- 元数据提取
- 扫描文档OCR（需依赖项）

**依赖项**：
```bash
pip install 'markitdown[pdf]'
```

**最佳适用场景**：
- 科研论文
- 报告
- 书籍
- 表单

**限制**：
- 复杂布局可能无法完美保留格式
- 扫描PDF需要OCR设置
- 部分PDF功能（注释/表单）可能无法转换

**示例**：
```python
from markitdown import MarkItDown

md = MarkItDown()
result = md.convert("research_paper.pdf")
print(result.text_content)
```

**Azure文档智能增强**：
```python
md = MarkItDown(docintel_endpoint="https://YOUR-ENDPOINT.cognitiveservices.azure.com/")
result = md.convert("complex_layout.pdf")
```

---

### Microsoft Word (.docx)

**功能**：
- 文本提取
- 表格转换
- 标题层级
- 列表格式化
- 基础文本格式（粗体/斜体）

**依赖项**：
```bash
pip install 'markitdown[docx]'
```

**最佳适用场景**：
- 科研论文
- 报告
- 文档
- 手稿

**保留元素**：
- 标题（转为Markdown标题）
- 表格（转为Markdown表格）
- 列表（项目符号和编号）
- 基础格式（粗体/斜体）
- 段落

**示例**：
```python
result = md.convert("manuscript.docx")
```

---

### PowerPoint (.pptx)

**功能**：
- 幻灯片内容提取
- 演讲者备注
- 表格提取
- 图像描述（AI支持）

**依赖项**：
```bash
pip install 'markitdown[pptx]'
```

**最佳适用场景**：
- 演示文稿
- 教学幻灯片
- 会议演讲

**输出格式**：
```markdown
# 幻灯片1：标题

幻灯片1内容...

**备注**：演讲者备注显示在此处

---

# 幻灯片2：下一主题

...
```

**AI图像描述功能**：
```python
from openai import OpenAI

client = OpenAI()
md = MarkItDown(llm_client=client, llm_model="gpt-4o")
result = md.convert("presentation.pptx")
```

---

### Excel (.xlsx, .xls)

**功能**：
- 工作表提取
- 表格格式化
- 数据保留
- 公式值（计算后）

**依赖项**：
```bash
pip install 'markitdown[xlsx]'  # 新版Excel
pip install 'markitdown[xls]'   # 旧版Excel
```

**最佳适用场景**：
- 数据表格
- 研究数据
- 统计结果
- 实验数据

**输出格式**：
```markdown
# 工作表：结果

| 样本 | 对照组 | 处理组 | P值 |
|------|--------|--------|-----|
| 1    | 10.2   | 12.5   | 0.023 |
| 2    | 9.8    | 11.9   | 0.031 |
```

**示例**：
```python
result = md.convert("experimental_data.xlsx")
```

---

## 图像格式

### 图像 (.jpg, .jpeg, .png, .gif, .webp)

**功能**：
- EXIF元数据提取
- OCR文本提取
- AI图像描述

**依赖项**：
```bash
pip install 'markitdown[all]'  # 包含图像支持
```

**最佳适用场景**：
- 扫描文档
- 图表
- 科学示意图
- 含文本的照片

**无AI输出**：
```markdown
![图像](image.jpg)

**EXIF数据**：
- 相机：Canon EOS 5D
- 日期：2024-01-15
- 分辨率：4000x3000
```

**AI增强输出**：
```python
from openai import OpenAI

client = OpenAI()
md = MarkItDown(
    llm_client=client,
    llm_model="gpt-4o",
    llm_prompt="详细描述此科学示意图"
)
result = md.convert("graph.png")
```

**OCR文本提取**：
需安装Tesseract OCR：
```bash
# macOS
brew install tesseract

# Ubuntu
sudo apt-get install tesseract-ocr
```

---

## 音频格式

### 音频 (.wav, .mp3)

**功能**：
- 元数据提取
- 语音转文字
- 时长和技术信息

**依赖项**：
```bash
pip install 'markitdown[audio-transcription]'
```

**最佳适用场景**：
- 讲座录音
- 访谈
- 播客
- 会议录音

**输出格式**：
```markdown
# 音频：interview.mp3

**元数据**：
- 时长：45:32
- 比特率：320kbps
- 采样率：44100Hz

**文字转录**：
[转录文本显示在此处...]
```

**示例**：
```python
result = md.convert("lecture.mp3")
```

---

## 网页格式

### HTML (.html, .htm)

**功能**：
- 纯净HTML转Markdown
- 链接保留
- 表格转换
- 列表格式化

**最佳适用场景**：
- 网页
- 文档
- 博客文章
- 在线文章

**输出格式**：保留链接和结构的纯净Markdown

**示例**：
```python
result = md.convert("webpage.html")
```

---

### YouTube网址

**功能**：
- 获取视频字幕
- 提取视频元数据
- 下载字幕

**依赖项**：
```bash
pip install 'markitdown[youtube-transcription]'
```

**最佳适用场景**：
- 教学视频
- 讲座
- 演讲
- 教程

**示例**：
```python
result = md.convert("https://www.youtube.com/watch?v=VIDEO_ID")
```

---

## 数据格式

### CSV (.csv)

**功能**：
- 自动表格转换
- 分隔符检测
- 表头保留

**输出格式**：Markdown表格

**示例**：
```python
result = md.convert("data.csv")
```

**输出**：
```markdown
| 列1 | 列2 | 列3 |
|-----|-----|-----|
| 值1 | 值2 | 值3 |
```

---

### JSON (.json)

**功能**：
- 结构化呈现
- 美观格式化
- 嵌套数据可视化

**最佳适用场景**：
- API响应
- 配置文件
- 数据导出

**示例**：
```python
result = md.convert("data.json")
```

---

### XML (.xml)

**功能**：
- 结构保留
- 属性提取
- 格式化输出

**最佳适用场景**：
- 配置文件
- 数据交换
- 结构化文档

**示例**：
```python
result = md.convert("config.xml")
```

---

## 归档格式

### ZIP (.zip)

**功能**：
- 遍历归档内容
- 单独转换每个文件
- 输出保留目录结构

**最佳适用场景**：
- 文档集合
- 项目归档
- 批量转换

**输出格式**：
```markdown
# 归档：documents.zip

## 文件：document1.pdf
[document1.pdf内容...]

---

## 文件：document2.docx
[document2.docx内容...]
```

**示例**：
```python
result = md.convert("archive.zip")
```

---

## 电子书格式

### EPUB (.epub)

**功能**：
- 全文提取
- 章节结构
- 元数据提取

**最佳适用场景**：
- 电子书
- 数字出版物
- 长文本内容

**输出格式**：保留章节结构的Markdown

**示例**：
```python
result = md.convert("book.epub")
```

---

## 其他格式

### Outlook邮件 (.msg)

**功能**：
- 邮件内容提取
- 附件列表
- 元数据（发件人/收件人/主题/日期）

**依赖项**：
```bash
pip install 'markitdown[outlook]'
```

**最佳适用场景**：
- 邮件归档
- 通信记录

**示例**：
```python
result = md.convert("message.msg")
```

---

## 格式优化建议

### PDF最佳实践

1. **复杂布局使用Azure文档智能**：
   ```python
   md = MarkItDown(docintel_endpoint="endpoint_url")
   ```

2. **扫描PDF需配置OCR**：
   ```bash
   brew install tesseract  # macOS
   ```

3. **超大PDF分割后转换**以提高性能

### PowerPoint最佳实践

1. **视觉内容使用AI**：
   ```python
   md = MarkItDown(llm_client=client, llm_model="gpt-4o")
   ```

2. **检查演讲者备注** - 会包含在输出中

3. **复杂动画无法捕获** - 仅静态内容

### Excel最佳实践

1. **大型表格**转换可能耗时

2. **公式转为计算值**

3. **多工作表**全部包含在输出中

4. **图表转为文字描述**（使用AI优化描述）

### 图像最佳实践

1. **使用AI生成描述**：
   ```python
   md = MarkItDown(
       llm_client=client,
       llm_model="gpt-4o",
       llm_prompt="详细描述此科学图表"
   )
   ```

2. **文字密集图像需安装OCR依赖项**

3. **高分辨率图像**处理时间较长

### 音频最佳实践

1. **清晰音频**转录效果更佳

2. **长录音**可能耗时显著

3. **考虑分割长音频**加速处理

---

## 不支持格式处理

如需转换不支持格式：

1. **创建自定义转换器**（参考`api_reference.md`）
2. **在GitHub搜索插件**（#markitdown-plugin）
3. **预转为支持格式**（如.rtf转.docx）

---

## 格式检测机制

MarkItDown自动检测依据：

1. **文件扩展名**（主要方式）
2. **MIME类型**（备用）
3. **文件签名**（魔数字节，备用）

**手动指定格式**：
```python
# 强制指定格式
result = md.convert("file_without_extension", file_extension=".pdf")

# 流处理模式
with open("file", "rb") as f:
    result = md.convert_stream(f, file_extension=".pdf")
```
