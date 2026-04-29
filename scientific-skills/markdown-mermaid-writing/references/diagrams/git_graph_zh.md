<!-- 来源：https://github.com/SuperiorByteWorks-LLC/agent-project | 许可证：Apache-2.0 | 作者：Clayton Young / Superior Byte Works, LLC (Boreal Bytes) -->

# Git 图谱

> **返回 [样式指南](../mermaid_style_guide.md)** — 请先阅读样式指南了解表情符号、颜色和无障碍规则

**语法关键字：** `gitGraph`  
**最佳适用场景：** 分支策略、合并工作流、发布流程、git-flow 可视化  
**不适用场景：** 通用流程（使用[流程图](flowchart.md)）、项目时间线（使用[甘特图](gantt.md)）

---

## 示例图

```mermaid
gitGraph
    accTitle: 主干开发工作流
    accDescr: 展示短期特性分支合并到主干的Git历史，含发布标签的主干开发示例

    commit id: "初始化"
    commit id: "配置CI"

    branch feature/auth
    checkout feature/auth
    commit id: "添加登录"
    commit id: "添加测试"

    checkout main
    merge feature/auth id: "合并认证" tag: "v1.0"

    commit id: "更新依赖"

    branch feature/dashboard
    checkout feature/dashboard
    commit id: "添加图表"
    commit id: "添加筛选器"

    checkout main
    merge feature/dashboard id: "合并面板"

    commit id: "性能修复" tag: "v1.1"
```

---

## 提示

- 为提交点使用描述性 `id:` 标签
- 通过 `tag:` 标记发布版本
- 分支名称需匹配实际规范（如 `feature/`, `fix/`, `release/`）
- 展示**理想**工作流 —— 这是规范性而非描述性
- 关键合并提交使用 `type: HIGHLIGHT` 高亮
- 保持 **10–15 个提交点** 以保证可读性

---

## 模板

```mermaid
gitGraph
    accTitle: 你的标题
    accDescr: 描述分支策略与合并模式

    commit id: "初始提交"
    commit id: "第二次提交"

    branch feature/你的特性
    checkout feature/你的特性
    commit id: "特性开发"
    commit id: "添加测试"

    checkout main
    merge feature/你的特性 id: "合并特性" tag: "v1.0"
```
