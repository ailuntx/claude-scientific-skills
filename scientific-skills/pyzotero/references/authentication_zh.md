# 认证与设置

## 凭证

从 https://www.zotero.org/settings/keys 获取：

| 凭证类型       | 位置说明                     |
|----------------|------------------------------|
| **用户 ID**    | "用于 API 调用的用户 ID" 区域 |
| **API 密钥**   | 在 /settings/keys/new 创建新密钥 |
| **群组库 ID**  | 群组 URL 中 `/groups/` 后的整数（例如 `https://www.zotero.org/groups/169947`） |

## 环境变量

存储在 `.env` 文件或在 shell 中导出：
```
ZOTERO_LIBRARY_ID=436
ZOTERO_API_KEY=ABC1234XYZ
ZOTERO_LIBRARY_TYPE=user
```

在 Python 中加载：
```python
import os
from dotenv import load_dotenv
from pyzotero import Zotero

load_dotenv()

zot = Zotero(
    library_id=os.environ['ZOTERO_LIBRARY_ID'],
    library_type=os.environ['ZOTERO_LIBRARY_TYPE'],
    api_key=os.environ['ZOTERO_API_KEY']
)
```

## 库类型

```python
# 个人库
zot = Zotero('436', 'user', 'ABC1234XYZ')

# 群组库
zot = Zotero('169947', 'group', 'ABC1234XYZ')
```

**重要提示**：每个 `Zotero` 实例仅绑定单个库。访问多个库需创建多个实例。

## 本地模式（只读）

无需 API 密钥连接本地 Zotero 安装。仅支持读取操作。

```python
zot = Zotero(library_id='436', library_type='user', local=True)
items = zot.items(limit=10)  # 从本地 Zotero 读取
```

## 可选参数

```python
zot = Zotero(
    library_id='436',
    library_type='user',
    api_key='ABC1234XYZ',
    preserve_json_order=True,   # 对 JSON 响应使用 OrderedDict
    locale='en-US',             # 本地化字段名称（例如法语用 'fr-FR'）
)
```

## 密钥权限

检查当前 API 密钥的访问范围：
```python
info = zot.key_info()
# 返回包含用户信息和群组权限的字典
```

检查可访问的群组：
```python
groups = zot.groups()
# 返回当前密钥可访问的群组库列表
```

## API 密钥作用域

在 https://www.zotero.org/settings/keys/new 创建密钥时选择适当权限：
- **只读**：用于检索条目和集合
- **写入权限**：用于创建、更新和删除条目
- **笔记权限**：在读写操作中包含笔记
- **文件权限**：上传附件所需
