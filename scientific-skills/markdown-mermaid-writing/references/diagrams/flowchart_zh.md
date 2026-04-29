<!-- 来源：https://github.com/SuperiorByteWorks-LLC/agent-project | 许可证：Apache-2.0 | 作者：Clayton Young / Superior Byte Works, LLC (Boreal Bytes) -->

# 流程图

> **返回[样式指南](../mermaid_style_guide.md)** — 请先阅读样式指南了解表情符号、颜色和无障碍规则

**语法关键词：** `flowchart`  
**最佳适用场景：** 顺序流程、工作流、决策逻辑、故障排查树  
**不适用场景：** 多角色复杂时序（使用[时序图](sequence.md)）、状态机（使用[状态图](state.md)）

---

## 示例图

```mermaid
flowchart TB
    accTitle: 功能开发生命周期
    accDescr: 从想法到设计、构建、测试、评审和发布的端到端功能流程，包含评审失败时的修订循环

    idea([💡 功能想法]) --> spec[📋 编写规范]
    spec --> design[🎨 设计方案]
    design --> build[🔧 实现功能]
    build --> test[🧪 运行测试]
    test --> review{🔍 评审通过？}
    review -->|是| release[🚀 发布到生产环境]
    review -->|否| revise[✏️ 修改代码]
    revise --> test
    release --> monitor([📊 监控指标])

    classDef start fill:#ede9fe,stroke:#7c3aed,stroke-width:2px,color:#3b0764
    classDef process fill:#dbeafe,stroke:#2563eb,stroke-width:2px,color:#1e3a5f
    classDef decision fill:#fef9c3,stroke:#ca8a04,stroke-width:2px,color:#713f12
    classDef success fill:#dcfce7,stroke:#16a34a,stroke-width:2px,color:#14532d

    class idea,monitor start
    class spec,design,build,test,revise process
    class review decision
    class release success
```

---

## 设计技巧

- 流程使用 `TB`（自上而下），流水线使用 `LR`（从左到右）
- 圆角矩形 `([文本])` 表示起点/终点，菱形 `{文本}` 表示决策点
- 节点上限 10 个——大型流程拆分为"阶段1"/"阶段2"子图
- 单图决策点不超过 3 个
- 连接线标签保持 1-4 个词：`-->|是|`，`-->|全部通过|`
- 使用 `classDef` 实现**语义化**着色——决策用琥珀色、成功用绿色、操作步骤用蓝色

## 子图模式

需要分组阶段时：

```mermaid
flowchart TB
    accTitle: CI/CD 流水线阶段
    accDescr: 三阶段流水线将代码质量检查、测试和部署划分为独立阶段

    trigger([⚡ 推送至主分支])

    subgraph quality ["🔍 代码质量"]
        lint[📝 代码检查] --> format[⚙️ 格式校验]
    end

    subgraph testing ["🧪 测试"]
        unit[🧪 单元测试] --> integration[🔗 集成测试]
    end

    subgraph deploy ["🚀 部署"]
        build[📦 构建产物] --> ship[☁️ 部署到预发环境]
    end

    trigger --> quality
    quality --> testing
    testing --> deploy
    deploy --> done([✅ 流水线完成])

    classDef trigger_style fill:#ede9fe,stroke:#7c3aed,stroke-width:2px,color:#3b0764
    classDef success fill:#dcfce7,stroke:#16a34a,stroke-width:2px,color:#14532d

    class trigger trigger_style
    class done success
```

---

## 模板

```mermaid
flowchart TB
    accTitle: 您的标题（3-8个词）
    accDescr: 用一两句话说明图表内容及读者可获得的洞察

    start([🏁 起点]) --> step1[⚙️ 第一步操作]
    step1 --> decision{🔍 检查条件？}
    decision -->|是| step2[✅ 正向路径]
    decision -->|否| step3[🔧 替代路径]
    step2 --> done([🏁 完成])
    step3 --> done
```

---

## 复杂案例

20+节点的电商订单流水线，划分为5个代表处理阶段的子图。子图通过内部节点连接，决策点将订单路由至异常处理，颜色类别实现阶段快速识别。

```mermaid
flowchart TB
    accTitle: 电商订单处理流水线
    accDescr: 从接单到履约、发货及通知的完整订单生命周期，包含支付失败、缺货和配送问题的异常处理路径

    order_in([📥 新订单]) --> validate_pay{💰 支付有效？}

    subgraph intake ["📥 订单接收"]
        validate_pay -->|是| check_fraud{🔐 欺诈检测}
        validate_pay -->|否| pay_fail[❌ 支付**失败**]
        check_fraud -->|安全| check_stock{📦 库存充足？}
        check_fraud -->|可疑| manual_review[🔍 人工**审核**]
        manual_review --> check_stock
    end

    subgraph fulfill ["📦 履约"]
        pick[📋 **拣货**] --> pack[📦 打包订单]
        pack --> label[🏷️ 生成**物流**标签]
    end

    subgraph ship ["🚚 发货"]
        handoff[🚚 交接**承运商**] --> transit[📍 运输中]
        transit --> deliver{✅ 送达？}
    end

    subgraph notify ["📤 通知"]
        confirm_email[📧 订单**确认**]
        ship_update[📧 物流**更新**]
        deliver_email[📧 送达**确认**]
    end

    subgraph exception ["⚠️ 异常处理"]
        pay_fail --> retry_pay[🔄 重试支付]
        retry_pay --> validate_pay
        out_of_stock[📦 创建**缺货订单**]
        deliver_fail[🔄 **重新**配送]
    end

    check_stock -->|是| pick
    check_stock -->|否| out_of_stock
    label --> handoff
    deliver -->|是| deliver_email
    deliver -->|否| deliver_fail
    deliver_fail --> transit

    check_stock -->|是| confirm_email
    handoff --> ship_update
    deliver_email --> complete([✅ 订单**完成**])

    classDef intake_style fill:#dbeafe,stroke:#2563eb,stroke-width:2px,color:#1e3a5f
    classDef fulfill_style fill:#ede9fe,stroke:#7c3aed,stroke-width:2px,color:#3b0764
    classDef ship_style fill:#dcfce7,stroke:#16a34a,stroke-width:2px,color:#14532d
    classDef warn_style fill:#fef9c3,stroke:#ca8a04,stroke-width:2px,color:#713f12
    classDef danger_style fill:#fee2e2,stroke:#dc2626,stroke-width:2px,color:#7f1d1d

    class validate_pay,check_fraud,check_stock,manual_review intake_style
    class pick,pack,label fulfill_style
    class handoff,transit,deliver ship_style
    class confirm_email,ship_update,deliver_email warn_style
    class pay_fail,retry_pay,out_of_stock,deliver_fail danger_style
```

### 设计优势

- **5个子图对应实际业务阶段**——接收、履约、发货、通知和异常处理符合运营团队的真实工作逻辑
- **异常处理独立成子图**——避免分散在各阶段，代理和读者可集中查看所有失败路径
- **颜色类别强化结构**——蓝色接收、紫色履约、绿色发货、琥珀色通知、红色异常。即使不读标签，色彩模式也能标识当前阶段
- **决策点连接子图**——菱形节点（`{支付有效？}`, `{库存充足？}`, `{送达？}`）是流程分支枢纽，每条分支指向清晰标记的目标
