# 读取 API 方法

## 检索条目

```python
# 库中所有条目（默认每次调用返回100条）
items = zot.items()

# 仅顶级条目（排除子附件/笔记）
top = zot.top(limit=25)

# 通过键值获取特定条目
item = zot.item('ITEMKEY')

# 获取多个特定条目（每次调用最多50条）
subset = zot.get_subset(['KEY1', 'KEY2', 'KEY3'])

# 回收站中的条目
trash = zot.trash()

# 已删除条目（需要'since'参数）
deleted = zot.deleted(since=1000)

# "我的出版物"中的条目
pubs = zot.publications()  # 仅限用户库

# 统计所有条目数量
count = zot.count_items()

# 统计顶级条目数量
n = zot.num_items()
```

## 条目数据结构

条目以字典形式返回，数据位于`item['data']`中：

```python
item = zot.item('VDNIEAPH')[0]
title = item['data']['title']
item_type = item['data']['itemType']
creators = item['data']['creators']
tags = item['data']['tags']
key = item['data']['key']
version = item['data']['version']
collections = item['data']['collections']
doi = item['data'].get('DOI', '')
```

## 子条目

```python
# 获取父条目的子条目（笔记、附件）
children = zot.children('PARENTKEY')
```

## 检索集合

```python
# 所有集合（包含子集合）
collections = zot.collections()

# 仅顶级集合
top_collections = zot.collections_top()

# 特定集合
collection = zot.collection('COLLECTIONKEY')

# 集合的子集合
sub = zot.collections_sub('COLLECTIONKEY')

# 平铺列表中的所有集合及子集合
all_cols = zot.all_collections()
# 或从特定集合开始向下检索：
all_cols = zot.all_collections('COLLECTIONKEY')

# 特定集合中的条目（不含子集合）
col_items = zot.collection_items('COLLECTIONKEY')

# 特定集合中的顶级条目
col_top = zot.collection_items_top('COLLECTIONKEY')

# 统计集合中的条目数量
n = zot.num_collectionitems('COLLECTIONKEY')
```

## 检索标签

```python
# 库中所有标签
tags = zot.tags()

# 特定条目的标签
item_tags = zot.item_tags('ITEMKEY')

# 集合中的标签
col_tags = zot.collection_tags('COLLECTIONKEY')
```

## 检索群组

```python
groups = zot.groups()
# 返回当前密钥可访问的群组库列表
```

## 版本信息

```python
# 库的最后修改版本
version = zot.last_modified_version()

# 条目版本字典 {key: version}
item_versions = zot.item_versions()

# 集合版本字典 {key: version}
col_versions = zot.collection_versions()

# 自已知版本以来的变更（用于同步）
changed_items = zot.item_versions(since=1000)
```

## 库设置

```python
settings = zot.settings()
# 返回同步设置（订阅源、PDF阅读进度等）
# 使用'since'仅获取变更：
new_settings = zot.settings(since=500)
```

## 保存的搜索

```python
searches = zot.searches()
# 检索保存的搜索元数据（非结果）
```
