# 搜索与请求参数

参数可直接传递给任何读取 API 调用，或通过 `add_parameters()` 全局设置。

```python
# 内联参数（仅对单次调用有效）
results = zot.items(q='climate change', limit=50, sort='date', direction='desc')

# 全局设置（会被下次调用的内联参数覆盖）
zot.add_parameters(limit=50, sort='dateAdded')
results = zot.items()
```

## 可用参数

| 参数 | 类型 | 描述 |
|-----------|------|-------------|
| `q` | str | 快速搜索——默认检索标题和创建者字段 |
| `qmode` | str | `'titleCreatorYear'`（默认）或 `'everything'`（全文检索） |
| `itemType` | str | 按条目类型过滤。运算符详见搜索语法 |
| `tag` | str 或 list | 按标签过滤。多标签采用 AND 逻辑 |
| `since` | int | 仅返回此库版本后修改的对象 |
| `sort` | str | 排序字段（见下文） |
| `direction` | str | `'asc'`（升序）或 `'desc'`（降序） |
| `limit` | int | 1–100 或 `None` |
| `start` | int | 结果集偏移量 |
| `format` | str | 响应格式（见 exports.md） |
| `itemKey` | str | 逗号分隔的条目键（最多 50 个） |
| `content` | str | `'bib'`、`'html'`、`'citation'` 或导出格式 |
| `style` | str | CSL 样式名称（配合 `content='bib'` 使用） |
| `linkwrap` | str | `'1'` 表示在参考文献输出中将 URL 包裹在 `<a>` 标签中 |

## 排序字段

`dateAdded`, `dateModified`, `title`, `creator`, `type`, `date`, `publisher`,
`publicationTitle`, `journalAbbreviation`, `language`, `accessDate`,
`libraryCatalog`, `callNumber`, `rights`, `addedBy`, `numItems`, `tags`

## 标签搜索语法

```python
# 单标签
zot.items(tag='machine learning')

# 多标签——AND 逻辑（条目必须包含所有标签）
zot.items(tag=['climate', 'adaptation'])

# OR 逻辑（包含任意标签）
zot.items(tag='climate OR adaptation')

# 排除标签
zot.items(tag='-retracted')
```

## 条目类型过滤

```python
# 单类型
zot.items(itemType='journalArticle')

# OR 多类型
zot.items(itemType='journalArticle || book')

# 排除类型
zot.items(itemType='-note')
```

常见条目类型：`journalArticle`, `book`, `bookSection`, `conferencePaper`,
`thesis`, `report`, `dataset`, `preprint`, `note`, `attachment`, `webpage`,
`patent`, `statute`, `case`, `hearing`, `interview`, `letter`, `manuscript`,
`map`, `artwork`, `audioRecording`, `videoRecording`, `podcast`, `film`,
`radioBroadcast`, `tvBroadcast`, `presentation`, `encyclopediaArticle`,
`dictionaryEntry`, `forumPost`, `blogPost`, `instantMessage`, `email`,
`document`, `computerProgram`, `bill`, `newspaperArticle`, `magazineArticle`

## 示例

```python
# 按日期倒序检索匹配查询的最新期刊文章
zot.items(q='CRISPR', itemType='journalArticle', sort='date', direction='desc', limit=20)

# 检索自已知库版本后新增的条目
zot.items(since=4000)

# 分页获取含特定标签的条目
zot.items(tag='to-read', limit=25, start=25)

# 全文检索
zot.items(q='gene editing', qmode='everything', limit=10)
```
