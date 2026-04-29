# 标签管理

## 检索标签

```python
# 库中的所有标签
tags = zot.tags()
# 返回字符串列表：['climate change', 'machine learning', ...]

# 特定条目的标签
item_tags = zot.item_tags('ITEMKEY')

# 特定集合中的标签
col_tags = zot.collection_tags('COLKEY')

# 按前缀过滤标签（例如所有以'bio'开头的标签）
filtered = zot.tags(q='bio')
```

## 为条目添加标签

```python
# 向条目添加一个或多个标签（先获取条目）
item = zot.item('ITEMKEY')
updated = zot.add_tags(item, 'tag1', 'tag2', 'tag3')

# 添加标签列表
tag_list = ['reviewed', 'high-priority', '2024']
updated = zot.add_tags(item, *tag_list)
```

## 删除标签

```python
# 从库中删除特定标签
zot.delete_tags('old-tag', 'unused-tag')

# 删除标签列表
tags_to_remove = ['deprecated', 'temp']
zot.delete_tags(*tags_to_remove)
```

## 按标签搜索条目

```python
# 包含单个标签的条目
items = zot.items(tag='machine learning')

# 包含多个标签的条目（AND逻辑）
items = zot.items(tag=['climate', 'adaptation'])

# 包含任意指定标签的条目（OR逻辑）
items = zot.items(tag='climate OR sea level')

# 不包含特定标签的条目
items = zot.items(tag='-retracted')
```

## 批量标签操作

```python
# 为集合中所有条目添加标签
items = zot.everything(zot.collection_items('COLKEY'))
for item in items:
    zot.add_tags(item, 'collection-reviewed')

# 查找包含特定标签的所有条目并重命名标签
old_tag_items = zot.everything(zot.items(tag='old-name'))
for item in old_tag_items:
    # 添加新标签
    item['data']['tags'].append({'tag': 'new-name'})
    # 移除旧标签
    item['data']['tags'] = [t for t in item['data']['tags'] if t['tag'] != 'old-name']
zot.update_items(old_tag_items)
```

## 标签类型

Zotero在`tag['type']`中存储两种标签类型：
- `0` — 用户添加的标签（默认）
- `1` — 自动导入的标签（来自文献数据库）

```python
item = zot.item('ITEMKEY')
for tag in item['data']['tags']:
    print(tag['tag'], tag.get('type', 0))
```
