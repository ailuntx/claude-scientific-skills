# 全文内容

Pyzotero 可检索和设置附件项目的全文索引内容。

## 检索全文内容

```python
# 获取特定附件项目的全文内容
data = zot.fulltext_item('ATTACHMENTKEY')
# 返回：
# {
#   "content": "文档全文...",
#   "indexedPages": 50,
#   "totalPages": 50
# }
# 文本文档使用 indexedChars/totalChars 替代 pages

text = data['content']
coverage = data['indexedPages'] / data['totalPages']
```

## 查找具有新全文内容的项目

```python
# 获取自指定库版本后更新全文内容的项目键
new_fulltext = zot.new_fulltext(since='1085')
# 返回字典：{'KEY1': 1090, 'KEY2': 1095, ...}
# 值为全文被索引时的库版本
```

## 设置全文内容

```python
# 为PDF附件设置全文
payload = {
    'content': '文档的完整文本内容。',
    'indexedPages': 50,
    'totalPages': 50
}
zot.set_fulltext('ATTACHMENTKEY', payload)

# 文本文档使用 indexedChars/totalChars
payload = {
    'content': '此处为全文内容。',
    'indexedChars': 15000,
    'totalChars': 15000
}
zot.set_fulltext('ATTACHMENTKEY', payload)
```

## 通过CLI进行全文搜索

CLI 提供对本地已索引PDF的全文搜索：

```bash
# 搜索全文内容
pyzotero search -q "CRISPR基因编辑" --fulltext

# 以JSON格式输出（检索附件的父书目项目）
pyzotero search -q "气候临界点" --fulltext --json
```

## 在API中搜索（qmode=everything）

```python
# 在标题/创建者 + 全文内容中搜索
results = zot.items(q='蛋白质折叠', qmode='everything', limit=20)
```
