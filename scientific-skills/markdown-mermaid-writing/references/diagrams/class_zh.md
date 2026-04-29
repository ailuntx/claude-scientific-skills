<!-- Source: https://github.com/SuperiorByteWorks-LLC/agent-project | License: Apache-2.0 | Author: Clayton Young / Superior Byte Works, LLC (Boreal Bytes) -->

# 类图

> **返回[样式指南](../mermaid_style_guide.md)** — 请先阅读样式指南了解表情符号、颜色和无障碍规则。

**语法关键字：** `classDiagram`  
**最佳适用场景：** 面向对象设计、类型层次结构、接口契约、领域模型  
**不适用场景：** 数据库模式（请使用[ER图](er.md)）、运行时行为（请使用[序列图](sequence.md)）

---

## 示例图

```mermaid
classDiagram
    accTitle: 支付处理类层次结构
    accDescr: 包含两个具体实现（信用卡与数字钱包）的接口和抽象基类

    class PaymentProcessor {
        <<interface>>
        +processPayment(amount) bool
        +refund(transactionId) bool
        +getStatus(transactionId) string
    }

    class BaseProcessor {
        <<abstract>>
        #apiKey: string
        #timeout: int
        +validateAmount(amount) bool
        #logTransaction(tx) void
    }

    class CreditCardProcessor {
        -gateway: string
        +processPayment(amount) bool
        +refund(transactionId) bool
        -tokenizeCard(card) string
    }

    class DigitalWalletProcessor {
        -provider: string
        +processPayment(amount) bool
        +refund(transactionId) bool
        -initiateHandshake() void
    }

    PaymentProcessor <|.. BaseProcessor : 实现
    BaseProcessor <|-- CreditCardProcessor : 继承
    BaseProcessor <|-- DigitalWalletProcessor : 继承

    style PaymentProcessor fill:#ede9fe,stroke:#7c3aed,stroke-width:2px,color:#3b0764
    style BaseProcessor fill:#dbeafe,stroke:#2563eb,stroke-width:2px,color:#1e3a5f
    style CreditCardProcessor fill:#dcfce7,stroke:#16a34a,stroke-width:2px,color:#14532d
    style DigitalWalletProcessor fill:#dcfce7,stroke:#16a34a,stroke-width:2px,color:#14532d
```

---

## 技巧

- 使用 `<<interface>>` 和 `<<abstract>>` 构造型增强清晰度
- 显示可见性：`+` 公有，`-` 私有，`#` 受保护
- 每张图保持 **4–6 个类** — 大型层次结构请拆分
- 使用 `style ClassName fill:...,stroke:...,color:...` 实现轻量语义着色：
  - 🟣 紫色表示接口/抽象
  - 🔵 蓝色表示基类/抽象类
  - 🟢 绿色表示具体实现
- 关系箭头：
  - `<|--` 继承（extends）
  - `<|..` 实现（implements）
  - `*--` 组合 · `o--` 聚合 · `-->` 依赖

---

## 模板

```mermaid
classDiagram
    accTitle: 此处填写标题
    accDescr: 描述类层次结构及类型间关键关系

    class InterfaceName {
        <<interface>>
        +methodOne() ReturnType
        +methodTwo(param) ReturnType
    }

    class ConcreteClass {
        -privateField: Type
        +methodOne() ReturnType
        +methodTwo(param) ReturnType
    }

    InterfaceName <|.. ConcreteClass : 实现
```

---

## 复杂示例

包含11个类的事件驱动通知平台，按3个`namespace`分组（核心编排层、传输通道层、数据模型层）。展示跨层的接口实现、组合和依赖关系。

```mermaid
classDiagram
    accTitle: 事件驱动通知平台
    accDescr: 多命名空间的类层次结构，展示通知系统的核心编排、四种传输通道实现及包含组合/依赖关系的支持数据模型

    namespace Core {
        class NotificationService {
            -queue: NotificationQueue
            -registry: ChannelRegistry
            +dispatch(notification) bool
            +scheduleDelivery(notification, time) void
            +getDeliveryStatus(id) DeliveryStatus
        }

        class NotificationQueue {
            -pending: List~Notification~
            -maxRetries: int
            +enqueue(notification) void
            +dequeue() Notification
            +retry(attempt) bool
        }

        class ChannelRegistry {
            -channels: Map~string, Channel~
            +register(name, channel) void
            +resolve(type) Channel
            +healthCheck() Map~string, bool~
        }
    }

    namespace Channels {
        class Channel {
            <<interface>>
            +send(notification, recipient) DeliveryAttempt
            +getStatus(attemptId) DeliveryStatus
            +validateRecipient(recipient) bool
        }

        class EmailChannel {
            -smtpHost: string
            -templateEngine: TemplateEngine
            +send(notification, recipient) DeliveryAttempt
            +getStatus(attemptId) DeliveryStatus
            +validateRecipient(recipient) bool
        }

        class SMSChannel {
            -provider: string
            -rateLimit: int
            +send(notification, recipient) DeliveryAttempt
            +getStatus(attemptId) DeliveryStatus
            +validateRecipient(recipient) bool
        }

        class PushChannel {
            -firebaseKey: string
            -apnsKey: string
            +send(notification, recipient) DeliveryAttempt
            +getStatus(attemptId) DeliveryStatus
            +validateRecipient(recipient) bool
        }

        class WebhookChannel {
            -signingSecret: string
            -timeout: int
            +send(notification, recipient) DeliveryAttempt
            +getStatus(attemptId) DeliveryStatus
            +validateRecipient(recipient) bool
        }
    }

    namespace Models {
        class Notification {
            +id: uuid
            +channel: string
            +subject: string
            +body: string
            +priority: string
            +createdAt: timestamp
        }

        class Recipient {
            +id: uuid
            +email: string
            +phone: string
            +deviceTokens: List~string~
            +preferences: Map~string, bool~
        }

        class DeliveryAttempt {
            +id: uuid
            +notificationId: uuid
            +recipientId: uuid
            +status: DeliveryStatus
            +attemptNumber: int
            +sentAt: timestamp
        }

        class DeliveryStatus {
            <<enumeration>>
            QUEUED
            SENDING
            DELIVERED
            FAILED
            BOUNCED
        }
    }

    NotificationService *-- NotificationQueue : 包含
    NotificationService *-- ChannelRegistry : 包含
    ChannelRegistry --> Channel : 解析

    Channel <|.. EmailChannel : 实现
    Channel <|.. SMSChannel : 实现
    Channel <|.. PushChannel : 实现
    Channel <|.. WebhookChannel : 实现

    Channel ..> Notification : 接收
    Channel ..> Recipient : 投递至
    Channel ..> DeliveryAttempt : 生成

    DeliveryAttempt --> DeliveryStatus : 包含状态

    style Channel fill:#ede9fe,stroke:#7c3aed,stroke-width:2px,color:#3b0764
    style DeliveryStatus fill:#ede9fe,stroke:#7c3aed,stroke-width:2px,color:#3b0764
    style NotificationService fill:#dbeafe,stroke:#2563eb,stroke-width:2px,color:#1e3a5f
    style NotificationQueue fill:#dbeafe,stroke:#2563eb,stroke-width:2px,color:#1e3a5f
    style ChannelRegistry fill:#dbeafe,stroke:#2563eb,stroke-width:2px,color:#1e3a5f
    style EmailChannel fill:#dcfce7,stroke:#16a34a,stroke-width:2px,color:#14532d
    style SMSChannel fill:#dcfce7,stroke:#16a34a,stroke-width:2px,color:#14532d
    style PushChannel fill:#dcfce7,stroke:#16a34a,stroke-width:2px,color:#14532d
    style WebhookChannel fill:#dcfce7,stroke:#16a34a,stroke-width:2px,color:#14532d
    style Notification fill:#f3f4f6,stroke:#6b7280,stroke-width:2px,color:#1f2937
    style Recipient fill:#f3f4f6,stroke:#6b7280,stroke-width:2px,color:#1f2937
    style DeliveryAttempt fill:#f3f4f6,stroke:#6b7280,stroke-width:2px,color:#1f2937
```

### 设计解析

- **三层命名空间对应架构层级** — 核心层（编排）、通道层（传输实现）、模型层（数据）。开发者可独立阅读各命名空间
- **颜色编码角色** — 紫色接口/枚举、蓝色核心服务、绿色具体实现、灰色数据模型。模式一目了然
- **关系类型精准定义** — 组合（`*--`）表示"拥有并管理"、实现（`<|..`）表示"履行契约"、依赖（`..>`）表示"运行时使用"。每种箭头均有语义含义
