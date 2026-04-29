# 编写API方法

## 创建条目

创建条目前务必使用 `item_template()` 获取有效模板。

```python
# 获取特定条目类型的模板
template = zot.item_template('journalArticle')

# 填写字段
template['title'] = '基因组学中的深度学习'
template['date'] = '2024'
template['publicationTitle'] = 'Nature Methods'
template['volume'] = '21'
template['DOI'] = '10.1038/s41592-024-02233-6'
template['creators'] = [
    {'creatorType': 'author', 'firstName': 'Jane', 'lastName': 'Doe'},
    {'creatorType': 'author', 'firstName': 'John', 'lastName': 'Smith'},
]

# 创建前验证字段（无效时会引发InvalidItemFields异常）
zot.check_items([template])

# 创建条目
resp = zot.create_items([template])
# resp: {'success': {'0': 'NEWITEMKEY'}, 'failed': {}, 'unchanged': {}}
new_key = resp['success']['0']
```

### 批量创建条目

```python
templates = []
for data in paper_data_list:
    t = zot.item_template('journalArticle')
    t['title'] = data['title']
    t['DOI'] = data['doi']
    templates.append(t)

resp = zot.create_items(templates)
```

### 创建子条目

```python
# 为现有条目创建笔记子项
note_template = zot.item_template('note')
note_template['note'] = '<p>我的注释内容</p>'
zot.create_items([note_template], parentid='PARENTKEY')
```

## 更新条目

```python
# 检索、修改、更新
item = zot.item('ITEMKEY')
item['data']['title'] = '更新后的标题'
item['data']['abstractNote'] = '新的摘要文本'
success = zot.update_item(item)  # 返回True或引发错误

# 批量更新条目（自动分块处理，每块50条）
items = zot.items(limit=10)
for item in items:
    item['data']['extra'] += '\n已处理'
zot.update_items(items)
```

## 删除条目

```python
# 必须先检索条目（需要version字段）
item = zot.item('ITEMKEY')
zot.delete_item([item])

# 批量删除条目
items = zot.items(tag='待删除')
zot.delete_item(items)
```

## 条目类型与字段

```python
# 所有可用条目类型
item_types = zot.item_types()
# [{'itemType': 'artwork', 'localized': '艺术作品'}, ...]

# 所有可用字段
fields = zot.item_fields()

# 特定条目类型的有效字段
journal_fields = zot.item_type_fields('journalArticle')

# 条目类型的有效创建者类型
creator_types = zot.item_creator_types('journalArticle')
# [{'creatorType': 'author', 'localized': '作者'}, ...]

# 所有本地化的创建者字段名
creator_fields = zot.creator_fields()

# 附件链接模式（附件模板所需）
link_modes = zot.item_attachment_link_modes()

# 附件模板
attach_template = zot.item_template('attachment', linkmode='imported_file')
```

## 乐观锁机制

使用 `last_modified` 防止覆盖并发修改：

```python
# 仅当库版本匹配时更新
zot.update_item(item, last_modified=4025)
# 若服务器版本不同则引发错误
```

## 注意事项

- `create_items()` 每次调用最多接受50个条目，需分批处理
- `update_items()` 自动按50条分块处理
- 若传入 `create_items()` 的字典包含与现有条目匹配的 `key`，将执行更新而非创建
- 在 `create_items()` 前务必调用 `check_items()` 提前捕获字段错误
