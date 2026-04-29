<!-- Source: https://github.com/SuperiorByteWorks-LLC/agent-project | License: Apache-2.0 | Author: Clayton Young / Superior Byte Works, LLC (Boreal Bytes) -->

# 构建复杂图表体系

> **返回[样式指南](../mermaid_style_guide.md)** — 本文档介绍如何组合多种图表类型来全面记录复杂系统。

**目的：** 单一图表仅能呈现单一视角。实际文档常需多种图表协同工作——概览流程图链接详细时序图，ER模型搭配展示实体生命周期的状态机，甘特图辅以架构前后对比视图。本文档将指导您何时及如何组合图表以实现最佳清晰度。

---

## 何时组合多张图表

| 记录对象                | 图表组合                                                                 | 优势分析                                                                         |
| ----------------------- | ------------------------------------------------------------------------ | -------------------------------------------------------------------------------- |
| 完整系统架构            | C4上下文图 + 架构图 + 核心流程时序图                                     | 面向利益相关者的上下文，面向运维的基础设施，面向开发者的交互流程                 |
| API设计文档             | ER模型（数据结构） + 请求流时序图 + 实体生命周期状态图                   | 数据库团队需结构模型，后端需交互逻辑，业务逻辑需状态流转                         |
| 功能规范                | 主流程流程图 + 服务交互时序图 + 用户体验旅程图                           | 产品经理看流程，工程师看实现，设计师看体验                                       |
| 迁移项目                | 甘特图（时间线） + 架构对比图（前后状态） + 迁移过程流程图               | 管理层看进度，基础设施团队看拓扑，迁移团队看步骤                                 |
| 入职文档                | 用户旅程图 + 设置步骤流程图 + 首次API调用时序图                          | 产品团队看体验地图，新员工看检查清单，开发者看技术指引                           |
| 事件响应                | 告警生命周期状态图 + 升级流程时序图 + 决策树流程图                       | 值班人员跟踪状态，管理层看沟通流程，响应人员按决策树处置                         |

---

## 模式一：概览图+细节图

**适用场景：** 需同时呈现全局框架与具体细节。管理层查看概览，工程师深入细节。

概览图展示高层级阶段或组件，细节图聚焦特定阶段呈现内部交互。

### 概览图——发布流水线

```mermaid
flowchart LR
    accTitle: 发布流水线概览
    accDescr: 四阶段发布流水线：从代码提交经构建、预发布到生产环境部署

    subgraph source ["📥 源码"]
        commit[📝 代码提交] --> pr_review[🔍 PR审核]
    end

    subgraph build ["🔧 构建"]
        compile[⚙️ 编译] --> test[🧪 测试套件]
        test --> scan[🔐 安全扫描]
    end

    subgraph staging ["🚀 预发布"]
        deploy_stg[☁️ 部署预发布] --> smoke[🧪 冒烟测试]
        smoke --> approval{👤 审批通过?}
    end

    subgraph production ["✅ 生产环境"]
        canary[🚀 金丝雀发布 **5%**] --> rollout[🚀 全量**发布**]
        rollout --> monitor[📊 监控指标]
    end

    source --> build
    build --> staging
    approval -->|是| production
    approval -->|否| source

    classDef phase_start fill:#dbeafe,stroke:#2563eb,stroke-width:2px,color:#1e3a5f
    classDef phase_test fill:#fef9c3,stroke:#ca8a04,stroke-width:2px,color:#713f12
    classDef phase_deploy fill:#dcfce7,stroke:#16a34a,stroke-width:2px,color:#14532d

    class commit,pr_review,compile phase_start
    class test,scan,smoke,approval phase_test
    class deploy_stg,canary,rollout,monitor phase_deploy
```

_生产部署阶段涉及多服务交互，金丝雀发布流程详见下方时序图。_

### 细节图——金丝雀发布时序

```mermaid
sequenceDiagram
    accTitle: 金丝雀发布服务交互时序
    accDescr: 详细展示CI服务器如何通过容器仓库、Kubernetes集群和监控栈协调金丝雀发布，并在失败时自动回滚

    participant ci as ⚙️ CI服务器
    participant registry as 📦 容器仓库
    participant k8s as ☁️ Kubernetes
    participant monitor as 📊 监控系统
    participant oncall as 👤 值班工程师

    ci->>registry: 📤 推送标记镜像
    registry-->>ci: ✅ 镜像已存储

    ci->>k8s: 🚀 部署金丝雀（5%流量）
    k8s-->>ci: ✅ 金丝雀Pod运行中

    ci->>monitor: 📊 启动金丝雀分析
    Note over monitor: ⏰ 观察15分钟

    loop 📊 每60秒
        monitor->>k8s: 🔍 查询错误率
        k8s-->>monitor: 📊 返回指标
    end

    alt ✅ 错误率低于阈值
        monitor-->>ci: ✅ 金丝雀健康
        ci->>k8s: 🚀 提升至100%
        k8s-->>ci: ✅ 全量发布完成
        ci->>monitor: 📊 持续监控
    else ❌ 错误率高于阈值
        monitor-->>ci: ❌ 金丝雀异常
        ci->>k8s: 🔄 回滚至上一版本
        k8s-->>ci: ✅ 回滚完成
        ci->>oncall: ⚠️ 告警：金丝雀失败
        Note over oncall: 📋 根因分析
    end
```

### 衔接方式

- **概览流程图**展示子图间连接的全流程——管理层借此理解发布过程
- **细节时序图**聚焦生产环境子图中的"金丝雀5%→全量发布"，呈现工程师需调试的实际服务交互
- **命名一致性**——"金丝雀"和"监控指标"在两张图表中统一出现，建立清晰的衔接桥梁

---

## 模式二：多视角文档

**适用场景：** 同一系统需为不同受众（数据库团队、后端工程师、产品经理）提供差异化视图。

本例从三个视角记录**用户认证**功能。

### 数据模型——面向数据库团队

```mermaid
erDiagram
    accTitle: 认证数据模型
    accDescr: 五实体认证模型：用户、会话、刷新令牌、登录尝试、MFA设备及其基数关系

    USER ||--o{ SESSION : "拥有"
    USER ||--o{ REFRESH_TOKEN : "持有"
    USER ||--o{ LOGIN_ATTEMPT : "产生"
    USER ||--o{ MFA_DEVICE : "注册"
    SESSION ||--|| REFRESH_TOKEN : "配对"

    USER {
        uuid id PK "🔑 主键"
        string email "📧 唯一登录名"
        string password_hash "🔐 Bcrypt哈希"
        boolean mfa_enabled "🔒 MFA启用标志"
        timestamp last_login "⏰ 最后登录时间"
    }

    SESSION {
        uuid id PK "🔑 主键"
        uuid user_id FK "👤 会话所有者"
        string ip_address "🌐 客户端IP"
        string user_agent "📋 浏览器信息"
        timestamp expires_at "⏰ 过期时间"
    }

    REFRESH_TOKEN {
        uuid id PK "🔑 主键"
        uuid user_id FK "👤 令牌所有者"
        uuid session_id FK "🔗 配对话"
        string token_hash "🔐 哈希令牌"
        boolean revoked "❌ 吊销标志"
        timestamp expires_at "⏰ 过期时间"
    }

    LOGIN_ATTEMPT {
        uuid id PK "🔑 主键"
        uuid user_id FK "👤 尝试用户"
        string ip_address "🌐 来源IP"
        boolean success "✅ 结果"
        string failure_reason "⚠️ 失败原因"
        timestamp attempted_at "⏰ 尝试时间"
    }

    MFA_DEVICE {
        uuid id PK "🔑 主键"
        uuid user_id FK "👤 设备所有者"
        string device_type "📱 TOTP或WebAuthn"
        string secret_hash "🔐 加密密钥"
        boolean verified "✅ 设置完成"
        timestamp registered_at "⏰ 注册时间"
    }
```

### 认证流程——面向后端团队

```mermaid
sequenceDiagram
    accTitle: 带MFA的登录流程
    accDescr: 分步认证时序：浏览器、API网关、认证服务与数据库间的凭证验证、条件式MFA挑战、令牌签发及异常处理

    participant B as 👤 浏览器
    participant API as 🌐 API网关
    participant Auth as 🔐 认证服务
    participant DB as 💾 数据库

    B->>API: 📤 POST /login (邮箱,密码)
    API->>Auth: 🔐 验证凭证
    Auth->>DB: 🔍 按邮箱查询用户
    DB-->>Auth: 👤 用户记录

    Auth->>Auth: 🔐 验证密码哈希

    alt ❌ 密码无效
        Auth->>DB: 📝 记录失败尝试
        Auth-->>API: ❌ 401 未授权
        API-->>B: ❌ 凭证无效
    else ✅ 密码有效
        alt 🔒 MFA已启用
            Auth-->>API: ⚠️ 202 需MFA验证
            API-->>B: 📱 显示MFA提示

            B->>API: 📤 POST /login/mfa (验证码)
            API->>Auth: 🔐 验证MFA码
            Auth->>DB: 🔍 查询MFA设备
            DB-->>Auth: 📱 设备记录
            Auth->>Auth: 🔐 验证TOTP

            alt ❌ 验证码无效
                Auth-->>API: ❌ 401 验证码错误
                API-->>B: ❌ 请重试
            else ✅ 验证码有效
                Auth->>DB: 📝 创建会话+令牌
                Auth-->>API: ✅ 200 + 令牌
                API-->>B: ✅ 设置Cookie并跳转
            end
        else 🔓 未启用MFA
            Auth->>DB: 📝 创建会话+令牌
            Auth-->>API: ✅ 200 + 令牌
            API-->>B: ✅ 设置Cookie并跳转
        end
    end
```

### 登录体验——面向产品团队

```mermaid
journey
    accTitle: 登录体验旅程图
    accDescr: 仅密码用户与MFA用户在登录过程中的满意度评分，揭示多因素流程中的摩擦点

    title 👤 登录体验
    section 🔐 登录阶段
        访问登录页          : 4 : 用户
        输入邮箱密码        : 3 : 用户
        点击登录按钮        : 4 : 用户
    section 📱 MFA验证阶段
        收到MFA提示        : 3 : MFA用户
        打开验证器应用      : 2 : MFA用户
        输入6位验证码      : 2 : MFA用户
        处理过期验证码      : 1 : MFA用户
    section ✅ 登录后阶段
        进入仪表盘          : 5 : 用户
        查看个性化内容      : 5 : 用户
        恢复历史会话        : 4 : 用户
```

### 衔接方式

- **相同实体，差异视角**——"用户"、"会话"、"MFA设备"在ER图中为表结构，在时序图中为参与者/操作，在旅程图中为体验触点
- **各受众获取可操作信息**——数据库团队关注索引与基数，后端团队关注API契约与错误码，产品团队关注满意度与摩擦点
- **旅程图揭示时序图未显信息**——时序图中MFA是清晰的条件分支，但旅程图显示其UX评分最低（1-2分）。这驱动产品决策投资WebAuthn/通行密钥

---

## 模式三：架构前后对比

**适用场景：** 迁移文档需向利益相关者展示当前状态、目标状态及转型过程。

### 当前状态——单体架构

```mermaid
flowchart TB
    accTitle: 当前单体架构状态
    accDescr: 单一Rails单体处理所有流量，通过单服务器连接单数据库，呈现扩展瓶颈

    client([👤 全部流量]) --> mono[🖥️ Rails**单体**]
    mono --> db[(💾 单一PostgreSQL)]
    mono --> jobs[⏰ 后台**任务**]
    jobs --> db

    classDef bottleneck fill:#fee2e2,stroke:#dc2626,stroke-width:2px,color:#7f1d1d
    classDef neutral fill:#f3f4f6,stroke:#6b7280,stroke-width:2px,color:#1f2937

    class mono,db bottleneck
    class client,jobs neutral
```

> ⚠️ **问题：** 单一数据库成瓶颈。单体无法水平扩展。部署需全局重启。

### 目标状态——微服务架构

```mermaid
flowchart TB
    accTitle: 目标微服务架构状态
    accDescr: 解耦的微服务架构：API网关路由至独立服务，各服务拥有专属数据存储，通过共享消息队列异步通信

    client([👤 全部流量]) --> gw[🌐 API**网关**]

    subgraph services ["⚙️ 服务集群"]
        user_svc[👤 用户服务]
        order_svc[📋 订单服务]
        product_svc[📦 商品服务]
    end

    subgraph data ["💾 数据存储"]
        user_db[(💾 用户数据库)]
        order_db[(💾 订单数据库)]
        product_db[(💾 商品数据库)]
    end

    gw --> user_svc
    gw --> order_svc
    gw --> product_svc

    user_svc --> user_db
    order_svc --> order_db
    product_svc --> product_db

    order_svc --> mq[📥 消息队列]
    mq --> user_svc
    mq --> product_svc

    classDef gateway fill:#ede9fe,stroke:#7c3aed,stroke-width:2px,color:#3b0764
    classDef service fill:#dbeafe,stroke:#2563eb,stroke-width:2px,color:#1e3a5f
    classDef datastore fill:#dcfce7,stroke:#16a34a,stroke-width:2px,color:#14532d
    classDef infra fill:#fef9c3,stroke:#ca8a04,stroke-width:2px,color:#713f12

    class gw gateway
    class user_svc,order_svc,product_svc service
    class user_db,order_db,product_db datastore
    class mq infra
```

> ✅ **成效：** 各服务独立扩展。专属数据库消除共享瓶颈。异步消息解耦服务依赖。

### 衔接方式

- **相同布局，差异复杂度**——均采用`flowchart TB`布局使结构转型直观可见。单体架构4节点，目标架构11节点含子图
- **色彩叙事**——单体架构用红色（危险）标注瓶颈组件，目标架构用蓝/绿/紫展示健康分治组件
- **文字衔接图表**——⚠️问题标注与✅成效标注解释架构变更动因，而非仅展示变化

---

## 文档中的图表衔接实践

组合图表时遵循以下准则：

| 实践要点               | 示例                                                                                                 |
| ---------------------- | ---------------------------------------------------------------------------------------------------- |
| **标题锚点定位**       | `完整登录流程见[认证流程](#认证流程——面向后端团队)`                                                  |
| **引用特定节点**       | "概览图中的**API网关**连接至下文详述的服务"                                                          |
| **命名一致性**         | 相同实体在所有图表中名称统一（如"用户服务"，而非此处用"User Svc"，彼处用"Users API"）                |
| **相邻排布**           | 相关图表置于连续章节，避免分散                                                                       |
| **衔接说明**

classDef result fill:#dbeafe,stroke:#2563eb,stroke-width:2px,color:#1e3a5f
    classDef start_style fill:#ede9fe,stroke:#7c3aed,stroke-width:2px,color:#3b0764

    class audience,depth,change decision
    class perspectives,overview,before_after,single result
    class start start_style
