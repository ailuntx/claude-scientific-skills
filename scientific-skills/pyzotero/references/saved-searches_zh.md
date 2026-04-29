# 保存的搜索

## 检索保存的搜索

```python
# 获取所有保存的搜索的元数据（而非结果）
searches = zot.searches()
# 返回包含 name、key、conditions、version 的字典列表

for search in searches:
    print(search['data']['name'], search['data']['key'])
```

**注意**：保存的搜索*结果*无法通过 API 获取（截至 2025 年）。仅返回元数据。

## 创建保存的搜索

每个条件字典必须包含 `condition`、`operator` 和 `value`：

```python
conditions = [
    {
        'condition': 'title',
        'operator': 'contains',
        'value': 'machine learning'
    }
]
zot.saved_search('ML Papers', conditions)
```

### 多条件（AND 逻辑）

```python
conditions = [
    {'condition': 'itemType', 'operator': 'is', 'value': 'journalArticle'},
    {'condition': 'tag', 'operator': 'is', 'value': 'unread'},
    {'condition': 'date', 'operator': 'isAfter', 'value': '2023-01-01'},
]
zot.saved_search('Recent Unread Articles', conditions)
```

## 删除保存的搜索

```python
# 首先获取搜索的 key
searches = zot.searches()
keys = [s['data']['key'] for s in searches if s['data']['name'] == 'Old Search']
zot.delete_saved_search(keys)
```

## 发现有效的运算符和条件

```python
# 所有可用运算符
operators = zot.show_operators()

# 所有可用条件
conditions = zot.show_conditions()

# 特定条件对应的有效运算符
title_operators = zot.show_condition_operators('title')
# 例如：['is', 'isNot', 'contains', 'doesNotContain', 'beginsWith']
```

## 常见的条件/运算符组合

| 条件 | 常用运算符 |
|-----------|-----------------|
| `title` | `contains`, `doesNotContain`, `is`, `beginsWith` |
| `tag` | `is`, `isNot` |
| `itemType` | `is`, `isNot` |
| `date` | `isBefore`, `isAfter`, `is` |
| `creator` | `contains`, `is` |
| `publicationTitle` | `contains`, `is` |
| `year` | `is`, `isBefore`, `isAfter` |
| `collection` | `is`, `isNot` |
| `fulltextContent` | `contains` |
