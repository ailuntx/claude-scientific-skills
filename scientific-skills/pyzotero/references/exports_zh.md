# 导出格式

## BibTeX

```python
zot.add_parameters(format='bibtex')
bibtex_db = zot.top(limit=50)
# 返回一个bibtexparser的BibDatabase对象

# 以字典列表形式访问条目
entries = bibtex_db.entries
for entry in entries:
    print(entry.get('title'), entry.get('author'))

# 写入.bib文件
import bibtexparser
with open('library.bib', 'w') as f:
    bibtexparser.dump(bibtex_db, f)
```

## CSL-JSON

```python
zot.add_parameters(content='csljson', limit=50)
csl_items = zot.items()
# 返回CSL-JSON格式的字典列表
```

## 参考文献HTML（格式化引文）

```python
# APA格式参考文献
zot.add_parameters(content='bib', style='apa')
bib_entries = zot.items(limit=50)
# 返回HTML <div>字符串列表

for entry in bib_entries:
    print(entry)  # 例如 '<div>Smith, J. (2024). Title. <i>Journal</i>...</div>'
```

**注意**：`format='bib'`会忽略`limit`参数。API强制限制最多150条条目。

### 可用引文格式

传递[Zotero样式库](https://www.zotero.org/styles)中任何有效的CSL样式名称：
- `'apa'`
- `'chicago-author-date'`
- `'chicago-note-bibliography'`
- `'mla'`
- `'vancouver'`
- `'ieee'`
- `'harvard-cite-them-right'`
- `'nature'`

## 文内引注

```python
zot.add_parameters(content='citation', style='apa')
citations = zot.items(limit=50)
# 返回HTML <span>元素列表：['<span>(Smith, 2024)</span>', ...]
```

## 其他格式

将`content`设置为任意Zotero导出格式：

| 格式 | `content`值 | 返回类型 |
|------|-------------|----------|
| BibTeX | `'bibtex'` | 通过`format='bibtex'`获取 |
| CSL-JSON | `'csljson'` | 字典列表 |
| RIS | `'ris'` | Unicode字符串列表 |
| RDF (Dublin Core) | `'rdf_dc'` | Unicode字符串列表 |
| Zotero RDF | `'rdf_zotero'` | Unicode字符串列表 |
| BibLaTeX | `'biblatex'` | Unicode字符串列表 |
| 维基百科引文模板 | `'wikipedia'` | Unicode字符串列表 |

**注意**：使用`content`导出格式时必须提供`limit`参数。不支持同时检索多种格式。

```python
# RIS格式导出
zot.add_parameters(content='ris', limit=50)
ris_data = zot.items()
with open('library.ris', 'w', encoding='utf-8') as f:
    f.write('\n'.join(ris_data))
```

## 仅键值

```python
# 获取换行符分隔的条目键值字符串
zot.add_parameters(format='keys')
keys_str = zot.items()
keys = keys_str.strip().split('\n')
```

## 版本信息（用于同步）

```python
# 所有条目的{键:版本}字典
zot.add_parameters(format='versions')
versions = zot.items()
```
