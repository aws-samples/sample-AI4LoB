---
name: protoforge
display_name: "ProtoForge"
icon: "🏗️"
description: "Converts natural language requirements into development-delivery-grade interactive HTML prototypes. Activates when the user wants to create a prototype, wireframe, or interactive mockup — good signals: 'prototype', 'create a prototype', 'generate UI', 'mockup', 'interactive HTML', '生成原型', '原型生成', 'protoforge'. Covers 15 UI states, multi-role permissions, and SaaS Web / Mobile App platform differences. Output is a single-file HTML that developers can validate without verbal explanation."
trigger: "protoforge prototype 生成原型 原型生成 generate prototype create prototype interactive mockup NL-to-prototype 交互原型 HTML原型"
inputs:
  - name: requirement_text
    description: "自然语言产品需求描述"
    type: string
    required: true
tools:
  - file_write
  - file_read
  - open_in_session_tab
  - run_javascript
id: 7f9bfb4e33b341539f320ccd90c637b4
---

# 🏗️ ProtoForge — 自然语言需求 → 研发交付级 HTML 原型

## Overview

ProtoForge 将用户的自然语言需求描述转化为**研发交付级**的可交互 HTML 原型。"研发交付级"指：研发和测试不依赖口头解释即可实现与验收。

**核心差异化**：覆盖 15 种 UI 状态（非仅 Happy Path）、多角色权限差异、异常场景注入，区分 SaaS Web 和 Mobile App 的导航与交互范式。

**技术栈**：单文件 HTML + Alpine.js (CDN) + Tailwind CSS + DaisyUI — 零构建步骤，双击即开。

**输出规模**：5-10 页面 × 全状态覆盖（一个功能模块）

---

## Workflow

### 整体流程（Multi-Step Generation Protocol）

```
用户输入自然语言需求
        ↓
   Phase 1: 需求解析
   (提取平台/角色/功能 → 输出设计规格概要)
        ↓
   用户确认概要 ✓
        ↓
   Phase 2: 分步生成 HTML
   Step 1: 骨架（导航+路由+控制面板）
   Step 2-N: 逐页填充（每页×全状态）
        ↓
   Phase 3: 质量验证
   (检查清单 + 验收标准生成)
        ↓
   输出: 单文件 .html + 验收标准注释

```

---

## Phase 1: 需求解析

# 需求解析器模块 (Requirement Parser)

> **模块职责**：从用户的自然语言描述中提取结构化信息，输出标准化需求规格，供后续模块（架构生成器、页面生成器）消费。

---

## 1. 平台检测规则 (Platform Detection)

### 关键词映射表

| 目标平台 | 触发关键词 |
| --- | --- |
| **Web** | `SaaS`、`后台`、`管理系统`、`dashboard`、`web app`、`B端`、`portal`、`网页`、`浏览器`、`中台`、`工作台` |
| **Mobile** | `App`、`移动端`、`iOS`、`Android`、`小程序`、`手机`、`原生应用`、`移动应用`、`客户端` |
| **Both** | `全端`、`跨平台`、`web+app`、`响应式`、`多端`、`全平台`、`H5+原生` |

### 检测逻辑

```
1. 扫描用户输入，匹配上述关键词（不区分大小写）
2. 如果命中 Both 类关键词 → platform = "Both"
3. 如果同时命中 Web 和 Mobile 类关键词 → platform = "Both"
4. 如果仅命中 Web 类 → platform = "Web"
5. 如果仅命中 Mobile 类 → platform = "Mobile"
6. 如果未命中任何关键词 → 触发追问

```

### 追问模板

```
未检测到明确的目标平台信息。请确认目标平台：
- **Web** — 浏览器端管理后台/SaaS应用
- **Mobile** — 手机App/小程序
- **Both** — 同时支持Web和移动端

请选择：Web / Mobile / Both？

```

---

## 2. 角色提取规则 (Role Extraction)

### 识别模式

| 模式 | 示例 | 提取方式 |
| --- | --- | --- |
| 显式列举 | "管理员和普通用户" | 直接提取命名角色 |
| 数量+类型 | "3种角色" / "分为X和Y" | 提取数量，结合上下文推断角色名 |
| 权限描述 | "有些人只能查看，有些人可以编辑" | 推断为 Viewer + Editor |
| 无角色信息 | 未提及任何角色相关内容 | 使用默认值 |

### 标准角色映射

| 用户表述 | 标准化角色名 |
| --- | --- |
| 管理员 / Admin / 超管 / 系统管理员 | `Admin` |
| 成员 / Member / 普通用户 / 员工 | `Member` |
| 访客 / Viewer / Guest / 游客 / 只读用户 | `Viewer` |
| 审批人 / Approver / 审核员 | `Approver` |
| 运营 / Operator / 编辑 | `Operator` |

### 默认回退策略

```
如果未检测到任何角色信息：
  → 默认使用: [Admin, Member, Viewer]
  → 在输出的规格摘要中声明: "⚠️ 未检测到角色信息，已使用默认三角色模型 (Admin/Member/Viewer)"

```

### 角色数量约束

```
- 最大角色数: 5
- 如果提取到 > 5 个角色:
  → 提示用户: "检测到 N 个角色，建议精简为 3-5 个核心角色以保证原型可读性。请确认优先保留哪些？"
  → 列出所有检测到的角色供用户勾选

```

---

## 3. 核心功能提取 (Core Function Extraction)

### 动词+名词对提取

扫描用户输入，提取以下结构的功能描述：

| 动词类型 | 示例动词 | 常见搭配名词 |
| --- | --- | --- |
| 创建类 | 创建、新建、添加、新增、发布 | 项目、订单、文章、任务 |
| 管理类 | 管理、维护、配置、设置 | 用户、权限、系统、数据 |
| 查看类 | 查看、浏览、搜索、筛选 | 报表、列表、详情、数据 |
| 编辑类 | 编辑、修改、更新、调整 | 订单、信息、资料、内容 |
| 删除类 | 删除、移除、归档、作废 | 记录、数据、内容 |
| 审批类 | 审批、审核、确认、驳回 | 申请、流程、工单 |
| 导出类 | 导出、下载、分享、推送 | 报表、数据、文件 |

### 功能→页面类型映射

```
功能类型        →  页面类型            →  页面组件
─────────────────────────────────────────────────────
CRUD操作        →  List + Detail + Form  →  表格/卡片列表 + 详情面板 + 表单
查看/报表       →  Dashboard            →  图表组合 + KPI卡片 + 筛选器
设置/配置       →  Settings             →  分组表单 + 开关 + 保存按钮
搜索/筛选       →  Search               →  搜索栏 + 筛选面板 + 结果列表
审批/流程       →  Workflow             →  状态流转图 + 操作按钮 + 历史记录
消息/通知       →  Notification         →  消息列表 + 详情 + 标记已读
登录/注册       →  Auth                 →  登录表单 + 注册表单 + 忘记密码

```

### 平台默认页面规则

**Web 平台默认包含：**

```
- 登录页 (Auth)
- 仪表盘/首页 (Dashboard)
- [按功能生成的业务页面]
- 系统设置页 (Settings)

```

**Mobile 平台默认包含：**

```
- 启动页/登录页 (Splash + Auth)
- 首页/Tab主页 (Home)
- [按功能生成的业务页面]
- 个人中心 (Profile)

```

**Both 平台：**

```
- Web端遵循 Web 规则
- Mobile端遵循 Mobile 规则
- 标注共享页面与平台独占页面

```

---

## 4. 行业/领域检测 (Industry/Domain Detection)

### 行业关键词表

| 行业标识 | 触发关键词 | 推荐术语风格 | 典型数据示例 |
| --- | --- | --- | --- |
| `ecommerce` | 电商、商城、购物、SKU、订单、库存、物流 | 商品、店铺、购物车 | 商品名、价格、库存量 |
| `crm` | CRM、客户、销售、跟进、线索、商机 | 客户、联系人、商机 | 公司名、联系方式、成交额 |
| `project` | 项目管理、任务、看板、迭代、Sprint | 项目、任务、里程碑 | 项目名、负责人、截止日期 |
| `saas` | SaaS、订阅、租户、多租户、计费 | 租户、订阅、计划 | 套餐名、价格、到期日 |
| `social` | 社交、社区、动态、关注、消息、帖子 | 动态、关注、粉丝 | 用户名、帖子内容、点赞数 |
| `education` | 教育、课程、学员、考试、学习 | 课程、学员、成绩 | 课程名、讲师、进度 |
| `healthcare` | 医疗、患者、预约、诊断、处方 | 患者、科室、诊断 | 姓名、科室、就诊日期 |
| `finance` | 金融、账户、交易、支付、理财 | 账户、交易、余额 | 账户号、金额、时间戳 |
| `general` | （未匹配任何行业关键词） | 通用术语 | 通用示例数据 |

### 检测逻辑

```
1. 统计各行业关键词命中次数
2. 命中最多的行业 → 主行业标识
3. 如果最高命中数 < 2 → industry = "general"
4. 如果两个行业命中数相同 → 取用户描述中最先出现的行业

```

---

## 5. 信息不足处理策略 (Insufficient Information Strategy)

### 分级处理机制

| 级别 | 判定条件 | 处理方式 |
| --- | --- | --- |
| **严重不足** | 描述 < 30 字符 | 必须追问 2 个问题：平台 + 核心功能 |
| **部分缺失** | 缺少角色信息 | 使用默认值，声明假设 |
| **轻度模糊** | 功能描述不明确 | 列出推断结果，请求确认 |
| **信息充足** | 平台+角色+功能均明确 | 直接输出结构化规格 |

### 严重不足时的追问模板

```
您的需求描述较为简短，需要补充以下信息以生成高质量原型：

1️⃣ **目标平台**：Web（浏览器） / Mobile（手机App） / Both（两端都要）？
2️⃣ **核心功能**：请列举 2-5 个最重要的功能，例如："用户管理、订单处理、数据报表"

可选补充：
- 用户角色（如：管理员、普通用户）
- 所属行业（如：电商、教育、SaaS）

```

### 部分缺失时的假设声明

```
在输出规格摘要时附加说明：
"ℹ️ 以下信息基于默认假设生成：
 - 角色: 使用默认三角色模型 [Admin, Member, Viewer]
 - [其他默认假设项]
 如需调整，请告知具体角色划分。"

```

### 轻度模糊时的确认模板

```
根据您的描述，我推断核心功能如下：
1. ✅ 用户管理 — CRUD操作
2. ✅ 订单处理 — 列表+详情+状态流转
3. ❓ "数据分析" — 不确定是仪表盘还是报表导出

请确认以上推断是否正确？第3项具体指哪种形式？

```

---

## 6. 输出格式 (Output Format)

### 结构化需求规格 (Structured Requirement Spec)

```json
{
  "platform": "Web | Mobile | Both",
  "roles": [
    {
      "name": "Admin",
      "label": "管理员",
      "permissions": "full_access"
    },
    {
      "name": "Member",
      "label": "成员",
      "permissions": "read_write"
    },
    {
      "name": "Viewer",
      "label": "访客",
      "permissions": "read_only"
    }
  ],
  "core_functions": [
    {
      "name": "用户管理",
      "verb": "管理",
      "noun": "用户",
      "page_type": "CRUD",
      "screens": ["用户列表", "用户详情", "新增用户"]
    },
    {
      "name": "数据报表",
      "verb": "查看",
      "noun": "报表",
      "page_type": "Dashboard",
      "screens": ["数据概览"]
    }
  ],
  "industry": "general | ecommerce | crm | project | saas | social | education | healthcare | finance",
  "screens": [
    "登录页",
    "仪表盘",
    "用户列表",
    "用户详情",
    "新增用户",
    "数据概览",
    "系统设置"
  ],
  "special_requirements": [
    "离线支持",
    "多语言",
    "暗色模式",
    "实时通知"
  ],
  "assumptions": [
    "未检测到角色信息，使用默认三角色模型"
  ],
  "confidence": {
    "platform": "high | medium | low",
    "roles": "high | medium | low",
    "functions": "high | medium | low",
    "industry": "high | medium | low"
  }
}

```

### 输出摘要模板（面向用户展示）

```
📋 **需求解析结果**

| 维度 | 解析结果 | 置信度 |
|------|---------|--------|
| 平台 | Web | ✅ 高 |
| 角色 | Admin, Member, Viewer | ⚠️ 默认值 |
| 核心功能 | 用户管理, 订单处理, 数据报表 | ✅ 高 |
| 行业 | 电商 (ecommerce) | ✅ 高 |
| 预计页面数 | 8 页 | — |

⚠️ 假设说明：[列出所有默认假设]

确认以上解析结果后，将进入架构生成阶段。需要修改请直接告知。

```

---

## 解析流水线 (Processing Pipeline)

```
用户输入
  │
  ├─→ [1] 平台检测 ──→ 命中？──→ 是 → 记录平台
  │                         └─→ 否 → 标记待追问
  │
  ├─→ [2] 角色提取 ──→ 命中？──→ 是 → 标准化角色名
  │                         └─→ 否 → 使用默认值 + 声明假设
  │
  ├─→ [3] 功能提取 ──→ 动词+名词对 → 映射页面类型 → 推断页面列表
  │
  ├─→ [4] 行业检测 ──→ 命中？──→ 是 → 设置行业标识
  │                         └─→ 否 → industry = "general"
  │
  ├─→ [5] 特殊需求扫描 ──→ 离线/多语言/实时 等关键词
  │
  └─→ [6] 信息完整度检查
           │
           ├─→ 完整 → 输出结构化规格
           ├─→ 部分缺失 → 输出规格 + 假设声明
           └─→ 严重不足 → 触发追问流程

```

---

## VERIFICATION

```
✅ 检查清单：
- [x] 平台检测规则：覆盖 Web/Mobile/Both 三种情况及未检测到的兜底策略
- [x] 角色提取规则：显式提取 + 默认回退 + 数量约束(最多5个)
- [x] 功能提取规则：动词+名词对模式 + 功能→页面类型映射 + 平台默认页面
- [x] 行业检测规则：8个行业 + general兜底 + 命中计数逻辑
- [x] 信息不足策略：4级分级处理 + 追问模板 + 假设声明模板
- [x] 输出格式：JSON结构化规格 + 用户友好摘要表格 + 置信度标注
- [x] 处理流水线：6步串行流程，清晰的决策分支
- [x] 全中文撰写，关键术语保留英文标识
- [x] 与后续模块（架构生成器）的接口契约明确：输出JSON可直接消费

```

---

## Phase 2: 状态覆盖模型（15 态）

# State Matrix 模块 — 15 状态覆盖模型

> **模块职责**: 定义原型生成中必须覆盖的 15 种页面状态，以及按页面类型判断每种状态是否适用的规则矩阵。确保生成的原型具备生产级状态覆盖，不遗漏边缘场景。

---

## 1. 完整状态目录 (15 States)

### 1.1 强制状态 — 每个原型页面必须覆盖的 7 种

| # | State ID | 英文名 | 中文名 | 描述 | 视觉模式 |
| --- | --- | --- | --- | --- | --- |
| 1 | `ideal` | Ideal | 理想状态 | 正常工作，数据充足，功能完整可用 | 完整数据展示，所有组件正常渲染 |
| 2 | `empty` | Empty | 空状态 | 无数据可展示，首次使用或数据被清空 | 插画占位 SVG + 引导文案 + CTA 按钮 |
| 3 | `loading` | Loading | 加载状态 | 数据加载中，等待服务器响应 | 骨架屏 (Skeleton)，保持布局结构不变 |
| 4 | `error` | Error | 错误状态 | 请求失败 / 系统错误 / 服务不可用 | 红色提示区域 + 错误原因文案 + 重试按钮 |
| 5 | `partial` | Partial | 部分加载 | 页面内部分模块成功、部分模块失败 | 成功区域正常渲染 + 失败区域显示错误提示卡片 |
| 6 | `success` | Success | 成功状态 | 用户操作完成（提交/保存/删除等） | Toast 或 Banner 成功提示 + 下一步引导 |
| 7 | `no-permission` | No-Permission | 无权限 (403) | 角色权限不足，无法访问该页面或功能 | 锁定图标 + "无权访问" 文案 + 联系管理员/申请权限按钮 |

### 1.2 上下文状态 — 按页面类型和场景按需覆盖的 8 种

| # | State ID | 英文名 | 中文名 | 触发条件 |
| --- | --- | --- | --- | --- |
| 8 | `offline` | Offline | 离线状态 | Mobile 平台必选；Web 按需（如明确指定离线支持） |
| 9 | `timeout` | Timeout | 超时状态 | 有长耗时操作的页面（文件上传、批量处理、报表生成） |
| 10 | `session-expired` | Session Expired | 会话过期 | 有认证的应用（默认包含，除非明确为公开页面） |
| 11 | `rate-limited` | Rate-Limited | 限流状态 | 有批量操作或高频 API 调用的页面 |
| 12 | `stale-data` | Stale Data | 数据过期 | 有缓存策略的页面（Mobile 常见；Web 中长轮询/WS 场景） |
| 13 | `maintenance` | Maintenance | 维护模式 | 全局级别 — 整个应用只需一个维护状态页面即可 |
| 14 | `degraded` | Degraded Mode | 降级模式 | 有依赖外部服务的功能（支付网关、地图 API、AI 接口等） |
| 15 | `validation-error` | Validation Error | 验证错误 | 有表单输入的页面（必选） |

---

## 2. 页面类型 × 状态适用性矩阵

### 2.1 SaaS Web 页面类型

| 页面类型 | ideal | empty | loading | error | partial | success | 403 | offline | timeout | session | rate-limit | stale | maint | degraded | validation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Dashboard (仪表盘) | ✅ | ✅ | ✅ | ✅ | ✅ | ⬜ | ✅ | ⬜ | ✅ | ✅ | ⬜ | ⬜ | ✅ | ✅ | ⬜ |
| Data Table/List (数据列表) | ✅ | ✅ | ✅ | ✅ | ✅ | ⬜ | ✅ | ⬜ | ⬜ | ⬜ | ✅ | ⬜ | ⬜ | ⬜ | ⬜ |
| Detail View (详情页) | ✅ | ⬜ | ✅ | ✅ | ✅ | ⬜ | ✅ | ⬜ | ⬜ | ⬜ | ⬜ | ✅ | ⬜ | ⬜ | ⬜ |
| Create/Edit Form (表单页) | ✅ | ⬜ | ✅ | ✅ | ⬜ | ✅ | ✅ | ⬜ | ✅ | ✅ | ⬜ | ⬜ | ⬜ | ⬜ | ✅ |
| Settings (设置页) | ✅ | ⬜ | ✅ | ✅ | ⬜ | ✅ | ✅ | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ | ✅ |
| Search Results (搜索结果) | ✅ | ✅ | ✅ | ✅ | ⬜ | ⬜ | ⬜ | ⬜ | ✅ | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ |

### 2.2 Mobile App 页面类型

| 页面类型 | ideal | empty | loading | error | partial | success | 403 | offline | timeout | session | rate-limit | stale | maint | degraded | validation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Home/Feed (首页/信息流) | ✅ | ✅ | ✅ | ✅ | ✅ | ⬜ | ✅ | ✅ | ⬜ | ⬜ | ⬜ | ✅ | ✅ | ⬜ | ⬜ |
| List/Detail (列表/详情) | ✅ | ✅ | ✅ | ✅ | ⬜ | ⬜ | ✅ | ✅ | ⬜ | ⬜ | ⬜ | ✅ | ⬜ | ⬜ | ⬜ |
| Action/Form (操作/表单) | ✅ | ⬜ | ✅ | ✅ | ⬜ | ✅ | ✅ | ✅ | ✅ | ✅ | ⬜ | ⬜ | ⬜ | ⬜ | ✅ |
| Profile (个人中心) | ✅ | ⬜ | ✅ | ✅ | ✅ | ✅ | ⬜ | ✅ | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ | ✅ |
| Notifications (通知中心) | ✅ | ✅ | ✅ | ✅ | ⬜ | ⬜ | ⬜ | ✅ | ⬜ | ⬜ | ⬜ | ✅ | ⬜ | ⬜ | ⬜ |

### 2.3 矩阵使用说明

- ✅ = 该状态**必须**为此页面类型生成对应 UI 变体
- ⬜ = 该状态对此页面类型**不适用**，跳过生成
- 如页面类型未列入矩阵，默认使用 **Dashboard** 行规则（最全面覆盖）

---

## 3. 状态视觉模式规则 (Visual Pattern Rules)

每种状态在代码生成时必须遵循的**精确视觉模式**：

### 3.1 Loading (加载状态)

```html
<!-- 骨架屏模式 — 保持与 Ideal 状态相同的布局结构 -->
<div class="space-y-4 animate-pulse">
  <div class="skeleton h-8 w-48"></div>        <!-- 标题占位 -->
  <div class="skeleton h-4 w-full"></div>      <!-- 段落占位 -->
  <div class="skeleton h-4 w-3/4"></div>       <!-- 段落占位 -->
  <div class="skeleton h-32 w-full"></div>     <!-- 内容区占位 -->
</div>

```

**规则**: 骨架屏必须与 Ideal 状态的布局结构一一对应，让用户感知到即将出现的内容结构。

### 3.2 Empty (空状态)

```html
<!-- 空状态模式 — 插画 + 引导文案 + 主操作按钮 -->
<div class="flex flex-col items-center justify-center py-16 text-center">
  <!-- 占位插画 SVG (64x64 或 128x128) -->
  <svg class="w-24 h-24 text-base-content/30 mb-4">...</svg>
  <h3 class="text-lg font-semibold text-base-content/70">暂无数据</h3>
  <p class="text-sm text-base-content/50 mt-1 max-w-sm">
    {上下文相关的引导文案，告诉用户为什么为空以及如何开始}
  </p>
  <button class="btn btn-primary mt-6">{主操作 CTA}</button>
</div>

```

**规则**: 插画使用简洁线条 SVG；文案需结合业务上下文解释为空原因；CTA 引导用户创建第一条数据。

### 3.3 Error (错误状态)

```html
<!-- 错误状态模式 — DaisyUI alert-error -->
<div class="alert alert-error shadow-lg">
  <svg xmlns="http://www.w3.org/2000/svg" class="stroke-current shrink-0 h-6 w-6" fill="none" viewBox="0 0 24 24">
    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z" />
  </svg>
  <div>
    <h3 class="font-bold">加载失败</h3>
    <p class="text-sm">{具体错误原因描述}</p>
  </div>
  <button class="btn btn-sm btn-ghost">重试</button>
</div>

```

**规则**: 必须包含错误图标 + 错误标题 + 错误原因文案 + 重试按钮。

### 3.4 No-Permission / 403 (无权限)

```html
<!-- 403 无权限模式 -->
<div class="flex flex-col items-center justify-center py-16 text-center">
  <div class="w-16 h-16 rounded-full bg-warning/10 flex items-center justify-center mb-4">
    <svg class="w-8 h-8 text-warning"><!-- 锁定图标 --></svg>
  </div>
  <h3 class="text-lg font-semibold">无权访问此页面</h3>
  <p class="text-sm text-base-content/50 mt-1">
    您的角色没有查看此内容的权限
  </p>
  <div class="flex gap-2 mt-6">
    <button class="btn btn-primary btn-sm">申请权限</button>
    <button class="btn btn-ghost btn-sm">联系管理员</button>
  </div>
</div>

```

**规则**: 锁定图标居中 + 明确的无权限文案 + 提供申请权限或联系管理员两个出口。

### 3.5 Success (成功状态)

```html
<!-- 成功 Toast 模式 — 固定右下角，3 秒自动消失 -->
<div class="toast toast-end">
  <div class="alert alert-success">
    <svg class="stroke-current shrink-0 h-6 w-6"><!-- 成功勾选图标 --></svg>
    <span>{操作成功文案} — {下一步引导}</span>
  </div>
</div>
<!-- JS: setTimeout(() => toast.remove(), 3000) -->

```

**规则**: 使用 Toast 定位右下角；包含成功图标 + 成功文案 + 下一步引导；3 秒自动消失。

### 3.6 Offline (离线状态)

```html
<!-- 离线横幅 — 页面顶部固定 -->
<div class="w-full bg-warning text-warning-content px-4 py-2 flex items-center gap-2 text-sm">
  <svg class="w-4 h-4"><!-- 断网/同步图标 --></svg>
  <span>当前处于离线模式，数据将在恢复连接后自动同步</span>
</div>

```

**规则**: 顶部固定黄色横幅；使用断网/同步图标；提示离线状态及数据同步策略。

### 3.7 Validation Error (验证错误)

```html
<!-- 表单验证错误 — 行内字段级别 -->
<div class="form-control w-full">
  <label class="label"><span class="label-text">邮箱</span></label>
  <input type="email" class="input input-bordered input-error w-full" value="invalid@" />
  <label class="label">
    <span class="label-text-alt text-error">请输入有效的邮箱地址</span>
  </label>
</div>

```

**规则**: 使用 `input-error` 红色边框；错误文案紧贴字段下方；逐字段独立校验提示。

### 3.8 其他上下文状态视觉模式

| State | 视觉模式概述 |
| --- | --- |
| Timeout | 类似 Error，但文案为 "请求超时" + 进度指示器 + 重试按钮 |
| Session Expired | 全屏遮罩 Modal，"会话已过期，请重新登录" + 登录按钮 |
| Rate-Limited | 类似 Error，文案 "请求过于频繁，请稍后再试" + 倒计时 |
| Stale Data | 顶部黄色 Banner "数据可能已过期" + 手动刷新按钮 |
| Maintenance | 全屏维护页面，维护插画 + 预计恢复时间 + 状态页链接 |
| Degraded | 受影响区域显示黄色 Warning 卡片 "部分功能暂时不可用" |

---

## 4. 状态枚举算法 (State Enumeration Algorithm)

给定一个页面规格说明，按以下算法确定该页面需要覆盖的状态列表：

```
输入: PageSpec { name, type, platform, features[] }
输出: ApplicableStates[]

算法步骤:

STEP 1 — 强制基础状态
  states = [ideal, empty, loading, error, partial, success, no-permission]

STEP 2 — 矩阵查表
  matrix_row = LOOKUP(PageSpec.type, PageSpec.platform)
  IF matrix_row 存在:
    FOR EACH contextual_state IN [offline, timeout, session, rate-limited, stale, maintenance, degraded, validation]:
      IF matrix_row[contextual_state] == ✅:
        states.ADD(contextual_state)

STEP 3 — 特征触发器 (覆盖矩阵结果)
  IF PageSpec.features CONTAINS "form" OR "input":
    states.ENSURE(validation-error)     // 有表单必须有验证错误
  IF PageSpec.platform == "mobile":
    states.ENSURE(offline)              // 移动端必须有离线
  IF PageSpec.features CONTAINS "auth" OR "login-required":
    states.ENSURE(session-expired)      // 需要认证就必须有会话过期
  IF PageSpec.features CONTAINS "external-api" OR "third-party":
    states.ENSURE(degraded)             // 依赖外部服务就必须有降级
  IF PageSpec.features CONTAINS "batch" OR "bulk" OR "export":
    states.ENSURE(timeout)              // 批量操作必须有超时
    states.ENSURE(rate-limited)         // 批量操作必须有限流
  IF PageSpec.features CONTAINS "cache" OR "local-storage":
    states.ENSURE(stale-data)           // 有缓存就必须有数据过期

STEP 4 — 去重排序
  states = DEDUPLICATE(states)
  states = SORT_BY(state_catalog_order)  // 按目录 1-15 排序

STEP 5 — 输出
  RETURN states

```

### 4.1 算法执行示例

**示例 1**: SaaS Web Dashboard（含第三方图表 API）

```
PageSpec: { type: "Dashboard", platform: "web", features: ["external-api"] }

Step 1: [ideal, empty, loading, error, partial, success, no-permission]
Step 2: 查 Dashboard 行 → +timeout, +session, +maintenance, +degraded
Step 3: features含"external-api" → ENSURE(degraded) ← 已有
Final:  [ideal, empty, loading, error, partial, success, no-permission,
         timeout, session-expired, maintenance, degraded]
总计: 11 个状态

```

**示例 2**: Mobile Action/Form（含文件上传）

```
PageSpec: { type: "Action/Form", platform: "mobile", features: ["form", "file-upload", "auth"] }

Step 1: [ideal, empty, loading, error, partial, success, no-permission]
Step 2: 查 Mobile Action/Form 行 → +offline, +timeout, +session, +validation
Step 3: features含"form" → ENSURE(validation) ← 已有
        platform=="mobile" → ENSURE(offline) ← 已有
        features含"auth" → ENSURE(session-expired) ← 已有
Final:  [ideal, empty, loading, error, partial, success, no-permission,
         offline, timeout, session-expired, validation-error]
总计: 11 个状态

```

**示例 3**: Web Data Table（含批量导出）

```
PageSpec: { type: "Data Table/List", platform: "web", features: ["batch", "export"] }

Step 1: [ideal, empty, loading, error, partial, success, no-permission]
Step 2: 查 Data Table 行 → +rate-limited
Step 3: features含"batch" → ENSURE(timeout) + ENSURE(rate-limited) ← rate-limited 已有
Final:  [ideal, empty, loading, error, partial, success, no-permission,
         timeout, rate-limited]
总计: 9 个状态

```

---

## 5. 状态间转换关系

定义状态之间的合法转换路径（用于交互原型中的状态机逻辑）：

```
┌─────────────────────────────────────────────────────────────┐
│                        状态转换图                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  [进入页面] ──→ Loading ──┬──→ Ideal (成功)                  │
│                          ├──→ Empty (无数据)                 │
│                          ├──→ Error (失败)                   │
│                          ├──→ Partial (部分失败)             │
│                          ├──→ No-Permission (403)           │
│                          ├──→ Timeout (超时)                 │
│                          └──→ Offline (断网)                 │
│                                                             │
│  Ideal ──→ [用户操作] ──→ Loading ──→ Success               │
│                                   ──→ Error                  │
│                                   ──→ Validation Error       │
│                                                             │
│  Error / Timeout ──→ [重试] ──→ Loading                      │
│  Session Expired ──→ [重新登录] ──→ Loading                   │
│  Offline ──→ [恢复连接] ──→ Loading ──→ Ideal / Stale Data   │
│  Stale Data ──→ [手动刷新] ──→ Loading ──→ Ideal             │
│                                                             │
│  [任意状态] ──→ Maintenance (全局触发)                        │
│  [任意状态] ──→ Rate-Limited (频率触发)                       │
│  [任意状态] ──→ Session Expired (过期触发)                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘

```

---

## 6. 代码生成集成指南

### 6.1 状态切换器组件

每个生成的原型页面必须包含状态切换器，用于演示查看：

```html
<!-- 状态切换器 — 放置在页面右上角 -->
<div class="dropdown dropdown-end fixed top-4 right-4 z-50">
  <label tabindex="0" class="btn btn-sm btn-outline gap-1">
    <span>状态</span>
    <svg class="w-4 h-4"><!-- 下拉箭头 --></svg>
  </label>
  <ul tabindex="0" class="dropdown-content menu p-2 shadow bg-base-100 rounded-box w-52">
    <li><a data-state="ideal" class="active">✅ 理想状态</a></li>
    <li><a data-state="empty">📭 空状态</a></li>
    <li><a data-state="loading">⏳ 加载中</a></li>
    <li><a data-state="error">❌ 错误</a></li>
    <li><a data-state="partial">⚠️ 部分加载</a></li>
    <li><a data-state="success">🎉 成功</a></li>
    <li><a data-state="no-permission">🔒 无权限</a></li>
    <!-- 以下按适用性动态添加 -->
    <li><a data-state="offline">📡 离线</a></li>
    <li><a data-state="validation">🚫 验证错误</a></li>
    <!-- ... -->
  </ul>
</div>

```

### 6.2 状态容器模式

```html
<!-- 每种状态对应一个容器，通过 data-state 属性切换显示 -->
<div data-state-view="ideal" class="">
  <!-- Ideal 状态内容 -->
</div>
<div data-state-view="empty" class="hidden">
  <!-- Empty 状态内容 -->
</div>
<div data-state-view="loading" class="hidden">
  <!-- Loading 状态内容 -->
</div>
<!-- ... 其他状态 ... -->

<script>
// 状态切换逻辑
document.querySelectorAll('[data-state]').forEach(btn => {
  btn.addEventListener('click', () => {
    const state = btn.dataset.state;
    document.querySelectorAll('[data-state-view]').forEach(view => {
      view.classList.toggle('hidden', view.dataset.stateView !== state);
    });
  });
});
</script>

```

---

## VERIFICATION

```yaml
verification:
  module_name: "State Matrix"
  completeness_checks:
    - "15 个状态全部定义（7 强制 + 8 上下文）": true
    - "每个状态有唯一 State ID": true
    - "每个状态有中英文名称": true
    - "每个状态有触发条件描述": true
    - "SaaS Web 页面类型矩阵完整（6 种页面类型）": true
    - "Mobile App 页面类型矩阵完整（5 种页面类型）": true
    - "所有 7 种视觉模式有 HTML 代码示例": true
    - "状态枚举算法步骤清晰（5 步）": true
    - "包含 3 个算法执行示例": true
    - "状态转换关系图完整": true
    - "代码生成集成指南（切换器 + 容器模式）": true

  cross_references:
    - "视觉模式使用 DaisyUI 组件类名 → 与 Tech Stack 模块一致"
    - "页面类型分类 → 与 Page Architecture 模块对齐"
    - "状态切换器 → 集成到 Code Generation 模块的模板中"

  usage_in_skill:
    - "接收 PageSpec → 执行状态枚举算法 → 输出 ApplicableStates[]"
    - "ApplicableStates[] 传递给代码生成模块 → 为每个状态生成对应 UI 变体"
    - "状态切换器组件自动插入每个生成页面 → 支持演示时快速切换查看"

  quality_gates:
    - "强制状态永远不少于 7 个"
    - "移动端页面必定包含 offline 状态"
    - "有表单的页面必定包含 validation-error 状态"
    - "有认证的页面必定包含 session-expired 状态"
    - "全局 maintenance 状态只需生成一次，不重复"

```

---

## Phase 3: 角色权限建模

# Role Permission 模块

> 定义多角色权限差异在生成原型中的建模与表达方式。

---

## 1. 角色层级模板（Role Hierarchy Template）

当用户未指定角色体系时，使用以下默认 RBAC 模型：

| 角色 | 权限范围 | 级别 |
| --- | --- | --- |
| **Owner** | 全部权限 + 组织管理 + 计费 | 5 |
| **Admin** | 全部功能权限 + 用户管理（无计费） | 4 |
| **Member** | CRUD 自有数据 + 查看共享数据 | 3 |
| **Viewer** | 只读访问（无创建/编辑/删除） | 2 |
| **Guest** | 有限只读（仅公开内容） | 1 |

**自定义角色映射规则：**

- 当用户指定自定义角色时，将其映射到层级中最接近的级别
- 例如："项目经理" → 映射到 Member 级别（可 CRUD 项目相关数据）
- 例如："外部审计员" → 映射到 Viewer 级别（只读，但可访问审计相关页面）
- 如果自定义角色跨多个级别，取其核心操作所需的最高级别

**生成时角色精简规则：**

- 原型默认展示 3 个角色（Admin / Member / Viewer）以保持 demo 简洁
- 用户明确要求 5 级时才展开全部
- Role Switcher 中仅显示原型中实际有差异化表现的角色

---

## 2. 三层权限模型（Three-Layer Permission Model）

### Layer 1: 页面级（Page-Level）

控制每个角色可访问哪些页面/模块。

**实现方式：**

```html
<!-- 页面级访问控制 -->
<div x-show="canAccess('page_name')">
  <!-- 页面内容 -->
</div>

```

**规则：**

- 无权访问的页面：从导航中完全隐藏（不是灰显，是移除）
- 用户直接切换到无权页面时：显示 403 视图
- 导航菜单根据当前角色动态过滤

### Layer 2: 操作级（Operation-Level）

控制每个角色在可访问页面上能执行哪些动作。

**实现方式：**

```html
<!-- 按钮级别控制 -->
<button x-show="can('operation')" class="btn btn-primary">操作按钮</button>

<!-- 行内操作控制（禁用而非隐藏） -->
<button :disabled="!can('operation')" class="btn btn-sm">
  行内操作
</button>

```

**UI 表达规则（三种状态）：**

| 场景 | 处理方式 | 实现 |
| --- | --- | --- |
| 该角色**永远**无此权限 | **隐藏**元素 | `x-show="can('op')"` |
| 更高角色有此权限，当前角色可升级 | **禁用** + tooltip | `:disabled` + DaisyUI tooltip |
| 需要临时提权（如 2FA 确认） | **显示**但点击时拦截 | `@click` 中增加权限门控 |

### Layer 3: 数据级（Data-Level）

控制每个角色能看到哪些记录/数据。

**实现方式：**

```javascript
// 根据 currentRole 过滤 mock 数据数组
filterDataByRole(data) {
  if (this.currentRole === 'admin') return data;
  if (this.currentRole === 'member') return data.filter(d => d.team === 'my_team' || d.isPublic);
  return data.filter(d => d.isPublic);
}

```

**数据级权限示例：**

- Admin：看到所有用户、所有项目、所有记录
- Member：看到自己团队的数据 + 公开数据
- Viewer：仅看到公开记录
- 数据条数变化应在 UI 上有明确反馈（如表格行数、统计数字联动更新）

---

## 3. 权限表达规则（Permission Expression Rules）

每个受限元素的决策树：

```
该元素是否属于主工作流（MAIN workflow）？
  ├── 是 → 显示为 DISABLED + tooltip: "需要 [Admin] 权限"
  │         实现: class="btn btn-disabled" + DaisyUI tooltip
  │         示例: 成员页面的"删除项目"按钮
  │
  └── 否（额外/高级功能） → 
      用户是否有可能获得此权限？
        ├── 是 → 显示 UPGRADE CTA: "升级到 Pro 解锁此功能"
        │         实现: 带升级引导的 card 或 banner
        │
        └── 否 → 完全隐藏（从 DOM 移除）
                  实现: x-show="false" 或不渲染

```

**tooltip 文案模板：**

```html
<!-- 禁用状态的 tooltip -->
<div class="tooltip" data-tip="需要管理员权限才能执行此操作">
  <button class="btn btn-disabled btn-sm">删除用户</button>
</div>

<!-- 升级引导 -->
<div class="tooltip" data-tip="升级到专业版解锁批量导出">
  <button class="btn btn-disabled btn-sm btn-outline">
    <svg><!-- lock icon --></svg>
    批量导出
  </button>
</div>

```

**视觉层次规则：**

- 禁用按钮保持原始位置（不重排布局）
- 使用 `opacity-50` + `cursor-not-allowed` 样式
- tooltip 使用 DaisyUI 内置 tooltip 组件，方向自适应

---

## 4. 角色切换器实现模板（Role Switcher）

角色切换器是权限演示的核心交互组件，固定在页面右上角。

```html
<!-- Role Switcher Component - 固定右上角 -->
<div class="fixed top-4 right-4 z-50 flex items-center gap-2 bg-base-100 p-2 rounded-lg shadow-lg border">
  <span class="text-xs font-bold text-base-content/60">角色:</span>
  <select x-model="currentRole" class="select select-xs select-bordered">
    <template x-for="role in roles" :key="role.id">
      <option :value="role.id" x-text="role.label"></option>
    </template>
  </select>
  <div class="badge badge-sm" :class="roleBadgeClass" x-text="roleLabel"></div>
</div>

```

**角色切换器行为规则：**

- 切换角色时，页面内容**即时**更新（无需刷新）
- 如果切换后当前页面不可访问 → 自动跳转到 dashboard
- Badge 颜色按角色级别变化：Admin=`badge-primary`，Member=`badge-secondary`，Viewer=`badge-accent`
- 切换时添加短暂的 transition 动画，让用户感知变化

**Badge 样式计算：**

```javascript
get roleBadgeClass() {
  const classes = {
    admin: 'badge-primary',
    member: 'badge-secondary',
    viewer: 'badge-accent',
    guest: 'badge-ghost'
  };
  return classes[this.currentRole] || 'badge-ghost';
},

get roleLabel() {
  return this.roles.find(r => r.id === this.currentRole)?.label || '未知';
}

```

---

## 5. 权限检查函数（Alpine.js Permission Functions）

以下为完整的权限系统实现模板，直接嵌入 Alpine.js `x-data` 中：

```javascript
// ===== 核心权限系统 =====
roles: [
  { id: 'admin', label: '管理员', level: 3 },
  { id: 'member', label: '成员', level: 2 },
  { id: 'viewer', label: '访客', level: 1 }
],
currentRole: 'admin',

// 页面访问权限检查
canAccess(page) {
  const pageAccess = {
    // 定义每个页面的最低角色级别
    dashboard: 1,     // 所有角色可访问
    projects: 1,      // 所有角色可访问
    settings: 2,      // Member 及以上
    team: 3,          // 仅 Admin
    billing: 3        // 仅 Admin
  };
  const currentLevel = this.roles.find(r => r.id === this.currentRole)?.level || 0;
  return currentLevel >= (pageAccess[page] || 999);
},

// 操作权限检查
can(operation) {
  const permissions = {
    admin: ['create', 'read', 'update', 'delete', 'manage_users', 'manage_billing', 'export', 'bulk_action'],
    member: ['create', 'read', 'update', 'delete_own', 'export'],
    viewer: ['read']
  };
  return permissions[this.currentRole]?.includes(operation) || false;
},

// 基于角色的数据过滤
filterDataByRole(data) {
  if (this.currentRole === 'admin') return data;
  if (this.currentRole === 'member') return data.filter(d => d.team === 'my_team' || d.isPublic);
  return data.filter(d => d.isPublic);
},

// 角色切换时的联动处理
switchRole(newRole) {
  this.currentRole = newRole;
  // 如果当前页面不可访问，自动跳转
  if (!this.canAccess(this.page)) {
    this.page = 'dashboard';
  }
}

```

**扩展：细粒度权限（当原型需要更复杂的权限矩阵时）：**

```javascript
// 资源级别权限（resource-level）
canOnResource(operation, resource) {
  // Admin 可操作任何资源
  if (this.currentRole === 'admin') return true;
  // Member 只能操作自己创建的资源
  if (this.currentRole === 'member') {
    if (['update', 'delete'].includes(operation)) {
      return resource.createdBy === 'current_user';
    }
    return this.can(operation);
  }
  return this.can(operation);
}

```

---

## 6. 403 无权限页面模板

当用户通过角色切换器切换到无权访问当前页面的角色时显示：

```html
<!-- Permission Denied View - 403 无权限页面 -->
<div x-show="currentState === '403'" class="flex flex-col items-center justify-center min-h-[400px] text-center">
  <!-- 锁定图标 -->
  <svg class="w-24 h-24 text-base-content/30 mb-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" 
          d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z"/>
  </svg>
  <h2 class="text-xl font-bold mb-2">无权访问此页面</h2>
  <p class="text-base-content/60 mb-4">
    您当前的角色（<span class="font-semibold" x-text="roleLabel"></span>）无法访问此内容
  </p>
  <!-- 操作按钮 -->
  <div class="flex gap-2">
    <button class="btn btn-primary btn-sm" @click="/* 模拟申请权限流程 */">
      申请权限
    </button>
    <button class="btn btn-ghost btn-sm" @click="page='dashboard'">
      返回首页
    </button>
  </div>
  
  <!-- 权限说明卡片 -->
  <div class="card bg-base-200 mt-6 max-w-sm">
    <div class="card-body text-left text-sm">
      <h3 class="font-bold text-sm">此页面需要以下权限：</h3>
      <ul class="list-disc list-inside text-base-content/60">
        <li>最低角色：<span class="badge badge-xs badge-primary">管理员</span></li>
        <li>联系组织管理员获取权限提升</li>
      </ul>
    </div>
  </div>
</div>

```

---

## 7. 导航过滤模板（Navigation Filtering）

侧边栏导航根据角色动态显示/隐藏菜单项：

```html
<!-- Sidebar with role-based filtering -->
<div class="drawer-side">
  <ul class="menu p-4 w-60 bg-base-200 min-h-full">
    <!-- Logo / 品牌区 -->
    <li class="menu-title">
      <span class="text-lg font-bold">ProtoApp</span>
    </li>
    <!-- 动态导航项 - 根据角色过滤 -->
    <template x-for="item in navItems" :key="item.id">
      <li x-show="canAccess(item.page)">
        <a :class="{'active': page === item.page}" 
           @click="page = item.page">
          <!-- x-html 等价于 innerHTML：仅允许渲染骨架内硬编码的可信 SVG 常量。
               任何来自用户输入或外部数据的内容必须用 x-text，否则构成 XSS。-->
          <span x-html="item.icon"></span>
          <span x-text="item.label"></span>
          <!-- Admin-only 标记 -->
          <span x-show="item.adminOnly" class="badge badge-xs badge-warning">Admin</span>
        </a>
      </li>
    </template>
  </ul>
</div>

```

**导航数据结构：**

```javascript
navItems: [
  { id: 'dashboard', page: 'dashboard', label: '仪表盘', icon: '📊', adminOnly: false },
  { id: 'projects', page: 'projects', label: '项目', icon: '📁', adminOnly: false },
  { id: 'settings', page: 'settings', label: '设置', icon: '⚙️', adminOnly: false },
  { id: 'team', page: 'team', label: '团队管理', icon: '👥', adminOnly: true },
  { id: 'billing', page: 'billing', label: '计费', icon: '💳', adminOnly: true }
]

```

---

## 8. 权限与其他模块的联动

### 与状态流转模块联动

- 某些状态转换仅特定角色可执行（如"审批通过"需 Admin）
- 状态流转按钮同时受操作权限和状态权限约束

### 与 Mock 数据模块联动

- Mock 数据中需包含 `createdBy`、`team`、`isPublic` 字段
- 切换角色时数据列表应即时过滤更新

### 与多状态模块联动

- 空状态（Empty State）对不同角色显示不同 CTA- Admin 看到："创建第一个项目"
- Viewer 看到："暂无可查看的内容"

---

## VERIFICATION

```yaml
module: role-permission
checklist:
  - 默认 RBAC 层级定义完整（5 级）
  - 自定义角色映射规则清晰
  - 三层权限模型（页面级/操作级/数据级）均有实现模板
  - 权限表达决策树覆盖三种状态（隐藏/禁用/拦截）
  - Role Switcher 组件固定右上角，含 badge 颜色联动
  - Alpine.js 权限函数完整可用（canAccess / can / filterDataByRole）
  - 403 页面模板含图标、说明文字、操作按钮
  - 导航过滤模板根据角色动态显示/隐藏
  - 角色切换后自动跳转逻辑已定义
  - 与其他模块（状态流转/Mock数据/多状态）的联动规则已说明
  - 代码示例使用 DaisyUI + Alpine.js 一致技术栈
  - 中文 UI 文案完整（tooltip / 403页面 / 导航标签）

integration_points:
  - state-flow: 状态转换的角色门控
  - mock-data: 数据字段需含权限相关属性
  - multi-state: 空状态 CTA 按角色差异化
  - navigation: 菜单项根据 canAccess 过滤

```

---

## Phase 4: 页面生成指令

# Page Generator 模块 — 全状态页面生成指令

> **模块职责**: 指导 LLM 为每种页面类型的每个适用状态生成完整的 HTML 变体。结合 State Matrix 模块确定的 ApplicableStates[] 列表，逐状态输出可交互的 DaisyUI + Alpine.js 原型代码。

---

## 1. 生成总则

### 1.1 核心约束

1. 每个状态变体 **必须** 包裹在 `<div x-show="currentState === 'stateName'" x-transition>` 中
2. **仅使用 DaisyUI 组件类**: `btn`, `card`, `alert`, `table`, `skeleton`, `badge`, `toast`, `modal`, `menu`, `dropdown`, `tabs`, `form-control`, `input`, `select`, `textarea`, `toggle`, `checkbox`, `radio`, `progress`, `stat`
3. 所有文本内容使用**占位中文**
4. 使用 Alpine.js `x-data` 管理本地组件状态
5. 角色相关元素使用 `x-show="can('permission')"` 或 `:disabled="!can('permission')"`
6. 状态切换使用 `x-transition` 类实现平滑过渡
7. 每个页面必须包含顶部状态切换器（参见 State Matrix 模块 §6.1）

### 1.2 全局 Alpine.js 数据结构

```html
<div x-data="{
  currentState: 'ideal',
  currentRole: 'admin',
  errors: {},
  can(permission) {
    const rolePerms = {
      admin: ['view', 'edit', 'delete', 'manage_team', 'manage_settings'],
      editor: ['view', 'edit'],
      viewer: ['view']
    };
    return rolePerms[this.currentRole]?.includes(permission) ?? false;
  }
}">
  <!-- 所有状态容器 -->
</div>

```

---

## 2. Web 页面类型生成模板（6 种）

### 2.1 Dashboard（仪表盘）

Ideal 状态

```html
<div x-show="currentState === 'ideal'" x-transition>
  <!-- 页面标题 -->
  <div class="flex justify-between items-center mb-6">
    <h1 class="text-2xl font-bold">工作台</h1>
    <button class="btn btn-primary btn-sm" x-show="can('edit')">+ 新建项目</button>
  </div>

  <!-- 统计卡片网格 2×2 -->
  <div class="grid grid-cols-2 lg:grid-cols-4 gap-4 mb-6">
    <div class="stat bg-base-100 shadow rounded-box">
      <div class="stat-title">总项目数</div>
      <div class="stat-value text-primary">128</div>
      <div class="stat-desc">较上周 ↑12%</div>
    </div>
    <div class="stat bg-base-100 shadow rounded-box">
      <div class="stat-title">活跃用户</div>
      <div class="stat-value text-secondary">1,024</div>
      <div class="stat-desc">较上周 ↑5%</div>
    </div>
    <div class="stat bg-base-100 shadow rounded-box">
      <div class="stat-title">待处理任务</div>
      <div class="stat-value text-accent">23</div>
      <div class="stat-desc">3 项紧急</div>
    </div>
    <div class="stat bg-base-100 shadow rounded-box">
      <div class="stat-title">本月收入</div>
      <div class="stat-value">¥89,400</div>
      <div class="stat-desc">目标完成 78%</div>
    </div>
  </div>

  <!-- 最近活动列表 -->
  <div class="card bg-base-100 shadow mb-6">
    <div class="card-body">
      <h2 class="card-title text-lg">最近活动</h2>
      <div class="divide-y">
        <div class="py-3 flex justify-between items-center">
          <div>
            <p class="font-medium">张三 更新了项目「后台重构」</p>
            <p class="text-sm text-base-content/50">2 分钟前</p>
          </div>
          <span class="badge badge-info">更新</span>
        </div>
        <div class="py-3 flex justify-between items-center">
          <div>
            <p class="font-medium">李四 提交了新工单 #1024</p>
            <p class="text-sm text-base-content/50">15 分钟前</p>
          </div>
          <span class="badge badge-success">新建</span>
        </div>
        <div class="py-3 flex justify-between items-center">
          <div>
            <p class="font-medium">系统 完成了数据备份</p>
            <p class="text-sm text-base-content/50">1 小时前</p>
          </div>
          <span class="badge badge-ghost">系统</span>
        </div>
      </div>
    </div>
  </div>

  <!-- 快捷操作按钮 -->
  <div class="flex gap-2">
    <button class="btn btn-outline btn-sm">查看报表</button>
    <button class="btn btn-outline btn-sm">导出数据</button>
    <button class="btn btn-outline btn-sm" x-show="can('manage_team')">团队管理</button>
  </div>
</div>

```

Loading 状态

```html
<div x-show="currentState === 'loading'" x-transition>
  <div class="flex justify-between items-center mb-6">
    <div class="skeleton h-8 w-32"></div>
    <div class="skeleton h-8 w-24"></div>
  </div>

  <!-- 统计卡片骨架 -->
  <div class="grid grid-cols-2 lg:grid-cols-4 gap-4 mb-6">
    <div class="stat bg-base-100 shadow rounded-box">
      <div class="skeleton h-4 w-16 mb-2"></div>
      <div class="skeleton h-8 w-20 mb-1"></div>
      <div class="skeleton h-3 w-24"></div>
    </div>
    <div class="stat bg-base-100 shadow rounded-box">
      <div class="skeleton h-4 w-16 mb-2"></div>
      <div class="skeleton h-8 w-20 mb-1"></div>
      <div class="skeleton h-3 w-24"></div>
    </div>
    <div class="stat bg-base-100 shadow rounded-box">
      <div class="skeleton h-4 w-16 mb-2"></div>
      <div class="skeleton h-8 w-20 mb-1"></div>
      <div class="skeleton h-3 w-24"></div>
    </div>
    <div class="stat bg-base-100 shadow rounded-box">
      <div class="skeleton h-4 w-16 mb-2"></div>
      <div class="skeleton h-8 w-20 mb-1"></div>
      <div class="skeleton h-3 w-24"></div>
    </div>
  </div>

  <!-- 活动列表骨架 -->
  <div class="card bg-base-100 shadow">
    <div class="card-body">
      <div class="skeleton h-6 w-24 mb-4"></div>
      <div class="space-y-4">
        <div class="flex justify-between"><div class="skeleton h-4 w-3/4"></div><div class="skeleton h-5 w-12"></div></div>
        <div class="flex justify-between"><div class="skeleton h-4 w-2/3"></div><div class="skeleton h-5 w-12"></div></div>
        <div class="flex justify-between"><div class="skeleton h-4 w-1/2"></div><div class="skeleton h-5 w-12"></div></div>
      </div>
    </div>
  </div>
</div>

```

Empty 状态

```html
<div x-show="currentState === 'empty'" x-transition>
  <div class="flex flex-col items-center justify-center py-16 text-center">
    <svg class="w-24 h-24 text-base-content/30 mb-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-6 0a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1m-6 0h6"/>
    </svg>
    <h3 class="text-lg font-semibold text-base-content/70">欢迎使用！</h3>
    <p class="text-sm text-base-content/50 mt-1 max-w-sm">
      您的工作台还没有任何数据。开始创建您的第一个项目，数据将在这里汇总展示。
    </p>
    <div class="flex gap-2 mt-6">
      <button class="btn btn-primary">创建第一个项目</button>
      <button class="btn btn-ghost">查看使用指南</button>
    </div>
  </div>
</div>

```

Error 状态

```html
<div x-show="currentState === 'error'" x-transition>
  <div class="alert alert-error shadow-lg max-w-lg mx-auto mt-12">
    <svg xmlns="http://www.w3.org/2000/svg" class="stroke-current shrink-0 h-6 w-6" fill="none" viewBox="0 0 24 24">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z"/>
    </svg>
    <div>
      <h3 class="font-bold">加载失败</h3>
      <p class="text-sm">无法获取仪表盘数据，请检查网络连接后重试。</p>
    </div>
    <button class="btn btn-sm btn-ghost" @click="currentState = 'loading'">重试</button>
  </div>
</div>

```

No-Permission (403) 状态

```html
<div x-show="currentState === 'no-permission'" x-transition>
  <div class="flex flex-col items-center justify-center py-16 text-center">
    <div class="w-16 h-16 rounded-full bg-warning/10 flex items-center justify-center mb-4">
      <svg class="w-8 h-8 text-warning" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z"/>
      </svg>
    </div>
    <h3 class="text-lg font-semibold">需要管理员权限查看此面板</h3>
    <p class="text-sm text-base-content/50 mt-1">您当前的角色无法访问仪表盘数据</p>
    <div class="flex gap-2 mt-6">
      <button class="btn btn-primary btn-sm">申请权限</button>
      <button class="btn btn-ghost btn-sm">联系管理员</button>
    </div>
  </div>
</div>

```

Partial 状态

```html
<div x-show="currentState === 'partial'" x-transition>
  <div class="flex justify-between items-center mb-6">
    <h1 class="text-2xl font-bold">工作台</h1>
  </div>

  <!-- 部分成功的卡片 -->
  <div class="grid grid-cols-2 lg:grid-cols-4 gap-4 mb-6">
    <div class="stat bg-base-100 shadow rounded-box">
      <div class="stat-title">总项目数</div>
      <div class="stat-value text-primary">128</div>
      <div class="stat-desc">较上周 ↑12%</div>
    </div>
    <div class="stat bg-base-100 shadow rounded-box">
      <div class="stat-title">活跃用户</div>
      <div class="stat-value text-secondary">1,024</div>
      <div class="stat-desc">较上周 ↑5%</div>
    </div>
    <!-- 失败的卡片 -->
    <div class="stat bg-base-100 shadow rounded-box border border-error/30">
      <div class="flex items-center gap-2 text-error">
        <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-2.5L13.732 4c-.77-.833-1.964-.833-2.732 0L4.082 16.5c-.77.833.192 2.5 1.732 2.5z"/>
        </svg>
        <span class="text-sm">加载失败</span>
      </div>
      <button class="btn btn-xs btn-ghost text-error mt-2">重试</button>
    </div>
    <div class="stat bg-base-100 shadow rounded-box border border-error/30">
      <div class="flex items-center gap-2 text-error">
        <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-2.5L13.732 4c-.77-.833-1.964-.833-2.732 0L4.082 16.5c-.77.833.192 2.5 1.732 2.5z"/>
        </svg>
        <span class="text-sm">服务不可用</span>
      </div>
      <button class="btn btn-xs btn-ghost text-error mt-2">重试</button>
    </div>
  </div>

  <!-- 活动列表正常加载 -->
  <div class="card bg-base-100 shadow">
    <div class="card-body">
      <h2 class="card-title text-lg">最近活动</h2>
      <p class="text-sm text-base-content/50">此区域正常加载</p>
    </div>
  </div>
</div>

```

---

### 2.2 Data Table / List（数据列表）

Ideal 状态

```html
<div x-show="currentState === 'ideal'" x-transition>
  <!-- 顶部工具栏 -->
  <div class="flex justify-between items-center mb-4">
    <h1 class="text-2xl font-bold">项目列表</h1>
    <div class="flex gap-2">
      <input type="text" placeholder="搜索项目…" class="input input-bordered input-sm w-64"/>
      <button class="btn btn-primary btn-sm" x-show="can('edit')">+ 新建</button>
    </div>
  </div>

  <!-- 数据表格 -->
  <div class="overflow-x-auto">
    <table class="table table-zebra">
      <thead>
        <tr>
          <th><input type="checkbox" class="checkbox checkbox-sm"/></th>
          <th>项目名称</th>
          <th>负责人</th>
          <th>状态</th>
          <th>更新时间</th>
          <th>操作</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td><input type="checkbox" class="checkbox checkbox-sm"/></td>
          <td class="font-medium">电商平台重构</td>
          <td>张三</td>
          <td><span class="badge badge-success badge-sm">进行中</span></td>
          <td>2024-01-15</td>
          <td>
            <button class="btn btn-ghost btn-xs" x-show="can('edit')">编辑</button>
            <button class="btn btn-ghost btn-xs text-error" x-show="can('delete')">删除</button>
          </td>
        </tr>
        <tr>
          <td><input type="checkbox" class="checkbox checkbox-sm"/></td>
          <td class="font-medium">用户中心改版</td>
          <td>李四</td>
          <td><span class="badge badge-warning badge-sm">待审核</span></td>
          <td>2024-01-14</td>
          <td>
            <button class="btn btn-ghost btn-xs" x-show="can('edit')">编辑</button>
            <button class="btn btn-ghost btn-xs text-error" x-show="can('delete')">删除</button>
          </td>
        </tr>
        <tr>
          <td><input type="checkbox" class="checkbox checkbox-sm"/></td>
          <td class="font-medium">支付系统升级</td>
          <td>王五</td>
          <td><span class="badge badge-info badge-sm">规划中</span></td>
          <td>2024-01-13</td>
          <td>
            <button class="btn btn-ghost btn-xs" x-show="can('edit')">编辑</button>
            <button class="btn btn-ghost btn-xs text-error" x-show="can('delete')">删除</button>
          </td>
        </tr>
        <!-- 更多行... -->
      </tbody>
    </table>
  </div>

  <!-- 分页 -->
  <div class="flex justify-between items-center mt-4">
    <span class="text-sm text-base-content/50">共 56 条记录，第 1/6 页</span>
    <div class="join">
      <button class="join-item btn btn-sm btn-disabled">«</button>
      <button class="join-item btn btn-sm btn-active">1</button>
      <button class="join-item btn btn-sm">2</button>
      <button class="join-item btn btn-sm">3</button>
      <button class="join-item btn btn-sm">»</button>
    </div>
  </div>
</div>

```

Loading 状态

```html
<div x-show="currentState === 'loading'" x-transition>
  <div class="flex justify-between items-center mb-4">
    <div class="skeleton h-8 w-32"></div>
    <div class="flex gap-2">
      <div class="skeleton h-8 w-64"></div>
      <div class="skeleton h-8 w-16"></div>
    </div>
  </div>

  <div class="overflow-x-auto">
    <table class="table">
      <thead>
        <tr>
          <th><div class="skeleton h-4 w-4"></div></th>
          <th><div class="skeleton h-4 w-20"></div></th>
          <th><div class="skeleton h-4 w-16"></div></th>
          <th><div class="skeleton h-4 w-12"></div></th>
          <th><div class="skeleton h-4 w-20"></div></th>
          <th><div class="skeleton h-4 w-12"></div></th>
        </tr>
      </thead>
      <tbody>
        <tr><td colspan="6"><div class="skeleton h-10 w-full"></div></td></tr>
        <tr><td colspan="6"><div class="skeleton h-10 w-full"></div></td></tr>
        <tr><td colspan="6"><div class="skeleton h-10 w-full"></div></td></tr>
        <tr><td colspan="6"><div class="skeleton h-10 w-full"></div></td></tr>
        <tr><td colspan="6"><div class="skeleton h-10 w-full"></div></td></tr>
        <tr><td colspan="6"><div class="skeleton h-10 w-full"></div></td></tr>
      </tbody>
    </table>
  </div>
</div>

```

Empty 状态

```html
<div x-show="currentState === 'empty'" x-transition>
  <div class="flex flex-col items-center justify-center py-16 text-center">
    <svg class="w-24 h-24 text-base-content/30 mb-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"/>
    </svg>
    <h3 class="text-lg font-semibold text-base-content/70">还没有任何项目</h3>
    <p class="text-sm text-base-content/50 mt-1 max-w-sm">
      创建您的第一个项目，开始管理和追踪工作进度。
    </p>
    <button class="btn btn-primary mt-6" x-show="can('edit')">创建第一个项目</button>
  </div>
</div>

```

Error 状态

```html
<div x-show="currentState === 'error'" x-transition>
  <div class="flex flex-col items-center justify-center py-16">
    <div class="alert alert-error shadow-lg max-w-md">
      <svg xmlns="http://www.w3.org/2000/svg" class="stroke-current shrink-0 h-6 w-6" fill="none" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z"/>
      </svg>
      <div>
        <h3 class="font-bold">无法加载列表</h3>
        <p class="text-sm">服务器响应异常，请稍后重试。</p>
      </div>
      <button class="btn btn-sm btn-ghost" @click="currentState = 'loading'">重试</button>
    </div>
  </div>
</div>

```

Search No Results 状态（Empty 子变体）

```html
<div x-show="currentState === 'empty' && searchQuery" x-transition>
  <div class="flex flex-col items-center justify-center py-16 text-center">
    <svg class="w-20 h-20 text-base-content/30 mb-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"/>
    </svg>
    <h3 class="text-lg font-semibold text-base-content/70">未找到匹配 '关键词' 的结果</h3>
    <p class="text-sm text-base-content/50 mt-1">请尝试不同的搜索词或清除筛选条件</p>
    <button class="btn btn-outline btn-sm mt-4" @click="searchQuery = ''">清除筛选条件</button>
  </div>
</div>

```

---

### 2.3 Detail View（详情页）

Ideal 状态

```html
<div x-show="currentState === 'ideal'" x-transition>
  <!-- 详情头部 -->
  <div class="flex justify-between items-start mb-6">
    <div>
      <div class="flex items-center gap-2 mb-1">
        <button class="btn btn-ghost btn-sm">← 返回</button>
        <span class="badge badge-success">进行中</span>
      </div>
      <h1 class="text-2xl font-bold">电商平台重构项目</h1>
      <p class="text-sm text-base-content/50 mt-1">创建于 2024-01-10 · 负责人: 张三</p>
    </div>
    <div class="flex gap-2">
      <button class="btn btn-outline btn-sm" x-show="can('edit')">编辑</button>
      <button class="btn btn-error btn-sm" x-show="can('delete')">删除</button>
    </div>
  </div>

  <!-- 标签页内容 -->
  <div class="tabs tabs-bordered mb-4">
    <a class="tab tab-active">概览</a>
    <a class="tab">任务</a>
    <a class="tab">文件</a>
    <a class="tab">活动记录</a>
  </div>

  <div class="card bg-base-100 shadow">
    <div class="card-body">
      <h2 class="card-title text-lg">项目概览</h2>
      <p class="text-base-content/70">
        本项目旨在对现有电商平台进行全面的技术架构重构，包括前端框架升级、后端微服务化改造、数据库优化等。
      </p>
      <div class="grid grid-cols-2 gap-4 mt-4">
        <div><span class="text-sm text-base-content/50">开始日期</span><p class="font-medium">2024-01-10</p></div>
        <div><span class="text-sm text-base-content/50">截止日期</span><p class="font-medium">2024-06-30</p></div>
        <div><span class="text-sm text-base-content/50">优先级</span><p><span class="badge badge-error badge-sm">高</span></p></div>
        <div><span class="text-sm text-base-content/50">完成度</span><p><progress class="progress progress-primary w-32" value="45" max="100"></progress> 45%</p></div>
      </div>
    </div>
  </div>
</div>

```

Loading 状态

```html
<div x-show="currentState === 'loading'" x-transition>
  <div class="mb-6">
    <div class="flex items-center gap-2 mb-2">
      <div class="skeleton h-6 w-16"></div>
      <div class="skeleton h-5 w-14"></div>
    </div>
    <div class="skeleton h-8 w-64 mb-1"></div>
    <div class="skeleton h-4 w-48"></div>
  </div>
  <div class="skeleton h-10 w-full mb-4"></div>
  <div class="card bg-base-100 shadow">
    <div class="card-body space-y-3">
      <div class="skeleton h-6 w-24"></div>
      <div class="skeleton h-4 w-full"></div>
      <div class="skeleton h-4 w-3/4"></div>
      <div class="grid grid-cols-2 gap-4 mt-4">
        <div class="skeleton h-12 w-full"></div>
        <div class="skeleton h-12 w-full"></div>
        <div class="skeleton h-12 w-full"></div>
        <div class="skeleton h-12 w-full"></div>
      </div>
    </div>
  </div>
</div>

```

Error 状态

```html
<div x-show="currentState === 'error'" x-transition>
  <div class="flex flex-col items-center justify-center py-16">
    <div class="alert alert-error shadow-lg max-w-md">
      <svg xmlns="http://www.w3.org/2000/svg" class="stroke-current shrink-0 h-6 w-6" fill="none" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z"/>
      </svg>
      <div>
        <h3 class="font-bold">无法加载详情</h3>
        <p class="text-sm">请求的资源不存在或加载失败。</p>
      </div>
    </div>
    <button class="btn btn-ghost btn-sm mt-4">← 返回列表</button>
  </div>
</div>

```

No-Permission (403) 状态

```html
<div x-show="currentState === 'no-permission'" x-transition>
  <div class="flex flex-col items-center justify-center py-16 text-center">
    <div class="w-16 h-16 rounded-full bg-warning/10 flex items-center justify-center mb-4">
      <svg class="w-8 h-8 text-warning" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z"/>
      </svg>
    </div>
    <h3 class="text-lg font-semibold">您无权查看此记录</h3>
    <p class="text-sm text-base-content/50 mt-1">此内容仅对特定角色开放</p>
    <div class="flex gap-2 mt-6">
      <button class="btn btn-primary btn-sm">申请访问权限</button>
      <button class="btn btn-ghost btn-sm">← 返回</button>
    </div>
  </div>
</div>

```

Stale Data 状态

```html
<div x-show="currentState === 'stale-data'" x-transition>
  <!-- 顶部过期提示 -->
  <div class="alert alert-warning mb-4">
    <svg class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"/>
    </svg>
    <span>数据可能已过期，上次更新于 5 分钟前</span>
    <button class="btn btn-sm btn-warning" @click="currentState = 'loading'">刷新</button>
  </div>

  <!-- 正常内容（带半透明遮罩暗示可能过期） -->
  <div class="opacity-75">
    <h1 class="text-2xl font-bold mb-4">电商平台重构项目</h1>
    <div class="card bg-base-100 shadow">
      <div class="card-body">
        <p class="text-base-content/70">项目内容加载中（数据可能不是最新）...</p>
      </div>
    </div>
  </div>
</div>

```

---

### 2.4 Form / Create-Edit（表单页）

Ideal 状态

```html
<div x-show="currentState === 'ideal'" x-transition>
  <div class="max-w-2xl mx-auto">
    <h1 class="text-2xl font-bold mb-6">创建新项目</h1>
    <form class="space-y-4">
      <div class="form-control w-full">
        <label class="label"><span class="label-text">项目名称 <span class="text-error">*</span></span></label>
        <input type="text" placeholder="请输入项目名称" class="input input-bordered w-full"/>
      </div>

      <div class="form-control w-full">
        <label class="label"><span class="label-text">项目描述</span></label>
        <textarea class="textarea textarea-bordered w-full h-24" placeholder="请输入项目描述…"></textarea>
      </div>

      <div class="grid grid-cols-2 gap-4">
        <div class="form-control">
          <label class="label"><span class="label-text">开始日期 <span class="text-error">*</span></span></label>
          <input type="date" class="input input-bordered"/>
        </div>
        <div class="form-control">
          <label class="label"><span class="label-text">截止日期</span></label>
          <input type="date" class="input input-bordered"/>
        </div>
      </div>

      <div class="form-control w-full">
        <label class="label"><span class="label-text">优先级</span></label>
        <select class="select select-bordered w-full">
          <option disabled selected>请选择优先级</option>
          <option>高</option>
          <option>中</option>
          <option>低</option>
        </select>
      </div>

      <div class="form-control">
        <label class="label"><span class="label-text">通知相关人员</span></label>
        <label class="label cursor-pointer justify-start gap-2">
          <input type="checkbox" class="toggle toggle-primary"/>
          <span class="label-text">创建后发送通知</span>
        </label>
      </div>

      <!-- 操作按钮 -->
      <div class="flex justify-end gap-2 pt-4 border-t">
        <button type="button" class="btn btn-ghost">取消</button>
        <button type="submit" class="btn btn-primary" :disabled="!can('edit')">提交</button>
      </div>
    </form>
  </div>
</div>

```

Loading 状态

```html
<div x-show="currentState === 'loading'" x-transition>
  <div class="max-w-2xl mx-auto space-y-4">
    <div class="skeleton h-8 w-32 mb-6"></div>
    <div class="space-y-4">
      <div><div class="skeleton h-4 w-20 mb-2"></div><div class="skeleton h-10 w-full"></div></div>
      <div><div class="skeleton h-4 w-20 mb-2"></div><div class="skeleton h-24 w-full"></div></div>
      <div class="grid grid-cols-2 gap-4">
        <div><div class="skeleton h-4 w-20 mb-2"></div><div class="skeleton h-10 w-full"></div></div>
        <div><div class="skeleton h-4 w-20 mb-2"></div><div class="skeleton h-10 w-full"></div></div>
      </div>
      <div><div class="skeleton h-4 w-20 mb-2"></div><div class="skeleton h-10 w-full"></div></div>
    </div>
  </div>
</div>

```

Validation Error 状态

```html
<div x-show="currentState === 'validation-error'" x-transition>
  <div class="max-w-2xl mx-auto">
    <h1 class="text-2xl font-bold mb-4">创建新项目</h1>
    <!-- 顶部错误摘要 -->
    <div class="alert alert-error mb-4">
      <svg class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-2.5L13.732 4c-.77-.833-1.964-.833-2.732 0L4.082 16.5c-.77.833.192 2.5 1.732 2.5z"/>
      </svg>
      <span>请修正以下 2 个错误后重新提交</span>
    </div>

    <form class="space-y-4">
      <!-- 错误字段 1 -->
      <div class="form-control w-full">
        <label class="label"><span class="label-text">项目名称 <span class="text-error">*</span></span></label>
        <input type="text" class="input input-bordered input-error w-full" value=""/>
        <label class="label"><span class="label-text-alt text-error">项目名称不能为空</span></label>
      </div>

      <div class="form-control w-full">
        <label class="label"><span class="label-text">项目描述</span></label>
        <textarea class="textarea textarea-bordered w-full h-24">这是一段有效的描述</textarea>
      </div>

      <div class="grid grid-cols-2 gap-4">
        <!-- 错误字段 2 -->
        <div class="form-control">
          <label class="label"><span class="label-text">开始日期 <span class="text-error">*</span></span></label>
          <input type="date" class="input input-bordered input-error"/>
          <label class="label"><span class="label-text-alt text-error">请选择开始日期</span></label>
        </div>
        <div class="form-control">
          <label class="label"><span class="label-text">截止日期</span></label>
          <input type="date" class="input input-bordered" value="2024-06-30"/>
        </div>
      </div>

      <div class="flex justify-end gap-2 pt-4 border-t">
        <button type="button" class="btn btn-ghost">取消</button>
        <button type="submit" class="btn btn-primary">提交</button>
      </div>
    </form>
  </div>
</div>

```

Success 状态

```html
<div x-show="currentState === 'success'" x-transition>
  <div class="max-w-2xl mx-auto">
    <!-- 保留表单内容表示已完成 -->
    <h1 class="text-2xl font-bold mb-6">创建新项目</h1>
    <p class="text-base-content/50">表单已成功提交，即将跳转至项目详情…</p>
  </div>

  <!-- 成功 Toast -->
  <div class="toast toast-end">
    <div class="alert alert-success">
      <svg class="stroke-current shrink-0 h-6 w-6" fill="none" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"/>
      </svg>
      <span>保存成功 — 正在跳转到项目详情</span>
    </div>
  </div>
</div>

```

Timeout 状态

```html
<div x-show="currentState === 'timeout'" x-transition>
  <div class="max-w-2xl mx-auto">
    <h1 class="text-2xl font-bold mb-4">创建新项目</h1>
    <div class="alert alert-warning mb-4">
      <svg class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"/>
      </svg>
      <div>
        <p class="font-medium">提交超时，请重试</p>
        <p class="text-sm">您的数据已保留，可以直接重新提交。</p>
      </div>
      <button class="btn btn-sm btn-warning" @click="currentState = 'loading'">重新提交</button>
    </div>

    <!-- 表单数据保留 -->
    <form class="space-y-4 opacity-75">
      <div class="form-control w-full">
        <label class="label"><span class="label-text">项目名称</span></label>
        <input type="text" class="input input-bordered w-full" value="电商平台重构" disabled/>
      </div>
      <!-- 其他字段保留... -->
    </form>
  </div>
</div>

```

Session Expired 状态

```html
<div x-show="currentState === 'session-expired'" x-transition>
  <!-- 全屏遮罩 Modal -->
  <div class="modal modal-open">
    <div class="modal-box text-center">
      <svg class="w-16 h-16 text-warning mx-auto mb-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z"/>
      </svg>
      <h3 class="font-bold text-lg">会话已过期</h3>
      <p class="py-4 text-base-content/70">您的登录会话已超时，请重新登录以继续操作。未保存的数据将在登录后恢复。</p>
      <div class="modal-action justify-center">
        <button class="btn btn-primary">重新登录</button>
      </div>
    </div>
    <div class="modal-backdrop bg-black/50"></div>
  </div>
</div>

```

---

### 2.5 Settings（设置页）

Ideal 状态

```html
<div x-show="currentState === 'ideal'" x-transition>
  <h1 class="text-2xl font-bold mb-6">设置</h1>
  <!-- 多标签设置 -->
  <div class="tabs tabs-bordered mb-6">
    <a class="tab tab-active">个人资料</a>
    <a class="tab">通知设置</a>
    <a class="tab">安全设置</a>
    <a class="tab" x-show="can('manage_settings')">计费管理</a>
  </div>

  <!-- 个人资料标签内容 -->
  <div class="card bg-base-100 shadow">
    <div class="card-body">
      <h2 class="card-title text-lg">个人资料</h2>
      <form class="space-y-4">
        <div class="flex items-center gap-4">
          <div class="avatar placeholder">
            <div class="bg-primary text-primary-content rounded-full w-16">
              <span class="text-xl">张</span>
            </div>
          </div>
          <button class="btn btn-outline btn-sm">更换头像</button>
        </div>
        <div class="form-control">
          <label class="label"><span class="label-text">显示名称</span></label>
          <input type="text" class="input input-bordered" value="张三"/>
        </div>
        <div class="form-control">
          <label class="label"><span class="label-text">邮箱地址</span></label>
          <input type="email" class="input input-bordered" value="zhangsan@example.com"/>
        </div>
        <div class="form-control">
          <label class="label"><span class="label-text">手机号码</span></label>
          <input type="tel" class="input input-bordered" value="138****1234"/>
        </div>
        <div class="flex justify-end pt-4">
          <button class="btn btn-primary" :disabled="!can('edit')">保存修改</button>
        </div>
      </form>
    </div>
  </div>
</div>

```

Loading 状态

```html
<div x-show="currentState === 'loading'" x-transition>
  <div class="skeleton h-8 w-16 mb-6"></div>
  <div class="skeleton h-10 w-full max-w-md mb-6"></div>
  <div class="card bg-base-100 shadow">
    <div class="card-body space-y-4">
      <div class="skeleton h-6 w-24"></div>
      <div class="flex items-center gap-4">
        <div class="skeleton w-16 h-16 rounded-full"></div>
        <div class="skeleton h-8 w-24"></div>
      </div>
      <div><div class="skeleton h-4 w-20 mb-2"></div><div class="skeleton h-10 w-full"></div></div>
      <div><div class="skeleton h-4 w-20 mb-2"></div><div class="skeleton h-10 w-full"></div></div>
      <div><div class="skeleton h-4 w-20 mb-2"></div><div class="skeleton h-10 w-full"></div></div>
    </div>
  </div>
</div>

```

Success 状态

```html
<div x-show="currentState === 'success'" x-transition>
  <!-- 正常设置页内容 + Toast -->
  <h1 class="text-2xl font-bold mb-6">设置</h1>
  <p class="text-base-content/50">设置内容已保存</p>
  <div class="toast toast-end">
    <div class="alert alert-success">
      <svg class="stroke-current shrink-0 h-6 w-6" fill="none" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"/>
      </svg>
      <span>设置已保存</span>
    </div>
  </div>
</div>

```

No-Permission (403) 状态

```html
<div x-show="currentState === 'no-permission'" x-transition>
  <h1 class="text-2xl font-bold mb-6">设置</h1>
  <div class="tabs tabs-bordered mb-6">
    <a class="tab tab-active">个人资料</a>
    <a class="tab">通知设置</a>
    <a class="tab">安全设置</a>
    <a class="tab">计费管理</a>
  </div>

  <!-- Tab 可见但内容被锁定 -->
  <div class="card bg-base-100 shadow">
    <div class="card-body">
      <div class="flex flex-col items-center justify-center py-8 text-center">
        <svg class="w-12 h-12 text-warning mb-3" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z"/>
        </svg>
        <h3 class="font-semibold">需要管理员权限修改此设置</h3>
        <p class="text-sm text-base-content/50 mt-1">请联系您的团队管理员获取权限</p>
        <button class="btn btn-outline btn-sm mt-4">联系管理员</button>
      </div>
    </div>
  </div>
</div>

```

Validation Error 状态

```html
<div x-show="currentState === 'validation-error'" x-transition>
  <h1 class="text-2xl font-bold mb-6">设置</h1>
  <div class="card bg-base-100 shadow">
    <div class="card-body">
      <h2 class="card-title text-lg">个人资料</h2>
      <form class="space-y-4">
        <div class="form-control">
          <label class="label"><span class="label-text">邮箱地址</span></label>
          <input type="email" class="input input-bordered input-error" value="invalid-email"/>
          <label class="label"><span class="label-text-alt text-error">请输入有效的邮箱地址</span></label>
        </div>
        <div class="form-control">
          <label class="label"><span class="label-text">手机号码</span></label>
          <input type="tel" class="input input-bordered input-error" value="123"/>
          <label class="label"><span class="label-text-alt text-error">手机号格式不正确</span></label>
        </div>
      </form>
    </div>
  </div>
</div>

```

---

### 2.6 Admin / Team Page（管理/团队页 — 仅 Web）

Ideal 状态

```html
<div x-show="currentState === 'ideal'" x-transition>
  <div class="flex justify-between items-center mb-6">
    <h1 class="text-2xl font-bold">团队管理</h1>
    <button class="btn btn-primary btn-sm" x-show="can('manage_team')">+ 邀请成员</button>
  </div>

  <!-- 成员列表 -->
  <div class="card bg-base-100 shadow">
    <div class="card-body p-0">
      <table class="table">
        <thead>
          <tr>
            <th>成员</th>
            <th>邮箱</th>
            <th>角色</th>
            <th>加入时间</th>
            <th x-show="can('manage_team')">操作</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td class="flex items-center gap-2">
              <div class="avatar placeholder"><div class="bg-primary text-primary-content rounded-full w-8"><span>张</span></div></div>
              <span class="font-medium">张三</span>
            </td>
            <td>zhangsan@example.com</td>
            <td>
              <select class="select select-bordered select-sm" x-show="can('manage_team')" :disabled="!can('manage_team')">
                <option selected>管理员</option>
                <option>编辑者</option>
                <option>查看者</option>
              </select>
              <span class="badge badge-primary badge-sm" x-show="!can('manage_team')">管理员</span>
            </td>
            <td>2024-01-01</td>
            <td x-show="can('manage_team')"><button class="btn btn-ghost btn-xs text-error">移除</button></td>
          </tr>
          <tr>
            <td class="flex items-center gap-2">
              <div class="avatar placeholder"><div class="bg-secondary text-secondary-content rounded-full w-8"><span>李</span></div></div>
              <span class="font-medium">李四</span>
            </td>
            <td>lisi@example.com</td>
            <td>
              <select class="select select-bordered select-sm" x-show="can('manage_team')">
                <option>管理员</option>
                <option selected>编辑者</option>
                <option>查看者</option>
              </select>
              <span class="badge badge-secondary badge-sm" x-show="!can('manage_team')">编辑者</span>
            </td>
            <td>2024-01-05</td>
            <td x-show="can('manage_team')"><button class="btn btn-ghost btn-xs text-error">移除</button></td>
          </tr>
          <tr>
            <td class="flex items-center gap-2">
              <div class="avatar placeholder"><div class="bg-accent text-accent-content rounded-full w-8"><span>王</span></div></div>
              <span class="font-medium">王五</span>
            </td>
            <td>wangwu@example.com</td>
            <td>
              <select class="select select-bordered select-sm" x-show="can('manage_team')">
                <option>管理员</option>
                <option>编辑者</option>
                <option selected>查看者</option>
              </select>
              <span class="badge badge-ghost badge-sm" x-show="!can('manage_team')">查看者</span>
            </td>
            <td>2024-01-10</td>
            <td x-show="can('manage_team')"><button class="btn btn-ghost btn-xs text-error">移除</button></td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</div>

```

Empty 状态

```html
<div x-show="currentState === 'empty'" x-transition>
  <h1 class="text-2xl font-bold mb-6">团队管理</h1>
  <div class="flex flex-col items-center justify-center py-16 text-center">
    <svg class="w-24 h-24 text-base-content/30 mb-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M17 20h5v-2a3 3 0 00-5.356-1.857M17 20H7m10 0v-2c0-.656-.126-1.283-.356-1.857M7 20H2v-2a3 3 0 015.356-1.857M7 20v-2c0-.656.126-1.283.356-1.857m0 0a5.002 5.002 0 019.288 0M15 7a3 3 0 11-6 0 3 3 0 016 0z"/>
    </svg>
    <h3 class="text-lg font-semibold text-base-content/70">还没有团队成员</h3>
    <p class="text-sm text-base-content/50 mt-1 max-w-sm">
      邀请您的同事加入团队，共同协作管理项目。
    </p>
    <button class="btn btn-primary mt-6" x-show="can('manage_team')">邀请第一位成员</button>
  </div>
</div>

```

No-Permission (403) 状态

```html
<div x-show="currentState === 'no-permission'" x-transition>
  <div class="flex flex-col items-center justify-center py-16 text-center">
    <div class="w-16 h-16 rounded-full bg-error/10 flex items-center justify-center mb-4">
      <svg class="w-8 h-8 text-error" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M18.364 18.364A9 9 0 005.636 5.636m12.728 12.728A9 9 0 015.636 5.636m12.728 12.728L5.636 5.636"/>
      </svg>
    </div>
    <h3 class="text-lg font-semibold">无权访问团队管理</h3>
    <p class="text-sm text-base-content/50 mt-1">此页面仅对管理员角色开放，非管理员用户无法查看。</p>
    <button class="btn btn-ghost btn-sm mt-4">← 返回首页</button>
  </div>
</div>

```

---

## 3. Mobile 页面类型生成模板（5 种）

### 3.1 Home Feed（首页/信息流）

Ideal 状态

```html
<div x-show="currentState === 'ideal'" x-transition>
  <!-- 顶部导航 -->
  <div class="navbar bg-base-100 shadow-sm sticky top-0 z-10">
    <div class="flex-1"><h1 class="text-lg font-bold px-2">首页</h1></div>
    <div class="flex-none">
      <button class="btn btn-ghost btn-circle"><span class="badge badge-primary badge-xs indicator-item"></span>🔔</button>
    </div>
  </div>

  <!-- 快捷操作 -->
  <div class="grid grid-cols-4 gap-2 p-4">
    <button class="btn btn-ghost flex-col h-auto py-3"><span class="text-2xl">📝</span><span class="text-xs">新建</span></button>
    <button class="btn btn-ghost flex-col h-auto py-3"><span class="text-2xl">📊</span><span class="text-xs">报表</span></button>
    <button class="btn btn-ghost flex-col h-auto py-3"><span class="text-2xl">👥</span><span class="text-xs">团队</span></button>
    <button class="btn btn-ghost flex-col h-auto py-3"><span class="text-2xl">⚙️</span><span class="text-xs">设置</span></button>
  </div>

  <!-- 信息流卡片 -->
  <div class="space-y-3 px-4 pb-4">
    <div class="card bg-base-100 shadow-sm">
      <div class="card-body p-4">
        <div class="flex items-center gap-2 mb-2">
          <div class="avatar placeholder"><div class="bg-primary text-primary-content rounded-full w-8"><span>张</span></div></div>
          <div><p class="text-sm font-medium">张三</p><p class="text-xs text-base-content/50">5 分钟前</p></div>
        </div>
        <p class="text-sm">完成了「用户中心」模块的代码审查，共修复 3 个问题。</p>
        <div class="flex gap-2 mt-2"><span class="badge badge-sm badge-ghost">代码审查</span></div>
      </div>
    </div>
    <div class="card bg-base-100 shadow-sm">
      <div class="card-body p-4">
        <div class="flex items-center gap-2 mb-2">
          <div class="avatar placeholder"><div class="bg-secondary text-secondary-content rounded-full w-8"><span>系</span></div></div>
          <div><p class="text-sm font-medium">系统通知</p><p class="text-xs text-base-content/50">1 小时前</p></div>
        </div>
        <p class="text-sm">您有 2 个待审批的请求需要处理。</p>
        <button class="btn btn-primary btn-xs mt-2">去处理</button>
      </div>
    </div>
  </div>
</div>

```

Loading 状态

```html
<div x-show="currentState === 'loading'" x-transition>
  <div class="navbar bg-base-100 shadow-sm sticky top-0 z-10">
    <div class="flex-1"><div class="skeleton h-6 w-16 mx-2"></div></div>
  </div>
  <div class="grid grid-cols-4 gap-2 p-4">
    <div class="skeleton h-16 w-full rounded-lg"></div>
    <div class="skeleton h-16 w-full rounded-lg"></div>
    <div class="skeleton h-16 w-full rounded-lg"></div>
    <div class="skeleton h-16 w-full rounded-lg"></div>
  </div>
  <div class="space-y-3 px-4">
    <div class="skeleton h-28 w-full rounded-lg"></div>
    <div class="skeleton h-28 w-full rounded-lg"></div>
    <div class="skeleton h-28 w-full rounded-lg"></div>
  </div>
</div>

```

Empty 状态

```html
<div x-show="currentState === 'empty'" x-transition>
  <div class="navbar bg-base-100 shadow-sm"><div class="flex-1"><h1 class="text-lg font-bold px-2">首页</h1></div></div>
  <div class="flex flex-col items-center justify-center py-16 text-center px-4">
    <svg class="w-20 h-20 text-base-content/30 mb-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M19 20H5a2 2 0 01-2-2V6a2 2 0 012-2h10a2 2 0 012 2v1m2 13a2 2 0 01-2-2V7m2 13a2 2 0 002-2V9a2 2 0 00-2-2h-2m-4-3H9M7 16h6M7 8h6v4H7V8z"/>
    </svg>
    <h3 class="text-lg font-semibold text-base-content/70">暂无动态</h3>
    <p class="text-sm text-base-content/50 mt-1">当团队有新活动时，将在这里展示。</p>
  </div>
</div>

```

Offline 状态

```html
<div x-show="currentState === 'offline'" x-transition>
  <!-- 离线横幅 -->
  <div class="w-full bg-warning text-warning-content px-4 py-2 flex items-center gap-2 text-sm">
    <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M18.364 5.636a9 9 0 010 12.728m-2.829-2.829a5 5 0 000-7.07m-2.828 2.828a1 1 0 010 1.414"/>
    </svg>
    <span>当前处于离线模式，数据将在恢复连接后自动同步</span>
  </div>

  <!-- 缓存内容 -->
  <div class="navbar bg-base-100 shadow-sm"><div class="flex-1"><h1 class="text-lg font-bold px-2">首页</h1></div></div>
  <div class="space-y-3 px-4 pt-4 opacity-75">
    <div class="card bg-base-100 shadow-sm">
      <div class="card-body p-4">
        <p class="text-sm text-base-content/50">（缓存数据）上次同步于 10 分钟前</p>
      </div>
    </div>
  </div>
</div>

```

Stale Data 状态

```html
<div x-show="currentState === 'stale-data'" x-transition>
  <div class="navbar bg-base-100 shadow-sm"><div class="flex-1"><h1 class="text-lg font-bold px-2">首页</h1></div></div>
  <div class="alert alert-warning rounded-none">
    <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"/>
    </svg>
    <span class="text-sm">数据可能已过期</span>
    <button class="btn btn-xs btn-warning" @click="currentState = 'loading'">刷新</button>
  </div>
  <div class="space-y-3 px-4 pt-4 opacity-75">
    <div class="card bg-base-100 shadow-sm"><div class="card-body p-4"><p class="text-sm">缓存内容展示…</p></div></div>
  </div>
</div>

```

---

### 3.2 List View（列表视图）

Ideal 状态

```html
<div x-show="currentState === 'ideal'" x-transition>
  <div class="navbar bg-base-100 shadow-sm sticky top-0 z-10">
    <div class="flex-1"><h1 class="text-lg font-bold px-2">我的任务</h1></div>
    <button class="btn btn-ghost btn-sm">筛选</button>
  </div>

  <!-- 卡片列表 -->
  <div class="space-y-2 p-4">
    <div class="card bg-base-100 shadow-sm">
      <div class="card-body p-4 flex-row items-center gap-3">
        <input type="checkbox" class="checkbox checkbox-primary checkbox-sm"/>
        <div class="flex-1">
          <p class="font-medium text-sm">完成首页设计稿评审</p>
          <p class="text-xs text-base-content/50">截止：明天 · 高优先级</p>
        </div>
        <span class="badge badge-error badge-xs">紧急</span>
      </div>
    </div>
    <div class="card bg-base-100 shadow-sm">
      <div class="card-body p-4 flex-row items-center gap-3">
        <input type="checkbox" class="checkbox checkbox-sm"/>
        <div class="flex-1">
          <p class="font-medium text-sm">更新 API 文档</p>
          <p class="text-xs text-base-content/50">截止：后天 · 中优先级</p>
        </div>
        <span class="badge badge-warning badge-xs">中</span>
      </div>
    </div>
    <div class="card bg-base-100 shadow-sm">
      <div class="card-body p-4 flex-row items-center gap-3">
        <input type="checkbox" class="checkbox checkbox-sm"/>
        <div class="flex-1">
          <p class="font-medium text-sm">整理会议纪要</p>
          <p class="text-xs text-base-content/50">截止：本周五 · 低优先级</p>
        </div>
        <span class="badge badge-ghost badge-xs">低</span>
      </div>
    </div>
  </div>
</div>

```

Empty 状态

```html
<div x-show="currentState === 'empty'" x-transition>
  <div class="navbar bg-base-100 shadow-sm"><div class="flex-1"><h1 class="text-lg font-bold px-2">我的任务</h1></div></div>
  <div class="flex flex-col items-center justify-center py-16 text-center px-4">
    <svg class="w-20 h-20 text-base-content/30 mb-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-6 9l2 2 4-4"/>
    </svg>
    <h3 class="text-lg font-semibold text-base-content/70">还没有任何任务</h3>
    <p class="text-sm text-base-content/50 mt-1">点击下方按钮创建您的第一个任务</p>
    <button class="btn btn-primary btn-sm mt-4">创建任务</button>
  </div>
</div>

```

Offline 状态

```html
<div x-show="currentState === 'offline'" x-transition>
  <div class="w-full bg-warning text-warning-content px-4 py-2 flex items-center gap-2 text-sm">
    <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M18.364 5.636a9 9 0 010 12.728"/>
    </svg>
    <span>离线模式 · 显示缓存数据</span>
  </div>
  <div class="navbar bg-base-100 shadow-sm"><div class="flex-1"><h1 class="text-lg font-bold px-2">我的任务</h1></div></div>
  <div class="space-y-2 p-4 opacity-75">
    <div class="card bg-base-100 shadow-sm"><div class="card-body p-4"><p class="text-sm">（缓存）完成首页设计稿评审</p></div></div>
    <div class="card bg-base-100 shadow-sm"><div class="card-body p-4"><p class="text-sm">（缓存）更新 API 文档</p></div></div>
  </div>
</div>

```

---

### 3.3 Action / Form（操作/表单 — Mobile）

Ideal 状态

```html
<div x-show="currentState === 'ideal'" x-transition>
  <div class="navbar bg-base-100 shadow-sm">
    <div class="flex-none"><button class="btn btn-ghost btn-sm">← 返回</button></div>
    <div class="flex-1"><h1 class="text-lg font-bold">新建任务</h1></div>
  </div>

  <form class="p-4 space-y-4">
    <div class="form-control">
      <label class="label"><span class="label-text">任务标题 <span class="text-error">*</span></span></label>
      <input type="text" class="input input-bordered w-full" placeholder="输入任务标题"/>
    </div>
    <div class="form-control">
      <label class="label"><span class="label-text">描述</span></label>
      <textarea class="textarea textarea-bordered w-full" placeholder="输入任务描述…"></textarea>
    </div>
    <div class="form-control">
      <label class="label"><span class="label-text">截止日期</span></label>
      <input type="date" class="input input-bordered w-full"/>
    </div>
    <div class="form-control">
      <label class="label"><span class="label-text">优先级</span></label>
      <select class="select select-bordered w-full">
        <option>高</option>
        <option selected>中</option>
        <option>低</option>
      </select>
    </div>

    <button class="btn btn-primary btn-block mt-6" :disabled="!can('edit')">提交</button>
  </form>
</div>

```

Success 状态

```html
<div x-show="currentState === 'success'" x-transition>
  <div class="navbar bg-base-100 shadow-sm">
    <div class="flex-1"><h1 class="text-lg font-bold">新建任务</h1></div>
  </div>
  <div class="flex flex-col items-center justify-center py-16 text-center">
    <div class="w-16 h-16 rounded-full bg-success/10 flex items-center justify-center mb-4">
      <svg class="w-8 h-8 text-success" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/>
      </svg>
    </div>
    <h3 class="font-semibold">创建成功！</h3>
    <p class="text-sm text-base-content/50 mt-1">任务已添加到您的列表</p>
    <button class="btn btn-ghost btn-sm mt-4">返回列表</button>
  </div>
</div>

```

Offline 状态

```html
<div x-show="currentState === 'offline'" x-transition>
  <div class="w-full bg-warning text-warning-content px-4 py-2 flex items-center gap-2 text-sm">
    <span>离线模式 · 提交将在恢复连接后同步</span>
  </div>
  <div class="navbar bg-base-100 shadow-sm">
    <div class="flex-none"><button class="btn btn-ghost btn-sm">← 返回</button></div>
    <div class="flex-1"><h1 class="text-lg font-bold">新建任务</h1></div>
  </div>
  <form class="p-4 space-y-4">
    <div class="form-control">
      <label class="label"><span class="label-text">任务标题</span></label>
      <input type="text" class="input input-bordered w-full" placeholder="输入任务标题"/>
    </div>
    <button class="btn btn-primary btn-block mt-6">提交（离线排队）</button>
  </form>
</div>

```

Validation Error 状态

```html
<div x-show="currentState === 'validation-error'" x-transition>
  <div class="navbar bg-base-100 shadow-sm">
    <div class="flex-none"><button class="btn btn-ghost btn-sm">← 返回</button></div>
    <div class="flex-1"><h1 class="text-lg font-bold">新建任务</h1></div>
  </div>
  <form class="p-4 space-y-4">
    <div class="form-control">
      <label class="label"><span class="label-text">任务标题 <span class="text-error">*</span></span></label>
      <input type="text" class="input input-bordered input-error w-full" value=""/>
      <label class="label"><span class="label-text-alt text-error">任务标题不能为空</span></label>
    </div>
    <button class="btn btn-primary btn-block mt-6">提交</button>
  </form>
</div>

```

---

### 3.4 Profile（个人中心）

Ideal 状态

```html
<div x-show="currentState === 'ideal'" x-transition>
  <div class="navbar bg-base-100 shadow-sm">
    <div class="flex-1"><h1 class="text-lg font-bold px-2">我的</h1></div>
    <button class="btn btn-ghost btn-sm">编辑</button>
  </div>

  <!-- 头像 + 信息 -->
  <div class="flex flex-col items-center py-6">
    <div class="avatar placeholder">
      <div class="bg-primary text-primary-content rounded-full w-20">
        <span class="text-2xl">张</span>
      </div>
    </div>
    <h2 class="font-bold text-lg mt-3">张三</h2>
    <p class="text-sm text-base-content/50">高级产品经理</p>
  </div>

  <!-- 功能菜单 -->
  <div class="px-4">
    <ul class="menu bg-base-100 rounded-box shadow w-full">
      <li><a>个人资料</a></li>
      <li><a>账户安全</a></li>
      <li><a>通知设置</a></li>
      <li><a>隐私设置</a></li>
      <li><a class="text-error">退出登录</a></li>
    </ul>
  </div>
</div>

```

Offline 状态

```html
<div x-show="currentState === 'offline'" x-transition>
  <div class="w-full bg-warning text-warning-content px-4 py-2 flex items-center gap-2 text-sm">
    <span>离线模式 · 部分功能不可用</span>
  </div>
  <div class="flex flex-col items-center py-6">
    <div class="avatar placeholder">
      <div class="bg-primary text-primary-content rounded-full w-20"><span class="text-2xl">张</span></div>
    </div>
    <h2 class="font-bold text-lg mt-3">张三</h2>
    <p class="text-sm text-base-content/50">（离线缓存）</p>
  </div>
  <div class="px-4">
    <ul class="menu bg-base-100 rounded-box shadow w-full">
      <li><a>个人资料</a></li>
      <li class="disabled"><a class="opacity-50">账户安全（需联网）</a></li>
      <li><a>通知设置</a></li>
      <li class="disabled"><a class="opacity-50">隐私设置（需联网）</a></li>
    </ul>
  </div>
</div>

```

---

### 3.5 Notifications（通知中心）

Ideal 状态

```html
<div x-show="currentState === 'ideal'" x-transition>
  <div class="navbar bg-base-100 shadow-sm sticky top-0 z-10">
    <div class="flex-1"><h1 class="text-lg font-bold px-2">通知</h1></div>
    <button class="btn btn-ghost btn-xs">全部已读</button>
  </div>

  <div class="divide-y">
    <div class="flex gap-3 p-4 bg-primary/5">
      <div class="w-2 h-2 rounded-full bg-primary mt-2 shrink-0"></div>
      <div class="flex-1">
        <p class="text-sm font-medium">张三 邀请您加入「新项目」</p>
        <p class="text-xs text-base-content/50">3 分钟前</p>
        <div class="flex gap-2 mt-2">
          <button class="btn btn-primary btn-xs">接受</button>
          <button class="btn btn-ghost btn-xs">拒绝</button>
        </div>
      </div>
    </div>
    <div class="flex gap-3 p-4">
      <div class="w-2 h-2 rounded-full bg-transparent mt-2 shrink-0"></div>
      <div class="flex-1">
        <p class="text-sm">您的任务「API 文档更新」已被审核通过</p>
        <p class="text-xs text-base-content/50">1 小时前</p>
      </div>
    </div>
    <div class="flex gap-3 p-4">
      <div class="w-2 h-2 rounded-full bg-transparent mt-2 shrink-0"></div>
      <div class="flex-1">
        <p class="text-sm">系统将于明天 02:00 进行例行维护</p>
        <p class="text-xs text-base-content/50">3 小时前</p>
      </div>
    </div>
  </div>
</div>

```

Empty 状态

```html
<div x-show="currentState === 'empty'" x-transition>
  <div class="navbar bg-base-100 shadow-sm"><div class="flex-1"><h1 class="text-lg font-bold px-2">通知</h1></div></div>
  <div class="flex flex-col items-center justify-center py-16 text-center px-4">
    <svg class="w-20 h-20 text-base-content/30 mb-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M15 17h5l-1.405-1.405A2.032 2.032 0 0118 14.158V11a6.002 6.002 0 00-4-5.659V5a2 2 0 10-4 0v.341C7.67 6.165 6 8.388 6 11v3.159c0 .538-.214 1.055-.595 1.436L4 17h5m6 0v1a3 3 0 11-6 0v-1m6 0H9"/>
    </svg>
    <h3 class="text-lg font-semibold text-base-content/70">暂无通知</h3>
    <p class="text-sm text-base-content/50 mt-1">当有新消息时，将在这里提醒您。</p>
  </div>
</div>

```

Offline 状态

```html
<div x-show="currentState === 'offline'" x-transition>
  <div class="w-full bg-warning text-warning-content px-4 py-2 flex items-center gap-2 text-sm">
    <span>离线 · 显示缓存通知</span>
  </div>
  <div class="navbar bg-base-100 shadow-sm"><div class="flex-1"><h1 class="text-lg font-bold px-2">通知</h1></div></div>
  <div class="divide-y opacity-75">
    <div class="flex gap-3 p-4">
      <div class="flex-1">
        <p class="text-sm">（缓存）上次同步的通知内容…</p>
        <p class="text-xs text-base-content/50">缓存于 10 分钟前</p>
      </div>
    </div>
  </div>
</div>

```

---

## 4. 生成流程集成指南

### 4.1 调用顺序

```
输入: PageSpec + ApplicableStates[]（来自 State Matrix 模块）
    ↓
Step 1: 确定页面类型 → 匹配本模块对应的模板节（§2 或 §3）
    ↓
Step 2: 遍历 ApplicableStates[] → 为每个状态生成对应 HTML 变体
    ↓
Step 3: 组装完整页面 → 包裹在全局 x-data 容器 + 状态切换器
    ↓
Step 4: 注入异常（调用 Exception Injection 模块）
    ↓
输出: 完整的单文件 HTML 原型

```

### 4.2 自定义规则

- 如果业务上下文指定了具体的**数据字段**，用实际字段名替换模板中的占位内容
- 如果业务指定了**特定操作**（如"审批流程"），在 Ideal 状态中添加对应操作按钮
- 如果页面有**多角色差异**，为每种角色生成同一状态的不同变体（通过 `x-show="can()"` 控制）

### 4.3 质量检查清单

生成完成后，逐项确认：

- [ ] 每个 ApplicableStates[] 中的状态都有对应的 `<div x-show="...">`
- [ ] Loading 骨架与 Ideal 布局结构一一对应
- [ ] Empty 状态有引导文案 + CTA 按钮
- [ ] Error 状态有重试机制
- [ ] 403 状态有申请权限/联系管理员出口
- [ ] 表单页包含 validation-error 状态
- [ ] Mobile 页面包含 offline 状态
- [ ] 所有按钮使用 DaisyUI 类名
- [ ] 角色敏感元素有 `x-show="can()"` 保护

---

## VERIFICATION

```yaml
verification:
  module_name: "Page Generator"
  completeness_checks:
    - "Web 6 种页面类型全部覆盖（Dashboard/DataTable/Detail/Form/Settings/Admin）": true
    - "Mobile 5 种页面类型全部覆盖（HomeFeed/ListView/ActionForm/Profile/Notifications）": true
    - "每种页面类型的 Ideal 状态有完整 HTML 模板": true
    - "每种页面类型的 Loading 状态骨架与 Ideal 布局对应": true
    - "每种页面类型的 Empty 状态有引导文案 + CTA": true
    - "每种页面类型的 Error 状态有重试按钮": true
    - "403 状态有双出口（申请权限 + 联系管理员）": true
    - "Form 类型包含 validation-error + timeout + session-expired 状态": true
    - "Detail 类型包含 stale-data 状态": true
    - "Mobile 类型包含 offline 状态": true
    - "所有 HTML 使用 DaisyUI 组件类": true
    - "所有状态切换使用 Alpine.js x-show + x-transition": true
    - "角色权限使用 can('permission') 函数": true
    - "生成流程与 State Matrix 模块集成说明完整": true

  cross_references:
    - "页面类型分类 → 与 State Matrix §2 页面类型矩阵对齐"
    - "状态 ID → 与 State Matrix §1 状态目录 ID 一致"
    - "DaisyUI 组件类 → 与 Tech Stack 模块一致"
    - "生成流程 Step 4 → 调用 Exception Injection 模块"

  usage_in_skill:
    - "接收 PageSpec + ApplicableStates[] → 按页面类型查找本模块模板"
    - "为每个适用状态生成完整 HTML 变体"
    - "组装为单文件 HTML 原型，包含状态切换器"
    - "输出传递给 Exception Injection 模块做异常增强"

  quality_gates:
    - "Loading 骨架必须与 Ideal 布局结构匹配"
    - "Empty 状态不能只显示空白，必须有引导"
    - "Error 状态必须有可操作的恢复路径"
    - "所有文本必须是中文"
    - "Mobile 页面不能使用 table 组件（用卡片列表替代）"

```

---

---

## Phase 4b: 异常场景注入

# Exception Injection 模块 — 自动异常注入规则

> **模块职责**: 定义当生成特定页面类型时，必须自动注入的异常场景及其对应的 HTML 模式。确保原型不仅覆盖基础状态，还能展示真实生产环境中的边缘异常交互。

---

## 1. 注入原则

### 1.1 自动注入 vs 手动注入

- **自动注入**: 根据页面类型自动附加的异常状态，无需用户显式指定
- **手动注入**: 用户可以在 PageSpec 中额外指定的特殊异常

### 1.2 注入时机

异常注入发生在 Page Generator 模块输出基础状态之后：

```
Page Generator 输出 → Exception Injection 模块 → 增强后的完整原型

```

### 1.3 注入规则

1. 每个异常注入为独立的 `<div x-show="...">`，使用复合状态 ID（如 `exception_validation_error`）
2. 异常状态可以叠加在某个基础状态之上（如 validation_error 叠加在 ideal/form 之上）
3. 使用 `x-show` + `x-transition` 控制异常的出现/消失
4. 异常 UI 不能破坏页面的基础布局结构

---

## 2. 注入矩阵

### 2.1 按页面类型自动注入

| 页面类型 | 自动注入的异常 | 注入位置 |
| --- | --- | --- |
| 任何表单（Form） | `validation_error`, `submit_timeout`, `session_expired`, `autosave_failure` | 表单内部/全局覆层 |
| 任何列表（List） | `empty_search`, `large_dataset_loading`, `filter_no_results` | 列表内容区 |
| 任何详情（Detail） | `not_found_404`, `data_stale`, `concurrent_edit_conflict` | 页面主体 |
| 仪表盘（Dashboard） | `partial_widget_failure`, `data_refresh_failure` | 卡片/组件级别 |
| 文件上传（File Upload） | `size_exceeded`, `format_invalid`, `network_interrupted`, `partial_batch_failure` | 上传区域 |
| 向导/多步骤（Wizard） | `step_data_loss`, `back_navigation_warning`, `timeout_mid_process` | 步骤容器 |
| 批量操作（Bulk Operation） | `partial_failure_report`, `rate_limit_warning`, `cancellation_confirmation` | 操作结果区 |

### 2.2 全局异常（任何页面均可触发）

| 异常 ID | 触发条件 | 表现形式 |
| --- | --- | --- |
| `network_disconnected` | 网络断开 | 顶部固定横幅 |
| `session_about_to_expire` | 会话即将过期（5分钟内） | 底部弹出提醒 |
| `system_announcement` | 系统公告/紧急通知 | 顶部蓝色横幅 |
| `concurrent_session` | 检测到异地登录 | 模态弹窗 |

---

## 3. 异常 HTML 模式库

### 3.1 表单类异常

`validation_error` — 字段级验证错误

```html
<!-- 单字段验证错误 — 注入到每个可能出错的表单字段下方 -->
<div class="form-control w-full">
  <label class="label"><span class="label-text">字段名称 <span class="text-error">*</span></span></label>
  <input type="text" 
    class="input input-bordered w-full" 
    :class="{ 'input-error': errors.fieldName }"
    x-model="formData.fieldName"/>
  <p class="text-error text-sm mt-1" x-show="errors.fieldName" x-transition>
    <span x-text="errors.fieldName">错误信息</span>
  </p>
</div>

<!-- 顶部错误摘要 — 当有多个字段错误时显示 -->
<div class="alert alert-error mb-4" x-show="Object.keys(errors).length > 0" x-transition>
  <svg class="w-5 h-5 shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor">
    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-2.5L13.732 4c-.77-.833-1.964-.833-2.732 0L4.082 16.5c-.77.833.192 2.5 1.732 2.5z"/>
  </svg>
  <div>
    <p class="font-medium">请修正以下 <span x-text="Object.keys(errors).length"></span> 个错误</p>
    <ul class="text-sm mt-1 list-disc list-inside">
      <template x-for="(msg, field) in errors">
        <li x-text="msg"></li>
      </template>
    </ul>
  </div>
</div>

```

`submit_timeout` — 提交超时

```html
<!-- 提交超时 Alert + 倒计时重试 -->
<div class="alert alert-warning shadow-lg" x-show="currentException === 'submit_timeout'" x-transition
  x-data="{ countdown: 10, interval: null }"
  x-init="interval = setInterval(() => { countdown--; if(countdown <= 0) clearInterval(interval) }, 1000)">
  <svg class="w-5 h-5 shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor">
    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"/>
  </svg>
  <div>
    <p class="font-medium">提交超时</p>
    <p class="text-sm">服务器响应时间过长，您的数据已自动保存。</p>
    <p class="text-xs mt-1">
      <span x-show="countdown > 0">将在 <span x-text="countdown"></span> 秒后自动重试…</span>
      <span x-show="countdown <= 0">正在重试…</span>
    </p>
  </div>
  <button class="btn btn-sm btn-warning" @click="currentException = null">立即重试</button>
</div>

```

`session_expired` — 会话过期

```html
<!-- 全屏模态 — 会话过期 -->
<div class="modal modal-open" x-show="currentException === 'session_expired'" x-transition>
  <div class="modal-box text-center max-w-sm">
    <svg class="w-16 h-16 text-warning mx-auto mb-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z"/>
    </svg>
    <h3 class="font-bold text-lg">会话已过期</h3>
    <p class="py-4 text-base-content/70">
      您的登录会话已超时，请重新登录以继续操作。<br/>
      未保存的数据将在重新登录后自动恢复。
    </p>
    <div class="modal-action justify-center">
      <button class="btn btn-primary">重新登录</button>
      <button class="btn btn-ghost btn-sm">返回首页</button>
    </div>
  </div>
  <div class="modal-backdrop bg-black/50"></div>
</div>

```

`autosave_failure` — 自动保存失败

```html
<!-- 底部浮动提示条 -->
<div class="fixed bottom-4 left-4 right-4 z-50" x-show="currentException === 'autosave_failure'" x-transition>
  <div class="alert alert-error shadow-lg">
    <svg class="w-5 h-5 shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01"/>
    </svg>
    <div>
      <p class="font-medium text-sm">自动保存失败</p>
      <p class="text-xs">最后成功保存于 3 分钟前，请手动保存或检查网络连接。</p>
    </div>
    <div class="flex gap-1">
      <button class="btn btn-xs btn-error">手动保存</button>
      <button class="btn btn-xs btn-ghost" @click="currentException = null">忽略</button>
    </div>
  </div>
</div>

```

---

### 3.2 列表类异常

`empty_search` — 搜索无结果

```html
<!-- 搜索无结果 — 替换列表内容区 -->
<div class="flex flex-col items-center justify-center py-12 text-center" 
  x-show="currentException === 'empty_search'" x-transition>
  <svg class="w-16 h-16 text-base-content/30 mb-3" fill="none" viewBox="0 0 24 24" stroke="currentColor">
    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"/>
  </svg>
  <h3 class="font-semibold text-base-content/70">未找到匹配 '<span x-text="searchQuery" class="text-primary"></span>' 的结果</h3>
  <p class="text-sm text-base-content/50 mt-1 max-w-sm">
    请尝试不同的关键词，或检查筛选条件是否过于严格。
  </p>
  <div class="flex gap-2 mt-4">
    <button class="btn btn-outline btn-sm" @click="searchQuery = ''; currentException = null">清除搜索</button>
    <button class="btn btn-ghost btn-sm" @click="filters = {}; currentException = null">重置所有筛选</button>
  </div>
</div>

```

`large_dataset_loading` — 大数据集加载

```html
<!-- 大数据集加载指示器 — 覆盖在列表上方 -->
<div class="w-full" x-show="currentException === 'large_dataset_loading'" x-transition>
  <div class="alert alert-info mb-4">
    <svg class="w-5 h-5 animate-spin" fill="none" viewBox="0 0 24 24">
      <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
      <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4z"></path>
    </svg>
    <div>
      <p class="font-medium text-sm">正在加载大量数据…</p>
      <p class="text-xs">共 12,847 条记录，已加载 4,200 条</p>
      <progress class="progress progress-info w-48 mt-1" value="33" max="100"></progress>
    </div>
    <button class="btn btn-xs btn-ghost">取消</button>
  </div>
</div>

```

`filter_no_results` — 筛选无结果

```html
<!-- 筛选条件过严导致无结果 -->
<div class="flex flex-col items-center justify-center py-12 text-center"
  x-show="currentException === 'filter_no_results'" x-transition>
  <svg class="w-16 h-16 text-base-content/30 mb-3" fill="none" viewBox="0 0 24 24" stroke="currentColor">
    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M3 4a1 1 0 011-1h16a1 1 0 011 1v2.586a1 1 0 01-.293.707l-6.414 6.414a1 1 0 00-.293.707V17l-4 4v-6.586a1 1 0 00-.293-.707L3.293 7.293A1 1 0 013 6.586V4z"/>
  </svg>
  <h3 class="font-semibold text-base-content/70">当前筛选条件下暂无数据</h3>
  <p class="text-sm text-base-content/50 mt-1">
    已应用 <span class="badge badge-sm badge-outline">状态: 已关闭</span> 
    <span class="badge badge-sm badge-outline">优先级: 紧急</span> 筛选
  </p>
  <button class="btn btn-outline btn-sm mt-4" @click="filters = {}; currentException = null">清除所有筛选</button>
</div>

```

---

### 3.3 详情类异常

`not_found_404` — 资源不存在

```html
<!-- 404 资源不存在 -->
<div class="flex flex-col items-center justify-center py-16 text-center"
  x-show="currentException === 'not_found_404'" x-transition>
  <div class="text-6xl font-bold text-base-content/10 mb-4">404</div>
  <h3 class="text-lg font-semibold">请求的资源不存在</h3>
  <p class="text-sm text-base-content/50 mt-1 max-w-sm">
    该记录可能已被删除，或链接已失效。请确认地址是否正确。
  </p>
  <div class="flex gap-2 mt-6">
    <button class="btn btn-primary btn-sm">返回列表</button>
    <button class="btn btn-ghost btn-sm">联系支持</button>
  </div>
</div>

```

`data_stale` — 数据过期提示

```html
<!-- 顶部黄色提示 Banner -->
<div class="alert alert-warning rounded-none" x-show="currentException === 'data_stale'" x-transition>
  <svg class="w-5 h-5 shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor">
    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"/>
  </svg>
  <div>
    <p class="font-medium text-sm">数据可能已过期</p>
    <p class="text-xs">上次同步于 5 分钟前，其他用户可能已做更新。</p>
  </div>
  <button class="btn btn-xs btn-warning" @click="currentException = null">立即刷新</button>
</div>

```

`concurrent_edit_conflict` — 并发编辑冲突

```html
<!-- 并发编辑冲突弹窗 -->
<div class="modal modal-open" x-show="currentException === 'concurrent_edit_conflict'" x-transition>
  <div class="modal-box">
    <h3 class="font-bold text-lg text-warning">⚠️ 编辑冲突</h3>
    <p class="py-2 text-sm text-base-content/70">
      用户 <span class="font-medium">李四</span> 在 2 分钟前也修改了此记录。请选择如何处理：
    </p>
    <div class="space-y-3 mt-4">
      <!-- 冲突对比 -->
      <div class="grid grid-cols-2 gap-3">
        <div class="card bg-base-200 p-3">
          <p class="text-xs font-semibold text-base-content/50 mb-1">您的版本</p>
          <p class="text-sm">项目状态：<span class="badge badge-success badge-sm">进行中</span></p>
        </div>
        <div class="card bg-base-200 p-3">
          <p class="text-xs font-semibold text-base-content/50 mb-1">李四的版本</p>
          <p class="text-sm">项目状态：<span class="badge badge-warning badge-sm">暂停</span></p>
        </div>
      </div>
    </div>

    <div class="modal-action">
      <button class="btn btn-primary btn-sm">使用我的版本</button>
      <button class="btn btn-outline btn-sm">使用对方版本</button>
      <button class="btn btn-ghost btn-sm" @click="currentException = null">稍后处理</button>
    </div>
  </div>
  <div class="modal-backdrop bg-black/30"></div>
</div>

```

---

### 3.4 仪表盘类异常

`partial_widget_failure` — 部分组件加载失败

```html
<!-- 单个 Widget 内联错误 — 注入到卡片网格中 -->
<div class="stat bg-base-100 shadow rounded-box border border-error/30">
  <div class="flex flex-col items-center justify-center h-full py-4">
    <svg class="w-6 h-6 text-error/50 mb-2" fill="none" viewBox="0 0 24 24" stroke="currentColor">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-2.5L13.732 4c-.77-.833-1.964-.833-2.732 0L4.082 16.5c-.77.833.192 2.5 1.732 2.5z"/>
    </svg>
    <p class="text-xs text-error/70">此组件加载失败</p>
    <button class="btn btn-xs btn-ghost text-error mt-1" @click="retryWidget('widgetId')">重试</button>
  </div>
</div>

```

`data_refresh_failure` — 数据刷新失败

```html
<!-- 顶部横幅 — 自动刷新失败 -->
<div class="alert alert-warning rounded-none mb-4" x-show="currentException === 'data_refresh_failure'" x-transition>
  <svg class="w-5 h-5 shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor">
    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15"/>
  </svg>
  <div>
    <p class="font-medium text-sm">自动刷新失败</p>
    <p class="text-xs">数据可能不是最新状态，上次成功刷新于 10 分钟前。</p>
  </div>
  <button class="btn btn-xs btn-warning" @click="currentException = null">手动刷新</button>
</div>

```

---

### 3.5 文件上传类异常

`size_exceeded` — 文件大小超限

```html
<!-- 文件大小超限提示 -->
<div class="alert alert-error" x-show="currentException === 'size_exceeded'" x-transition>
  <svg class="w-5 h-5 shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor">
    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 21h10a2 2 0 002-2V9.414a1 1 0 00-.293-.707l-5.414-5.414A1 1 0 0012.586 3H7a2 2 0 00-2 2v14a2 2 0 002 2z"/>
  </svg>
  <div>
    <p class="font-medium text-sm">文件大小超过限制</p>
    <p class="text-xs">最大允许 10MB，您选择的文件为 25.3MB。</p>
  </div>
  <button class="btn btn-xs btn-ghost" @click="currentException = null">知道了</button>
</div>

```

`format_invalid` — 文件格式不支持

```html
<!-- 格式不支持提示 -->
<div class="alert alert-error" x-show="currentException === 'format_invalid'" x-transition>
  <svg class="w-5 h-5 shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor">
    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M18.364 18.364A9 9 0 005.636 5.636m12.728 12.728A9 9 0 015.636 5.636m12.728 12.728L5.636 5.636"/>
  </svg>
  <div>
    <p class="font-medium text-sm">不支持的文件格式</p>
    <p class="text-xs">仅支持 .jpg, .png, .pdf 格式，您上传的是 .exe 文件。</p>
  </div>
  <button class="btn btn-xs btn-ghost" @click="currentException = null">重新选择</button>
</div>

```

`network_interrupted` — 网络中断（上传过程中）

```html
<!-- 上传中断 Banner + 自动重试指示器 -->
<div class="card bg-base-100 shadow border border-warning/50" x-show="currentException === 'network_interrupted'" x-transition>
  <div class="card-body p-4">
    <div class="flex items-center gap-3">
      <svg class="w-8 h-8 text-warning" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M18.364 5.636a9 9 0 010 12.728m-2.829-2.829a5 5 0 000-7.07"/>
      </svg>
      <div class="flex-1">
        <p class="font-medium text-sm">网络连接中断</p>
        <p class="text-xs text-base-content/50">上传进度：67%（已上传 6.7MB / 10MB）</p>
        <progress class="progress progress-warning w-full mt-1" value="67" max="100"></progress>
      </div>
    </div>
    <div class="flex items-center gap-2 mt-3">
      <span class="loading loading-spinner loading-xs text-warning"></span>
      <span class="text-xs text-base-content/50">等待网络恢复后自动继续…</span>
      <button class="btn btn-xs btn-ghost ml-auto">取消上传</button>
    </div>
  </div>
</div>

```

`partial_batch_failure` — 批量上传部分失败

```html
<!-- 批量上传结果报告 -->
<div class="card bg-base-100 shadow" x-show="currentException === 'partial_batch_failure'" x-transition>
  <div class="card-body p-4">
    <h3 class="font-medium text-sm mb-3">上传结果（3/5 成功）</h3>
    <div class="space-y-2">
      <!-- 成功项 -->
      <div class="flex items-center gap-2 text-sm">
        <span class="text-success">✅</span>
        <span>报告_v1.pdf</span>
        <span class="text-xs text-base-content/50 ml-auto">2.1MB</span>
      </div>
      <div class="flex items-center gap-2 text-sm">
        <span class="text-success">✅</span>
        <span>设计稿.png</span>
        <span class="text-xs text-base-content/50 ml-auto">1.5MB</span>
      </div>
      <div class="flex items-center gap-2 text-sm">
        <span class="text-success">✅</span>
        <span>数据表.xlsx</span>
        <span class="text-xs text-base-content/50 ml-auto">0.8MB</span>
      </div>
      <!-- 失败项 -->
      <div class="flex items-center gap-2 text-sm text-error">
        <span>❌</span>
        <span>视频素材.mp4</span>
        <span class="text-xs ml-auto">文件过大 (156MB)</span>
      </div>
      <div class="flex items-center gap-2 text-sm text-error">
        <span>❌</span>
        <span>未命名.tmp</span>
        <span class="text-xs ml-auto">格式不支持</span>
      </div>
    </div>

    <div class="flex gap-2 mt-4 pt-3 border-t">
      <button class="btn btn-sm btn-primary">重试失败项</button>
      <button class="btn btn-sm btn-ghost" @click="currentException = null">跳过</button>
    </div>
  </div>
</div>

```

---

### 3.6 向导/多步骤类异常

`step_data_loss` — 步骤数据丢失警告

```html
<!-- 数据丢失确认弹窗 -->
<div class="modal modal-open" x-show="currentException === 'step_data_loss'" x-transition>
  <div class="modal-box max-w-sm">
    <h3 class="font-bold text-lg text-warning">⚠️ 数据可能丢失</h3>
    <p class="py-3 text-sm text-base-content/70">
      检测到第 2 步的填写数据未能保存。如果继续前进，之前填写的内容将丢失。
    </p>
    <div class="modal-action">
      <button class="btn btn-warning btn-sm">返回第 2 步</button>
      <button class="btn btn-ghost btn-sm" @click="currentException = null">继续前进</button>
    </div>
  </div>
  <div class="modal-backdrop bg-black/30"></div>
</div>

```

`back_navigation_warning` — 返回导航警告

```html
<!-- 返回确认弹窗 -->
<div class="modal modal-open" x-show="currentException === 'back_navigation_warning'" x-transition>
  <div class="modal-box max-w-sm">
    <h3 class="font-bold text-lg">确认返回？</h3>
    <p class="py-3 text-sm text-base-content/70">
      返回上一步将清除当前步骤已填写的内容。确定要返回吗？
    </p>
    <div class="modal-action">
      <button class="btn btn-primary btn-sm" @click="currentException = null">留在当前步</button>
      <button class="btn btn-ghost btn-sm">确认返回</button>
    </div>
  </div>
  <div class="modal-backdrop bg-black/30"></div>
</div>

```

`timeout_mid_process` — 流程中途超时

```html
<!-- 中途超时提示 -->
<div class="alert alert-error" x-show="currentException === 'timeout_mid_process'" x-transition>
  <svg class="w-5 h-5 shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor">
    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"/>
  </svg>
  <div>
    <p class="font-medium text-sm">操作超时</p>
    <p class="text-xs">第 3/5 步处理超时。已完成的步骤数据已保存，您可以从当前步骤继续。</p>
  </div>
  <div class="flex gap-1">
    <button class="btn btn-xs btn-error">从第 3 步重试</button>
    <button class="btn btn-xs btn-ghost">保存草稿退出</button>
  </div>
</div>

```

---

### 3.7 批量操作类异常

`partial_failure_report` — 部分失败报告

```html
<!-- 批量操作结果报告 -->
<div class="card bg-base-100 shadow" x-show="currentException === 'partial_failure_report'" x-transition>
  <div class="card-body">
    <h3 class="card-title text-base">批量操作结果</h3>
    <!-- 摘要统计 -->
    <div class="stats stats-horizontal shadow mb-4">
      <div class="stat px-4 py-2">
        <div class="stat-title text-xs">成功</div>
        <div class="stat-value text-success text-lg">47</div>
      </div>
      <div class="stat px-4 py-2">
        <div class="stat-title text-xs">失败</div>
        <div class="stat-value text-error text-lg">3</div>
      </div>
      <div class="stat px-4 py-2">
        <div class="stat-title text-xs">跳过</div>
        <div class="stat-value text-warning text-lg">2</div>
      </div>
    </div>

    <!-- 失败详情 -->
    <div class="collapse collapse-arrow bg-base-200 rounded-box">
      <input type="checkbox" checked/>
      <div class="collapse-title font-medium text-sm text-error">
        查看失败项详情（3 项）
      </div>
      <div class="collapse-content">
        <div class="space-y-2 text-sm">
          <div class="flex justify-between items-center">
            <span>❌ 记录 #1024 — 权限不足</span>
          </div>
          <div class="flex justify-between items-center">
            <span>❌ 记录 #1031 — 数据格式错误</span>
          </div>
          <div class="flex justify-between items-center">
            <span>❌ 记录 #1045 — 关联记录已删除</span>
          </div>
        </div>
      </div>
    </div>

    <div class="flex gap-2 mt-4">
      <button class="btn btn-sm btn-primary">重试失败项</button>
      <button class="btn btn-sm btn-outline">导出报告</button>
      <button class="btn btn-sm btn-ghost" @click="currentException = null">关闭</button>
    </div>
  </div>
</div>

```

`rate_limit_warning` — 限流警告

```html
<!-- 限流警告 Banner -->
<div class="alert alert-warning" x-show="currentException === 'rate_limit_warning'" x-transition
  x-data="{ waitSeconds: 30, interval: null }"
  x-init="interval = setInterval(() => { waitSeconds--; if(waitSeconds <= 0) { clearInterval(interval); currentException = null } }, 1000)">
  <svg class="w-5 h-5 shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor">
    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-2.5L13.732 4c-.77-.833-1.964-.833-2.732 0L4.082 16.5c-.77.833.192 2.5 1.732 2.5z"/>
  </svg>
  <div>
    <p class="font-medium text-sm">请求过于频繁</p>
    <p class="text-xs">已达到 API 调用限制，请等待 <span x-text="waitSeconds"></span> 秒后继续操作。</p>
    <progress class="progress progress-warning w-48 mt-1" :value="30 - waitSeconds" max="30"></progress>
  </div>
</div>

```

`cancellation_confirmation` — 取消操作确认

```html
<!-- 取消确认弹窗 -->
<div class="modal modal-open" x-show="currentException === 'cancellation_confirmation'" x-transition>
  <div class="modal-box max-w-sm">
    <h3 class="font-bold text-lg">确认取消批量操作？</h3>
    <p class="py-3 text-sm text-base-content/70">
      已处理 23/50 项。取消后：
    </p>
    <ul class="text-sm text-base-content/70 list-disc list-inside mb-3">
      <li>已成功的 23 项将保留</li>
      <li>未处理的 27 项将跳过</li>
      <li>此操作不可恢复</li>
    </ul>
    <div class="modal-action">
      <button class="btn btn-error btn-sm">确认取消</button>
      <button class="btn btn-ghost btn-sm" @click="currentException = null">继续执行</button>
    </div>
  </div>
  <div class="modal-backdrop bg-black/30"></div>
</div>

```

---

### 3.8 全局异常模式

`network_disconnected` — 全局网络断开

```html
<!-- 顶部固定横幅 — 全局网络断开 -->
<div class="fixed top-0 left-0 right-0 z-[100] w-full bg-error text-error-content px-4 py-2 flex items-center justify-center gap-2 text-sm"
  x-show="globalException === 'network_disconnected'" x-transition.opacity>
  <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M18.364 5.636a9 9 0 010 12.728M5.636 18.364a9 9 0 010-12.728"/>
  </svg>
  <span>网络连接已断开 · 正在尝试重新连接…</span>
  <span class="loading loading-dots loading-xs"></span>
</div>

```

`session_about_to_expire` — 会话即将过期

```html
<!-- 底部浮动提醒 -->
<div class="fixed bottom-4 right-4 z-[100]" x-show="globalException === 'session_about_to_expire'" x-transition>
  <div class="alert alert-warning shadow-lg max-w-sm">
    <svg class="w-5 h-5 shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"/>
    </svg>
    <div>
      <p class="font-medium text-sm">会话即将过期</p>
      <p class="text-xs">您的登录将在 5 分钟后失效</p>
    </div>
    <button class="btn btn-xs btn-warning">续期</button>
  </div>
</div>

```

`system_announcement` — 系统公告

```html
<!-- 顶部蓝色系统公告 -->
<div class="w-full bg-info text-info-content px-4 py-2 flex items-center justify-center gap-2 text-sm"
  x-show="globalException === 'system_announcement'" x-transition>
  <svg class="w-4 h-4 shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor">
    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 5.882V19.24a1.76 1.76 0 01-3.417.592l-2.147-6.15M18 13a3 3 0 100-6M5.436 13.683A4.001 4.001 0 017 6h1.832c4.1 0 7.625-1.234 9.168-3v14c-1.543-1.766-5.067-3-9.168-3H7a3.988 3.988 0 01-1.564-.317z"/>
  </svg>
  <span>系统将于今晚 23:00-01:00 进行维护升级，届时服务将短暂不可用。</span>
  <button class="btn btn-xs btn-ghost" @click="globalException = null">✕</button>
</div>

```

`concurrent_session` — 异地登录检测

```html
<!-- 异地登录检测弹窗 -->
<div class="modal modal-open" x-show="globalException === 'concurrent_session'" x-transition>
  <div class="modal-box max-w-sm text-center">
    <svg class="w-16 h-16 text-error mx-auto mb-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-2.5L13.732 4c-.77-.833-1.964-.833-2.732 0L4.082 16.5c-.77.833.192 2.5 1.732 2.5z"/>
    </svg>
    <h3 class="font-bold text-lg">检测到异地登录</h3>
    <p class="py-3 text-sm text-base-content/70">
      您的账号在其他设备上登录（IP: 203.*.*.45，位置: 上海）。如果不是您本人操作，请立即修改密码。
    </p>
    <div class="modal-action justify-center flex-col gap-2">
      <button class="btn btn-error btn-sm btn-block">不是我，修改密码</button>
      <button class="btn btn-ghost btn-sm btn-block" @click="globalException = null">是我本人，继续使用</button>
    </div>
  </div>
  <div class="modal-backdrop bg-black/50"></div>
</div>

```

---

## 4. 注入执行算法

```
输入: GeneratedPage（Page Generator 输出）, PageSpec
输出: EnhancedPage（注入异常后的完整原型）

算法步骤:

STEP 1 — 确定页面特征标签
  tags = []
  IF PageSpec.features CONTAINS "form" → tags.ADD("form")
  IF PageSpec.type IN ["Data Table/List", "List/Detail", "Search Results"] → tags.ADD("list")
  IF PageSpec.type IN ["Detail View", "List/Detail"] → tags.ADD("detail")
  IF PageSpec.type IN ["Dashboard", "Home/Feed"] → tags.ADD("dashboard")
  IF PageSpec.features CONTAINS "file-upload" → tags.ADD("file_upload")
  IF PageSpec.features CONTAINS "wizard" OR "multi-step" → tags.ADD("wizard")
  IF PageSpec.features CONTAINS "batch" OR "bulk" → tags.ADD("bulk")

STEP 2 — 查表获取注入列表
  exceptions = []
  FOR EACH tag IN tags:
    exceptions.ADD_ALL( INJECTION_MATRIX[tag] )
  exceptions = DEDUPLICATE(exceptions)

STEP 3 — 注入全局异常
  exceptions.ADD("network_disconnected")
  exceptions.ADD("session_about_to_expire")

STEP 4 — 为每个异常生成 HTML
  FOR EACH exception IN exceptions:
    html = EXCEPTION_PATTERNS[exception]  // 从 §3 查询对应模式
    INJECT(GeneratedPage, html, position=INJECTION_MATRIX[tag].position)

STEP 5 — 更新状态切换器
  将所有异常 ID 添加到状态切换器的下拉列表中（分组显示为"异常场景"）

STEP 6 — 输出
  RETURN EnhancedPage

```

---

## 5. 异常状态切换器扩展

将异常场景添加到状态切换器中，与基础状态分组显示：

```html
<!-- 扩展的状态切换器 — 包含异常分组 -->
<div class="dropdown dropdown-end fixed top-4 right-4 z-50">
  <label tabindex="0" class="btn btn-sm btn-outline gap-1">
    <span>状态切换</span>
    <svg class="w-4 h-4"><!-- 下拉箭头 --></svg>
  </label>
  <ul tabindex="0" class="dropdown-content menu p-2 shadow bg-base-100 rounded-box w-64 max-h-96 overflow-y-auto">
    <!-- 基础状态分组 -->
    <li class="menu-title"><span>基础状态</span></li>
    <li><a data-state="ideal">✅ 理想状态</a></li>
    <li><a data-state="empty">📭 空状态</a></li>
    <li><a data-state="loading">⏳ 加载中</a></li>
    <li><a data-state="error">❌ 错误</a></li>
    <li><a data-state="partial">⚠️ 部分加载</a></li>
    <li><a data-state="success">🎉 成功</a></li>
    <li><a data-state="no-permission">🔒 无权限</a></li>
    <!-- 异常场景分组 -->
    <li class="menu-title"><span>异常场景</span></li>
    <li><a data-exception="validation_error">🚫 验证错误</a></li>
    <li><a data-exception="submit_timeout">⏱️ 提交超时</a></li>
    <li><a data-exception="session_expired">🔐 会话过期</a></li>
    <li><a data-exception="network_interrupted">📡 网络中断</a></li>
    <li><a data-exception="partial_failure_report">📊 部分失败</a></li>
    <li><a data-exception="concurrent_edit_conflict">⚔️ 编辑冲突</a></li>
    <!-- 根据注入列表动态添加 -->
  </ul>
</div>

```

---

## 6. 注入优先级与冲突规则

### 6.1 优先级排序

当多个异常可能同时显示时，按以下优先级决定显示顺序：

| 优先级 | 异常类型 | 原因 |
| --- | --- | --- |
| P0（最高） | `session_expired`, `concurrent_session` | 安全相关，阻断所有操作 |
| P1 | `network_disconnected`, `network_interrupted` | 基础能力丧失 |
| P2 | `rate_limit_warning`, `timeout_mid_process` | 操作受限 |
| P3 | `validation_error`, `concurrent_edit_conflict` | 数据完整性 |
| P4（最低） | `data_stale`, `system_announcement` | 信息性提示 |

### 6.2 冲突规则

- 同一时刻最多显示 **1 个 Modal**（P0 优先）
- 顶部 Banner 可与 Modal 共存，但最多 **2 条** Banner 堆叠
- Toast 独立于其他异常，始终可以显示
- 全局异常（§3.8）覆盖页面级异常的同类提示

---

## VERIFICATION

```yaml
verification:
  module_name: "Exception Injection"
  completeness_checks:
    - "注入矩阵覆盖 7 种页面特征类型": true
    - "全局异常覆盖 4 种全局场景": true
    - "每个异常类型有完整 HTML 模式代码": true
    - "表单类异常 4 种全部有 HTML 模式": true
    - "列表类异常 3 种全部有 HTML 模式": true
    - "详情类异常 3 种全部有 HTML 模式": true
    - "仪表盘类异常 2 种全部有 HTML 模式": true
    - "文件上传类异常 4 种全部有 HTML 模式": true
    - "向导/多步骤类异常 3 种全部有 HTML 模式": true
    - "批量操作类异常 3 种全部有 HTML 模式": true
    - "全局异常 4 种全部有 HTML 模式": true
    - "注入执行算法步骤完整（6 步）": true
    - "异常状态切换器扩展定义完整": true
    - "优先级与冲突规则定义清晰": true

  cross_references:
    - "注入矩阵页面类型 → 与 State Matrix §2 页面类型分类对齐"
    - "异常 HTML 使用 DaisyUI 类 → 与 Tech Stack 模块一致"
    - "Alpine.js x-show/x-transition → 与 Page Generator 模块模式一致"
    - "注入时机 → 在 Page Generator 输出之后执行"

  usage_in_skill:
    - "接收 Page Generator 输出 + PageSpec → 执行注入算法"
    - "为每个匹配的异常类型注入对应 HTML 模式"
    - "扩展状态切换器以包含异常场景选项"
    - "输出增强后的完整原型 HTML"

  quality_gates:
    - "每个异常必须有用户可操作的恢复路径（按钮/链接）"
    - "异常 UI 不能破坏页面基础布局"
    - "Modal 类异常必须有关闭/取消选项"
    - "所有异常文案必须是中文"
    - "倒计时/自动重试必须有手动覆盖选项"
    - "批量操作失败必须列出具体失败项"

```

---

## Phase 5: 质量检查与验收标准

### 质量检查清单

# Quality Checklist 模块 — 页面生成质量验证

> **模块职责**: 定义原型页面生成后的质量验证清单，包括逐页检查（10 项）和全局一致性检查（5 项），以及检查失败后的修复协议。确保每个生成产物达到生产级原型标准，不遗漏关键状态和交互细节。

---

## 1. 页面生成质量检查清单（Per-Page Validation）

对每个生成的页面执行以下 **10 项**验证，全部通过方可标记该页面为"已完成"：

### 1.1 必选状态覆盖 (4 项)

| # | 检查项 | 验证标准 | 失败判定 |
| --- | --- | --- | --- |
| 1 | ✅ Ideal 状态展示了该页面的完整功能和数据 | 所有业务组件可见；数据使用具体中文示例（非占位符）；交互元素均可感知 | 页面存在空白区域、placeholder 文字、或未渲染的组件插槽 |
| 2 | ✅ Loading 状态使用骨架屏，保持布局结构一致 | 骨架屏块数量和尺寸与 Ideal 状态的内容区域一一对应；使用 `animate-pulse` + `skeleton` 类 | 使用 spinner 居中代替；骨架屏布局与 Ideal 不匹配；无动画效果 |
| 3 | ✅ Empty 状态包含：说明文案 + 引导操作(CTA按钮) + (可选)插画占位 | 文案解释为空原因（结合业务上下文）；CTA 按钮文字具体（如"创建第一个项目"而非"开始"）；有 SVG 插画或图标占位 | 仅显示"暂无数据"无引导；CTA 文字过于笼统；完全空白无任何视觉元素 |
| 4 | ✅ Error 状态包含：错误原因说明 + 恢复操作按钮(重试/返回) | 使用 `alert-error` 样式；错误文案具体描述原因（如"服务器连接超时"而非"出错了"）；包含"重试"按钮 | 仅显示通用错误文字；无恢复操作按钮；未使用错误色系样式 |

### 1.2 角色权限验证 (2 项)

| # | 检查项 | 验证标准 | 失败判定 |
| --- | --- | --- | --- |
| 5 | ✅ 切换到最低权限角色后，受限功能正确隐藏或禁用 | 角色切换器切换后：受限按钮添加 `btn-disabled` 或 `hidden`；受限菜单项不可见或灰显标注"无权限" | 切换角色后页面无变化；受限按钮仍可点击；无视觉区分 |
| 6 | ✅ 403 状态提供了清晰的权限说明和操作路径 | 显示锁定图标 + "无权访问此页面" 文案；提供至少两个出口：申请权限 / 联系管理员 / 返回上一页 | 仅显示空白页面；无操作出口；文案为英文或 Lorem ipsum |

### 1.3 交互完整性 (2 项)

| # | 检查项 | 验证标准 | 失败判定 |
| --- | --- | --- | --- |
| 7 | ✅ 所有可点击元素有明确的视觉反馈 | 按钮有 `hover:` 样式变化；链接有 `hover:underline` 或颜色变化；卡片有 `hover:shadow` 提升；光标为 `cursor-pointer` | 悬停无任何视觉变化；光标保持默认箭头；无法区分可点击与不可点击元素 |
| 8 | ✅ 表单提交有即时反馈 | 提交按钮点击后：按钮文字变为"提交中..."或显示 loading spinner；按钮添加 `btn-disabled` 防止重复点击 | 点击后无任何变化；可重复点击；无加载指示器 |

### 1.4 导航一致性 (1 项)

| # | 检查项 | 验证标准 | 失败判定 |
| --- | --- | --- | --- |
| 9 | ✅ 当前页面在导航中高亮标记，面包屑/标题栏正确 | 侧边栏或底部导航中当前项有 `active` 类高亮；Web 端有面包屑路径；Mobile 端标题栏显示当前页面名称 | 所有导航项样式相同无法区分当前位置；面包屑缺失或路径错误；标题栏为空 |

### 1.5 内容完整性 (1 项)

| # | 检查项 | 验证标准 | 失败判定 |
| --- | --- | --- | --- |
| 10 | ✅ 所有文案使用具体的中文描述 | 标题、按钮、说明文字、表格数据均为有意义的中文内容；模拟数据贴合业务场景 | 出现 Lorem ipsum；出现 "xxx"/"placeholder" 占位；按钮文字为英文（除品牌词外）；数据为无意义随机字符 |

---

## 2. 全局一致性检查（Global Validation）

在**所有页面**生成完毕后，执行以下 **5 项**全局验证：

| # | 检查项 | 验证标准 | 失败判定 |
| --- | --- | --- | --- |
| G1 | ✅ 角色切换器在所有页面间保持状态一致 | 切换角色一次后，所有页面（通过 hash 路由导航）均响应角色变化；使用全局变量 `window.currentRole` 统一管理 | 某些页面未响应角色切换；角色状态在页面切换后重置 |
| G2 | ✅ 状态模拟器正确切换所有页面的状态展示 | 状态切换器在每个页面都存在（右上角固定位置）；点击后正确显示对应 `data-state-view` 容器 | 某页面缺少状态切换器；切换后显示空白或多个状态同时可见 |
| G3 | ✅ 页面间导航跳转正确 | 侧边栏/底部导航中每个菜单项点击后：URL hash 变化、对应页面内容显示、其他页面隐藏 | hash 路由不生效；点击后无反应；显示错误页面内容 |
| G4 | ✅ 视觉风格统一 | 所有页面使用同一 DaisyUI theme（`data-theme` 属性一致）；间距使用统一的 Tailwind spacing scale；颜色仅使用 DaisyUI 语义色（primary/secondary/accent/neutral 等） | 页面间配色不一致；混用自定义颜色和语义色；间距不规则 |
| G5 | ✅ HTML 文件在主流浏览器中双击打开正常显示 | 文件使用 CDN 引入 Tailwind + DaisyUI（无本地依赖）；无 CORS 限制资源；viewport 设置正确 | 打开后样式丢失；JS 报错阻断渲染；需要本地服务器才能运行 |

---

## 3. 检查执行协议

### 3.1 逐页检查流程

```
生成页面 N 完毕
    ↓
执行 10 项 Per-Page Validation
    ↓
┌─── 全部通过 ───┐      ┌─── 存在失败项 ───┐
│                │      │                  │
│  标记页面 N     │      │  进入修复协议     │
│  为"已完成"    │      │  (见第 4 节)      │
│                │      │                  │
└────────────────┘      └──────────────────┘

```

### 3.2 全局检查时机

```
所有页面均标记为"已完成"
    ↓
执行 5 项 Global Validation
    ↓
┌─── 全部通过 ───┐      ┌─── 存在失败项 ───┐
│                │      │                  │
│  原型生成完毕   │      │  定位问题页面     │
│  输出交付物    │      │  进入修复协议     │
│                │      │                  │
└────────────────┘      └──────────────────┘

```

---

## 4. 失败修复协议 (Failure Recovery Protocol)

当任何检查项未通过时，执行以下修复流程：

### 4.1 修复步骤

```
Step 1: 识别具体缺失元素
  ├── 确定失败的检查项编号（如 #3 Empty 状态缺失 CTA）
  ├── 定位缺失发生的文件和代码区域
  └── 记录期望的正确表现

Step 2: 生成定向修复代码
  ├── 仅生成缺失部分的 HTML 片段（非整页重写）
  ├── 确保修复片段符合 State Matrix 视觉模式规则
  └── 保持与现有代码风格一致（缩进、类名命名等）

Step 3: 精准插入修复
  ├── 使用 file_edit 定位到正确的插入位置
  ├── 插入修复片段
  └── 不修改其他正常工作的代码

Step 4: 重新验证
  ├── 仅重新执行失败的检查项（非全部 10 项）
  ├── 确认修复通过
  └── 如仍失败 → 回到 Step 1（最多重试 3 次）

```

### 4.2 最大重试限制

| 场景 | 最大重试次数 | 超限处理 |
| --- | --- | --- |
| 单个检查项修复 | 3 次 | 在 HTML 中添加 `<!-- TODO: [检查项描述] 需要人工修复 -->` 注释，继续后续检查 |
| 全局检查修复 | 2 次 | 输出修复建议清单，标记为"需人工确认" |

### 4.3 常见失败场景与修复模式

| 失败检查项 | 典型原因 | 标准修复模式 |
| --- | --- | --- |
| #1 Ideal 状态不完整 | 组件未生成或数据为空 | 补充缺失组件的完整 HTML + 填充示例数据 |
| #2 Loading 骨架屏不匹配 | 骨架块数量与 Ideal 布局不对应 | 参照 Ideal 状态的 DOM 结构重新生成匹配的 skeleton 块 |
| #3 Empty 缺失 CTA | 只有文案没有按钮 | 在文案下方添加 `<button class="btn btn-primary mt-6">` |
| #4 Error 缺失重试按钮 | alert 内无操作按钮 | 在 alert-error 容器末尾添加重试按钮 |
| #5 角色切换无效 | JS 未监听角色变化事件 | 补充 `onRoleChange` 事件处理，添加元素的 `data-role-min` 属性 |
| #6 403 无操作出口 | 只有文案没有按钮组 | 添加"申请权限" + "联系管理员"双按钮组 |
| #7 无 hover 反馈 | 缺少 hover 类 | 为可点击元素添加 `hover:bg-base-200` 或 `hover:shadow-md` |
| #8 表单无提交反馈 | 缺少 onclick 处理 | 添加按钮点击事件：disable + 文字变更 + loading 指示 |
| #9 导航未高亮 | 缺少 active 类逻辑 | 在路由切换 JS 中添加 `menu-active` 类的动态切换 |
| #10 英文/占位文案 | 生成时未替换 | 逐一替换为符合业务上下文的中文文案 |

---

## 5. 检查结果记录模板

每次检查完毕后，以如下格式记录结果（用于生成报告或调试）：

```markdown
### 质量检查报告 — [页面名称]

**检查时间**: [ISO 时间戳]
**页面类型**: [页面类型（来自 State Matrix）]
**适用状态数**: [N] / 15

#### Per-Page Validation 结果

| # | 检查项 | 结果 | 备注 |
|---|--------|:----:|------|
| 1 | Ideal 状态完整 | ✅/❌ | |
| 2 | Loading 骨架屏匹配 | ✅/❌ | |
| 3 | Empty 状态完整 | ✅/❌ | |
| 4 | Error 状态完整 | ✅/❌ | |
| 5 | 角色权限隐藏/禁用 | ✅/❌ | |
| 6 | 403 操作路径完整 | ✅/❌ | |
| 7 | 点击元素视觉反馈 | ✅/❌ | |
| 8 | 表单提交即时反馈 | ✅/❌ | N/A if no form |
| 9 | 导航高亮正确 | ✅/❌ | |
| 10 | 中文文案完整 | ✅/❌ | |

**通过率**: [N]/10
**修复次数**: [N]
**最终状态**: ✅ 全部通过 / ⚠️ 部分需人工确认

```

---

## 6. 与其他模块的协作关系

```
State Matrix (状态矩阵)
    ↓ 提供：适用状态列表 + 视觉模式规则
Quality Checklist (本模块)
    ↓ 验证：生成结果是否覆盖所有必要状态
    ↓ 触发：修复协议填补缺失
Acceptance Criteria (验收标准)
    ↓ 前置：验收标准定义了期望行为
    ↓ 本模块验证实际输出是否满足验收标准

```

### 协作规则

1. **检查项 #1-#4** 直接对应 State Matrix 的 4 种强制状态的视觉模式规则
2. **检查项 #5-#6** 验证 `no-permission` 状态在角色切换场景下的表现
3. **检查项 #7-#8** 确保交互级别的原型保真度
4. **检查项 #9** 确保多页面原型的导航框架正确
5. **检查项 #10** 是面向中文用户的原型的基本完整性保证

---

## VERIFICATION

```yaml
verification:
  module_name: "Quality Checklist"
  completeness_checks:
    - "Per-Page Validation 包含 10 个检查项": true
    - "检查项覆盖 4 大类别（状态覆盖/权限/交互/导航/内容）": true
    - "每个检查项有明确的验证标准": true
    - "每个检查项有明确的失败判定": true
    - "Global Validation 包含 5 个检查项": true
    - "失败修复协议步骤清晰（4 步）": true
    - "定义了最大重试限制": true
    - "包含常见失败场景与修复模式（10 种）": true
    - "包含检查结果记录模板": true
    - "与 State Matrix 的协作关系明确": true

  cross_references:
    - "检查项 #1-#4 对应 State Matrix 的 7 种强制状态中的 4 种核心状态"
    - "检查项 #5-#6 对应 State Matrix 的 no-permission 状态"
    - "检查项 #2 的骨架屏标准引用 State Matrix 3.1 Loading 视觉模式规则"
    - "检查项 #3 的 Empty 标准引用 State Matrix 3.2 Empty 视觉模式规则"
    - "检查项 #4 的 Error 标准引用 State Matrix 3.3 Error 视觉模式规则"
    - "全局检查 G2 引用 State Matrix 6.1 状态切换器组件"
    - "修复协议使用 file_edit 精准插入 → 与 Skill 工具链一致"

  usage_in_skill:
    - "页面生成完毕后自动触发 Per-Page Validation"
    - "全部页面完成后触发 Global Validation"
    - "失败时自动进入 Failure Recovery Protocol"
    - "检查结果写入检查报告（可输出给用户）"
    - "最大重试 3 次后标记为人工修复项，不阻塞后续流程"

  quality_gates:
    - "Per-Page: 10/10 通过才标记页面为已完成"
    - "Global: 5/5 通过才标记原型为可交付"
    - "修复协议最多 3 次重试，超限降级为人工标记"
    - "检查项 #8（表单提交反馈）对无表单页面标记 N/A，不计入通过率"

```

### 验收标准模板

# Acceptance Criteria 模块 — 验收标准生成模板

> **模块职责**: 定义验收标准（Given-When-Then）的生成模板和规则。Skill 在生成每个 HTML 页面时，将验收标准作为 HTML 注释嵌入文件顶部，确保每个页面的期望行为有明确的可验证描述。验收标准同时服务于质量检查清单（Quality Checklist）的验证依据。

---

## 1. 验收标准注释模板

以下为嵌入 HTML 文件顶部的验收标准注释格式：

```html
<!--
=== 验收标准 (Acceptance Criteria) ===

页面: [PageName]
生成日期: [YYYY-MM-DD]
覆盖状态: [N]/15
适用角色: [Role1], [Role2], ...

--- AC-[PageName]-1: 理想状态 ---
Given: 用户以 [Role] 角色访问 [Page]
When: 数据加载完成
Then: 显示 [具体内容描述，包含关键数据项和组件名称]
And: [补充条件或视觉要求]

--- AC-[PageName]-2: 空状态 ---
Given: [Role] 角色的 [Page] 无数据
When: 页面加载完成
Then: 显示空状态视图，包含 "[引导文案原文]" 和 "[CTA按钮文字原文]"
And: 显示 [占位插画描述]

--- AC-[PageName]-3: 加载状态 ---
Given: 用户访问 [Page]
When: 数据正在加载中
Then: 显示骨架屏，保持 [布局描述：如"左侧边栏 + 右侧内容区 + 顶部统计卡片"] 结构
And: 骨架块数量与理想状态的内容区域一一对应

--- AC-[PageName]-4: 错误状态 ---
Given: [Page] 数据请求失败
When: 页面尝试加载
Then: 显示错误提示 "[错误文案原文]" + 重试按钮
And: 错误提示使用红色 alert 样式，包含错误图标

--- AC-[PageName]-5: 权限不足 ---
Given: [Low-role] 角色尝试访问 [Page/Action]
When: 权限检查未通过
Then: [隐藏/禁用/显示403页面] + "[说明文案原文]"
And: 提供 [操作出口：申请权限/联系管理员/返回]

--- AC-[PageName]-6: 部分加载 ---
Given: [Page] 中 [ModuleA] 加载成功但 [ModuleB] 失败
When: 页面渲染完成
Then: [ModuleA] 正常显示，[ModuleB] 位置显示错误提示卡片 + 单独重试按钮
And: 错误卡片不影响其他模块的正常使用

--- AC-[PageName]-7: 操作成功 ---
Given: 用户在 [Page] 完成 [操作描述]
When: 操作执行成功
Then: 显示成功 Toast "[成功文案原文]"，3秒后自动消失
And: [下一步引导描述]

--- AC-[PageName]-N: [状态名称] ---
Given: [前置条件]
When: [触发动作]
Then: [期望结果]
And: [补充条件]

--- AC-[PageName]-NEG-1: [反向测试名称] ---
Given: [前置条件]
When: [触发动作]
Then: 不应该 [不应发生的行为描述]

-->

```

---

## 2. 验收标准生成规则

### 2.1 基本规则

| # | 规则 | 说明 |
| --- | --- | --- |
| 1 | **一状态一条 AC** | State Matrix 中每个适用状态对应一条独立的验收标准 |
| 2 | **业务上下文具体化** | 所有 AC 必须结合页面的业务场景，禁止通用模板化描述 |
| 3 | **角色边界覆盖** | 每个权限边界点（角色 A 可见但角色 B 不可见的功能）必须有独立的 AC |
| 4 | **包含反向测试** | 每个页面至少 1 条 "不应该" 类型的 AC（负面测试用例） |
| 5 | **中文具体文案** | Then 子句中的文案必须使用实际中文描述，不可使用占位符 |

### 2.2 AC 编号规则

```
格式: AC-[PageName]-[序号]
  - PageName: 使用页面的英文短名，如 Dashboard, OrderList, UserProfile
  - 序号: 从 1 开始，按状态优先级排列
  - 反向测试: AC-[PageName]-NEG-[序号]

排列顺序（固定）:
  1. 理想状态 (ideal)
  2. 空状态 (empty)
  3. 加载状态 (loading)
  4. 错误状态 (error)
  5. 权限不足 (no-permission)
  6. 部分加载 (partial)
  7. 操作成功 (success)
  8-N. 上下文状态（按 State Matrix 目录顺序）
  NEG-1~N. 反向测试用例

```

### 2.3 Given-When-Then 写法规范

Given（前置条件）

```
✅ 正确: Given: 用户以"运营经理"角色访问"订单管理"页面，且系统中已有 50 条订单
❌ 错误: Given: 用户访问页面
❌ 错误: Given: User visits the page

规范:
- 必须指定角色名称
- 必须指定页面名称
- 如有数据量或状态前提，必须说明

```

When（触发动作）

```
✅ 正确: When: 数据接口返回 200 且包含订单列表数据
✅ 正确: When: 点击"导出报表"按钮
❌ 错误: When: 页面加载
❌ 错误: When: something happens

规范:
- 动作必须是单一、可观察的事件
- 对于系统行为，描述具体的触发条件
- 对于用户操作，描述具体的交互动作

```

Then（期望结果）

```
✅ 正确: Then: 页面显示订单列表表格，包含"订单号/客户/金额/状态/操作"5列，每页展示 20 条
✅ 正确: Then: 显示空状态插画 + "暂无待处理订单" + "去创建订单"蓝色按钮
❌ 错误: Then: 显示数据
❌ 错误: Then: 页面正常工作

规范:
- 必须描述可观察的具体视觉结果
- 包含关键 UI 元素（按钮文字、标题文案、图标类型）
- 如有颜色/样式要求，明确说明

```

And（补充条件，可选）

```
✅ 正确: And: 表格支持按"订单号"列升序/降序排列
✅ 正确: And: Loading 动画持续期间，提交按钮保持 disabled 状态
规范:
- 用于补充 Then 无法完全描述的附加行为
- 每条 AC 最多 2 个 And 子句

```

---

## 3. 按页面类型的 AC 生成示例

### 3.1 Dashboard（仪表盘）页面示例

```html
<!--
=== 验收标准 (Acceptance Criteria) ===

页面: Dashboard
生成日期: 2026-08-02
覆盖状态: 11/15
适用角色: 管理员, 运营经理, 普通员工

--- AC-Dashboard-1: 理想状态 ---
Given: 用户以"管理员"角色访问"数据概览"仪表盘，系统中有近 30 天运营数据
When: 所有数据接口返回成功
Then: 显示 4 个统计卡片（今日订单数/总收入/活跃用户/转化率）+ 趋势折线图 + 最近订单列表（前 5 条）
And: 统计卡片显示同比/环比变化百分比，正增长为绿色，负增长为红色

--- AC-Dashboard-2: 空状态 ---
Given: "管理员"角色的仪表盘无任何历史数据（新账户首次使用）
When: 页面加载完成
Then: 显示空状态视图，包含"您的数据面板尚无数据"说明 + "开始配置数据源"主按钮
And: 统计卡片区域显示为灰色 "—" 占位

--- AC-Dashboard-3: 加载状态 ---
Given: 用户访问"数据概览"仪表盘
When: 数据正在加载中
Then: 显示骨架屏，保持"顶部4卡片 + 中间图表区 + 底部列表"三层布局结构
And: 骨架块使用 pulse 动画，卡片区为 4 个等宽矩形块

--- AC-Dashboard-4: 错误状态 ---
Given: "数据概览"仪表盘数据接口返回 500 错误
When: 页面尝试加载
Then: 显示错误提示"数据加载失败，服务器暂时无法响应" + "重新加载"按钮
And: 错误提示使用红色 alert 样式，居中显示

--- AC-Dashboard-5: 权限不足 ---
Given: "普通员工"角色尝试访问"收入分析"模块
When: 权限检查未通过
Then: "收入分析"卡片显示锁定图标覆盖 + "升级为管理员可查看" 提示
And: 其他有权限的模块正常显示，仅受限模块被遮挡

--- AC-Dashboard-6: 部分加载 ---
Given: 仪表盘中"统计卡片"接口成功但"趋势图表"接口失败
When: 页面渲染完成
Then: 统计卡片正常显示数据，图表区域显示"图表加载失败" + "重试"按钮
And: 失败模块的重试不影响已成功加载的其他模块

--- AC-Dashboard-NEG-1: 非管理员不应看到删除操作 ---
Given: "普通员工"角色访问仪表盘
When: 页面加载完成
Then: 不应该显示任何"删除"或"清空数据"类型的危险操作按钮

-->

```

### 3.2 表单页面 AC 示例

```html
<!--
=== 验收标准 (Acceptance Criteria) ===

页面: CreateOrder
生成日期: 2026-08-02
覆盖状态: 9/15
适用角色: 管理员, 运营经理

--- AC-CreateOrder-1: 理想状态 ---
Given: "运营经理"角色访问"创建订单"表单页
When: 页面加载完成
Then: 显示完整的订单创建表单，包含"客户信息/商品选择/配送方式/备注"4个区块
And: 所有必填字段标注红色星号(*)，提交按钮初始为蓝色可点击状态

--- AC-CreateOrder-7: 操作成功 ---
Given: "运营经理"在创建订单表单中填写完整信息
When: 点击"提交订单"按钮且服务端返回成功
Then: 显示成功 Toast "订单创建成功，订单号 #ORD-2026001"，3秒后自动消失
And: 自动跳转至订单详情页

--- AC-CreateOrder-8: 验证错误 ---
Given: "运营经理"未填写必填字段"客户名称"
When: 点击"提交订单"按钮
Then: "客户名称"输入框变为红色边框 + 下方显示"请输入客户名称"错误提示
And: 页面自动滚动到第一个错误字段位置，提交按钮恢复可点击状态

--- AC-CreateOrder-NEG-1: 重复提交防护 ---
Given: "运营经理"点击"提交订单"按钮
When: 请求正在处理中（尚未返回结果）
Then: 不应该允许再次点击提交按钮（按钮显示"提交中..."且为禁用状态）

--- AC-CreateOrder-NEG-2: 普通员工不应访问 ---
Given: "普通员工"角色尝试访问"创建订单"页面
When: 页面路由匹配
Then: 不应该显示表单内容，而是显示 403 无权限页面

-->

```

---

## 4. 上下文状态的 AC 生成模式

对于 State Matrix 中 8 种上下文状态，按以下模式生成 AC：

### 4.1 Offline（离线状态）

```
--- AC-[Page]-N: 离线状态 ---
Given: 用户正在使用 [Page]
When: 设备网络连接中断
Then: 页面顶部显示黄色横幅 "当前处于离线模式，数据将在恢复连接后自动同步"
And: 已加载的数据保持可见，新的操作请求被缓存

```

### 4.2 Timeout（超时状态）

```
--- AC-[Page]-N: 超时状态 ---
Given: 用户在 [Page] 触发 [长耗时操作]
When: 请求超过 [N] 秒未响应
Then: 显示"请求处理时间较长，请耐心等待"提示 + 进度指示器 + "取消"按钮
And: 超过 [M] 秒后显示"请求超时" + "重试"按钮

```

### 4.3 Session Expired（会话过期）

```
--- AC-[Page]-N: 会话过期 ---
Given: 用户在 [Page] 操作期间
When: 后端返回 401（Token 已过期）
Then: 弹出全屏遮罩 Modal "您的登录已过期，请重新登录" + "去登录"按钮
And: Modal 不可关闭（无 X 按钮），防止在未认证状态下继续操作

```

### 4.4 Rate-Limited（限流状态）

```
--- AC-[Page]-N: 限流状态 ---
Given: 用户在 [Page] 短时间内连续操作
When: 后端返回 429 Too Many Requests
Then: 显示"操作过于频繁，请 [N] 秒后再试"提示 + 倒计时显示
And: 倒计时结束后操作按钮自动恢复可用

```

### 4.5 Stale Data（数据过期）

```
--- AC-[Page]-N: 数据过期 ---
Given: 用户在 [Page] 查看缓存数据
When: 缓存时间超过 [N] 分钟
Then: 页面顶部显示黄色 Banner "数据最后更新于 [时间]，可能已过期" + "刷新"按钮
And: 页面内容仍可正常浏览和操作

```

### 4.6 Maintenance（维护模式）

```
--- AC-[Page]-N: 维护模式 ---
Given: 系统进入计划维护
When: 用户尝试访问任何页面
Then: 显示全屏维护页面，包含维护插画 + "系统正在升级维护中" + "预计恢复时间: [时间]"
And: 提供"查看系统状态"链接

```

### 4.7 Degraded（降级模式）

```
--- AC-[Page]-N: 降级模式 ---
Given: [Page] 依赖的 [第三方服务名] 不可用
When: 页面加载时检测到服务异常
Then: 受影响区域显示黄色 Warning 卡片 "[功能名] 暂时不可用，其他功能正常使用"
And: 非依赖该服务的功能模块正常工作

```

### 4.8 Validation Error（验证错误）

```
--- AC-[Page]-N: 验证错误 ---
Given: 用户在 [Page] 的 [表单名] 输入不合法数据
When: 失去焦点或点击提交
Then: 错误字段输入框变为红色边框 + 紧贴字段下方显示 "[具体错误提示]"
And: 错误提示与字段关联明确，修正输入后错误提示实时消失

```

---

## 5. 反向测试用例生成规则

每个页面**至少**生成 1 条反向 AC，推荐按以下类别覆盖：

### 5.1 反向测试分类

| 类别 | 描述 | 示例 |
| --- | --- | --- |
| **权限越界** | 低权限角色不应看到/操作高权限功能 | 普通员工不应看到"系统设置"入口 |
| **重复操作** | 防止重复提交/重复创建 | 提交中不应允许再次点击 |
| **数据泄露** | 不应展示超出角色数据范围的信息 | 员工 A 不应看到员工 B 的薪资 |
| **状态污染** | 一个页面的错误不应影响其他页面 | 订单页的错误不应导致仪表盘也显示错误 |
| **UI 残留** | 状态切换后不应有上一状态的残留 | 切换到空状态后不应仍显示数据表格 |

### 5.2 反向 AC 写法

```
--- AC-[Page]-NEG-[N]: [反向测试名称] ---
Given: [前置条件]
When: [触发动作]
Then: 不应该 [具体描述不应发生的行为]

```

**注意**: "不应该" 是 Then 子句的固定前缀，后面接具体可观察的行为描述。

---

## 6. AC 覆盖计数规则

### 6.1 计算公式

```
覆盖状态数 = 该页面实际生成的 AC 条数中对应的唯一状态数
总状态数 = 15（固定）

显示格式: 覆盖状态: [N]/15

```

### 6.2 状态到 AC 的映射表

| State ID | AC 标题关键词 | 必选/按需 |
| --- | --- | --- |
| ideal | 理想状态 | 必选 |
| empty | 空状态 | 必选 |
| loading | 加载状态 | 必选 |
| error | 错误状态 | 必选 |
| partial | 部分加载 | 必选 |
| success | 操作成功 | 必选 |
| no-permission | 权限不足 | 必选 |
| offline | 离线状态 | 按需（State Matrix 矩阵判定） |
| timeout | 超时状态 | 按需 |
| session-expired | 会话过期 | 按需 |
| rate-limited | 限流状态 | 按需 |
| stale-data | 数据过期 | 按需 |
| maintenance | 维护模式 | 按需（全局只需一次） |
| degraded | 降级模式 | 按需 |
| validation-error | 验证错误 | 按需（有表单时必选） |

---

## 7. Skill 生成流程中的 AC 使用

### 7.1 生成时机

```
Step 1: 解析 PageSpec（页面规格）
Step 2: 执行 State Matrix 状态枚举算法 → 获得 ApplicableStates[]
Step 3: 为每个 ApplicableState 生成对应的 AC 条目
Step 4: 生成反向测试 AC（至少 1 条）
Step 5: 组装完整 AC 注释块
Step 6: 将 AC 注释插入 HTML 文件顶部（<!DOCTYPE html> 之前）
Step 7: 开始生成 HTML 页面内容

```

### 7.2 AC 与质量检查的关系

```
AC 注释 (本模块生成)
    ↓ 定义了期望行为
Quality Checklist (质量检查清单)
    ↓ 逐项验证实际 HTML 是否满足 AC 描述
    ↓ 失败时引用 AC 编号定位问题
修复协议
    ↓ 根据失败的 AC 精准定位缺失内容
    ↓ 修复后重新验证对应 AC

```

### 7.3 输出位置

AC 注释块嵌入 HTML 文件的**最顶部**，格式如下：

```html
<!-- 验收标准注释块 -->
<!--
=== 验收标准 (Acceptance Criteria) ===
...
-->
<!DOCTYPE html>
<html lang="zh-CN" data-theme="light">
<head>
  ...
</head>
<body>
  ...
</body>
</html>

```

---

## VERIFICATION

```yaml
verification:
  module_name: "Acceptance Criteria"
  completeness_checks:
    - "完整的 AC 注释模板结构": true
    - "5 条 AC 生成基本规则": true
    - "AC 编号规则和排列顺序": true
    - "Given-When-Then 写法规范（含正反例）": true
    - "Dashboard 页面完整 AC 示例": true
    - "表单页面完整 AC 示例": true
    - "8 种上下文状态的 AC 生成模式": true
    - "反向测试用例生成规则和分类": true
    - "AC 覆盖计数规则和映射表": true
    - "与 Skill 生成流程的集成说明": true
    - "AC 与 Quality Checklist 的协作关系": true

  cross_references:
    - "AC 状态列表来源 → State Matrix 的 15 状态目录"
    - "AC 适用性判断 → State Matrix 的状态枚举算法"
    - "AC 验证执行 → Quality Checklist 的 Per-Page Validation"
    - "AC 编号中的 PageName → 与页面路由的 hash 名称一致"
    - "AC 中的角色名 → 与 Role Matrix（角色权限模块）定义一致"
    - "AC 视觉描述 → 与 State Matrix 的视觉模式规则对应"

  usage_in_skill:
    - "PageSpec 解析后自动触发 AC 生成"
    - "AC 注释作为 HTML 文件的第一部分输出"
    - "Quality Checklist 引用 AC 编号进行逐项验证"
    - "修复协议通过 AC 编号定位需要修复的功能点"
    - "AC 覆盖计数用于最终质量报告的完整性指标"

  quality_gates:
    - "每个页面的 AC 必须覆盖 7 种强制状态"
    - "每个页面至少 1 条反向测试 AC"
    - "所有 AC 的 Then 子句使用具体中文描述（非占位符）"
    - "AC 编号全局唯一，无重复"
    - "有表单的页面必须包含 validation-error 的 AC"

```

---

## HTML 骨架模板

### 生成步骤

1. **根据平台选择骨架模板**（Web 或 Mobile）
2. **填入页面清单**：根据 Phase 1 解析结果，为每个页面创建 `<section x-show="currentPage === 'pageName'">`
3. **逐页填充状态**：为每个页面，按照 Phase 4 的页面生成指令，为所有适用状态生成对应 HTML
4. **注入异常场景**：根据页面类型，自动注入对应异常场景 HTML
5. **更新导航和路由**：确保所有页面在导航中可切换
6. **嵌入验收标准**：在文件顶部以 HTML 注释形式嵌入 Given-When-Then 验收标准

### 技术栈固定配置

第三方资源必须锁定精确版本并带 Subresource Integrity（SRI）哈希 —— 浮动版本号
（`3.x.x`、`@4`）意味着上游任何一次发布都会直接进入客户拿到的原型，CDN 被投毒时
没有任何校验拦得住。

```html
<!DOCTYPE html>
<html lang="zh-CN" data-theme="light">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>[应用名称] - 交互原型</title>
  <script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3.16.0/dist/cdn.min.js"
          integrity="sha384-7qqNHWB62JqNIpl7h3HbRqJfUKJZgCFtPGHurPMRhJ6/eFmGMzQ7BJiMDRyX6nyK"
          crossorigin="anonymous" referrerpolicy="no-referrer"></script>
  <!-- Tailwind Play CDN 是运行时 JIT 编译器，产物随请求变化，无法施加 SRI。
       Tailwind 官方明示它仅供开发/原型使用，不得用于生产。生产环境请改用
       构建期生成的静态 CSS。-->
  <script src="https://cdn.tailwindcss.com/3.4.16"></script>
  <link href="https://cdn.jsdelivr.net/npm/daisyui@4.12.24/dist/full.min.css" rel="stylesheet"
        integrity="sha384-2V5uSMIWpBK7suX6yRDZH6ll7ktPJF2O58y0HSz+HiFCBCsmqZpxX1AZB4qAHuYI"
        crossorigin="anonymous" referrerpolicy="no-referrer">
</head>

```

**升级依赖版本时**：SRI 哈希必须重新计算，不能沿用旧值，否则浏览器会直接拒绝加载。

```bash
curl -sL https://cdn.jsdelivr.net/npm/alpinejs@<版本>/dist/cdn.min.js \
  | openssl dgst -sha384 -binary | openssl base64 -A
```

**生成的原型是演示工件，不是生产代码**。原型 HTML 里应保留一句提示：此原型仅用于
需求确认与交互演示，不得用于生产环境。

**渲染文本一律用 `x-text`**。`x-html` 等价于 `innerHTML`，只允许用于骨架内硬编码的
可信常量（例如导航图标的 SVG 字面量）。任何来自用户输入、URL 参数、localStorage 或
外部数据的值一旦经 `x-html` 渲染即构成 XSS。

### Web 骨架核心结构

- 固定左侧边栏导航（角色过滤）
- 顶部控制面板（角色切换 + QA 模式 + 状态模拟器）
- Hash 路由系统
- 响应式 max-w-7xl 内容区

### Mobile 骨架核心结构

- 底部 Tab Bar（3-5 项）
- 顶部标题栏
- max-w-md 全宽布局
- 离线状态 Banner

---

## Lessons Learned

### Do

- 每个页面生成后立即运行质量检查清单（10 项）
- 使用具体的中文业务文案，不使用 Lorem ipsum
- 角色切换后验证视图差异是否正确
- 生成前先输出设计概要供用户确认
- 分步生成：骨架先行，逐页填充
- 为每个页面的每种状态使用独立的 `<div x-show="...">`

### Don't

- 不要一次性生成全部页面（会超出 token 限制）
- 不要跳过 Empty/Error 状态（这是核心差异化价值）
- 不要使用需要构建步骤的框架（React/Vue/Svelte）
- 不要生成超过 10 个页面（建议拆分为多个原型）
- 不要忘记角色权限差异（切换角色后必须有可见变化）
- 不要使用 `x-if` 代替 `x-show`（避免 DOM 重建开销）

### Common Failures

- **状态遗漏**: LLM 跳过低频状态 → 用检查清单强制验证
- **token 超限**: 页面过多 → 分步生成，每次一个页面
- **角色切换无效果**: 忘记添加 `x-show="can()"` → 检查清单第 5 项
- **导航断裂**: Hash route 与 page 变量不匹配 → 全局验证
- **移动端遗漏离线状态**: → 移动端强制包含 Offline 状态

### When to Ask the User

- 需求描述 < 30 字且无法推断平台/功能时
- 角色数量 > 5 时（请用户精简到 3-5 个）
- 页面数量 > 10 时（建议拆分模块）
- 存在行业特定术语需要确认含义时