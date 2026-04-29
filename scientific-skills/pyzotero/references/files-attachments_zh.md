# 文件与附件

## 下载文件

```python
# 获取附件的原始二进制内容
raw = zot.file('ATTACHMENTKEY')
with open('paper.pdf', 'wb') as f:
    f.write(raw)

# 便捷封装：将文件转储到磁盘
# 使用存储的文件名，保存到当前目录
zot.dump('ATTACHMENTKEY')

# 转储到指定路径和文件名
zot.dump('ATTACHMENTKEY', 'renamed_paper.pdf', '/home/user/papers/')
# 成功时返回完整文件路径
```

**注意**：HTML快照会以条目键命名的`.zip`文件形式转储。

## 查找附件

```python
# 获取父条目的子项（附件、笔记）
children = zot.children('PARENTKEY')
attachments = [c for c in children if c['data']['itemType'] == 'attachment']

# 获取附件键值
for att in attachments:
    key = att['data']['key']
    filename = att['data']['filename']
    content_type = att['data']['contentType']
    link_mode = att['data']['linkMode']  # 'imported_file', 'linked_file', 'imported_url', 'linked_url'
```

## 上传附件

**注意**：附件上传方法处于测试阶段。

```python
# 简单上传：通过路径上传一个或多个文件
result = zot.attachment_simple(['/path/to/paper.pdf', '/path/to/notes.docx'])

# 作为父条目的子项上传
result = zot.attachment_simple(['/path/to/paper.pdf'], parentid='PARENTKEY')

# 使用自定义文件名上传：(名称, 路径)元组列表
result = zot.attachment_both([
    ('Paper 2024.pdf', '/path/to/paper.pdf'),
    ('Supplementary.pdf', '/path/to/supp.pdf'),
], parentid='PARENTKEY')

# 向现有附件条目上传文件
result = zot.upload_attachments(attachment_items, basedir='/path/to/files/')
```

上传结果结构：
```python
{
    'success': [attachment_item1, ...],  # 成功项
    'failure': [attachment_item2, ...],  # 失败项
    'unchanged': [attachment_item3, ...] # 未变更项
}
```

## 附件模板

```python
# 获取文件附件模板
template = zot.item_template('attachment', linkmode='imported_file')
# linkmode选项: 'imported_file', 'linked_file', 'imported_url', 'linked_url'

# 可用的链接模式
modes = zot.item_attachment_link_modes()
```

## 从集合下载所有PDF文件

```python
import os

collection_key = 'COLKEY'
output_dir = '/path/to/output/'
os.makedirs(output_dir, exist_ok=True)

items = zot.everything(zot.collection_items(collection_key))
for item in items:
    children = zot.children(item['data']['key'])
    for child in children:
        if child['data']['itemType'] == 'attachment' and \
           child['data'].get('contentType') == 'application/pdf':
            try:
                zot.dump(child['data']['key'], path=output_dir)
            except Exception as e:
                print(f"下载失败 {child['data']['key']}: {e}")
```
