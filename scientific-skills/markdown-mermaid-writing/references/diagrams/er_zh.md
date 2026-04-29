<!-- 来源：https://github.com/SuperiorByteWorks-LLC/agent-project | 许可证：Apache-2.0 | 作者：Clayton Young / Superior Byte Works, LLC (Boreal Bytes) -->

# 实体关系（ER）图

> **返回[样式指南](../mermaid_style_guide.md)** — 请先阅读样式指南了解表情符号、颜色和无障碍规则。

**语法关键字：** `erDiagram`  
**最佳适用场景：** 数据库模式、数据模型、实体关系、API数据结构  
**不适用场景：** 包含方法的类层次结构（使用[类图](class.md)）、流程设计（使用[流程图](flowchart.md)）

---

## 示例图

```mermaid
erDiagram
    accTitle: 项目管理数据模型
    accDescr: 项目管理系统的实体关系图，展示团队、项目、任务、成员和评论的基数关系

    TEAM ||--o{ PROJECT : "拥有"
    PROJECT ||--o{ TASK : "包含"
    TASK ||--o{ COMMENT : "包含"
    TEAM ||--o{ MEMBER : "包含"
    MEMBER ||--o{ TASK : "分配给"
    MEMBER ||--o{ COMMENT : "撰写"

    TEAM {
        uuid id PK "🔑 主键"
        string name "👥 团队名称"
        string department "🏢 所属部门"
    }

    PROJECT {
        uuid id PK "🔑 主键"
        uuid team_id FK "🔗 团队引用"
        string title "📋 项目标题"
        string status "📊 当前状态"
        date deadline "⏰ 截止日期"
    }

    TASK {
        uuid id PK "🔑 主键"
        uuid project_id FK "🔗 项目引用"
        uuid assignee_id FK "👤 分配成员"
        string title "📝 任务标题"
        string priority "⚠️ 优先级"
        string status "📊 当前状态"
    }

    MEMBER {
        uuid id PK "🔑 主键"
        uuid team_id FK "🔗 团队引用"
        string name "👤 姓名"
        string email "📧 电子邮箱"
        string role "🏷️ 职位角色"
    }

    COMMENT {
        uuid id PK "🔑 主键"
        uuid task_id FK "🔗 任务引用"
        uuid author_id FK "👤 作者引用"
        text body "📝 评论内容"
        timestamp created_at "⏰ 创建时间"
    }
```

---

## 设计建议

- 包含数据类型、`PK`/`FK`标注及带表情符号的**注释文本**  
- 使用明确的动词短语关系标签：`"拥有"`、`"包含"`、`"分配给"`  
- 基数表示法：  
  - `||--o{` 一对多  
  - `||--||` 一对一  
  - `}o--o{` 多对多  
  - `o` = 零或多个，`|` = 严格一个  
- 每图限制 **5–7 个实体** — 大型模式按领域拆分  
- 实体命名：`大写字母`（遵循 SQL 惯例）  

---

## 模板

```mermaid
erDiagram
    accTitle: 此处填写标题
    accDescr: 描述数据模型及实体间关键关系

    ENTITY_A ||--o{ ENTITY_B : "拥有多个"
    ENTITY_B ||--|| ENTITY_C : "属于"

    ENTITY_A {
        uuid id PK "🔑 主键"
        string name "📝 显示名称"
    }

    ENTITY_B {
        uuid id PK "🔑 主键"
        uuid entity_a_id FK "🔗 引用"
        string value "📊 数值字段"
    }
```

---

## 复杂案例

包含 10 个实体的多租户 SaaS 平台模式，覆盖身份认证、计费订阅和安全审计三大领域。关系线完整呈现了从租户隔离到用户权限再到账单生成的基数约束。

```mermaid
erDiagram
    accTitle: SaaS多租户平台模式
    accDescr: 十实体数据模型，覆盖多租户SaaS平台的身份管理、基于角色的访问控制、订阅计费及审计日志，包含完整基数关系

    TENANT ||--o{ ORGANIZATION : "包含"
    ORGANIZATION ||--o{ USER : "雇佣"
    ORGANIZATION ||--|| SUBSCRIPTION : "持有"
    USER }o--o{ ROLE : "分配"
    ROLE ||--o{ PERMISSION : "授予"
    SUBSCRIPTION ||--|| PLAN : "订阅"
    SUBSCRIPTION ||--o{ INVOICE : "生成"
    USER ||--o{ AUDIT_LOG : "产生"
    TENANT ||--o{ AUDIT_LOG : "作用域"
    USER ||--o{ API_KEY : "拥有"

    TENANT {
        uuid id PK "🔑 主键"
        string name "🏢 租户名称"
        string subdomain "🌐 唯一子域名"
        string tier "🏷️ 服务层级"
        boolean active "✅ 激活状态"
        timestamp created_at "⏰ 创建时间"
    }

    ORGANIZATION {
        uuid id PK "🔑 主键"
        uuid tenant_id FK "🔗 租户引用"
        string name "👥 组织名称"
        string billing_email "📧 账单联系人"
        int seat_count "📊 许可席位数量"
    }

    USER {
        uuid id PK "🔑 主键"
        uuid org_id FK "🔗 组织引用"
        string email "📧 登录邮箱"
        string display_name "👤 显示名称"
        string status "📊 账户状态"
        timestamp last_login "⏰ 最后活跃时间"
    }

    ROLE {
        uuid id PK "🔑 主键"
        uuid tenant_id FK "🔗 租户作用域"
        string name "🏷️ 角色名称"
        string description "📝 角色用途"
        boolean system_role "🔒 内置标识"
    }

    PERMISSION {
        uuid id PK "🔑 主键"
        uuid role_id FK "🔗 角色引用"
        string resource "🎯 目标资源"
        string action "⚙️ 允许操作"
        string scope "🔒 权限范围"
    }

    PLAN {
        uuid id PK "🔑 主键"
        string name "🏷️ 方案名称"
        int price_cents "💰 月费（分）"
        int seat_limit "👥 最大席位"
        jsonb features "📋 功能开关"
        boolean active "✅ 可用标识"
    }

    SUBSCRIPTION {
        uuid id PK "🔑 主键"
        uuid org_id FK "🔗 组织引用"
        uuid plan_id FK "🔗 方案引用"
        string status "📊 订阅状态"
        date current_period_start "📅 周期起始"
        date current_period_end "📅 周期结束"
    }

    INVOICE {
        uuid id PK "🔑 主键"
        uuid subscription_id FK "🔗 订阅引用"
        int amount_cents "💰 总金额（分）"
        string currency "💱 货币代码"
        string status "📊 支付状态"
        timestamp issued_at "⏰ 开具日期"
    }

    AUDIT_LOG {
        uuid id PK "🔑 主键"
        uuid tenant_id FK "🔗 租户作用域"
        uuid user_id FK "👤 操作用户"
        string action "⚙️ 执行操作"
        string resource_type "🎯 目标类型"
        uuid resource_id "🔗 目标ID"
        jsonb metadata "📋 事件详情"
        timestamp created_at "⏰ 发生时间"
    }

    API_KEY {
        uuid id PK "🔑 主键"
        uuid user_id FK "👤 所有者"
        string prefix "🏷️ 密钥前缀"
        string hash "🔐 哈希密文"
        string name "📝 密钥名称"
        timestamp expires_at "⏰ 过期时间"
        boolean revoked "❌ 吊销标识"
    }
```

### 设计优势

- **10 个实体按领域组织** — 身份（租户、组织、用户、角色、权限）、计费（方案、订阅、账单）、安全（审计日志、API密钥）。关系线自然聚类相关实体  
- **完整基数体现业务规则** — `||--||`（一对一）表示组织-订阅关系（每组织单订阅）。`}o--o{`（多对多）实现灵活RBAC用户-角色分配。每个关系符号编码一种约束  
- **字段含类型/标注/用途说明** — PK/FK 支持模式生成，表情注释便于人工扫描。开发者可直接根据此图编写迁移脚本
