# 命令行界面

pyzotero CLI 连接至您的**本地 Zotero 安装**（非远程 API）。它要求本地 Zotero 桌面应用处于运行状态。

## 安装

```bash
uv add "pyzotero[cli]"
# 或无需安装直接运行：
uvx --from "pyzotero[cli]" pyzotero search -q "your query"
```

## 搜索

```bash
# 搜索标题和元数据
pyzotero search -q "机器学习"

# 全文搜索（包含 PDF 内容）
pyzotero search -q "气候变化" --fulltext

# 按条目类型过滤
pyzotero search -q "方法论" --itemtype journalArticle --itemtype book

# 按标签过滤（AND 逻辑）
pyzotero search -q "进化论" --tag "已审核" --tag "高优先级"

# 在特定集合内搜索
pyzotero search --collection ABC123 -q "测试"

# 分页显示结果
pyzotero search -q "深度学习" --limit 20 --offset 40

# 输出 JSON 格式（供机器处理）
pyzotero search -q "蛋白质" --json
```

## 获取单个条目

```bash
# 通过键值获取单个条目
pyzotero item ABC123

# 获取 JSON 格式
pyzotero item ABC123 --json

# 获取子条目（附件、笔记）
pyzotero children ABC123 --json

# 批量获取多个条目（最多 50 个）
pyzotero subset ABC123 DEF456 GHI789 --json
```

## 集合与标签

```bash
# 列出所有集合
pyzotero listcollections

# 列出所有标签
pyzotero tags

# 特定集合中的标签
pyzotero tags --collection ABC123
```

## 全文内容

```bash
# 获取附件的全文内容
pyzotero fulltext ABC123
```

## 条目类型

```bash
# 列出所有可用条目类型
pyzotero itemtypes
```

## DOI 索引

```bash
# 获取完整的 DOI 到键值映射（适用于缓存）
pyzotero doiindex > doi_cache.json
# 返回 JSON 格式：{"10.1038/s41592-024-02233-6": {"key": "ABC123", "doi": "..."}}
```

## 输出格式

默认输出人类可读文本，包含标题、作者、日期、出版物、卷号、期号、DOI、URL 及 PDF 附件路径。

使用 `--json` 参数输出结构化 JSON 格式，便于通过管道传递给其他工具。

## 搜索行为说明

- 默认搜索仅覆盖顶层条目标题和元数据字段
- `--fulltext` 将搜索扩展至 PDF 内容；结果展示父级文献条目（非原始附件）
- 多个 `--tag` 标志使用 AND 逻辑
- 多个 `--itemtype` 标志使用 OR 逻辑
