# LaminDB 安装与部署

本文档涵盖 LaminDB 的安装、配置、实例管理、存储选项及部署策略。

## 安装

### 基础安装

```bash
# 安装 LaminDB
pip install lamindb

# 或使用 pip3
pip3 install lamindb
```

### 扩展功能安装

安装特定功能的可选依赖：

```bash
# Google Cloud Platform 支持
pip install 'lamindb[gcp]'

# 流式细胞术格式支持
pip install 'lamindb[fcs]'

# 数组存储与流处理（Zarr 支持）
pip install 'lamindb[zarr]'

# AWS S3 支持（通常默认包含）
pip install 'lamindb[aws]'

# 多扩展组合安装
pip install 'lamindb[gcp,zarr,fcs]'
```

### 模块插件

```bash
# 生物本体（Bionty）
pip install bionty

# 湿实验室功能
pip install lamindb-wetlab

# 临床数据（OMOP CDM）
pip install lamindb-clinical
```

### 验证安装

```python
import lamindb as ln
print(ln.__version__)

# 检查可用模块
import bionty as bt
print(bt.__version__)
```

## 身份认证

### 创建账户

1. 访问 https://lamin.ai
2. 注册免费账户
3. 进入账户设置生成 API 密钥

### 登录操作

```bash
# 使用 API 密钥登录
lamin login

# 系统将提示输入 API 密钥
# API 密钥存储在本地 ~/.lamin/ 目录
```

### 认证详情

**数据隐私：** LaminDB 认证仅收集基础元数据（邮箱、用户信息）。您的实际数据保持私有，不会发送至 LaminDB 服务器。

**本地与云端：** 即使仅限本地使用也需认证，以启用协作功能与实例管理。

## 实例初始化

### 本地 SQLite 实例

适用于本地开发和小型数据集：

```bash
# 在当前目录初始化
lamin init --storage ./mydata

# 在指定目录初始化
lamin init --storage /path/to/data

# 加载特定模块初始化
lamin init --storage ./mydata --modules bionty

# 加载多模块初始化
lamin init --storage ./mydata --modules bionty,wetlab
```

### 云端存储 + SQLite

使用云端存储但本地 SQLite 数据库：

```bash
# AWS S3
lamin init --storage s3://my-bucket/path

# Google 云存储
lamin init --storage gs://my-bucket/path

# S3 兼容存储（MinIO, Cloudflare R2）
lamin init --storage 's3://bucket?endpoint_url=http://endpoint:9000'
```

### 云端存储 + PostgreSQL

适用于生产环境部署：

```bash
# S3 + PostgreSQL
lamin init --storage s3://my-bucket/path \
  --db postgresql://user:password@hostname:5432/dbname \
  --modules bionty

# GCS + PostgreSQL
lamin init --storage gs://my-bucket/path \
  --db postgresql://user:password@hostname:5432/dbname \
  --modules bionty
```

### 实例命名

```bash
# 指定实例名称
lamin init --storage ./mydata --name my-project

# 默认使用目录名
lamin init --storage ./mydata  # 实例名称: "mydata"
```

## 连接实例

### 连接自有实例

```bash
# 按名称连接
lamin connect my-project

# 按完整路径连接
lamin connect account_handle/my-project
```

### 连接共享实例

```bash
# 连接他人实例
lamin connect other-user/their-project

# 需具备相应权限
```

### 切换实例

```bash
# 列出可用实例
lamin info

# 切换实例
lamin connect another-instance

# 关闭当前实例
lamin close
```

## 存储配置

### 本地存储

**优势：**
- 访问速度快
- 无需网络连接
- 配置简单

**配置：**
```bash
lamin init --storage ./data
```

### AWS S3 存储

**优势：**
- 可扩展性强
- 支持协作
- 数据持久化

**配置：**
```bash
# 设置凭证
export AWS_ACCESS_KEY_ID=your_key_id
export AWS_SECRET_ACCESS_KEY=your_secret_key
export AWS_DEFAULT_REGION=us-east-1

# 初始化
lamin init --storage s3://my-bucket/project-data \
  --db postgresql://user:pwd@host:5432/db
```

**所需 S3 权限：**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket/*",
        "arn:aws:s3:::my-bucket"
      ]
    }
  ]
}
```

### Google 云存储

**配置：**
```bash
# 身份认证
gcloud auth application-default login

# 或使用服务账号
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/credentials.json

# 初始化
lamin init --storage gs://my-bucket/project-data \
  --db postgresql://user:pwd@host:5432/db
```

### S3 兼容存储

适用于 MinIO、Cloudflare R2 等 S3 兼容服务：

```bash
# MinIO 示例
export AWS_ACCESS_KEY_ID=minioadmin
export AWS_SECRET_ACCESS_KEY=minioadmin

lamin init --storage 's3://my-bucket?endpoint_url=http://minio.example.com:9000'

# Cloudflare R2 示例
export AWS_ACCESS_KEY_ID=your_r2_access_key
export AWS_SECRET_ACCESS_KEY=your_r2_secret_key

lamin init --storage 's3://bucket?endpoint_url=https://account-id.r2.cloudflarestorage.com'
```

## 数据库配置

### SQLite（默认）

**优势：**
- 无需独立数据库服务
- 配置简单
- 适合开发环境

**限制：**
- 不支持并发写入
- 扩展性有限

**配置：**
```bash
# SQLite 为默认选项
lamin init --storage ./data
# 数据库存储在 ./data/.lamindb/
```

### PostgreSQL

**优势：**
- 满足生产需求
- 支持并发访问
- 大规模性能更优

**配置：**
```bash
# 完整连接字符串
lamin init --storage s3://bucket/path \
  --db postgresql://username:password@hostname:5432/database

# 启用 SSL
lamin init --storage s3://bucket/path \
  --db "postgresql://user:pwd@host:5432/db?sslmode=require"
```

**PostgreSQL 版本：** 兼容 PostgreSQL 12+

### 数据库模式管理

```bash
# 检查当前模式版本
lamin migrate check

# 升级模式
lamin migrate deploy

# 查看迁移历史
lamin migrate history
```

## 缓存配置

### 缓存目录

LaminDB 为云端文件维护本地缓存：

```python
import lamindb as ln

# 查看缓存位置
print(ln.settings.cache_dir)
```

### 配置缓存位置

```bash
# 设置缓存目录
lamin cache set /path/to/cache

# 查看当前缓存设置
lamin cache get
```

### 系统级缓存（多用户）

适用于多用户共享系统：

```bash
# 创建系统设置文件
sudo mkdir -p /system/settings
sudo nano /system/settings/system.env
```

在 `system.env` 中添加：
```bash
lamindb_cache_path=/shared/cache/lamindb
```

确保权限：
```bash
sudo chmod 755 /shared/cache/lamindb
sudo chown -R shared-user:shared-group /shared/cache/lamindb
```

### 缓存管理

```python
import lamindb as ln

# 清除特定工件缓存
artifact = ln.Artifact.get(key="data.h5ad")
artifact.delete_cache()

# 检查是否已缓存
if artifact.is_cached():
    print("已缓存")

# 手动清除完整缓存
import shutil
shutil.rmtree(ln.settings.cache_dir)
```

## 设置管理

### 查看当前设置

```python
import lamindb as ln

# 用户设置
print(ln.setup.settings.user)
# User(handle='username', email='user@email.com', name='Full Name')

# 实例设置
print(ln.setup.settings.instance)
# Instance(name='my-project', storage='s3://bucket/path')
```

### 配置设置

```bash
# 为相对路径键设置开发目录
lamin settings set dev-dir /path/to/project

# 配置 Git 同步
lamin settings set sync-git-repo https://github.com/user/repo.git

# 查看所有设置
lamin settings
```

### 环境变量

```bash
# 缓存目录
export LAMIN_CACHE_DIR=/path/to/cache

# 设置目录
export LAMIN_SETTINGS_DIR=/path/to/settings

# Git 同步
export LAMINDB_SYNC_GIT_REPO=https://github.com/user/repo.git
```

## 实例管理

### 查看实例信息

```bash
# 当前实例信息
lamin info

# 列出所有实例
lamin ls

# 查看实例详情
lamin instance details
```

### 实例协作

```bash
# 设置实例可见性（需 LaminHub）
lamin instance set-visibility public
lamin instance set-visibility private

# 邀请协作者（需 LaminHub）
lamin instance invite user@email.com
```

### 实例迁移

```bash
# 创建实例备份
lamin backup create

# 从备份恢复
lamin backup restore backup_id

# 导出实例元数据
lamin export instance-metadata.json
```

### 删除实例

```bash
# 删除实例（保留数据，移除元数据）
lamin delete --force instance-name

# 此操作仅移除 LaminDB 元数据
# 存储位置的实际数据仍保留
```

## 生产部署模式

### 模式 1：本地开发 → 云端生产

**开发环境：**
```bash
# 本地开发
lamin init --storage ./dev-data --modules bionty
```

**生产环境：**
```bash
# 云端生产
lamin init --storage s3://prod-bucket/data \
  --db postgresql://user:pwd@db-host:5432/prod-db \
  --modules bionty \
  --name production
```

**迁移流程：** 从开发环境导出工件，导入生产环境
```python
# 从开发环境导出
artifacts = ln.Artifact.filter().all()
for artifact in artifacts:
    artifact.export("/tmp/export/")

# 切换至生产环境
lamin connect production

# 导入生产环境
for file in Path("/tmp/export/").glob("*"):
    ln.Artifact(str(file), key=file.name).save()
```

### 模式 2：多区域部署

为满足数据主权要求部署多区域实例：

```bash
# 美国实例
lamin init --storage s3://us-bucket/data \
  --db postgresql://user:pwd@us-db:5432/db \
  --name us-production

# 欧盟实例
lamin init --storage s3://eu-bucket/data \
  --db postgresql://user:pwd@eu-db:5432/db \
  --name eu-production
```

### 模式 3：共享存储 + 个人实例

多用户共享数据场景：

```bash
# 共享存储 + 用户专属数据库
lamin init --storage s3://shared-bucket/data \
  --db postgresql://user1:pwd@host:5432/user1_db \
  --name user1-workspace

lamin init --storage s3://shared-bucket/data \
  --db postgresql://user2:pwd@host:5432/user2_db \
  --name user2-workspace
```

## 性能优化

### 数据库性能

```python
# PostgreSQL 使用连接池
# 在数据库服务器配置中设置

# 通过索引优化查询
# LaminDB 自动为常用查询创建索引
```

### 存储性能

```bash
# 使用合适的存储类别
# S3：频繁访问用 STANDARD，混合访问用 INTELLIGENT_TIERING

# 配置分段上传阈值
export AWS_CLI_FILE_IO_BANDWIDTH=100MB
```

### 缓存优化

```python
# 预缓存常用工件
artifacts = ln.Artifact.filter(key__startswith="reference/")
for artifact in artifacts:
    artifact.cache()  # 下载至缓存

# 大型数组使用后备模式
adata = artifact.backed()  # 不加载到内存
```

## 安全最佳实践

1. **凭证管理：**
   - 使用环境变量而非硬编码凭证
   - 在 AWS/GCP 使用 IAM 角色替代访问密钥
   - 定期轮换凭证

2. **访问控制：**
   - 使用 PostgreSQL 实现多用户访问控制
   - 配置存储桶策略
   - 启用审计日志

3. **网络安全：**
   - 数据库连接使用 SSL/TLS
   - 云端部署使用 VPC
   - 尽可能限制 IP 地址

4. **数据保护：**
   - 启用静态加密（S3, GCS）
   - 使用传输中加密（HTTPS, SSL）
   - 实施备份策略

## 监控与维护

### 健康检查

```python
import lamindb as ln

# 检查数据库连接
try:
    ln.Artifact.filter().count()
    print("✓ 数据库连接正常")
except Exception as e:
    print(f"✗ 数据库错误: {e}")

# 检查存储访问
try:
    test_artifact = ln.Artifact("test.txt", key="healthcheck.txt").save()
    test_artifact.delete(permanent=True)
    print("✓ 存储访问正常")
except Exception as e:
    print(f"✗ 存储错误: {e}")
```

### 日志记录

```python
# 启用调试日志
import logging
logging.basicConfig(level=logging.DEBUG)

# LaminDB 操作将生成详细日志
```

### 备份策略

```bash
# 定期数据库备份（PostgreSQL）
pg_dump -h hostname -U username -d database > backup_$(date +%Y%m%d).sql

# 存储备份（S3 版本控制）
aws s3api put-bucket-versioning \
  --bucket my-bucket \
  --versioning-configuration Status=Enabled

# 元数据导出
lamin export metadata_backup.json
```

## 故障排除

### 常见问题

**问题：无法连接实例**
```bash
# 检查实例是否存在
lamin ls

# 验证认证状态
lamin login

# 重新连接
lamin connect instance-name
```

**问题：存储权限拒绝**
```bash
# 检查 AWS 凭证
aws s3 ls s3://your-bucket/

# 检查 GCS 凭证
gsutil ls gs://your-bucket/

# 验证 IAM 权限
```

**问题：数据库连接错误**
```bash
# 测试 PostgreSQL 连接
psql postgresql://user:pwd@host:5432/db

# 检查数据库版本兼容性
lamin migrate check
```

**问题：缓存已满**
```python
# 清除缓存
import lamindb as ln
import shutil
shutil.rmtree(ln.settings.cache_dir)

# 设置更大缓存位置
lamin cache set /larger/disk/cache
```

## 升级与迁移

### 升级 LaminDB

```bash
# 升级至最新版
pip install --upgrade lamindb

# 升级数据库模式
lamin migrate deploy
```

### 模式兼容性

检查兼容性矩阵确保数据库模式版本与安装的 LaminDB 版本兼容。

### 重大变更

主版本升级可能需要迁移：

```bash
# 检查重大变更
lamin migrate check

# 查看迁移计划
lamin migrate plan

# 执行迁移
lamin migrate deploy
```

## 最佳实践

1. **本地起步，云端扩展：** 本地开发，生产环境部署至云端
2. **生产环境使用 PostgreSQL：** SQLite 仅限开发使用
3. **合理配置缓存：** 根据工作集大小调整缓存
4. **启用版本控制：** 使用 S3/GCS 版本控制保护数据
5. **监控成本：** 跟踪云端部署的存储与计算成本
6. **文档化配置：** 保持
