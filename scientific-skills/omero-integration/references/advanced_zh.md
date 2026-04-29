# 高级功能

本参考涵盖高级 OMERO 操作，包括权限管理、删除操作、文件集管理及管理任务。

## 删除对象

### 带等待的删除

```python
# 删除对象并等待完成
project_ids = [1, 2, 3]
conn.deleteObjects("Project", project_ids, wait=True)
print("删除完成")

# 异步删除（无需等待）
conn.deleteObjects("Dataset", [dataset_id], wait=False)
```

### 带回调监控的删除

```python
from omero.callbacks import CmdCallbackI

# 启动删除操作
handle = conn.deleteObjects("Project", [project_id])

# 创建回调监控进度
cb = CmdCallbackI(conn.c, handle)
print("正在删除，请稍候...")

# 轮询完成状态
while not cb.block(500):  # 每500毫秒检查一次
    print(".", end="", flush=True)

print("\n删除完成")

# 检查错误
response = cb.getResponse()
if isinstance(response, omero.cmd.ERR):
    print("发生错误：")
    print(response)
else:
    print("删除成功")

# 清理资源
cb.close(True)  # 同时关闭句柄
```

### 删除不同类型对象

```python
# 删除图像
image_ids = [101, 102, 103]
conn.deleteObjects("Image", image_ids, wait=True)

# 删除数据集
dataset_ids = [10, 11]
conn.deleteObjects("Dataset", dataset_ids, wait=True)

# 删除ROI
roi_ids = [201, 202]
conn.deleteObjects("Roi", roi_ids, wait=True)

# 删除注释
annotation_ids = [301, 302]
conn.deleteObjects("Annotation", annotation_ids, wait=True)
```

### 级联删除

```python
# 删除项目将级联删除包含的数据集
# 具体行为取决于服务器配置
project_id = 123
conn.deleteObjects("Project", [project_id], wait=True)

# 数据集和图像可能被删除或成为孤立对象
# 取决于删除配置
```

## 文件集

文件集代表原始导入文件的集合，在 OMERO 5.0 中引入。

### 检查图像是否关联文件集

```python
image = conn.getObject("Image", image_id)

fileset = image.getFileset()
if fileset:
    print(f"图像属于文件集 {fileset.getId()}")
else:
    print("图像未关联文件集（OMERO 5.0 之前版本）")
```

### 访问文件集信息

```python
image = conn.getObject("Image", image_id)
fileset = image.getFileset()

if fileset:
    fs_id = fileset.getId()
    print(f"文件集 ID: {fs_id}")

    # 列出文件集中所有图像
    print("文件集中的图像：")
    for fs_image in fileset.copyImages():
        print(f"  {fs_image.getId()}: {fs_image.getName()}")

    # 列出原始导入文件
    print("\n原始文件：")
    for orig_file in fileset.listFiles():
        print(f"  {orig_file.getPath()}/{orig_file.getName()}")
        print(f"    大小: {orig_file.getSize()} 字节")
```

### 直接获取文件集

```python
# 获取文件集对象
fileset = conn.getObject("Fileset", fileset_id)

if fileset:
    # 访问图像
    for image in fileset.copyImages():
        print(f"图像: {image.getName()}")

    # 访问文件
    for orig_file in fileset.listFiles():
        print(f"文件: {orig_file.getName()}")
```

### 下载原始文件

```python
import os

fileset = image.getFileset()

if fileset:
    download_dir = "./original_files"
    os.makedirs(download_dir, exist_ok=True)

    for orig_file in fileset.listFiles():
        file_name = orig_file.getName()
        file_path = os.path.join(download_dir, file_name)

        print(f"下载中: {file_name}")

        # 获取RawFileStore对象
        raw_file_store = conn.createRawFileStore()
        raw_file_store.setFileId(orig_file.getId())

        # 分块下载
        with open(file_path, 'wb') as f:
            offset = 0
            chunk_size = 1024 * 1024  # 1MB分块
            size = orig_file.getSize()

            while offset < size:
                chunk = raw_file_store.read(offset, chunk_size)
                f.write(chunk)
                offset += len(chunk)

        raw_file_store.close()
        print(f"保存至: {file_path}")
```

## 组权限

OMERO 使用基于组的权限控制数据访问。

### 权限级别

- **私有** (`rw----`): 仅所有者可读写
- **只读** (`rwr---`): 组成员可读，仅所有者可写
- **读取-注释** (`rwra--`): 组成员可读和注释
- **读写** (`rwrw--`): 组成员可读写

### 检查当前组权限

```python
# 获取当前组
group = conn.getGroupFromContext()

# 获取权限
permissions = group.getDetails().getPermissions()
perm_string = str(permissions)

# 映射为可读名称
permission_names = {
    'rw----': '私有',
    'rwr---': '只读',
    'rwra--': '读取-注释',
    'rwrw--': '读写'
}

perm_name = permission_names.get(perm_string, '未知')
print(f"组: {group.getName()}")
print(f"权限: {perm_name} ({perm_string})")
```

### 列出用户所属组

```python
# 获取当前用户所有组
print("用户所属组：")
for group in conn.getGroupsMemberOf():
    print(f"  {group.getName()} (ID: {group.getId()})")

    # 获取组权限
    perms = group.getDetails().getPermissions()
    print(f"    权限: {perms}")
```

### 获取组成员

```python
# 获取组
group = conn.getObject("ExperimenterGroup", group_id)

# 列出成员
print(f"{group.getName()} 的成员：")
for member in group.getMembers():
    print(f"  {member.getFullName()} ({member.getOmeName()})")
```

## 跨组查询

### 查询所有可访问组

```python
# 设置上下文以查询所有可访问组
conn.SERVICE_OPTS.setOmeroGroup('-1')

# 此时查询将跨越所有组
image = conn.getObject("Image", image_id)
if image:
    group = image.getDetails().getGroup()
    print(f"图像所在组: {group.getName()}")

# 列出所有组中的项目
for project in conn.getObjects("Project"):
    group = project.getDetails().getGroup()
    print(f"项目: {project.getName()} (组: {group.getName()})")
```

### 切换到特定组

```python
# 获取图像所属组
image = conn.getObject("Image", image_id)
group_id = image.getDetails().getGroup().getId()

# 切换到该组上下文
conn.SERVICE_OPTS.setOmeroGroup(group_id)

# 后续操作使用此组
projects = conn.listProjects()  # 仅限此组
```

### 重置到默认组

```python
# 获取默认组
default_group_id = conn.getEventContext().groupId

# 切换回默认组
conn.SERVICE_OPTS.setOmeroGroup(default_group_id)
```

## 管理操作

### 检查管理员状态

```python
# 检查当前用户是否为管理员
if conn.isAdmin():
    print("用户具有管理员权限")

# 检查是否完全管理员
if conn.isFullAdmin():
    print("用户为完全管理员")
else:
    # 检查具体权限
    privileges = conn.getCurrentAdminPrivileges()
    print(f"管理员权限: {privileges}")
```

### 列出管理员

```python
# 获取所有管理员
print("管理员列表：")
for admin in conn.getAdministrators():
    print(f"  ID: {admin.getId()}")
    print(f"  用户名: {admin.getOmeName()}")
    print(f"  全名: {admin.getFullName()}")
```

### 设置对象所有者（仅管理员）

```python
import omero.model

# 创建带指定所有者的注释（需管理员权限）
tag_ann = omero.gateway.TagAnnotationWrapper(conn)
tag_ann.setValue("管理员创建的标签")

# 设置所有者
user_id = 5
tag_ann._obj.details.owner = omero.model.ExperimenterI(user_id, False)
tag_ann.save()

print(f"创建了用户 {user_id} 拥有的注释")
```

### 代用用户连接（仅管理员）

```python
# 以管理员身份连接
admin_conn = BlitzGateway(admin_user, admin_pass, host=host, port=4064)
admin_conn.connect()

# 获取目标用户
target_user_id = 10
user = admin_conn.getObject("Experimenter", target_user_id)
username = user.getOmeName()

# 以该用户身份创建连接
user_conn = admin_conn.suConn(username)

print(f"已作为 {username} 连接")

# 以该用户身份执行操作
for project in user_conn.listProjects():
    print(f"  {project.getName()}")

# 关闭连接
user_conn.close()
admin_conn.close()
```

### 列出所有用户

```python
# 获取所有用户（管理员操作）
print("所有用户：")
for user in conn.getObjects("Experimenter"):
    print(f"  ID: {user.getId()}")
    print(f"  用户名: {user.getOmeName()}")
    print(f"  全名: {user.getFullName()}")
    print(f"  邮箱: {user.getEmail()}")
    print()
```

## 服务访问

OMERO 提供多种服务用于特定操作。

### 更新服务

```python
# 获取更新服务
updateService = conn.getUpdateService()

# 保存并返回对象
roi = omero.model.RoiI()
roi.setImage(image._obj)
saved_roi = updateService.saveAndReturnObject(roi)

# 保存多个对象
objects = [obj1, obj2, obj3]
saved_objects = updateService.saveAndReturnArray(objects)
```

### ROI 服务

```python
# 获取ROI服务
roi_service = conn.getRoiService()

# 查找图像的ROI
result = roi_service.findByImage(image_id, None)

# 获取形状统计信息
shape_ids = [shape.id.val for roi in result.rois
             for shape in roi.copyShapes()]
stats = roi_service.getShapeStatsRestricted(shape_ids, 0, 0, [0])
```

### 元数据服务

```python
# 获取元数据服务
metadataService = conn.getMetadataService()

# 按类型和命名空间加载注释
ns_to_include = ["mylab.analysis"]
ns_to_exclude = []

annotations = metadataService.loadSpecifiedAnnotations(
    'omero.model.FileAnnotation',
    ns_to_include,
    ns_to_exclude,
    None
)

for ann in annotations:
    print(f"注释: {ann.getId().getValue()}")
```

### 查询服务

```python
# 获取查询服务
queryService = conn.getQueryService()

# 构建查询（复杂查询）
params = omero.sys.ParametersI()
params.addLong("image_id", image_id)

query = "select i from Image i where i.id = :image_id"
image = queryService.findByQuery(query, params)
```

### 缩略图服务

```python
# 获取缩略图服务
thumbnailService = conn.createThumbnailStore()

# 设置当前图像
thumbnailService.setPixelsId(image.getPrimaryPixels().getId())

# 获取缩略图
thumbnail = thumbnailService.getThumbnail(96, 96)

# 关闭服务
thumbnailService.close()
```

### 原始文件存储

```python
# 获取原始文件存储
rawFileStore = conn.createRawFileStore()

# 设置文件ID
rawFileStore.setFileId(orig_file_id)

# 读取文件
data = rawFileStore.read(0, rawFileStore.size())

# 关闭
rawFileStore.close()
```

## 对象所有权与详情

### 获取对象详情

```python
image = conn.getObject("Image", image_id)

# 获取详情
details = image.getDetails()

# 所有者信息
owner = details.getOwner()
print(f"所有者 ID: {owner.getId()}")
print(f"用户名: {owner.getOmeName()}")
print(f"全名: {owner.getFullName()}")

# 组信息
group = details.getGroup()
print(f"组: {group.getName()} (ID: {group.getId()})")

# 创建信息
creation_event = details.getCreationEvent()
print(f"创建时间: {creation_event.getTime()}")

# 更新信息
update_event = details.getUpdateEvent()
print(f"更新时间: {update_event.getTime()}")
```

### 获取权限

```python
# 获取对象权限
details = image.getDetails()
permissions = details.getPermissions()

# 检查具体权限
can_edit = permissions.canEdit()
can_annotate = permissions.canAnnotate()
can_link = permissions.canLink()
can_delete = permissions.canDelete()

print(f"可编辑: {can_edit}")
print(f"可注释: {can_annotate}")
print(f"可链接: {can_link}")
print(f"可删除: {can_delete}")
```

## 事件上下文

### 获取当前事件上下文

```python
# 获取事件上下文（当前会话信息）
ctx = conn.getEventContext()

print(f"用户 ID: {ctx.userId}")
print(f"用户名: {ctx.userName}")
print(f"组 ID: {ctx.groupId}")
print(f"组名: {ctx.groupName}")
print(f"会话 ID: {ctx.sessionId}")
print(f"是否管理员: {ctx.isAdmin}")
```

## 完整管理员示例

```python
from omero.gateway import BlitzGateway

# 以管理员身份连接
ADMIN_USER = 'root'
ADMIN_PASS = 'password'
HOST = 'omero.example.com'
PORT = 4064

with BlitzGateway(ADMIN_USER, ADMIN_PASS, host=HOST, port=PORT) as admin_conn:
    print("=== 管理员操作 ===\n")

    # 列出所有用户
    print("所有用户：")
    for user in admin_conn.getObjects("Experimenter"):
        print(f"  {user.getOmeName()}: {user.getFullName()}")

    # 列出所有组
    print("\n所有组：")
    for group in admin_conn.getObjects("ExperimenterGroup"):
        perms = group.getDetails().getPermissions()
        print(f"  {group.getName()}: {perms}")

        # 列出成员
        for member in group.getMembers():
            print(f"    - {member.getOmeName()}")

    # 跨组查询
    print("\n所有项目（全组）：")
    admin_conn.SERVICE_OPTS.setOmeroGroup('-1')

    for project in admin_conn.getObjects("Project"):
        owner = project.getDetails().getOwner()
        group = project.getDetails().getGroup()
        print(f"  {project.getName()}")
        print(f"    所有者: {owner.getOmeName()}")
        print(f"    所属组: {group.getName()}")

    # 以其他用户身份连接
    target_user_id = 5
    user = admin_conn.getObject("Experimenter", target_user_id)

    if user:
        print(f"\n=== 以 {user.getOmeName()} 身份操作 ===\n")

        user_conn = admin_conn.suConn(user.getOmeName())

        # 列出该用户的项目
        for project in user_conn.listProjects():
            print(f"  {project.getName()}")

        user_conn.close()
```

## 最佳实践

1. **权限检查**：操作前始终检查权限
2. **组上下文**：为查询设置合适的

# 在访问前检查对象是否存在

```python
obj = conn.getObject("Image", image_id)
if obj is None:
    print(f"Image {image_id} not found or not accessible")
else:
    # 处理对象
    pass
```

### 组上下文问题

```python
# 如果未找到对象，尝试跨组查询
conn.SERVICE_OPTS.setOmeroGroup('-1')
obj = conn.getObject("Image", image_id)

if obj:
    # 切换到对象所在组以进行后续操作
    group_id = obj.getDetails().getGroup().getId()
    conn.SERVICE_OPTS.setOmeroGroup(group_id)
```
