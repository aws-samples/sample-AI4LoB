# 核心 Skill

[🇬🇧 English](README.md) ｜ **中文**

AI4LoB 流水线的五个 Skill。安装步骤见
[仓库根目录 README 中文版](../README.zh-CN.md#3-在-amazon-quick-desktop-安装)。

---

## 索引

| Skill | 定位 | 输出 | references/ | 安装顺序 |
| --- | --- | --- | --- | --- |
| [`DeepResearch`](DeepResearch/) | 多智能体深度研究编排器 | `deepresearch-{slug}-{date}.md` | 无 | 1 |
| [`brainstorm`](brainstorm/) | 研究报告 → PRD | `{slug}-prd-{date}.md` | 3 个 | 2 |
| [`plan`](plan/) | PRD → DAG 实施计划 | `...-plan.md` | 3 个 | 3 |
| [`execute`](execute/) | 计划 → 工件 | 状态回写 + 工件 | 3 个 | 4 |
| [`AI4LoB`](AI4LoB/) | 四步流水线编排 | 编排上述四步 | 无 | 5（最后） |

⚠️ AI4LoB 声明 `depends-on: [deepresearch, brainstorm, plan, execute]`，**必须最后
安装**。

---

## 文件组织

每个 Skill 提供两份内容，用途不同：

```
core-skills/
├── DeepResearch.zip     ┐
├── brainstorm.zip       │ 安装用
├── plan.zip             │ 下载 → 解压 → 导入 Quick desktop
├── execute.zip          │
├── AI4LoB.zip            ┘
│
├── DeepResearch/        ┐
│   └── SKILL.md         │
├── brainstorm/          │ 协作用
│   ├── SKILL.md         │ GitHub 可 diff / review / 全文搜索
│   └── references/      │
│       ├── adversarial-review.md
│       ├── prd-template.md
│       └── pressure-test.md
├── plan/
│   ├── SKILL.md
│   └── references/
│       ├── forcing-questions.md
│       ├── plan-template.md
│       └── validator-checklist.md
├── execute/
│   ├── SKILL.md
│   └── references/
│       ├── oracle-review.md
│       ├── plan-parser.md
│       └── skill-generation.md
└── AI4LoB/
    └── SKILL.md
```

两份内容是同一个 Skill 的两种形态，**内容必须保持一致**。修改流程见
[CONTRIBUTING.md](../CONTRIBUTING.zh-CN.md#修改核心-skill)。

---

## 关于 references/

`brainstorm`、`plan`、`execute` 各带 3 个参考文件。这些文件是**懒加载**的——Skill
规定不要在会话开始时读取，只在对应阶段才加载，理由是"context budget is finite"
（上下文预算有限），提前加载会污染质询与推理所需的上下文。

导入 Quick desktop 时，`SKILL.md` 之外还要把 `references/` 下的文件作为参考文件
附加上去，否则 Skill 在需要它们的阶段会失败。

---

## 校验 zip 与源文件一致

```bash
cd core-skills
for s in DeepResearch brainstorm plan execute AI4LoB; do
  echo "=== $s ==="
  diff -r <(unzip -p "$s.zip" SKILL.md) "$s/SKILL.md" > /dev/null \
    && echo "SKILL.md OK" || echo "SKILL.md MISMATCH"
done
```

如报 MISMATCH，说明有人只改了一边。以解压目录里的源文件为准重新打包。
