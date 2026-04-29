---
name: pyzotero
description: 使用 pyzotero Python 客户端与 Zotero 文献管理库交互。通过 Zotero Web API v3 检索、创建、更新和删除条目、集合、标签及附件。当需要编程操作 Zotero 库、管理参考文献、导出引文、搜索库内容、上传 PDF 附件或构建与 Zotero 集成的研究自动化工作流时使用此技能。
allowed-tools: Read Write Edit Bash
license: MIT License
metadata:
    skill-author: K-Dense Inc.
---

# Pyzotero

Pyzotero 是 [Zotero API v3](https://www.zotero.org/support/dev/web_api/v3/start) 的 Python 封装库。用于编程管理 Zotero 库：读取条目和集合、创建更新参考文献、上传附件、管理标签及导出引文。

## 认证设置

**必需凭证** — 从 https://www.zotero.org/settings/keys 获取：
- **用户 ID**：显示为 "Your userID for use in API calls"
- **API 密钥**：在 https://www.zotero.org/settings/keys/new 创建
- **库 ID**：对于群组库，即群组 URL 中 `/groups/` 后的整数值

将凭证存储在环境变量或 `.env` 文件中：
```
ZOTERO_LIBRARY_ID=your_user_id
ZOTERO_API_KEY=your_api_key
ZOTERO_LIBRARY_TYPE=user  # 或 "group"
```

完整设置详见 [references/authentication.md](references/authentication.md)。

## 安装

```bash
uv add pyzotero
# 或安装 CLI 支持：
uv add "pyzotero[cli]"
```

## 快速入门

```python
from pyzotero import Zotero

zot = Zotero(library_id='123456', library_type='user', api_key='ABC1234XYZ')

# 检索顶层条目（默认返回100条）
items = zot.top(limit=10)
for item in items:
    print(item['data']['title'], item['data']['itemType'])

# 关键词搜索
results = zot.items(q='machine learning', limit=20)

# 检索所有条目（使用 everything() 获取完整结果）
all_items = zot.everything(zot.items())
```

## 核心概念

- `Zotero` 实例绑定到单个库（用户或群组），所有方法均操作该库
- 条目数据位于 `item['data']` 中，访问字段如 `item['data']['title']`、`item['data']['creators']`
- Pyzotero 默认返回100条（API 默认25条），使用 `zot.everything(zot.items())` 获取全部条目
- 写入方法成功时返回 `True`，失败则抛出 `ZoteroError`

## 参考文档

| 文件 | 内容 |
|------|----------|
| [references/authentication.md](references/authentication.md) | 凭证、库类型、本地模式 |
| [references/read-api.md](references/read-api.md) | 检索条目、集合、标签、群组 |
| [references/search-params.md](references/search-params.md) | 过滤、排序、搜索参数 |
| [references/write-api.md](references/write-api.md) | 创建、更新、删除条目 |
| [references/collections.md](references/collections.md) | 集合的增删改查操作 |
| [references/tags.md](references/tags.md) | 标签检索与管理 |
| [references/files-attachments.md](references/files-attachments.md) | 文件检索与附件上传 |
| [references/exports.md](references/exports.md) | BibTeX、CSL-JSON、参考文献导出 |
| [references/pagination.md](references/pagination.md) | follow()、everything()、生成器 |
| [references/full-text.md](references/full-text.md) | 全文内容索引与检索 |
| [references/saved-searches.md](references/saved-searches.md) | 保存的搜索管理 |
| [references/cli.md](references/cli.md) | 命令行界面用法 |
| [references/error-handling.md](references/error-handling.md) | 错误与异常处理 |

## 常用模式

### 获取并修改条目
```python
item = zot.item('ITEMKEY')
item['data']['title'] = '新标题'
zot.update_item(item)
```

### 通过模板创建条目
```python
template = zot.item_template('journalArticle')
template['title'] = '我的论文'
template['creators'][0] = {'creatorType': 'author', 'firstName': 'Jane', 'lastName': 'Doe'}
zot.create_items([template])
```

### 导出为 BibTeX
```python
zot.add_parameters(format='bibtex')
bibtex = zot.top(limit=50)
# bibtex 是 bibtexparser 的 BibDatabase 对象
print(bibtex.entries)
```

### 本地模式（只读，无需 API 密钥）
```python
zot = Zotero(library_id='123456', library_type='user', local=True)
items = zot.items()
```
