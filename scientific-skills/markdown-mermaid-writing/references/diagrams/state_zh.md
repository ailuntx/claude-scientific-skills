<!-- 来源：https://github.com/SuperiorByteWorks-LLC/agent-project | 许可证：Apache-2.0 | 作者：Clayton Young / Superior Byte Works, LLC (Boreal Bytes) -->

# 状态图

> **返回[样式指南](../mermaid_style_guide.md)** — 请先阅读样式指南了解表情符号、颜色和无障碍规则

**语法关键字：** `stateDiagram-v2`  
**最佳适用场景：** 状态机、生命周期流程、状态转换、对象生命周期  
**不适用场景：** 多步骤顺序流程（使用[流程图](flowchart.md)），时间敏感交互（使用[时序图](sequence.md)）

---

## 示例图

```mermaid
stateDiagram-v2
    accTitle: 订单履行生命周期
    accDescr: 电商订单状态机，涵盖下单、支付、履行到交付全流程，含取消路径

    [*] --> Placed: 📋 客户提交

    Placed --> PaymentPending: 💰 发起支付
    PaymentPending --> PaymentFailed: ❌ 支付失败
    PaymentPending --> Confirmed: ✅ 支付成功

    PaymentFailed --> Placed: 🔄 重试支付
    PaymentFailed --> Cancelled: 🚫 客户取消

    Confirmed --> Picking: 📦 仓库拣货
    Picking --> Shipped: 🚚 物流揽收
    Shipped --> Delivered: ✅ 交付凭证
    Delivered --> [*]: 🏁 完成

    Cancelled --> [*]: 🏁 关闭

    note right of Confirmed
        📋 库存已预留
        💰 发票已生成
    end note
```

---

## 设计建议

- 始终以`[*]`（初始状态）开始，以`[*]`（终止状态）结束
- 用**表情符号+动作**标注转换以提升可读性
- 使用`note right of`/`note left of`添加上下文细节
- 状态命名：`驼峰式`（Mermaid状态图规范）
- 谨慎使用嵌套状态：`state "名称" as s1 { ... }`
- 状态数量控制在**8-10个**以内

---

## 模板

```mermaid
stateDiagram-v2
    accTitle: 此处填写标题
    accDescr: 描述实体生命周期及关键状态转换

    [*] --> InitialState: ⚡ 触发事件

    InitialState --> ActiveState: ▶️ 执行操作
    ActiveState --> CompleteState: ✅ 成功
    ActiveState --> FailedState: ❌ 错误

    CompleteState --> [*]: 🏁 完成
    FailedState --> [*]: 🏁 关闭
```

---

## 复杂示例

CI/CD流水线状态机，包含3个复合（嵌套）状态，每个状态含内部子状态。展示代码变更如何通过构建、测试、部署阶段，含故障恢复和回滚路径。

```mermaid
stateDiagram-v2
    accTitle: CI/CD流水线状态机
    accDescr: CI/CD流水线复合状态图，展示代码检测、构建测试阶段（含并行扫描）及三阶段部署（含审批门控和回滚路径）

    [*] --> Source: ⚡ 代码提交

    state "📥 代码源" as Source {
        [*] --> Idle
        Idle --> Fetching: 🔄 检测到变更
        Fetching --> Validating: 📋 检出完成
        Validating --> [*]: ✅ 配置有效
    }

    Source --> Build: ⚙️ 触发流水线

    state "🔧 构建与测试" as Build {
        [*] --> Compiling
        Compiling --> UnitTests: ✅ 构建产物就绪
        UnitTests --> IntegrationTests: ✅ 单元测试通过
        IntegrationTests --> SecurityScan: ✅ 集成测试通过
        SecurityScan --> [*]: ✅ 无漏洞

        note right of Compiling
            📦 构建Docker镜像
            🏷️ 标记提交SHA值
        end note
    }

    Build --> Deploy: 📦 发布产物
    Build --> Failed: ❌ 构建或测试失败

    state "🚀 部署" as Deploy {
        [*] --> Staging
        Staging --> WaitApproval: ✅ 预发环境正常
        WaitApproval --> Production: ✅ 审批通过
        WaitApproval --> Cancelled: 🚫 审批拒绝
        Production --> Monitoring: 🚀 已部署
        Monitoring --> [*]: ✅ 稳定运行30分钟

        note right of WaitApproval
            👤 需团队负责人审批
            ⏰ 24小时后自动拒绝
        end note
    }

    Deploy --> Rollback: ❌ 健康检查失败
    Rollback --> Deploy: 🔄 回退至前一版本
    Deploy --> Complete: 🏁 流水线完成
    Failed --> Source: 🔧 修复后提交
    Cancelled --> [*]: 🏁 流水线中止
    Complete --> [*]: 🏁 完成

    state Failed {
        [*] --> AnalyzeFailure
        AnalyzeFailure --> NotifyTeam: 📤 发送告警
        NotifyTeam --> [*]
    }

    state Rollback {
        [*] --> RevertArtifact
        RevertArtifact --> RestorePrevious: 🔄 恢复旧版本
        RestorePrevious --> VerifyRollback: 🔍 健康检查
        VerifyRollback --> [*]
    }
```

### 为何这样设计有效

- **复合状态分组流水线阶段** — 代码源、构建与测试、部署各自包含内部流程，既可独立阅读又可整体理解
- **故障和回滚作为独立状态** — 不仅是转换标签。Failed和Rollback状态含内部子状态，清晰展示恢复过程
- **关键状态注释补充操作上下文** — 审批门控含超时规则，编译步骤记录产物格式。这些是运维必需细节
- **复合状态间转换体现高层流程**（代码源→构建→部署→完成），而复合状态内部转换展示详细步骤。双重阅读层级适配不同受众
