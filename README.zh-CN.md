# AI-PLC Practices

[🇬🇧 English](README.md) ｜ **中文**

面向 **Amazon Quick desktop** 的一套 Skill 集合，把"研究 → 产品定义 → 工程规划 →
交付执行"的完整产品生命周期固化成可复用的流水线。

> **这是示例代码，不适用于生产环境。** 部署前请与你所在组织的安全与法务团队确认，
> 以满足组织自身的安全、监管与合规要求。这些 Skill 产出的工件（研究报告、PRD、
> 计划、HTML 原型）是供评审的草稿，不是可直接交付的成品。

> **⚠️ 你输入的内容会离开本机。** `DeepResearch` 和 `AI4LoB` 会把研究主题以及由它
> 衍生的检索词发送给第三方搜索与推理服务。**不要把机密信息作为研究主题** —— 包括
> 未发布的产品或发布计划、内部指标、客户名称等客户可识别信息、凭证，以及任何被你
> 所在组织归类为机密的内容。请先改写为脱敏表述。改写成同义说法没有用 —— 同义
> 表述照样会被发送出去。

---

## 目录

| 目录 | 说明 |
| --- | --- |
| [`core-skills/`](core-skills/README.zh-CN.md) | 五个核心 Skill，构成 AI-PLC 流水线 |
| [`extend-skills/`](extend-skills/README.zh-CN.md) | 扩展 Skill 存放位置 |

---

## 1. 这套 Skill 是什么

五个 Skill 构成一条 **AI 产品生命周期（AI Product Lifecycle, AI-PLC）流水线**。
前四个是功能 Skill，第五个（AI4LoB）是把前四个串联起来的编排 Skill。

```
用户主题 topic
    │
    ▼
┌─────────────────┐  research.md（研究报告）
│  DeepResearch   │───────────────┐
└─────────────────┘               ▼
                       ┌─────────────────┐  PRD（含 R#/A#/F#/AE# 稳定 ID）
                       │   brainstorm    │───────────────┐
                       └─────────────────┘               ▼
                                            ┌─────────────────┐  plan.md（含 U# 单元 + DAG + 波次）
                                            │      plan       │───────────────┐
                                            └─────────────────┘               ▼
                                                                   ┌─────────────────┐
                                                                   │     execute     │──▶ 工件
                                                                   └─────────────────┘   （代码/文档/skill）
     ▲                        ▲                      ▲                      ▲
     └────────────────────────┴──────────────────────┴──────────────────────┘
                  AI4LoB（编排层，每步之间设用户确认门）
```

### 1.1 DeepResearch — 多智能体深度研究编排器

把一个研究主题分解为多个角度，并行派发 5-8 个研究代理（网页搜索、文档查询、GitHub
代码搜索），交叉验证结果构建共识矩阵，再用苏格拉底式质询挑战假设、暴露论证缺口，
最后产出带置信度分级、开放问题和来源附录的 markdown 报告。

这是一个**只读** Skill——不编辑任何代码。核心纪律是"NEVER guess. NEVER fabricate."：
每个事实性断言都必须能追溯到来源 URL，找不到证据就写"No evidence found"，绝不编造。

- **输入** — 一个研究主题（自然语言）
- **输出** — `deepresearch-{topic-slug}-{YYYY-MM-DD}.md`
- **触发词** — `deep research`、`research [topic]`、`deepresearch`

### 1.2 brainstorm — 研究报告到 PRD 的转化器

扮演"驻场产品负责人"（Product-Lead-in-residence）。先质询再综合：一次只问一个问题，
先给选项再给推荐，然后经过压力测试与对抗性审查，产出一份带稳定 ID 的 PRD
（`R#` 需求 / `A#` 角色 / `F#` 流程 / `AE#` 验收示例）。

它**绝不写代码**。Skill 自述："I am the user's thinking partner, not their
yes-person. Flattery is forbidden. Friction is the service."（我是用户的思考伙伴，
不是应声虫。禁止奉承，摩擦即服务。）PRD 里每一条需求都必须追溯到研究报告的具体段落，
或用户在质询阶段的逐字回答。

- **输入** — `deepresearch*.md` 路径（可选；不给则进入冷启动模式，把想法描述当研究输入）
- **输出** — `{slug}-prd-{YYYY-MM-DD}.md` + 口头交接菜单
- **触发词** — `brainstorm`、`turn research into PRD`、`write PRD`

### 1.3 plan — PRD 到 DAG 实施计划的规划器

扮演"驻场资深工程师"（Staff-Engineer-in-residence）。把 PRD 转化为 DAG 形状的实施
计划：拆出 `U#` 实施单元，标注依赖关系、并行波次、涉及文件、验收条件与执行分类提示。
PRD 质量越高，它需要追问的问题越少（0-8 题）。

**只做计划**——不写代码、不构建、不调用 execute。计划里每个 `R#` 引用都必须对应 PRD
中真实存在的 ID，缺失即阻塞交接。

- **输入** — 带稳定 ID 的 PRD 路径（可选；无则冷启动）
- **输出** — `docs/plans/YYYY-MM-DD-NNN-<type>-<slug>-plan.md`（若仓库有该目录），
  否则 `./{slug}-plan-{YYYY-MM-DD}.md`
- **触发词** — `plan`、`implementation plan`、`计划`、`做计划`、`实施计划`、
  `根据 PRD 做计划`（中英双语触发）

### 1.4 execute — 计划到工件的并行波次执行器

扮演纯"派发者"（delegator）。按计划的 DAG 波次并行派发分类专家子代理，收集结构化
验证结果，失败自动重试一次，仍失败则沿 DAG 传播性跳过依赖者、其余分支继续，最后统一
做一次 Oracle 对抗性审查，并把完成状态回写进计划文件、追加执行日志。

**自己从不编写工件代码**——唯一例外是用编辑工具修改计划文件里的状态行和执行日志。
计划文件格式有问题时（DAG 有环、依赖悬空、同波次文件冲突、未知分类值等），它以
`NEEDS_CONTEXT` 中止且**不派发任何单元**，禁止"编造替代方案"。

执行期间基本免打扰（"No mid-run questions"）。

- **输入** — plan 产出的计划文件路径
- **输出** — 状态回写后的同一份计划文件 + 子代理产出的各类工件
- **触发词** — `execute`、`run plan`、`执行计划`、`施工`、`实施`、`开干`、`按计划执行`

### 1.5 AI4LoB — 四步流水线总编排

把上面四个 Skill 串成 `deepresearch → brainstorm → plan → execute` 的流水线，
**每一步完成后暂停等待用户确认**。用户可以提出修改意见，agent review 后给出反馈，
形成迭代循环，直到满意才进入下一步。

相比裸用 deepresearch，AI4LoB 引入亚马逊式的 PR/FAQ 与 One-way / Two-way Door
决策框架；当主题涉及特定 AWS 服务时，还可选接入 AWS Documentation MCP server
补充官方文档。

- **输入** — 一个必填参数 `topic`
- **声明依赖** — `depends-on: [deepresearch, brainstorm, plan, execute]`
- **触发词** — `ai4lob <主题>`、`AI-PLC <topic>`、"从研究到交付"

---

## 2. 设计哲学

这套 Skill 与普通 prompt 模板的区别在下面几条纪律上。

| 原则 | 说明 |
| --- | --- |
| **硬边界 Hard gate** | 每个 Skill 只产出一种东西。DeepResearch 只读，brainstorm 只出 PRD，plan 只出计划，execute 只派发。职责单一、输出可预期、可独立替换。 |
| **禁止编造 No fabrication** | 每条断言都要能追溯到来源：研究报告的段落、用户的逐字回答、或上游真实存在的 ID。 |
| **稳定 ID 永不重编号** | `R#/A#/F#/AE#/U#` 一经分配永不回收。删除条目留下的编号空洞是正确的。拆分 `R3` 得到 `R3a`/`R3b` 或新分配下一个未用整数，绝不复用旧 ID。 |
| **六值状态机** | `PLANNED` / `IN_PROGRESS` / `DONE` / `DONE_WITH_CONCERNS` / `BLOCKED` / `NEEDS_CONTEXT`，其他值非法。升级时用统一四行格式 STATUS / REASON / ATTEMPTED / RECOMMENDATION。 |
| **双向门禁** | 不只有前向数据流：PRD 有未解决的阻塞问题 → 不许进 planning；计划校验不过 → 不许进 execute。 |
| **懒加载参考** | `references/` 下的文件只在对应阶段才读取，"context budget is finite"（上下文预算有限）。会话开始就全量加载被明确列为反模式。 |
| **Oracle 对抗审查** | 关键节点派 oracle 子代理做对抗性审查，主动找漏洞而非确认结论。 |
| **反奉承** | 明文禁止奉承。摩擦是服务的一部分。 |
| **并行优先** | 研究代理、写作代理、执行波次都尽可能并行；同一文件禁止两个代理同时写——那是竞态条件。 |

### 追溯链条

体系的可审计性建立在一条不断链的 ID 引用上：

```
研究报告的发现/引文
  → PRD 的 R#
    → 计划的 Requirements Trace（每个 R# 必须出现，缺失即阻塞交接）
      → U-block 的 Requirements 字段
        → 子代理 VERIFICATION 块（命令、退出码、输出）
          → Execution Log 的 Artifacts 清单与 Oracle 裁决
```

这正是"永不重编号"在四个 Skill 里都是 CRITICAL 级规则的原因——链条上任何一环重编号，
下游的引用全部失效。

---

## 3. 在 Amazon Quick desktop 安装

> 官方文档：
> <https://docs.aws.amazon.com/quick/latest/userguide/skills-and-agents-desktop.html>

### 3.1 前提

- 已安装并登录 **Amazon Quick desktop** 应用
- 已从本仓库 `core-skills/` 取得 Skill 文件

### 3.2 方式 A — 加目录后让 Agent 安装（推荐）

这是 Workshop 实地验证过的路径，一次装完五个 Skill，且不需要手工补挂
参考文件。

1. 克隆本仓库获取 Skill：

   ```bash
   git clone https://github.com/aws-samples/sample-AI4LoB.git
   ```

   `core-skills/` 下已是解压好的 Skill 目录，无需再解压。若你拿的是 `.zip`，先把它们
   解压到同一个文件夹里。

2. 打开 Amazon Quick，进入 **settings** → **My computer**。

3. 点击 **Add Folder**，把存放 Skill 的文件夹加进来（例如
   `sample-AI4LoB/core-skills`），并确认该目录是**已启用**状态。

4. 点击 **New Chat**，用提示词让 Agent 安装：

   ```
   请帮我安装：<目录>下的 5 个 skill
   ```

5. 等待安装完成提示，然后到 Skills 标签页确认——装好的 Skill 会显示在这里。

因为是由 Agent 执行安装，它会自行识别 `references/` 子目录，也能兼容两种指令文件
命名（`SKILL.md` 或 `<skill-name>.md`）。

### 3.3 方式 B — Import from file

官方文档描述的路径，适合只装单个 Skill。

1. 打开 Amazon Quick desktop，在左侧导航选择 **Agents & skills**，切换到
   **Skills** 标签页。

2. 点击 **+ Create**，选择 **Import from file**。

3. 选择该 Skill 的指令文件（`SKILL.md`，`extend-skills/` 下的 Skill 是
   `<skill-name>.md`）。

4. 检查并按需编辑，然后保存。Skill 会出现在 **MY SKILLS** 分组下。

5. **补挂参考文件（brainstorm / plan / execute 必做）**：这三个 Skill 各自带 3 个
   `references/*.md`。Quick 的 Skill 本质是一个文件夹，支持附加参考文件——导入指令
   文件后，进入该 Skill 的详情页，把 `references/` 下的文件作为参考文件附加上去。
   漏掉这一步，Skill 会在需要懒加载参考的阶段失败。方式 A 会自动处理这一步。

### 3.4 安装顺序与验证

**按依赖顺序安装**：AI4LoB 声明了
`depends-on: [deepresearch, brainstorm, plan, execute]`。请先装完四个功能 Skill，
再装 AI4LoB，否则 AI4LoB 编排到某一步时会找不到对应 Skill：

`DeepResearch` → `brainstorm` → `plan` → `execute` → `AI4LoB`

**验证**：在 Skills 标签页找到该 Skill，点 **Run** 开启一个已预加载该 Skill 的会话；
或直接在对话里说"use the DeepResearch skill"。

### 3.5 Kiro CLI

这套 Skill 也能在 Kiro CLI 下运行。Kiro CLI 里一个 Skill 就是 `.kiro/skills/` 下的
一个目录，把指令文件放进去即可自动加载：

```bash
mkdir -p .kiro/skills/deepresearch
cp core-skills/DeepResearch/SKILL.md .kiro/skills/deepresearch/
```

在会话里用 `/context` 确认已加载。`.kiro/skills/` 是工作区级的，只在该目录生效；
放到 `~/.kiro/skills/` 则全局可用。

`DeepResearch` 已用这种方式验证过；其余四个 Skill 目前只在 Amazon Quick desktop
上运行过。

### 3.6 更新已安装的 Skill

Quick desktop 目前没有"重新导入覆盖"的入口。更新方式：在 Skills 标签页选中该 Skill，
进入详情页点 **Edit**，用仓库里的新版 `SKILL.md` 内容替换；或删除旧 Skill 后重新导入。
建议记录你安装时的 commit SHA，便于日后比对。

---

## 4. 使用方法

### 4.1 全流程（推荐入口）

```
ai4lob <主题>
```

自动串起四步，每步之间有确认门。适合"从一个想法到可交付工件"的完整旅程。

### 4.2 单独使用各 Skill

| 场景 | 调用 | 说明 |
| --- | --- | --- |
| 只想要一份严谨的调研报告 | `deep research: <主题>` | 产出带共识矩阵和苏格拉底分析的报告 |
| 已有研究报告，要 PRD | `brainstorm <research.md 路径>` | 质询 → 方案 → 对抗审查 → PRD |
| 没有研究，直接从想法出发 | `brainstorm <想法描述>` | 冷启动：不跳过任何阶段，问全量问题 |
| 已有 PRD，要实施计划 | `plan <PRD 路径>` | 缺口扫描问 0-4 题后出 DAG 计划 |
| 没有 PRD，直接规划 | `plan`（冷启动） | 最多 8 问后出计划 |
| 已有计划，要执行 | `execute <计划路径>` 或"执行计划" | 并行波次派发，结束回写状态 |

### 4.3 各环节的用户参与度

| Skill | 参与度 |
| --- | --- |
| **DeepResearch** | 只需给主题，其余全自动。报告尾部的 DISPUTED 项和开放问题需人工判断。 |
| **brainstorm** | **最高**。需逐一回答 2-6 个强制性问题并接受顶回追问；在方案选择、审查发现、交接菜单上做决定。不耐烦时有逃生舱，但方案探索与对抗审查不可豁免。 |
| **plan** | 回答 0-8 个问题（PRD 质量越高问得越少）。只有"close call 且 one-way door"的技术决策才需要你拍板。 |
| **execute** | 基本免打扰。只在 `NEEDS_CONTEXT` 中止和最终汇报时介入。 |
| **AI4LoB** | 每步之间的确认门都需表态；研究报告可无限轮迭代修改。 |

### 4.4 中间产物出问题时

- **计划 DAG 有环 / 依赖悬空 / 同波次文件冲突** — execute 以 `NEEDS_CONTEXT`
  中止并指明问题，重跑 `plan` 修正。
- **PRD 阻塞问题未解决** — brainstorm 关门；回答阻塞问题后自动走重审回路。
- **单元执行失败** — 自动重试一次；仍失败则传递性跳过依赖者、其余分支继续。终态为
  `DONE_WITH_CONCERNS` / `BLOCKED` + 四行升级格式，由你决定是否局部重跑。

---

## 5. 关于 zip 与源文件并存

`core-skills/` 下每个 Skill **同时**提供两份内容：

```
core-skills/
├── brainstorm.zip        ← 安装用：下载、解压、导入 Quick desktop
└── brainstorm/           ← 协作用：GitHub 可 diff、可 code review、可全文搜索
    ├── SKILL.md
    └── references/
```

| 用途 | 用哪个 |
| --- | --- |
| 安装到 Quick desktop | `.zip`，或直接用解压目录里的 `SKILL.md` |
| 阅读、审查、改进 | 解压目录 |

**为什么这么做。** zip 对 Git 是二进制黑盒：改一个字，`git diff` 只会告诉你"文件变了"，
没法 code review，也搜不到内容。解压的源文件解决了这些问题，zip 则保留了给同事
"下载即用"的便利。

### ⚠️ 修改 Skill 时必须同步两边

**先改解压目录里的源文件（这是权威版本），再重新打包 zip**：

```bash
cd core-skills

# 1. 编辑源文件
vim brainstorm/SKILL.md

# 2. 重新打包，-r 递归以包含 references/
rm brainstorm.zip
cd brainstorm && zip -r ../brainstorm.zip . -x '.*' && cd ..

# 3. 确认 zip 内容
unzip -l brainstorm.zip
```

提交时两边一起提交，并在 commit message 里说明改了什么。只改一边会导致下载 zip 的
同事拿到旧版本。

---

## 6. 运行环境依赖

这套 Skill 依赖运行环境提供的若干能力，迁移到其他 Agent 平台时需要适配。

1. **子代理派发原语** — 全体系依赖一个 `task()` 原语，支持 `subagent_type`
   （`oracle` / `explore` / `librarian` / `skill-creator`）与 `category`
   （`quick` / `writing` / `unspecified-low` / `unspecified-high` /
   `visual-engineering` / `ultrabrain` / `deep` / `artistry`）两种寻址方式，
   以及后台运行、会话续用、完成通知等机制。
2. **研究工具** — DeepResearch 指名使用网页搜索、网页抓取、库文档查询、GitHub
   代码搜索等工具。
3. **AWS Documentation MCP server（可选）** — 当主题涉及特定 AWS 服务时，AI4LoB
   可接入 AWS Documentation MCP server 补充官方文档。若该 connector 未连接，
   Skill 会跳过该数据源并在报告中注明。
4. **会话交互能力** — AI4LoB 使用会话标签页展示文件、decision card 决策卡片语法等。
5. **skill-creator 校验脚本**（仅当计划要生成 Skill 时） — `execute` 会 shell out
   执行 skill-creator 的 `python3 -m scripts.quick_validate`，路径假定为
   `~/.opencode/skills/skill-creator/`。该路径是对宿主环境的假设，本仓库并不提供
   这份代码；请改成 skill-creator 在你机器上的实际位置，并注意这相当于执行一个
   用户可写目录下的代码。若未安装 skill-creator，请跳过生成 Skill 的 Unit。

---

## 7. 延伸阅读

| 文件 | 内容 |
| --- | --- |
| [`extend-skills/`](extend-skills/README.zh-CN.md) | 扩展 Skill，含 `protoforge`（自然语言 → 可交互 HTML 原型）与 `research-to-prd` |
| [`CONTRIBUTING.zh-CN.md`](CONTRIBUTING.zh-CN.md) | 如何新增 Skill |

---

## 8. 一句话总结

这套体系把"研究 → 产品定义 → 工程规划 → 交付执行"的完整产品生命周期拆成四个边界严格、
只产出单一工件、互相以稳定 ID 引用的 Skill，用并行子代理提速、用苏格拉底质询与
Oracle 对抗审查控质、用门禁与六值状态机防止半成品向下游泄漏，最后由 AI4LoB 编排成
一条从一个主题短语到可交付工件的自动化流水线。

---

## 维护

问题与建议请提 Issue 或 Pull Request。

---

## License

本项目基于 MIT-0 协议开源，详见 [`LICENSE`](LICENSE)。
