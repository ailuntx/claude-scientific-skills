<!-- 来源：https://github.com/SuperiorByteWorks-LLC/agent-project | 许可证：Apache-2.0 | 作者：Clayton Young / Superior Byte Works, LLC (Boreal Bytes) -->

# 甘特图

> **返回[样式指南](../mermaid_style_guide.md)** — 请先阅读样式指南了解表情符号、颜色和无障碍规则

**语法关键字：** `gantt`  
**最佳适用场景：** 项目时间线、路线图、阶段规划、里程碑跟踪、任务依赖关系  
**不适用场景：** 简单时序事件（使用[时间线](timeline.md)）、流程逻辑（使用[流程图](flowchart.md)）

---

## 示例图表

```mermaid
gantt
    accTitle: 第一季度产品发布路线图
    accDescr: 为期八周的项目时间线，涵盖探索、设计、构建和发布阶段，包含设计评审和启动决策里程碑

    title 🚀 第一季度产品发布路线图
    dateFormat YYYY-MM-DD
    axisFormat %b %d

    section 📋 探索阶段
        用户调研          :done, research, 2026-01-05, 7d
        竞品分析          :done, compete, 2026-01-05, 5d
        需求文档          :done, reqs, after compete, 3d

    section 🎨 设计阶段
        线框图            :done, wire, after reqs, 5d
        视觉设计          :active, visual, after wire, 7d
        🏁 设计评审       :milestone, review, after visual, 0d

    section 🔧 构建阶段
        核心功能          :crit, core, after visual, 10d
        API集成          :api, after visual, 8d
        测试              :test, after core, 5d

    section 🚀 发布阶段
        预发布部署        :staging, after test, 3d
        🏁 启动决策       :milestone, decision, after staging, 0d
        生产环境发布      :crit, release, after staging, 2d
```

---

## 使用技巧

- 使用带表情符号前缀的 `section` 按阶段或团队分组
- 用 `:milestone` 和 `0d` 时长标记里程碑——前缀添加 🏁
- 状态标签：`:done`（已完成）、`:active`（进行中）、`:crit`（关键路径，高亮显示）
- 使用 `after taskId` 建立任务依赖关系
- 保持总时间线**在3个月内**以确保可读性
- 使用 `axisFormat` 控制日期显示（`%b %d` = "1月05日"，`%m/%d` = "01/05"）

---

## 模板

```mermaid
gantt
    accTitle: 你的标题
    accDescr: 描述时间线范围和关键里程碑

    title 📋 你的路线图标题
    dateFormat YYYY-MM-DD
    axisFormat %b %d

    section 📋 第一阶段
        任务一       :done, t1, 2026-01-01, 5d
        任务二       :active, t2, after t1, 3d

    section 🔧 第二阶段
        任务三       :crit, t3, after t2, 7d
        🏁 里程碑     :milestone, m1, after t3, 0d
```

---

## 复杂案例

跨团队平台迁移项目，历时4个月，包含6个阶段、24项任务和3个里程碑。展示团队间依赖（后端迁移阻塞前端迁移）、关键路径任务，以及从规划到上线监控的全生命周期。

```mermaid
gantt
    accTitle: 多团队平台迁移路线图
    accDescr: 为期四个月的迁移项目，涵盖规划、后端、前端、数据、质量保证和发布团队，包含跨团队依赖、关键路径任务和三个里程碑节点

    title 🚀 平台迁移——2026年第一/二季度
    dateFormat YYYY-MM-DD
    axisFormat %b %d

    section 📋 规划阶段
        启动会议                :done, plan1, 2026-01-05, 2d
        架构评审                :done, plan2, after plan1, 5d
        迁移方案文档            :done, plan3, after plan2, 5d
        风险评估                :done, plan4, after plan2, 3d
        🏁 规划完成             :milestone, m_plan, after plan3, 0d

    section 🔧 后端团队
        API重设计               :crit, be1, after m_plan, 12d
        数据迁移脚本            :be2, after m_plan, 10d
        新服务部署              :crit, be3, after be1, 8d
        向后兼容层              :be4, after be1, 6d

    section 🎨 前端团队
        组件库更新              :fe1, after m_plan, 10d
        页面迁移                :crit, fe2, after be3, 12d
        A/B测试配置             :fe3, after fe2, 5d
        功能对等验证            :fe4, after fe2, 4d

    section 🗄️ 数据团队
        结构迁移                :crit, de1, after be2, 8d
        ETL管道更新             :de2, after de1, 7d
        数据验证套件            :de3, after de2, 5d
        回滚脚本                :de4, after de1, 4d

    section 🧪 质量保证团队
        测试方案制定            :qa1, after m_plan, 7d
        回归测试套件            :qa2, after be3, 10d
        性能测试                :crit, qa3, after qa2, 7d
        用户验收测试协调        :qa4, after qa3, 5d
        🏁 质量保证签核         :milestone, m_qa, after qa4, 0d

    section 🚀 发布阶段
        预发布环境部署          :crit, l1, after m_qa, 3d
        🏁 启动决策             :milestone, m_go, after l1, 0d
        生产环境切换            :crit, l2, after m_go, 2d
        上线后监控              :l3, after l2, 10d
        旧系统停用              :l4, after l3, 5d
```

### 设计优势解析

- **6个阶段对应实际团队**——各团队可快速定位工作流。通过`after taskId`显式标注跨团队依赖（前端等待后端API，质量保证等待后端部署）
- **`:crit`标记关键路径**——决定项目总时长的任务链。任何关键任务延误都将导致发布日期推迟，Mermaid会将其标红突出
- **3个里程碑作为决策关卡**——规划完成、质量保证签核、启动决策。这些是利益相关方进行实质性决策的节点，而非单纯进度更新
- **4个月跨度24项任务仍具可读性**——按团队分组展示。若无阶段划分，将形成难以阅读的任务墙
