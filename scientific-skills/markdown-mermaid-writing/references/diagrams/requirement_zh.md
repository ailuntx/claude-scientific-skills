<!-- 来源：https://github.com/SuperiorByteWorks-LLC/agent-project | 许可证：Apache-2.0 | 作者：Clayton Young / Superior Byte Works, LLC (Boreal Bytes) -->

# 需求图

> **返回 [样式指南](../mermaid_style_guide.md)** — 请先阅读样式指南以了解表情符号、颜色和无障碍规则。

**语法关键字：** `requirementDiagram`  
**最适用于：** 系统需求追溯、合规性映射、正式需求工程  
**何时不应使用：** 非正式任务跟踪（使用 [看板](kanban.md)），通用关系描述（使用 [实体关系图](er.md)）

---

## 示例图

```mermaid
requirementDiagram

    requirement high_availability {
        id: 1
        text: 系统应保持99.9%的正常运行时间
        risk: high
        verifymethod: test
    }

    requirement data_encryption {
        id: 2
        text: 所有静态数据应使用AES-256加密
        risk: medium
        verifymethod: inspection
    }

    requirement session_timeout {
        id: 3
        text: 会话在空闲30分钟后过期
        risk: low
        verifymethod: test
    }

    element auth_service {
        type: service
        docref: auth-service-v2
    }

    element crypto_module {
        type: module
        docref: crypto-lib-v3
    }

    auth_service - satisfies -> high_availability
    auth_service - satisfies -> session_timeout
    crypto_module - satisfies -> data_encryption
```

---

## 提示

- 每个需求需包含：`id`、`text`、`risk`、`verifymethod`  
- **`id` 必须是数字** — 使用 `id: 1`、`id: 2` 等（带连字符的格式如 `REQ-001` 可能导致解析错误）  
- 风险等级：`low`、`medium`、`high`（全小写）  
- 验证方法：`analysis`、`inspection`、`test`、`demonstration`（全小写）  
- 使用 `element` 表示满足需求的设计组件  
- 关系类型：`- satisfies ->`、`- traces ->`、`- contains ->`、`- derives ->`、`- refines ->`、`- copies ->`  
- 每张图限制在 **3–5 个需求**  
- 避免在文本字段使用特殊字符 — 拼写符号（例如用"99.9 percent"而非"99.9%"）  
- 在 `{ }` 代码块内使用 4 空格缩进  

---

## 模板

```mermaid
requirementDiagram

    requirement your_requirement {
        id: 1
        text: 此处填写需求陈述
        risk: medium
        verifymethod: test
    }

    element your_component {
        type: service
        docref: component-ref
    }

    your_component - satisfies -> your_requirement
```
