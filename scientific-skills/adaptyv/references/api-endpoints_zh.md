# Adaptyv Bio Foundry API — 完整端点参考

基础 URL: `https://foundry-api-public.adaptyvbio.com/api/v1`
OpenAPI 规范: `GET /openapi.json`

## 目录

- [实验](#实验)
- [序列](#序列)
- [结果](#结果)
- [靶标](#靶标)
- [报价](#报价)
- [令牌](#令牌)
- [更新](#更新)
- [反馈](#反馈)

---

## 实验

### POST /experiments — 创建实验

创建新实验。默认以`草稿`状态启动。

**请求体:**

| 字段 | 类型 | 必填 | 描述 |
|---|---|---|---|
| `name` | string | 是 | 可读名称 |
| `experiment_spec` | ExperimentSpec | 是 | 实验定义（见下文） |
| `skip_draft` | boolean | 否（默认 false） | 跳过草稿状态，直接进入等待确认 |
| `auto_accept_quote` | boolean | 否（默认 false） | 自动接受报价并创建发票 |
| `webhook_url` | string/null | 否 | 状态变更通知的 POST 回调 URL |

**ExperimentSpec:**

| 字段 | 类型 | 必填 | 描述 |
|---|---|---|---|
| `experiment_type` | string | 是 | `affinity`（亲和力）, `screening`（筛选）, `thermostability`（热稳定性）, `fluorescence`（荧光）或 `expression`（表达） |
| `method` | string | 结合实验必填 | `bli` 或 `spr` |
| `target_id` | uuid | 结合实验必填 | 目录中的靶标 UUID |
| `sequences` | object | 是 | 名称 → 氨基酸字符串或富对象映射 |
| `n_replicates` | integer | 推荐（默认 3） | 技术重复次数（最小值 1） |
| `antigen_concentrations` | number[] | 否（仅亲和力） | 默认 `[1000.0, 316.2, 100.0, 31.6, 0.0]` nM |
| `parameters` | object | 否 | 实验特定设置 |

**实验类型字段要求:**

| 字段 | 亲和力 | 筛选 | 热稳定性 | 荧光 | 表达 |
|---|---|---|---|---|---|
| `experiment_type` | 必填 | 必填 | 必填 | 必填 | 必填 |
| `method` | 必填 | 必填 | — | — | — |
| `target_id` | 必填 | 必填 | — | — | — |
| `sequences` | 必填 | 必填 | 必填 | 必填 | 必填 |
| `n_replicates` | 推荐 | 推荐 | 可选 | 可选 | 可选 |
| `antigen_concentrations` | 可选 | — | — | — | — |

**响应 (201):**

| 字段 | 类型 | 描述 |
|---|---|---|
| `experiment_id` | string | 新实验 UUID |
| `error` | string/null | 验证失败时的错误信息 |
| `stripe_hosted_invoice_url` | string/null | 当`auto_accept_quote`创建发票时出现 |
| `stripe_invoice_id` | string/null | Stripe 发票 ID |

**状态码:** 201, 400, 401, 403, 404

---

### GET /experiments — 列出实验

列出调用者可访问的实验，按创建日期排序（最新优先）。

**查询参数:** `limit`, `offset`, `filter`, `search`, `sort`

**响应项:**

| 字段 | 类型 | 描述 |
|---|---|---|
| `id` | uuid | 唯一标识符 |
| `code` | string | 例如 "EXP-2024-001" |
| `name` | string/null | 可读名称 |
| `status` | ExperimentStatus | 当前生命周期状态 |
| `experiment_type` | ExperimentType | affinity/screening/thermostability/fluorescence/expression |
| `results_status` | ResultsStatus | none/partial/all |
| `created_at` | datetime | ISO 8601 格式 |
| `experiment_url` | string | Foundry 门户 URL |
| `stripe_invoice_url` | string/null | 发票 URL |
| `stripe_quote_url` | string/null | 报价单 URL |

**状态码:** 200, 401

---

### GET /experiments/{experiment_id} — 获取实验

返回单个实验的完整元数据。

**路径参数:** `experiment_id` (uuid)

**响应:**

| 字段 | 类型 | 描述 |
|---|---|---|
| `id` | uuid | 唯一标识符 |
| `code` | string | 实验代码 |
| `status` | ExperimentStatus | 当前状态 |
| `experiment_spec` | ExperimentSpec | 完整实验定义 |
| `results_status` | ResultsStatus | none/partial/all |
| `created_at` | datetime | ISO 8601 格式 |
| `experiment_url` | string | 门户 URL |
| `costs` | object | 成本明细 |

**状态码:** 200, 401, 404, 500

---

### PATCH /experiments/{experiment_id} — 更新实验

修改现有实验。草稿实验允许全字段编辑；报价生成后仅`name`, `description`, `webhook_url`可编辑。

**路径参数:** `experiment_id` (uuid)

**请求体:** 所有字段可选 — 仅更新提供的字段。

**状态码:** 200, 400, 401, 404, 409

---

### POST /experiments/{experiment_id}/submit — 提交实验

提交草稿实验进行审核。状态从`草稿`转为`等待确认`。

**路径参数:** `experiment_id` (uuid)

**响应:**

| 字段 | 类型 | 描述 |
|---|---|---|
| `experiment_id` | string | 实验 UUID |

**状态码:** 200, 401, 403, 404, 409, 500

---

### POST /experiments/cost-estimate — 成本估算

计算成本而不创建实验。

**请求体:**
```json
{
  "experiment_spec": {
    "experiment_type": "screening",
    "method": "bli",
    "target_id": "...",
    "sequences": {"seq1": "MKTL..."},
    "n_replicates": 3
  }
}
```

**响应:**

| 字段 | 类型 | 描述 |
|---|---|---|
| `pricing_version` | string | 例如 "v1_2026-01-20" |
| `assay` | object | 按类型成本（含基础价和重复定价） |
| `materials` | object | 靶标材料成本（结合实验） |
| `total_cents` | integer | 美元美分总和 |

所有价格不含增值税；税费在开票时计算。无自助定价的靶标返回不完整估算。

**状态码:** 200, 400, 401

---

### GET /experiments/{experiment_id}/quote — 获取报价

返回报价元数据（总额、货币、状态、有效期）。

**路径参数:** `experiment_id` (uuid)

**响应:**

| 字段 | 类型 | 描述 |
|---|---|---|
| `experiment_id` | string | 实验 UUID |
| `stripe_quote_url` | string | Stripe 报价 URL |
| `amount_total` | int64 | 最小货币单位总额 |
| `amount_subtotal` | int64 | 小计 |
| `currency` | string | ISO 货币代码（如 "usd"） |
| `status` | string | 报价状态 |
| `expires_at` | datetime/null | 过期时间 |

**状态码:** 200, 401, 403, 404, 500

---

### GET /experiments/{experiment_id}/quote/pdf — 获取报价 PDF

返回 PDF 格式的报价文件 (`application/pdf`)。

**路径参数:** `experiment_id` (uuid)

**状态码:** 200, 401, 403, 404, 500

---

### POST /experiments/{experiment_id}/quote/confirm — 接受报价（按实验）

接受 Stripe 报价，创建发票草稿，状态转为`等待材料`。

**路径参数:** `experiment_id` (uuid)

**请求体:**

| 字段 | 类型 | 必填 | 描述 |
|---|---|---|---|
| `purchase_order_number` | string/null | 否 | 采购订单号 |
| `notes` | string/null | 否 | 保留字段 |

**响应:**

| 字段 | 类型 | 描述 |
|---|---|---|
| `id` | string | 报价 ID |
| `status` | StripeQuoteStatus | 新状态 |
| `hosted_invoice_url` | string/null | Stripe 支付 URL |
| `invoice_id` | string/null | 生成的发票 ID |

**状态码:** 200, 401, 403, 404, 409

---

### GET /experiments/{experiment_id}/invoice — 获取发票

返回发票元数据（含支付 URL）。

**路径参数:** `experiment_id` (uuid)

**状态码:** 200, 401, 403, 404, 500

---

### GET /experiments/{experiment_id}/results — 列出实验结果

返回特定实验的所有分析结果。

**路径参数:** `experiment_id` (uuid)
**查询参数:** `limit`, `offset`, `filter`, `sort`

**状态码:** 200, 400, 401, 403, 404

---

### GET /experiments/{experiment_id}/sequences — 列出实验序列

返回特定实验的所有序列，最新优先。

**路径参数:** `experiment_id` (uuid)
**查询参数:** `limit`, `offset`, `search`, `sort`

**状态码:** 200, 400, 401, 403, 404

---

### GET /experiments/{experiment_id}/updates — 列出实验更新

返回单实验的更新记录，最早优先。类型: `status_change`, `progress`, `error`。

**路径参数:** `experiment_id` (uuid)
**查询参数:** `limit`, `offset`, `filter`, `sort`

筛选示例: `filter=eq(type,status_change)`

---

## 序列

### GET /sequences — 列出序列

返回所有实验的序列，最新优先。

**查询参数:** `limit`, `offset`, `search`, `sort`, `experiment_id`（按实验 UUID 筛选）

**响应项:**

| 字段 | 类型 | 描述 |
|---|---|---|
| `id` | uuid | 唯一标识符 |
| `name` | string/null | 可选名称 |
| `aa_preview` | string/null | 截取预览（前 50 字符） |
| `length` | int32 | 氨基酸序列长度 |
| `experiment_id` | uuid | 父实验 |
| `experiment_code` | string | 可读实验代码 |
| `is_control` | boolean | 是否为对照 |
| `created_at` | datetime | 创建时间戳 |

**状态码:** 200, 401

---

### GET /sequences/{sequence_id} — 获取序列

返回完整详情（含氨基酸序列）。

**路径参数:** `sequence_id` (uuid)

**响应:**

| 字段 | 类型 | 描述 |
|---|---|---|
| `id` | uuid | 唯一标识符 |
| `aa_string` | string/null | 完整氨基酸序列 |
| `length` | int32 | 氨基酸长度 |
| `is_control` | boolean | 对照标记 |
| `metadata` | object | 序列级注释 |
| `experiment` | object | 父实验引用 |
| `created_at` | datetime | 创建时间戳 |

**状态码:** 200, 401, 403, 404, 500

---

### POST /sequences — 向实验添加序列

向**草稿**状态实验追加序列（通过可读代码标识）。

**请求体:**

| 字段 | 类型 | 必填 | 描述 |
|---|---|---|---|
| `experiment_code` | string | 是 | 例如 "PROJ-001" |
| `sequences` | array | 是 | 序列条目数组 |

**序列条目:**

| 字段 | 类型 | 必填 | 描述 |
|---|---|---|---|
| `aa_string` | string | 是 | 氨基酸序列 |
| `name` | string | 否 | 可读名称 |
| `control` | boolean | 否 | 是否为对照 |
| `metadata` | object | 否 | 注释信息 |

**响应 (201):**

| 字段 | 类型 | 描述 |
|---|---|---|
| `added_count` | int32 | 添加的序列数量 |
| `experiment_id` | string | 实验 UUID |
| `experiment_code` | string | 实验代码 |
| `sequence_ids` | array | 新增序列 ID 列表 |

**状态码:** 201, 400, 404, 409（实验非草稿状态）, 500

---

## 结果

### GET /results — 列出结果

列出已完成的分析结果，最新优先。当`results_status`为`partial`或`all`时显示结果。

**查询参数:** `limit`, `offset`, `filter`, `search`, `sort`

**响应项:**

| 字段 | 类型 | 描述 |
|---|---|---|
| `id` | uuid | 结果标识符 |
| `title` | string | 可读标题 |
| `experiment_id` | uuid | 关联实验 |
| `result_type` | string | 例如 "affinity", "thermostability" |
| `summary` | array | 关键结果（类型特定，见下文） |
| `metadata` | object | 扩展元数据（如仪器信息） |
| `data_package_url` | string/null | 原始数据包下载 URL |
| `created_at` | datetime | 结果生成时间 |

**亲和力结果字段:** `kd_mean`, `kd_std`, `kon_mean`, `kon_log_std`, `koff_mean`, `koff_std`, `replicates`（含每重复`kd`, `kon`, `koff`, `binding_strength`, `kon_method`, `koff_method`, `replicate`索引）, `sequence`, `target_id`

**热稳定性结果字段:** Tm 值和熔解曲线

**状态码:** 200, 401

---

### GET /results/{result_id} — 获取结果

返回详细结果数据（含完整摘要数组）。

**路径参数:** `result_id` (uuid)

**状态码:** 200, 401, 403, 404, 500

---

## 靶标

### GET /targets — 列出靶标

列出可用于实验的已验证抗原。

**查询参数:**

| 参数 | 类型 | 描述 |
|---|---|---|
| `limit` | int | 最大条目数（1-100，默认 50） |
| `offset` | int | 跳过数量 |
| `search` | string | 产品名称全文搜索 |
| `sort` | string | 排序表达式 |
| `selfservice_only` | boolean | 仅含自助定价的靶标 |
| `show_conjugated` | boolean | 包含偶联靶标（默认：

| `note` | string/null | 否 | 附加说明 |

**状态码：** 201, 400, 401, 403, 500

---

### GET /targets/request-custom — 列出定制靶点申请

返回您所在组织的定制靶点申请，按最新优先排序。

**查询参数：** `limit`, `offset`, `filter`, `sort`

筛选示例：`filter=eq(status,pending_review)`

---

### GET /targets/request-custom/{request_id} — 获取定制靶点申请详情

**路径参数：** `request_id` (uuid)

**响应：**

| 字段 | 类型 | 描述 |
|---|---|---|
| `id` | uuid | 申请标识符 |
| `name` | string | 靶点名称 |
| `product_id` | string | 您的产品ID |
| `status` | string | 状态（如"pending_review"） |
| `material_id` | string/null | 批准后关联的目录ID |
| `molecular_weight` | number/null | 分子量（单位kDa） |
| `note` | string/null | 用户备注 |
| `created_at` | datetime | 创建时间 |
| `updated_at` | datetime | 最后更新时间 |

**状态码：** 200, 401, 403, 404, 500

---

## 报价单

### GET /quotes — 列出报价单

返回调用方组织的所有报价单。

**查询参数：** `limit`, `offset`, `filter`, `sort`

**响应项：**

| 字段 | 类型 | 描述 |
|---|---|---|
| `id` | string | 报价单标识符 |
| `quote_number` | string | 可读报价单编号 |
| `organization_id` | uuid | 所属组织 |
| `amount_cents` | int | 金额（单位：分） |
| `currency` | string | ISO 4217货币代码 |
| `status` | StripeQuoteStatus | 报价单状态 |
| `valid_until` | datetime | 有效期截止时间 |
| `created_at` | datetime | 创建时间戳 |

---

### GET /quotes/{quote_id} — 获取报价单详情

返回含明细定价的完整报价单文档。

**路径参数：** `quote_id` (字符串，如"qt_1Abc2DefGhi")

**响应：**

| 字段 | 类型 | 描述 |
|---|---|---|
| `id` | string | 报价单标识符 |
| `quote_number` | string | 参考编号 |
| `organization_id` | uuid | 所属组织 |
| `organization_name` | string | 组织名称 |
| `line_items` | array | 明细定价项 |
| `subtotal_cents` | int | 小计（单位：分） |
| `tax_cents` | int | 税费（单位：分） |
| `total_cents` | int | 总计（单位：分） |
| `currency` | string | ISO 4217货币代码 |
| `status` | StripeQuoteStatus | 当前状态 |
| `valid_until` | datetime | 有效期截止时间 |
| `notes` | string | 特殊定价说明 |
| `terms_and_conditions` | string | 条款与条件 |
| `stripe_quote_url` | string | Stripe链接 |
| `created_at` | datetime | 创建时间 |

**状态码：** 200, 401, 403, 404, 500

---

### POST /quotes/{quote_id}/confirm — 接受报价单

确认报价单，创建发票草稿，将实验推进至`WaitingForMaterials`状态。

**路径参数：** `quote_id` (字符串)

**请求体：**

| 字段 | 类型 | 必填 | 描述 |
|---|---|---|---|
| `purchase_order_number` | string/null | 否 | 采购订单号 |
| `notes` | string/null | 否 | 预留字段 |

**响应：** `id`, `status`, `hosted_invoice_url`, `invoice_id`

**状态码：** 200, 403, 404, 409, 500

---

### POST /quotes/{quote_id}/reject — 拒绝报价单

取消报价单；关联实验回退至`Draft`状态。

**路径参数：** `quote_id` (字符串)

**请求体：**

| 字段 | 类型 | 必填 | 描述 |
|---|---|---|---|
| `reason` | QuoteRejectionReason | 是 | 主要拒绝原因 |
| `feedback` | string/null | 否 | 补充反馈 |

**响应：** `id`, `status` (canceled)

**状态码：** 200, 403, 404, 409, 500

---

## 令牌

### GET /tokens — 列出令牌

返回调用方拥有的所有令牌（根令牌与衰减令牌）。

**查询参数：** `limit`, `offset`

**响应项：**

| 字段 | 类型 | 描述 |
|---|---|---|
| `id` | string | 令牌标识符 |
| `name` | string | 可读标签 |
| `kind` | string | "root"（根令牌）或"attenuated"（衰减令牌） |
| `created_at` | datetime | 创建时间 |
| `expires_at` | datetime/null | 过期时间（null表示永不过期） |
| `revoked_at` | datetime/null | 撤销时间戳 |
| `parent_token_id` | string/null | 父令牌（根令牌为null） |
| `root_token_id` | string/null | 派生树根令牌 |
| `attenuation_spec` | object/null | 限制规则（根令牌为null） |

---

### POST /tokens/attenuate — 创建衰减令牌

通过Biscuit加密衰减机制创建现有令牌的限制版本。

**请求体：**

| 字段 | 类型 | 必填 | 描述 |
|---|---|---|---|
| `token` | string | 是 | 现有令牌（格式：`abs0_{slug}{biscuit_base64}`） |
| `attenuation` | AttenuationSpec | 是 | 待应用的限制规则 |
| `name` | string | 是 | 可读标签 |
| `attenuated_parent_token_id` | uuid/null | 否 | 链式衰减的父令牌ID |

**限制类型：** 组织、资源（实验/结果）、操作（读取/创建/更新）、有效期

**响应（201）：** `id`（数据库ID）, `token`（新衰减令牌字符串）

**状态码：** 201, 400, 401, 403

---

### POST /tokens/revoke — 撤销令牌及派生链

撤销调用令牌的根令牌及其所有衰减派生令牌。幂等操作。

**响应：**

| 字段 | 类型 | 描述 |
|---|---|---|
| `token_id` | string | 被撤销的根令牌ID |
| `revoked_at` | datetime | 撤销时间戳 |
| `children_revoked` | int64 | 新撤销的子令牌数量 |

**状态码：** 200, 403, 404

---

## 实验更新

### GET /updates — 获取实验更新

返回实验更新动态（最新优先）：状态变更、进度、错误信息。

**查询参数：** `limit`, `offset`, `filter`, `sort`

**筛选示例：**
- `filter=eq(experiment_id,<uuid>)`
- `filter=in(experiment_id,uuid1,uuid2)`
- `filter=eq(type,status_change)`

**响应项：**

| 字段 | 类型 | 描述 |
|---|---|---|
| `id` | string | 更新标识符 |
| `experiment_id` | uuid | 关联实验 |
| `experiment_code` | string | 可读实验代码 |
| `name` | string | 更新描述 |
| `timestamp` | datetime | 更新时间 |

---

## 反馈

### POST /feedback/submit — 提交反馈

用于提交错误报告、功能请求或通用反馈。

**请求体：**

| 字段 | 类型 | 必填 | 描述 |
|---|---|---|---|
| `request_uuid` | uuid | 是 | 问题API请求的UUID |
| `feedback_type` | FeedbackType | 是 | `feature_request`（功能请求）, `feedback`（反馈）或`bug_report`（错误报告） |
| `title` | string/null | 否 | 简短标题 |
| `json_body` | object/null | 至少一项 | 结构化错误详情 |
| `human_note` | string/null | 至少一项 | 自由描述文本 |

**响应（201）：** `reference`（反馈编号）, `message`（确认信息）

**状态码：** 201, 400, 401, 500
