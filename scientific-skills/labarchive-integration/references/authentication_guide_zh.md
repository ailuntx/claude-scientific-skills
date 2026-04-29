# LabArchives 认证指南

## 先决条件

### 1. 企业版许可证

API访问需要企业版LabArchives许可证。请联系您的LabArchives管理员或sales@labarchives.com以：
- 确认您的机构具备企业版访问权限
- 为您的账户申请启用API访问
- 获取机构级API凭证

### 2. API凭证

您需要两组凭证：

#### 机构级API凭证（来自LabArchives管理员）
- **访问密钥ID**：机构级标识符
- **访问密码**：机构级密钥

#### 用户认证凭证（自行配置）
- **邮箱**：您的LabArchives账户邮箱（例如researcher@university.edu）
- **外部应用专用密码**：在LabArchives账户设置中创建

## 设置外部应用专用密码

外部应用专用密码不同于常规LabArchives登录密码，它可在不暴露主凭证的情况下提供API访问权限。

**创建外部应用专用密码步骤：**

1. 登录您的LabArchives账户（mynotebook.labarchives.com或机构专属URL）
2. 进入**账户设置**（点击右上角您的姓名）
3. 选择**安全与隐私**标签页
4. 找到**外部应用**区域
5. 点击**生成新密码**或**重置密码**
6. 复制并安全存储该密码（后续不可见）
7. 所有API认证均使用此密码

**安全提示：** 将此密码视为API令牌。若泄露，请立即在账户设置中重新生成。

## 配置文件设置

创建`config.yaml`文件安全存储凭证：

```yaml
# 区域API端点
api_url: https://api.labarchives.com/api

# 机构凭证（管理员提供）
access_key_id: YOUR_ACCESS_KEY_ID_HERE
access_password: YOUR_ACCESS_PASSWORD_HERE

# 用户凭证（用户级操作）
user_email: researcher@university.edu
user_external_password: YOUR_EXTERNAL_APP_PASSWORD_HERE
```

**替代方案：环境变量**

为增强安全性，可使用环境变量替代配置文件：

```bash
export LABARCHIVES_API_URL="https://api.labarchives.com/api"
export LABARCHIVES_ACCESS_KEY_ID="your_key_id"
export LABARCHIVES_ACCESS_PASSWORD="your_access_password"
export LABARCHIVES_USER_EMAIL="researcher@university.edu"
export LABARCHIVES_USER_PASSWORD="your_external_app_password"
```

## 区域端点

根据机构选择正确的区域API端点：

| 区域 | 端点 | 适用场景 |
|--------|----------|--------------------------------|
| 美国/国际 | `https://api.labarchives.com/api` | `mynotebook.labarchives.com` |
| 澳大利亚 | `https://auapi.labarchives.com/api` | `aunotebook.labarchives.com` |
| 英国 | `https://ukapi.labarchives.com/api` | `uknotebook.labarchives.com` |

使用错误的区域端点将导致认证失败（即使凭证正确）。

## 认证流程

### 方案1：使用labarchives-py Python封装库

```python
from labarchivespy.client import Client
import yaml

# 加载配置
with open('config.yaml', 'r') as f:
    config = yaml.safe_load(f)

# 使用机构凭证初始化客户端
client = Client(
    config['api_url'],
    config['access_key_id'],
    config['access_password']
)

# 以特定用户身份认证获取UID
login_params = {
    'login_or_email': config['user_email'],
    'password': config['user_external_password']
}
response = client.make_call('users', 'user_access_info', params=login_params)

# 解析响应获取UID
import xml.etree.ElementTree as ET
uid = ET.fromstring(response.content)[0].text
print(f"已认证用户ID: {uid}")
```

### 方案2：Python requests直接HTTP请求

```python
import requests
import yaml

# 加载配置
with open('config.yaml', 'r') as f:
    config = yaml.safe_load(f)

# 构造API调用
url = f"{config['api_url']}/users/user_access_info"
params = {
    'access_key_id': config['access_key_id'],
    'access_password': config['access_password'],
    'login_or_email': config['user_email'],
    'password': config['user_external_password']
}

# 发起认证请求
response = requests.get(url, params=params)

if response.status_code == 200:
    print("认证成功！")
    print(response.content.decode('utf-8'))
else:
    print(f"认证失败: {response.status_code}")
    print(response.content.decode('utf-8'))
```

### 方案3：使用R语言

```r
library(httr)
library(xml2)

# 配置
api_url <- "https://api.labarchives.com/api"
access_key_id <- "YOUR_ACCESS_KEY_ID"
access_password <- "YOUR_ACCESS_PASSWORD"
user_email <- "researcher@university.edu"
user_external_password <- "YOUR_EXTERNAL_APP_PASSWORD"

# 发起认证请求
response <- GET(
    paste0(api_url, "/users/user_access_info"),
    query = list(
        access_key_id = access_key_id,
        access_password = access_password,
        login_or_email = user_email,
        password = user_external_password
    )
)

# 解析响应
if (status_code(response) == 200) {
    content <- content(response, as = "text", encoding = "UTF-8")
    xml_data <- read_xml(content)
    uid <- xml_text(xml_find_first(xml_data, "//uid"))
    print(paste("已认证用户ID:", uid))
} else {
    print(paste("认证失败:", status_code(response)))
}
```

## OAuth认证（新集成方案）

LabArchives新第三方集成现采用OAuth 2.0。传统API密钥认证（上述方案）仍适用于直接API访问。

**OAuth流程（开发者适用）：**

1. 向LabArchives注册应用
2. 获取客户端ID和密钥
3. 实现OAuth 2.0授权码流程
4. 用授权码交换访问令牌
5. 使用访问令牌发起API请求

联系LabArchives开发者支持获取OAuth集成文档。

## 认证问题排查

### 401未授权错误

**可能原因及解决方案：**

1. **access_key_id或access_password错误**
   - 向LabArchives管理员核实凭证
   - 检查配置文件中的拼写错误或多余空格

2. **外部应用专用密码错误**
   - 确认使用的是外部应用专用密码而非常规登录密码
   - 在账户设置中重新生成外部应用专用密码

3. **API访问未启用**
   - 联系管理员为您的账户启用API访问
   - 确认机构持有企业版许可证

4. **区域端点错误**
   - 确认api_url与机构LabArchives实例匹配
   - 检查是否使用.com/.auapi/.ukapi域名

### 403禁止访问错误

**可能原因及解决方案：**

1. **权限不足**
   - 确认账户角色具备必要权限
   - 检查是否拥有特定笔记本(nbid)的访问权

2. **账户停用或过期**
   - 联系管理员确认账户状态

### 网络连接问题

**防火墙/代理配置：**

若机构使用防火墙或代理：

```python
import requests

# 配置代理
proxies = {
    'http': 'http://proxy.university.edu:8080',
    'https': 'http://proxy.university.edu:8080'
}

# 通过代理发起请求
response = requests.get(url, params=params, proxies=proxies)
```

**SSL证书验证：**

自签名证书处理方案（生产环境不推荐）：

```python
# 禁用SSL验证（仅限测试）
response = requests.get(url, params=params, verify=False)
```

## 安全最佳实践

1. **切勿将凭证提交至版本控制系统**
   - 将`config.yaml`加入`.gitignore`
   - 使用环境变量或密钥管理系统

2. **定期轮换凭证**
   - 每90天更换外部应用专用密码
   - 每年重新生成API密钥

3. **遵循最小权限原则**
   - 仅申请必要API权限
   - 为不同应用创建独立API凭证

4. **监控API使用**
   - 定期审查API访问日志
   - 设置异常活动告警

5. **安全存储**
   - 静态配置文件加密
   - 使用系统密钥链或密钥管理工具（如AWS Secrets Manager/Azure Key Vault）

## 认证测试

使用以下脚本验证认证配置：

```python
#!/usr/bin/env python3
"""测试LabArchives API认证"""

from labarchivespy.client import Client
import yaml
import sys

def test_authentication():
    try:
        # 加载配置
        with open('config.yaml', 'r') as f:
            config = yaml.safe_load(f)

        print("配置加载成功")
        print(f"API地址: {config['api_url']}")

        # 初始化客户端
        client = Client(
            config['api_url'],
            config['access_key_id'],
            config['access_password']
        )
        print("客户端初始化完成")

        # 测试认证
        login_params = {
            'login_or_email': config['user_email'],
            'password': config['user_external_password']
        }
        response = client.make_call('users', 'user_access_info', params=login_params)

        if response.status_code == 200:
            print("✅ 认证成功！")

            # 提取UID
            import xml.etree.ElementTree as ET
            uid = ET.fromstring(response.content)[0].text
            print(f"用户ID: {uid}")

            # 获取用户信息
            user_response = client.make_call('users', 'user_info_via_id', params={'uid': uid})
            print("✅ 用户信息获取成功")

            return True
        else:
            print(f"❌ 认证失败: {response.status_code}")
            print(response.content.decode('utf-8'))
            return False

    except Exception as e:
        print(f"❌ 错误: {str(e)}")
        import traceback
        traceback.print_exc()
        return False

if __name__ == '__main__':
    success = test_authentication()
    sys.exit(0 if success else 1)
```

运行脚本验证配置：

```bash
python3 test_auth.py
```

## 获取帮助

若认证问题持续存在：

1. 联系机构LabArchives管理员
2. 发送邮件至LabArchives支持：support@labarchives.com
3. 邮件需包含：
   - 机构名称
   - LabArchives账户邮箱
   - 错误信息及响应代码
   - 使用的区域端点
   - 编程语言及库版本信息
