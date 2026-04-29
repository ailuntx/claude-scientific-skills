# 分页：follow()、everything()、生成器

Pyzotero 默认返回 100 条记录。使用以下方法可获取更多结果。

## everything() — 获取所有结果

获取全部条目的最简单方式：

```python
# 文库中的所有条目
all_items = zot.everything(zot.items())

# 所有顶层条目
all_top = zot.everything(zot.top())

# 集合中的所有条目
all_col = zot.everything(zot.collection_items('COLKEY'))

# 匹配搜索的所有条目
all_results = zot.everything(zot.items(q='machine learning', itemType='journalArticle'))
```

`everything()` 适用于所有可返回多个条目的读取 API 调用。

## follow() — 顺序分页

```python
# 分批检索条目，手动推进分页
first_batch = zot.top(limit=25)
second_batch = zot.follow()   # 后续 25 条
third_batch = zot.follow()    # 再后续 25 条
```

**警告**：当无更多条目时，`follow()` 会抛出 `StopIteration`。在单条目调用（如 `zot.item()`）后无效。

## iterfollow() — 生成器

```python
# 创建基于 follow() 的生成器
first = zot.top(limit=10)
lazy = zot.iterfollow()

# 获取后续分页
second = next(lazy)
third = next(lazy)
```

## makeiter() — 任意方法的生成器封装

```python
# 直接从方法调用创建生成器
gen = zot.makeiter(zot.top(limit=25))

page1 = next(gen)  # 前 25 条
page2 = next(gen)  # 后续 25 条
# 数据耗尽时抛出 StopIteration
```

## 手动 start/limit 分页

```python
page_size = 50
offset = 0

while True:
    batch = zot.items(limit=page_size, start=offset)
    if not batch:
        break
    # 处理批次数据
    for item in batch:
        process(item)
    offset += page_size
```

## 性能说明

- `everything()` 会顺序发起多次 API 调用；大型文库可能耗时较长
- 对于包含数千条目的文库，使用 `since=version` 仅获取变更条目（适用于同步场景）
- `follow()`、`everything()` 和 `makeiter()` 仅适用于返回多个条目的方法
