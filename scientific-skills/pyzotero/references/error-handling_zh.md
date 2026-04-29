# 错误处理

## 异常类型

Pyzotero 针对 API 错误会抛出 `ZoteroError` 子类。请从 `pyzotero.zotero_errors` 导入：

```python
from pyzotero import zotero_errors
```

常见异常：

| 异常 | 原因 |
|-----------|-------|
| `UserNotAuthorised` | API 密钥无效或缺失 |
| `HTTPError` | 通用 HTTP 错误 |
| `ParamNotPassed` | 缺少必需参数 |
| `CallDoesNotExist` | 库类型不支持的 API 方法 |
| `ResourceNotFound` | 条目/集合键未找到 |
| `Conflict` | 版本冲突（乐观锁） |
| `PreConditionFailed` | `If-Unmodified-Since-Version` 检查失败 |
| `TooManyItems` | 批量操作超过 50 条目限制 |
| `TooManyRequests` | API 请求频率超限 |
| `InvalidItemFields` | 条目字典包含未知字段 |

## 基础错误处理

```python
from pyzotero import Zotero
from pyzotero import zotero_errors

zot = Zotero('123456', 'user', 'APIKEY')

try:
    item = zot.item('BADKEY')
except zotero_errors.ResourceNotFound:
    print('条目未找到')
except zotero_errors.UserNotAuthorised:
    print('无效 API 密钥')
except Exception as e:
    print(f'意外错误: {e}')
    if hasattr(e, '__cause__'):
        print(f'引发原因: {e.__cause__}')
```

## 版本冲突处理

```python
try:
    zot.update_item(item)
except zotero_errors.PreConditionFailed:
    # 条目自您获取后已被修改 — 请重新获取并重试
    fresh_item = zot.item(item['data']['key'])
    fresh_item['data']['title'] = new_title
    zot.update_item(fresh_item)
```

## 检查无效字段

```python
from pyzotero import zotero_errors

template = zot.item_template('journalArticle')
template['badField'] = 'bad value'

try:
    zot.check_items([template])
except zotero_errors.InvalidItemFields as e:
    print(f'无效字段: {e}')
    # 在调用 create_items 前修复字段
```

## 频率限制

Zotero API 会限制请求频率。若收到 `TooManyRequests` 错误：

```python
import time
from pyzotero import zotero_errors

def safe_request(func, *args, **kwargs):
    retries = 3
    for attempt in range(retries):
        try:
            return func(*args, **kwargs)
        except zotero_errors.TooManyRequests:
            wait = 2 ** attempt
            print(f'请求受限，等待 {wait} 秒...')
            time.sleep(wait)
    raise RuntimeError('超出最大重试次数')

items = safe_request(zot.items, limit=100)
```

## 访问底层错误

```python
try:
    zot.item('BADKEY')
except Exception as e:
    print(e.__cause__)    # 原始 HTTP 错误
    print(e.__context__)  # 异常上下文
```
