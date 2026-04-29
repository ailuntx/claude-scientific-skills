# 连接与会话管理

本文档介绍如何使用 BlitzGateway 建立和管理与 OMERO 服务器的连接。

## 基础连接

### 标准连接模式

```python
from omero.gateway import BlitzGateway

# 创建连接
conn = BlitzGateway(username, password, host=host, port=4064)

# 连接服务器
if conn.connect():
    print("连接成功")
    # 执行操作
    conn.close()
else:
    print("连接失败")
```

### 连接参数

- **username** (str): OMERO 用户账户名
- **password** (str): 用户密码
- **host** (str): OMERO 服务器主机名或 IP 地址
- **port** (int): 服务器端口（默认：4064）
- **secure** (bool): 强制加密连接（默认：False）

### 安全连接

确保所有数据传输加密：

```python
conn = BlitzGateway(username, password, host=host, port=4064, secure=True)
conn.connect()
```

## 上下文管理器模式（推荐）

使用上下文管理器实现自动连接管理和清理：

```python
from omero.gateway import BlitzGateway

with BlitzGateway(username, password, host=host, port=4064) as conn:
    # 自动建立连接
    for project in conn.getObjects('Project'):
        print(project.getName())
    # 退出时自动关闭连接
```

**优势：**
- 自动调用 `connect()`
- 退出时自动调用 `close()`
- 异常安全的资源清理
- 代码更简洁

## 会话管理

### 从现有客户端创建连接

通过已有的 `omero.client` 会话创建 BlitzGateway：

```python
import omero.clients
from omero.gateway import BlitzGateway

# 创建客户端和会话
client = omero.client(host, port)
session = client.createSession(username, password)

# 从现有客户端创建 BlitzGateway
conn = BlitzGateway(client_obj=client)

# 使用连接
# ...

# 完成后关闭
conn.close()
```

### 获取会话信息

```python
# 获取当前用户信息
user = conn.getUser()
print(f"用户ID: {user.getId()}")
print(f"用户名: {user.getName()}")
print(f"全名: {user.getFullName()}")
print(f"管理员状态: {conn.isAdmin()}")

# 获取当前组
group = conn.getGroupFromContext()
print(f"当前组: {group.getName()}")
print(f"组ID: {group.getId()}")
```

### 检查管理员权限

```python
if conn.isAdmin():
    print("用户拥有管理员权限")

if conn.isFullAdmin():
    print("用户是完整管理员")
else:
    # 检查特定管理员权限
    privileges = conn.getCurrentAdminPrivileges()
    print(f"管理员权限: {privileges}")
```

## 组上下文管理

OMERO 使用组管理数据访问权限，用户可属于多个组。

### 获取当前组上下文

```python
# 获取当前组上下文
group = conn.getGroupFromContext()
print(f"当前组: {group.getName()}")
print(f"组ID: {group.getId()}")
```

### 跨所有组查询

使用组 ID `-1` 查询所有可访问组：

```python
# 设置为跨所有组查询
conn.SERVICE_OPTS.setOmeroGroup('-1')

# 此时查询将覆盖所有可访问组
image = conn.getObject("Image", image_id)
projects = conn.listProjects()
```

### 切换到特定组

切换上下文到指定组内操作：

```python
# 从对象获取组ID
image = conn.getObject("Image", image_id)
group_id = image.getDetails().getGroup().getId()

# 切换到该组上下文
conn.SERVICE_OPTS.setOmeroGroup(group_id)

# 后续操作使用此组上下文
projects = conn.listProjects()
```

### 列出可用组

```python
# 获取当前用户的所有组
for group in conn.getGroupsMemberOf():
    print(f"组名: {group.getName()} (ID: {group.getId()})")
```

## 高级连接功能

### 代理用户连接（仅管理员）

管理员可创建代理其他用户的连接：

```python
# 以管理员身份连接
admin_conn = BlitzGateway(admin_user, admin_pass, host=host, port=4064)
admin_conn.connect()

# 获取目标用户
target_user = admin_conn.getObject("Experimenter", user_id).getName()

# 创建代理用户连接
user_conn = admin_conn.suConn(target_user)

# 以目标用户身份执行操作
for project in user_conn.listProjects():
    print(project.getName())

# 关闭代理连接
user_conn.close()
admin_conn.close()
```

### 列出管理员

```python
# 获取所有管理员
for admin in conn.getAdministrators():
    print(f"ID: {admin.getId()}, 姓名: {admin.getFullName()}, "
          f"用户名: {admin.getOmeName()}")
```

## 连接生命周期

### 关闭连接

始终关闭连接以释放服务器资源：

```python
try:
    conn = BlitzGateway(username, password, host=host, port=4064)
    conn.connect()

    # 执行操作

except Exception as e:
    print(f"错误: {e}")
finally:
    if conn:
        conn.close()
```

### 检查连接状态

```python
if conn.isConnected():
    print("连接处于活动状态")
else:
    print("连接已关闭")
```

## 错误处理

### 健壮连接模式

```python
from omero.gateway import BlitzGateway
import traceback

def connect_to_omero(username, password, host, port=4064):
    """
    建立到 OMERO 服务器的连接（含错误处理）
    
    返回:
        BlitzGateway 连接对象（失败时返回 None）
    """
    try:
        conn = BlitzGateway(username, password, host=host, port=port, secure=True)
        if conn.connect():
            print(f"已连接到 {host}:{port} 用户 {username}")
            return conn
        else:
            print("建立连接失败")
            return None
    except Exception as e:
        print(f"连接错误: {e}")
        traceback.print_exc()
        return None

# 使用示例
conn = connect_to_omero(username, password, host)
if conn:
    try:
        # 执行操作
        pass
    finally:
        conn.close()
```

## 常用连接模式

### 模式1：简单脚本

```python
from omero.gateway import BlitzGateway

# 连接参数
HOST = 'omero.example.com'
PORT = 4064
USERNAME = 'user'
PASSWORD = 'pass'

# 连接
with BlitzGateway(USERNAME, PASSWORD, host=HOST, port=PORT) as conn:
    print(f"当前用户: {conn.getUser().getName()}")
    # 执行操作
```

### 模式2：基于配置的连接

```python
import yaml
from omero.gateway import BlitzGateway

# 加载配置
with open('omero_config.yaml', 'r') as f:
    config = yaml.safe_load(f)

# 使用配置连接
with BlitzGateway(
    config['username'],
    config['password'],
    host=config['host'],
    port=config.get('port', 4064),
    secure=config.get('secure', True)
) as conn:
    # 执行操作
    pass
```

### 模式3：环境变量

```python
import os
from omero.gateway import BlitzGateway

# 从环境变量获取凭据
USERNAME = os.environ.get('OMERO_USER')
PASSWORD = os.environ.get('OMERO_PASSWORD')
HOST = os.environ.get('OMERO_HOST', 'localhost')
PORT = int(os.environ.get('OMERO_PORT', 4064))

# 连接
with BlitzGateway(USERNAME, PASSWORD, host=HOST, port=PORT) as conn:
    # 执行操作
    pass
```

## 最佳实践

1. **使用上下文管理器**：始终优先使用自动清理的上下文管理器
2. **安全连接**：生产环境使用 `secure=True`
3. **错误处理**：用 try-except 块包裹连接代码
4. **关闭连接**：完成后始终关闭连接
5. **组上下文**：查询前设置正确的组上下文
6. **凭据安全**：避免硬编码凭据，使用环境变量或配置文件
7. **连接池**：Web 应用需实现连接池
8. **超时控制**：长时操作考虑实现连接超时

## 故障排除

### 连接被拒绝

```
无法联系 ORB
```

**解决方案：**
- 确认主机和端口正确
- 检查防火墙设置
- 确保 OMERO 服务器正在运行
- 验证网络连通性

### 认证失败

```
无法连接服务器
```

**解决方案：**
- 验证用户名和密码
- 检查用户账户是否激活
- 确认组成员关系
- 查看服务器日志获取详情

### 会话超时

**解决方案：**
- 增加服务器会话超时时间
- 实现会话保活机制
- 超时后重新连接
- 长时运行应用使用连接池
