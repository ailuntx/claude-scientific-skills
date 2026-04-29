# Benchling 身份验证参考

## 身份验证方法

Benchling 支持三种身份验证方法，每种都适用于不同的用例。

### 1. API 密钥身份验证 (Basic Auth)

**最适合：** 个人脚本、原型设计、单用户集成

**工作原理：**
- 在 HTTP Basic 身份验证中，将您的 API 密钥用作用户名
- 将密码字段留空
- 所有请求都必须使用 HTTPS

**获取 API 密钥：**
1. 登录您的 Benchling 账户
2. 导航到“个人资料设置”
3. 找到“API 密钥”部分
4. 生成新的 API 密钥
5. 安全地存储它（它只会显示一次）

**Python SDK 用法：**
```python
from benchling_sdk.benchling import Benchling
from benchling_sdk.auth.api_key_auth import ApiKeyAuth

benchling = Benchling(
    url="https://your-tenant.benchling.com",
    auth_method=ApiKeyAuth("your_api_key_here")
)
```

**直接 HTTP 用法：**
```bash
curl -X GET \
  https://your-tenant.benchling.com/api/v2/dna-sequences \
  -u "your_api_key_here:"
```

请注意 API 密钥后的冒号，且无密码。

**环境变量模式：**
```python
import os
from benchling_sdk.benchling import Benchling
from benchling_sdk.auth.api_key_auth import ApiKeyAuth

api_key = os.environ.get("BENCHLING_API_KEY")
tenant_url = os.environ.get("BENCHLING_TENANT_URL")

benchling = Benchling(
    url=tenant_url,
    auth_method=ApiKeyAuth(api_key)
)
```

### 2. OAuth 2.0 客户端凭据

**最适合：** 多用户应用程序、服务账户、生产集成

**工作原理：**
1. 在 Benchling 的开发者控制台中注册一个应用程序
2. 获取客户端 ID 和客户端密钥
3. 交换凭据以获取访问令牌
4. 将访问令牌用于 API 请求
5. 令牌过期时刷新

**注册应用程序：**
1. 以管理员身份登录 Benchling
2. 导航到“开发者控制台”
3. 创建一个新应用程序
4. 记录客户端 ID 和客户端密钥
5. 配置 OAuth 重定向 URI 和权限

**Python SDK 用法：**
```python
from benchling_sdk.benchling import Benchling
from benchling_sdk.auth.client_credentials_oauth2 import ClientCredentialsOAuth2

auth_method = ClientCredentialsOAuth2(
    client_id="your_client_id",
    client_secret="your_client_secret"
)

benchling = Benchling(
    url="https://your-tenant.benchling.com",
    auth_method=auth_method
)
```

SDK 会自动处理令牌刷新。

**直接 HTTP 令牌流程：**
```bash
# 获取访问令牌
curl -X POST \
  https://your-tenant.benchling.com/api/v2/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials" \
  -d "client_id=your_client_id" \
  -d "client_secret=your_client_secret"

# 响应：
# {
#   "access_token": "token_here",
#   "token_type": "Bearer",
#   "expires_in": 3600
# }

# 使用访问令牌
curl -X GET \
  https://your-tenant.benchling.com/api/v2/dna-sequences \
  -H "Authorization: Bearer access_token_here"
```

### 3. OpenID Connect (OIDC)

**最适合：** 与现有身份提供商进行企业集成、SSO 场景

**工作原理：**
- 通过您的身份提供商（Okta、Azure AD 等）对用户进行身份验证
- 身份提供商颁发包含电子邮件声明的 ID 令牌
- Benchling 根据 OpenID 配置端点验证令牌
- 通过电子邮件匹配已验证的用户

**要求：**
- 企业 Benchling 账户
- 已配置的身份提供商 (IdP)
- IdP 必须颁发包含电子邮件声明的令牌
- 令牌中的电子邮件必须与 Benchling 用户电子邮件匹配

**身份提供商配置：**
1. 配置您的 IdP 以颁发 OpenID Connect 令牌
2. 确保令牌包含 `email` 声明
3. 向 Benchling 提供您的 IdP 的 OpenID 配置 URL
4. Benchling 将根据此配置验证令牌

**Python 用法：**
```python
# 假设您从 IdP 获取了一个 ID 令牌
from benchling_sdk.benchling import Benchling
from benchling_sdk.auth.oidc_auth import OidcAuth

auth_method = OidcAuth(id_token="id_token_from_idp")

benchling = Benchling(
    url="https://your-tenant.benchling.com",
    auth_method=auth_method
)
```

**直接 HTTP 用法：**
```bash
curl -X GET \
  https://your-tenant.benchling.com/api/v2/dna-sequences \
  -H "Authorization: Bearer id_token_here"
```

## 安全最佳实践

### 凭据存储

**应该：**
- 将凭据存储在环境变量中
- 使用密码管理器或秘密管理服务（AWS Secrets Manager, HashiCorp Vault）
- 加密静态凭据
- 为开发/测试/生产环境使用不同的凭据

**不应该：**
- 将凭据提交到版本控制
- 在源文件中硬编码凭据
- 通过电子邮件或聊天共享凭据
- 将凭据存储在纯文本文件中

**环境变量示例：**
```python
import os
from dotenv import load_dotenv  # python-dotenv package

# 从 .env 文件加载（将 .env 添加到 .gitignore！）
load_dotenv()

api_key = os.environ["BENCHLING_API_KEY"]
tenant = os.environ["BENCHLING_TENANT_URL"]
```

### 凭据轮换

**API 密钥轮换：**
1. 在“个人资料设置”中生成新的 API 密钥
2. 更新您的应用程序以使用新密钥
3. 验证新密钥是否有效
4. 删除旧的 API 密钥

**应用程序密钥轮换：**
1. 导航到“开发者控制台”
2. 选择您的应用程序
3. 生成新的客户端密钥
4. 更新您的应用程序配置
5. 验证后删除旧密钥

**最佳实践：** 定期轮换凭据（例如，每 90 天），如果凭据泄露，应立即轮换。

### 访问控制

**最小权限原则：**
- 仅授予最低限度的必要权限
- 对于自动化，使用服务账户（应用程序）而不是个人账户
- 定期审查和审计权限

**应用程序权限：**
应用程序需要明确授予对以下各项的访问权限：
- 组织
- 团队
- 项目
- 文件夹

在“开发者控制台”中设置应用程序时配置这些权限。

**用户权限：**
API 访问权限与 UI 权限相同：
- 用户只能访问他们在 UI 中有权查看/编辑的数据
- 被暂停的用户将失去 API 访问权限
- 已归档的应用程序在解除归档之前将失去 API 访问权限

### 网络安全

**仅限 HTTPS：**
所有 Benchling API 请求都必须使用 HTTPS。HTTP 请求将被拒绝。

**IP 白名单（企业版）：**
某些企业账户可以将 API 访问限制在特定的 IP 范围。请联系 Benchling 支持部门进行配置。

**速率限制：**
Benchling 实施速率限制以防止滥用：
- 默认：每个用户/应用程序每 10 秒 100 个请求
- 超出速率限制时返回 429 状态码
- SDK 会自动使用指数退避重试

### 审计日志

**跟踪 API 使用情况：**
- 所有 API 调用都记录了用户/应用程序身份
- OAuth 应用程序显示带有用户归属的正确审计跟踪
- API 密钥调用归属于密钥所有者
- 在 Benchling 的管理控制台中查看审计日志

**应用程序的最佳实践：**
当多个用户通过您的应用程序进行交互时，请使用 OAuth 而非 API 密钥。这可以确保审计正确归属于实际用户，而不仅仅是应用程序。

## 故障排除

### 常见身份验证错误

**401 未授权：**
- 无效或过期的凭据
- API 密钥格式不正确
- 缺少 "Authorization" 头部

**解决方案：**
- 验证凭据是否正确
- 检查 API 密钥是否未过期或被删除
- 确保头部格式正确：`Authorization: Bearer <token>`

**403 禁止：**
- 凭据有效但权限不足
- 用户无权访问请求的资源
- 应用程序未被授予对组织/项目的访问权限

**解决方案：**
- 在 Benchling 中检查用户/应用程序权限
- 在“开发者控制台”中授予必要的访问权限（针对应用程序）
- 验证资源是否存在且用户有访问权限

**429 请求过多：**
- 超出速率限制
- 短时间内请求过多

**解决方案：**
- 实施指数退避
- SDK 会自动处理此问题
- 考虑缓存结果
- 分散请求时间

### 测试身份验证

**使用 curl 进行快速测试：**
```bash
# 测试 API 密钥
curl -X GET \
  https://your-tenant.benchling.com/api/v2/users/me \
  -u "your_api_key:" \
  -v

# 测试 OAuth 令牌
curl -X GET \
  https://your-tenant.benchling.com/api/v2/users/me \
  -H "Authorization: Bearer your_token" \
  -v
```

`/users/me` 端点返回已验证用户的信息，可用于验证凭据。

**Python SDK 测试：**
```python
from benchling_sdk.benchling import Benchling
from benchling_sdk.auth.api_key_auth import ApiKeyAuth

try:
    benchling = Benchling(
        url="https://your-tenant.benchling.com",
        auth_method=ApiKeyAuth("your_api_key")
    )

    # 测试身份验证
    user = benchling.users.get_me()
    print(f"Authenticated as: {user.name} ({user.email})")

except Exception as e:
    print(f"Authentication failed: {e}")
```

## 多租户考量

如果使用多个 Benchling 租户：

```python
# 多个租户的配置
tenants = {
    "production": {
        "url": "https://prod.benchling.com",
        "api_key": os.environ["PROD_API_KEY"]
    },
    "staging": {
        "url": "https://staging.benchling.com",
        "api_key": os.environ["STAGING_API_KEY"]
    }
}

# 初始化客户端
clients = {}
for name, config in tenants.items():
    clients[name] = Benchling(
        url=config["url"],
        auth_method=ApiKeyAuth(config["api_key"])
    )

# 使用特定客户端
prod_sequences = clients["production"].dna_sequences.list()
```

## 高级：自定义 HTTPS 客户端

对于具有自签名证书或公司代理的环境：

```python
import httpx
from benchling_sdk.benchling import Benchling
from benchling_sdk.auth.api_key_auth import ApiKeyAuth

# 带有证书验证的自定义 httpx 客户端
custom_client = httpx.Client(
    verify="/path/to/custom/ca-bundle.crt",
    timeout=30.0
)

benchling = Benchling(
    url="https://your-tenant.benchling.com",
    auth_method=ApiKeyAuth("your_api_key"),
    http_client=custom_client
)
```

## 参考资料

- **官方身份验证文档：** https://docs.benchling.com/docs/authentication
- **开发者控制台：** https://your-tenant.benchling.com/developer
- **SDK 文档：** https://benchling.com/sdk-docs/
