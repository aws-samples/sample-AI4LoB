# 贡献指南

[🇬🇧 English](CONTRIBUTING.md) ｜ **中文**

欢迎贡献新的 Skill。请先读完对应小节再动手。

---

## 快速开始

```bash
git clone https://github.com/aws-samples/sample-AI4LoB.git
cd sample-AI4LoB
git checkout -b feature/<简短描述>
# ... 改动 ...
git add .
git commit -m "core-skills: 修正 plan 的波次示例"
git push -u origin feature/<简短描述>
# 在 GitHub 上创建 Pull Request
```

---

## 分支与提交

- 不要直接推 `main`，走 feature 分支 + Pull Request。
- 分支命名：`feature/...`、`fix/...`、`docs/...`。
- Commit message 用 `<范围>: <做了什么>` 格式，范围取
  `core-skills` / `extend-skills` / `docs` / `readme`。

改动说明写清"为什么"，而不只是"改了什么"——`git diff` 已经能告诉别人改了什么。

---

## 修改核心 Skill

`core-skills/` 下的 Skill 影响所有使用者，改动需要 Pull Request 评审。

### ⚠️ 必须同步 zip 与源文件

每个 Skill 以两种形态存在：`<name>.zip`（安装用）和 `<name>/`（协作用）。**内容必须
一致**，只改一边会导致下载 zip 的同事拿到旧版本。

正确流程：

```bash
cd core-skills

# 1. 先改解压目录里的源文件（这是权威版本）
vim brainstorm/SKILL.md

# 2. 重新打包，-r 递归以包含 references/
rm brainstorm.zip
cd brainstorm && zip -r ../brainstorm.zip . -x '.*' && cd ..

# 3. 确认 zip 内容
unzip -l brainstorm.zip

# 4. 两边一起提交
cd .. && git add core-skills/brainstorm core-skills/brainstorm.zip
git commit -m "core-skills: <改了什么，为什么>"
```

`-x '.*'` 用于排除 macOS 产生的 `.DS_Store` 等隐藏文件。

### 校验一致性

```bash
cd core-skills
for s in DeepResearch brainstorm plan execute AI4LoB; do
  diff <(unzip -p "$s.zip" SKILL.md) "$s/SKILL.md" > /dev/null \
    && echo "$s OK" || echo "$s MISMATCH"
done
```

### 改动核心 Skill 前请先想清楚

这套 Skill 的若干设计是**刻意**的，不是遗漏。改之前请确认你理解了为什么：

- **稳定 ID 永不重编号** — 编号空洞是正确的。`R#/A#/F#/AE#/U#` 跨会话、跨 Skill
  被引用，重编号会切断追溯链。
- **六值状态机** — `PLANNED` / `IN_PROGRESS` / `DONE` / `DONE_WITH_CONCERNS` /
  `BLOCKED` / `NEEDS_CONTEXT`。新增第七个值会让下游解析器失效。
- **懒加载参考文件** — 不在会话开始时读取 `references/`，是为了保护上下文预算。
- **硬边界** — brainstorm 不写代码、plan 不执行、execute 不亲自编写工件。放宽任何
  一条都会让流水线的输出变得不可预期。

如果确实要改上述任何一条，请在 Pull Request 描述里说明影响面。

改完后请**实机验证**：导入 Quick desktop 跑一遍，在 MR 里贴出验证结果。

---

## 新增扩展 Skill

新 Skill 放 `extend-skills/`，不要直接进 `core-skills/`。分界见
[`extend-skills/README.zh-CN.md`](extend-skills/README.zh-CN.md)。

### 目录结构

```
extend-skills/
├── my-skill.zip
└── my-skill/
    ├── SKILL.md             # 或 my-skill.md，两种都可
    └── references/          # 可选
```

命名用小写连字符，目录名与 zip 名一致。指令文件可以叫 `SKILL.md`，也可以叫
`<skill-name>.md`——两种都能被正确安装（`core-skills/` 用前者，`protoforge` /
`research-to-prd` 用后者）。保持一个目录内只有一个指令文件。

### SKILL.md 应包含

参照 `core-skills/` 里的现有 Skill。至少要有：

- **name / description** — description 要写清触发条件，Quick 靠它自动选择 Skill。
- **role** — 这个 Skill 扮演什么角色，一两句。
- **触发词 triggers** — 明确列出，也明确列出**不应该**触发的情况。
- **输入 / 输出 inputs / outputs** — 输出文件的命名规则要确定，不要让 Agent 自己
  发挥。
- **阶段划分 phases** — 分步骤，每步的进入条件与产出。
- **中止条件 abort conditions** — 什么情况下应该停下来问用户，而不是猜。
- **反模式 anti-patterns** — 明确禁止什么。

### 提交前

1. 在 Quick desktop 上实际导入并跑通。
2. zip 与源文件一致。
3. 在 [`extend-skills/README.zh-CN.md`](extend-skills/README.zh-CN.md) 的索引表
   登记一行。
4. Pull Request 里说明：解决什么问题、怎么验证的。

---

## ⚠️ 禁止提交敏感信息

这是一个公开仓库。禁止提交：客户名称、合同金额、AWS 账号 ID / ARN / 真实端点、
任何凭据、人员姓名邮箱电话、NDA 材料、任何一方的业务数据样本。

判断标准：假设仓库里的内容被截图发到任何地方，会不会造成问题？

**已提交的敏感信息不能靠"再提交一次删掉"解决**——Git 历史里还在。一旦发现请立即
提 Issue。

---

## 评审标准

Pull Request 会按以下几点评审：

| 项 | 要求 |
| --- | --- |
| 功能正确 | 改动的 Skill 在 Quick desktop 上实机跑通过 |
| zip 一致性 | zip 与源文件内容一致 |
| 脱敏 | 改动中不含任何敏感信息 |
| 双语 | 每份面向使用者的文档都成对发布——`X.md`（英文）与 `X.zh-CN.md`（中文）——第 3 行放语言切换器，改动需同步两份。 |
| 索引更新 | 新增内容已在对应 README 索引表登记 |
| 说明清楚 | PR 描述讲了"为什么"，不只是"改了什么" |

---

## 联系

问题与建议请提 Issue 或 Pull Request。
