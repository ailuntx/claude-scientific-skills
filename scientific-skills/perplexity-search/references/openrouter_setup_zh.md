# OpenRouter 设置指南

完整指南：设置并使用 OpenRouter 访问 Perplexity 模型。

## 什么是 OpenRouter？

OpenRouter 是一个统一的 API 网关，通过单一 API 接口提供对 100+ 个来自不同供应商的 AI 模型的访问。它提供：

- **单一 API 密钥**：用一个密钥访问多个模型
- **统一格式**：兼容 OpenAI 的 API 格式
- **成本追踪**：内置使用监控和计费功能
- **模型路由**：智能回退和负载均衡
- **按量付费**：无需订阅，仅按实际使用付费

对于 Perplexity 模型，OpenRouter 独家提供特定模型（如 Sonar Pro Search）的访问权限。

## 快速入门

### 步骤 1：创建 OpenRouter 账户

1. 访问 https://openrouter.ai/
2. 点击右上角 "Sign Up"
3. 通过 Google、GitHub 或邮箱注册
4. 邮箱注册需验证邮箱

### 步骤 2：添加支付方式

OpenRouter 采用按量计费：

1. 访问 https://openrouter.ai/account
2. 点击 "Credits" 标签页
3. 添加支付方式（信用卡）
4. 充值初始额度（建议至少 $5）
5. 可选设置自动充值

**定价说明：**
- 不同模型按 token 计费标准不同
- Perplexity 定价见 https://openrouter.ai/perplexity
- 使用监控见 https://openrouter.ai/activity

### 步骤 3：生成 API 密钥

1. 访问 https://openrouter.ai/keys
2. 点击 "Create Key"
3. 输入描述性名称（如 "perplexity-search-skill"）
4. 可选设置使用限额
5. 复制密钥（以 `sk-or-v1-...` 开头）
6. **重要**：安全保存密钥 - 后续无法再次查看！

**安全提示：**
- 切勿公开分享 API 密钥
- 不要将密钥提交到版本控制系统
- 不同项目使用独立密钥
- 设置使用限额防止意外扣费
- 定期轮换密钥

### 步骤 4：配置环境

两种 API 密钥设置方式：

#### 选项 A：环境变量（推荐）

**Linux/macOS：**
```bash
export OPENROUTER_API_KEY='sk-or-v1-your-key-here'
```

永久生效配置：
```bash
# bash 用户：添加到 ~/.bashrc 或 ~/.bash_profile
echo 'export OPENROUTER_API_KEY="sk-or-v1-your-key-here"' >> ~/.bashrc
source ~/.bashrc

# zsh 用户：添加到 ~/.zshrc
echo 'export OPENROUTER_API_KEY="sk-or-v1-your-key-here"' >> ~/.zshrc
source ~/.zshrc
```

**Windows (PowerShell)：**
```powershell
$env:OPENROUTER_API_KEY = "sk-or-v1-your-key-here"
```

永久生效配置：
```powershell
[System.Environment]::SetEnvironmentVariable('OPENROUTER_API_KEY', 'sk-or-v1-your-key-here', 'User')
```

#### 选项 B：.env 文件

在项目目录创建 `.env` 文件：

```bash
# 创建 .env 文件
cat > .env << EOF
OPENROUTER_API_KEY=sk-or-v1-your-key-here
EOF
```

或使用安装脚本：
```bash
python scripts/setup_env.py --api-key sk-or-v1-your-key-here
```

运行脚本前加载环境变量：
```bash
# 从 .env 加载环境变量
source .env

# 或使用 python-dotenv
pip install python-dotenv
```

**在脚本中使用 python-dotenv：**
```python
from dotenv import load_dotenv
load_dotenv()  # 自动加载 .env 文件

import os
api_key = os.environ.get("OPENROUTER_API_KEY")
```

### 步骤 5：安装依赖

使用 uv 安装 LiteLLM：

```bash
uv pip install litellm
```

或使用常规 pip：
```bash
pip install litellm
```

**可选依赖：**
```bash
# 支持 .env 文件
uv pip install python-dotenv

# 附加功能支持
uv pip install litellm[proxy]  # 使用 LiteLLM 代理服务器时
```

### 步骤 6：验证配置

测试配置状态：

```bash
# 使用安装脚本验证
python scripts/setup_env.py --validate

# 或使用搜索脚本验证
python scripts/perplexity_search.py --check-setup
```

预期输出：
```
✓ OPENROUTER_API_KEY 已设置 (sk-or-v1-...xxxx)
✓ LiteLLM 已安装 (版本 X.X.X)
✓ 配置完成！可开始使用 Perplexity 搜索。
```

### 步骤 7：首次搜索测试

运行简单测试查询：

```bash
python scripts/perplexity_search.py "什么是 CRISPR 基因编辑？"
```

预期输出：
```
================================================================================
答案
================================================================================
CRISPR（成簇规律间隔短回文重复序列）是一项革命性基因编辑技术，
允许对 DNA 进行精确修改...
[详细答案继续]
================================================================================
```

## 使用监控

### 查看使用情况

监控 OpenRouter 使用量和成本：

1. 访问 https://openrouter.ai/activity
2. 查看请求数、token 量和费用
3. 按日期范围、模型或密钥筛选
4. 导出使用数据进行分析

### 设置使用限额

防止意外扣费：

1. 访问 https://openrouter.ai/keys
2. 点击目标密钥
3. 设置"速率限制"（每分钟请求数）
4. 设置"消费限额"（最高总消费额）
5. 可选启用带限额的"自动充值"

**开发环境推荐限额：**
- 速率限制：10-20 次请求/分钟
- 消费限额：$10-50（根据使用情况调整）

### 成本优化

降低费用技巧：

1. **选择合适的模型**：简单查询用 Sonar，而非 Sonar Pro Search
2. **设置 max_tokens**：通过 `--max-tokens` 参数限制响应长度
3. **批量查询**：合并多个简单问题
4. **监控使用**：高强度开发期间每日检查成本
5. **使用缓存**：存储重复查询结果

## 故障排除

### 错误："未配置 OpenRouter API 密钥"

**原因**：环境变量未设置

**解决方案**：
```bash
# 检查变量是否设置
echo $OPENROUTER_API_KEY

# 若为空则设置
export OPENROUTER_API_KEY='sk-or-v1-your-key-here'

# 或使用安装脚本
python scripts/setup_env.py --api-key sk-or-v1-your-key-here
```

### 错误："无效 API 密钥"

**可能原因**：
- 密钥已删除或吊销
- 密钥已过期
- 密钥输入错误
- 密钥格式错误

**解决方案**：
1. 在 https://openrouter.ai/keys 验证密钥
2. 检查多余空格或引号
3. 必要时生成新密钥
4. 确保密钥以 `sk-or-v1-` 开头

### 错误："额度不足"

**原因**：OpenRouter 账户余额不足

**解决方案**：
1. 访问 https://openrouter.ai/account
2. 点击 "Credits" 标签页
3. 充值额度
4. 考虑启用自动充值

### 错误："超出速率限制"

**原因**：短时间内请求过多

**解决方案**：
1. 等待数秒后重试
2. 在 https://openrouter.ai/keys 提高速率限制
3. 在代码中实现指数退避
4. 批量请求或降低频率

### 错误："找不到模型"

**原因**：模型名称错误或模型已停用

**解决方案**：
1. 在 https://openrouter.ai/models 查看可用模型
2. 使用正确格式：`openrouter/perplexity/sonar-pro`
3. 确认模型仍受支持

### 错误："未安装 LiteLLM"

**原因**：未安装 LiteLLM 包

**解决方案**：
```bash
uv pip install litellm
```

### LiteLLM 导入错误

**可能原因**：Python 路径问题或版本冲突

**解决方案**：
1. 验证安装：`pip list | grep litellm`
2. 重新安装：`uv pip install --force-reinstall litellm`
3. 检查 Python 版本：`python --version` (需 3.8+)
4. 使用虚拟环境避免冲突

## 高级配置

### 使用多密钥

适用于不同项目或团队成员：

```bash
# 项目 1
export OPENROUTER_API_KEY='sk-or-v1-project1-key'

# 项目 2
export OPENROUTER_API_KEY='sk-or-v1-project2-key'
```

或在不同目录使用独立的 .env 文件。

### 自定义基础 URL

使用 OpenRouter 代理或自定义端点时：

```python
from litellm import completion

response = completion(
    model="openrouter/perplexity/sonar-pro",
    messages=[{"role": "user", "content": "query"}],
    api_base="https://custom-endpoint.com/v1"  # 自定义 URL
)
```

### 请求头配置

添加跟踪用自定义头信息：

```python
from litellm import completion

response = completion(
    model="openrouter/perplexity/sonar-pro",
    messages=[{"role": "user", "content": "query"}],
    extra_headers={
        "HTTP-Referer": "https://your-app.com",
        "X-Title": "Your App Name"
    }
)
```

### 超时配置

为长时间查询设置超时：

```python
from litellm import completion

response = completion(
    model="openrouter/perplexity/sonar-pro-search",
    messages=[{"role": "user", "content": "complex query"}],
    timeout=120  # 120 秒超时
)
```

## 安全最佳实践

### API 密钥管理

1. **永不提交密钥**：将 `.env` 加入 `.gitignore`
2. **定期轮换密钥**：每 3-6 个月轮换
3. **密钥分离**：开发/预发/生产环境使用不同密钥
4. **监控使用**：检查未授权访问
5. **设置限额**：配置消费和速率限制

### .gitignore 模板

添加到 `.gitignore`：
```
# 环境变量
.env
.env.local
.env.*.local

# API 密钥
*api_key*
*apikey*
*.key

# 敏感配置
config/secrets.yaml
```

### 密钥吊销

密钥泄露时：

1. 立即访问 https://openrouter.ai/keys
2. 点击泄露密钥的 "Delete"
3. 生成新密钥
4. 更新所有使用旧密钥的应用
5. 审查使用日志确认未授权访问
6. 必要时联系 OpenRouter 支持

## 常见问题

**问：通过 OpenRouter 使用 Perplexity 的费用是多少？**

答：不同模型价格不同。Sonar 最便宜（约 $0.001-0.002/次查询），Sonar Pro 中等（约 $0.002-0.005），Sonar Pro Search 最贵（约 $0.02-0.05+/次查询）。具体定价见 https://openrouter.ai/perplexity。

**问：需要单独的 Perplexity API 密钥吗？**

答：不需要！OpenRouter 仅需您的 OpenRouter 密钥即可访问 Perplexity 模型。

**问：OpenRouter 能用于 Perplexity 之外的其他模型吗？**

答：可以！OpenRouter 通过同一 API 密钥提供对 100+ 个模型的访问，包括来自 OpenAI、Anthropic、Google、Meta 等的模型。

**问：是否有免费额度？**

答：OpenRouter 需付费使用，但定价极具竞争力。初始 $5 额度可满足大量测试需求。

**问：如何注销 OpenRouter 账户？**

答：联系 OpenRouter 支持。注意未使用额度可能无法退款。

**问：能否在生产应用中使用 OpenRouter？**

答：可以，OpenRouter 专为生产环境设计，提供稳健基础设施、SLA 协议和企业级支持。

## 资源

**官方文档：**
- OpenRouter：https://openrouter.ai/docs
- Perplexity 模型：https://openrouter.ai/perplexity
- LiteLLM：https://docs.litellm.ai/

**账户管理：**
- 控制台：https://openrouter.ai/account
- API 密钥：https://openrouter.ai/keys
- 使用情况：https://openrouter.ai/activity
- 账单：https://openrouter.ai/credits

**社区：**
- OpenRouter Discord：https://discord.gg/openrouter
- GitHub Issues：https://github.com/OpenRouter
- LiteLLM GitHub：https://github.com/BerriAI/litellm

## 总结

配置 OpenRouter 访问 Perplexity 的步骤：

1. 在 https://openrouter.ai 创建账户
2. 添加支付方式并充值
3. 在 https://openrouter.ai/keys 生成 API 密钥
4. 设置 `OPENROUTER_API_KEY` 环境变量
5. 安装 LiteLLM：`uv pip install litellm`
6. 验证配置：`python scripts/setup_env.py --validate`
7. 开始搜索：`python scripts/perplexity_search.py "您的查询"`

定期监控使用情况和成本，优化支出并确保安全。
