# 集合管理

## 读取集合

```python
# 所有集合（包含嵌套的扁平列表）
all_cols = zot.collections()

# 仅顶级集合
top_cols = zot.collections_top()

# 特定集合
col = zot.collection('COLKEY')

# 某个集合的子集合
sub_cols = zot.collections_sub('COLKEY')

# 给定集合下的所有集合（递归）
tree = zot.all_collections('COLKEY')
# 或库中的所有集合：
tree = zot.all_collections()
```

## 集合数据结构

```python
col = zot.collection('5TSDXJG6')
name = col['data']['name']
key = col['data']['key']
parent = col['data']['parentCollection']  # 如果是顶级则为False，否则为父集合键
version = col['data']['version']
n_items = col['meta']['numItems']
n_sub_collections = col['meta']['numCollections']
```

## 创建集合

```python
# 创建顶级集合
zot.create_collections([{'name': 'My New Collection'}])

# 创建嵌套集合
zot.create_collections([{
    'name': 'Sub-Collection',
    'parentCollection': 'PARENTCOLKEY'
}])

# 批量创建
zot.create_collections([
    {'name': 'Collection A'},
    {'name': 'Collection B'},
    {'name': 'Sub-B', 'parentCollection': 'BKEY'},
])
```

## 更新集合

```python
cols = zot.collections()
# 重命名第一个集合
cols[0]['data']['name'] = 'Renamed Collection'
zot.update_collection(cols[0])

# 更新多个集合（自动分块，每块50个）
zot.update_collections(cols)
```

## 删除集合

```python
# 删除单个集合
col = zot.collection('COLKEY')
zot.delete_collection(col)

# 删除多个集合
cols = zot.collections()
zot.delete_collection(cols)  # 传入字典列表
```

## 管理集合中的条目

```python
# 将条目添加到集合
item = zot.item('ITEMKEY')
zot.addto_collection('COLKEY', item)

# 从集合中移除条目
zot.deletefrom_collection('COLKEY', item)

# 获取集合中所有条目
items = zot.collection_items('COLKEY')

# 仅获取集合中的顶级条目
top_items = zot.collection_items_top('COLKEY')

# 统计集合中的条目数量
n = zot.num_collectionitems('COLKEY')

# 获取集合中的标签
tags = zot.collection_tags('COLKEY')
```

## 按名称查找集合键

```python
def find_collection(zot, name):
    for col in zot.everything(zot.collections()):
        if col['data']['name'] == name:
            return col['data']['key']
    return None

key = find_collection(zot, 'Machine Learning Papers')
```
