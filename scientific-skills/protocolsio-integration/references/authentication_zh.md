# protocols.io 认证

## 概述

protocols.io API 支持两种访问令牌进行认证，可访问公共内容和私有内容。

## 访问令牌类型

### 1. CLIENT_ACCESS_TOKEN

- **用途**：访问公共内容及客户端用户的私有内容  
- **使用场景**：访问个人协议和公共协议时  
- **权限范围**：仅限令牌持有者的私有内容及所有公共内容  

### 2. OAUTH_ACCESS_TOKEN

- **用途**：访问特定用户的私有内容及所有公共内容  
- **使用场景**：构建需要经用户授权访问其内容的应用程序时  
- **权限范围**：被授权用户的完整私有内容及所有公共内容  

## 认证请求头

所有 API 请求必须包含 Authorization 请求头：

```
Authorization: Bearer [ACCESS_TOKEN]
```

## OAuth 流程

### 步骤 1：生成授权链接

引导用户访问授权 URL 授予权限：

```
GET https://protocols.io/api/v3/oauth/authorize
```

**参数：**  
- `client_id`（必填）：应用客户端 ID  
- `redirect_uri`（必填）：授权后重定向用户的 URL  
- `response_type`（必填）：固定为 "code"  
- `state`（推荐）：随机字符串防止 CSRF 攻击  

**示例：**  
```
https://protocols.io/api/v3/oauth/authorize?client_id=YOUR_CLIENT_ID&redirect_uri=YOUR_REDIRECT_URI&response_type=code&state=RANDOM_STRING
```

### 步骤 2：兑换授权码获取令牌

用户授权后，protocols.io 将携带授权码重定向至您的 `redirect_uri`。兑换该授权码获取访问令牌：

```
POST https://protocols.io/api/v3/oauth/token
```

**参数：**  
- `grant_type`：固定为 "authorization_code"  
- `code`：收到的授权码  
- `client_id`：应用客户端 ID  
- `client_secret`：应用客户端密钥  
- `redirect_uri`：必须与步骤 1 中的 redirect_uri 完全匹配  

**响应包含：**  
- `access_token`：用于 API 请求的 OAuth 访问令牌  
- `token_type`："Bearer"  
- `expires_in`：令牌有效期（秒，通常为 1 年）  
- `refresh_token`：用于刷新访问令牌的令牌  

### 步骤 3：刷新访问令牌

在访问令牌过期前（通常为 1 年），使用刷新令牌获取新访问令牌：

```
POST https://protocols.io/api/v3/oauth/token
```

**参数：**  
- `grant_type`：固定为 "refresh_token"  
- `refresh_token`：步骤 2 中获取的刷新令牌  
- `client_id`：应用客户端 ID  
- `client_secret`：应用客户端密钥  

## 速率限制

API 请求需注意速率限制：

- **标准端点**：每用户每分钟 100 次请求  
- **PDF 端点**（`/view/[protocol-uri].pdf`）：  
  - 登录用户：每分钟 5 次请求  
  - 未登录用户：每分钟 3 次请求  

## 最佳实践

1. **安全存储令牌**：切勿在客户端代码或版本控制中暴露访问令牌  
2. **处理令牌过期**：在过期前实现自动令牌刷新机制  
3. **遵守速率限制**：对限流错误实施指数退避策略  
4. **使用 state 参数**：OAuth 流程中始终包含 state 参数保障安全  
5. **验证 redirect_uri**：确保授权请求与令牌请求中的重定向 URI 完全一致
