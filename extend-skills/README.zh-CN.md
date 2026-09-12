# 扩展 Skill

[🇬🇧 English](README.md) ｜ **中文**

存放核心流水线之外的 Skill：单点工具、特定客户或场景的定制 Skill、以及对核心 Skill
的实验性改版。

---

## 与 core-skills 的分界

| 放 `core-skills/` | 放 `extend-skills/` |
| --- | --- |
| 属于 AI-PLC 四步流水线本体 | 其他一切 |
| 改动影响所有使用者，需 Merge Request 评审 | 新增互不干扰，评审可从简 |
| 五个，数量稳定 | 数量随需求增长 |

**不要**为了"改进"直接覆盖 `core-skills/` 里的 Skill。先在 `extend-skills/` 里做变体
验证，跑通并有实际使用证据后，再提 Merge Request 合并回核心。

---

## 新增一个扩展 Skill

保持与 `core-skills/` 相同的结构，zip 与源文件并存：

```
extend-skills/
├── my-skill.zip
└── my-skill/
    ├── SKILL.md
    └── references/          # 可选
        └── whatever.md
```

命名约定：

- 目录名与 zip 名一致，用小写连字符。
- 一个 Skill 一个目录，不要嵌套。
- 指令文件可以叫 `SKILL.md`，也可以叫 `<skill-name>.md`——两种都能被正确安装。
  `core-skills/` 下用的是 `SKILL.md`，`protoforge` / `research-to-prd` 用的是
  `<name>.md`。保持一个目录内只有一个指令文件即可。

详细步骤与 `SKILL.md` 应包含的内容见
[CONTRIBUTING.md](../CONTRIBUTING.zh-CN.md#新增扩展-skill)。

---

## 索引

新增 Skill 后请在此登记一行。

| Skill | 用途 | 触发词 |
| --- | --- | --- |
| [`protoforge`](protoforge/) | 自然语言需求 → 研发交付级可交互 HTML 原型。15 态覆盖、三层权限模型、Web/Mobile 双范式，单文件 Alpine.js + Tailwind + DaisyUI，零构建 | `protoforge`, `生成原型`, `原型生成`, `create prototype`, `interactive mockup`, `交互原型`, `HTML原型` |
| [`research-to-prd`](research-to-prd/) | `deepresearch → brainstorm` 两段流水线，带深度档位（light 3 代理 / standard 6 / deep 8） | `research to prd`, `从研究到PRD`, `研究生成PRD`, `topic to PRD`, `主题转PRD` |

两个 Skill 都是用 AI4LoB 流水线**生成**出来的，而非手写——这本身是 AI4LoB 的一种用法：
用它造领域专用工具，而不只是造最终交付物。
