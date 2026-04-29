<!-- Source: https://github.com/SuperiorByteWorks-LLC/agent-project | License: Apache-2.0 | Author: Clayton Young / Superior Byte Works, LLC (Boreal Bytes) -->

# 时序图

> **返回[样式指南](../mermaid_style_guide.md)** — 请先阅读样式指南了解表情符号、颜色和无障碍规则。

**语法关键词：** `sequenceDiagram`  
**最佳适用场景：** API交互、时序流程、多角色通信、请求/响应模式  
**不适用场景：** 简单线性流程（使用[流程图](flowchart.md)）、静态关系（使用[类图](class.md)或[实体关系图](er.md)）

---

## 示例图

```mermaid
sequenceDiagram
    accTitle: OAuth 2.0 授权码流程
    accDescr: 用户浏览器、应用服务器与身份提供方之间的OAuth流程，展示令牌交换和错误路径

    participant U as 👤 用户浏览器
    participant A as 🖥️ 应用服务器
    participant I as 🔐 身份提供方

    U->>A: 点击登录
    A-->>U: 重定向至身份提供方

    U->>I: 输入凭证
    I->>I: 🔍 验证凭证

    alt ✅ 凭证有效
        I-->>U: 携带授权码重定向
        U->>A: 发送授权码
        A->>I: 兑换令牌
        I-->>A: 🔐 访问令牌+刷新令牌
        A-->>U: ✅ 设置会话Cookie
        Note over U,A: 🔒 用户完成认证
    else ❌ 凭证无效
        I-->>U: ⚠️ 显示错误信息
    end
```

---

## 技巧

- 限制在**4-5个参与者** — 过多会导致可读性下降
- 实线箭头(`->>`)表示请求，虚线(`-->>`)表示响应
- 使用`alt/else/end`处理条件分支
- 使用`Note over X,Y:`添加带表情符号的上下文注释
- 使用`par/end`表示并行操作
- 使用`loop/end`表示重复交互
- **消息文本**中的表情符号能清晰标识状态(✅, ❌, ⚠️, 🔐)

## 常用模式

**并行调用：**

```
par 📥 获取用户
    A->>B: GET /user
and 📥 获取订单
    A->>C: GET /orders
end
```

**循环：**

```
loop ⏰ 每30秒
    A->>B: 健康检查
    B-->>A: ✅ 200 OK
end
```

---

## 模板

```mermaid
sequenceDiagram
    accTitle: 此处填写标题
    accDescr: 描述参与者间的交互及流程演示内容

    participant A as 👤 操作者
    participant B as 🖥️ 系统
    participant C as 💾 数据库

    A->>B: 📤 请求操作
    B->>C: 🔍 查询数据
    C-->>B: 📥 返回结果
    B-->>A: ✅ 交付响应
```

---

## 复杂示例

包含6个参与者的微服务结算流程，使用`box`区域分组。展示并行调用、条件分支、`break`错误处理、重试逻辑和上下文注释——复杂时序图的完整工具集。

```mermaid
sequenceDiagram
    accTitle: 微服务结算流程
    accDescr: 多服务结算时序，展示库存与支付的并行处理、错误重试恢复机制，以及客户端/网关/后端服务层的异步通知分发

    box rgb(237,233,254) 🌐 客户端层
        participant browser as 👤 浏览器
    end

    box rgb(219,234,254) 🖥️ API层
        participant gw as 🌐 API网关
        participant order as 📋 订单服务
    end

    box rgb(220,252,231) ⚙️ 后端服务
        participant inventory as 📦 库存服务
        participant payment as 💰 支付服务
        participant notify as 📤 通知服务
    end

    browser->>gw: 🛒 提交结算
    gw->>gw: 🔐 验证JWT令牌
    gw->>order: 📋 创建订单

    Note over order: 📊 订单状态: 待处理

    par ⚡ 并行验证
        order->>inventory: 📦 预留商品
        inventory-->>order: ✅ 商品预留成功
    and
        order->>payment: 💰 预授权卡片
        payment-->>order: ✅ 支付预授权成功
    end

    alt ✅ 全部成功
        order->>payment: 💰 扣款操作
        payment-->>order: ✅ 扣款成功
        order->>inventory: 📦 确认预留

        Note over order: 📊 订单状态: 已确认

        par 📤 异步通知
            order->>notify: 📧 发送确认邮件
        and
            order->>notify: 📱 发送推送通知
        end

        order-->>gw: ✅ 订单确认
        gw-->>browser: ✅ 显示确认页面

    else ❌ 库存不足
        order->>payment: 🔄 取消预授权
        order-->>gw: ⚠️ 商品缺货
        gw-->>browser: ⚠️ 显示库存错误

    else ❌ 支付拒绝
        order->>inventory: 🔄 释放预留

        loop 🔄 最多重试2次
            order->>payment: 💰 重试预授权
            payment-->>order: ❌ 仍被拒绝
        end

        order-->>gw: ❌ 支付失败
        gw-->>browser: ❌ 显示支付错误
    end
```

### 设计优势

- **`box`分组**按架构层聚合参与者——读者立即区分客户端与后端服务
- **`par`区块**展示库存与支付检查的并行处理，符合实际结算系统的高性能需求
- **嵌套`alt/else`**覆盖成功路径和两种独立失败场景，均含清理操作（取消预授权/释放预留）
- **`loop`重试逻辑**在不干扰主流程前提下展示支付重试机制
- **消息中的表情符号**提升可读性——📦代表库存，💰代表支付，✅/❌标识结果
