# URL 提取

从以下来源提取内容：$ARGUMENTS

## 命令

根据 URL 或内容选择一个简短、描述性的文件名（例如 `vespa-docs`、`react-hooks-api`）。使用小写字母和连字符，不要空格。

```bash
parallel-cli extract "$ARGUMENTS" --json -o "$FILENAME.json"
```

可选选项：
- `--objective "关注领域"` 以聚焦特定内容

## 学术内容处理

从学术来源（arXiv、PubMed、期刊网站、会议论文集）提取时，使用 `--objective` 聚焦最有价值的部分：

```bash
parallel-cli extract "$URL" --json --objective "extract abstract, methodology, key findings, and conclusions" -o "$FILENAME.json"
```

对于 arXiv 论文，优先使用 `/abs/` URL（包含结构化元数据），而不是原始 PDF URL（如果可用）。如果用户提供 PDF 链接，直接提取——parallel-cli 可以处理 PDF。

## 响应格式

以如下格式返回内容：

**[页面标题](URL)**

对于学术论文，如果可用，包括结构化元数据：
- **作者：** 作者列表
- **发表：** 日期和期刊/会议
- **DOI：** 如果可用
- **摘要：** 论文摘要

然后逐字提取内容，遵循以下规则：
- 保留内容原样——不要转述或总结
- 详尽解析列表——提取每一个编号/项目符号项
- 仅去除明显噪音：导航菜单、页脚、广告
- 保留所有事实、名称、数字、日期、引用
- 对于学术论文，保留图表标题和参考文献

在响应之后，提及输出文件路径（`$FILENAME.json`），以便用户知道它可用于后续问题。
