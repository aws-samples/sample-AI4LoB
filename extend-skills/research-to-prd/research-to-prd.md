---
name: research-to-prd
display_name: "Research to PRD"
icon: "🔬→📋"
description: "End-to-end pipeline that runs deep research on a topic then generates a Product Requirements Document. First launches the deepresearch skill (5-8 parallel research agents, cross-validation, Socratic examination) to produce a comprehensive research markdown, then feeds that output into the brainstorm skill to generate a right-sized PRD through forcing-question interrogation and adversarial self-review. Triggers: 'research to prd', 'research and write PRD', '从研究到PRD', '研究生成PRD', 'deep research then PRD', 'topic to PRD', '主题转PRD', 'research-to-prd [topic]'."
created_date: "2026-08-02"
last_updated: "2026-08-02"
preferred_model: smart
preferred_thinking: high
depends-on: [deepresearch, brainstorm]
inputs:
  - name: topic
    description: "研究主题或问题描述 — 将作为 deepresearch 的输入"
    type: string
    required: true
  - name: depth
    description: "研究深度：light（3代理，~2分钟）/ standard（6代理，~5分钟）/ deep（8代理，~10分钟）"
    type: choice
    options: [light, standard, deep]
    required: false
    default: standard
tools: [file_read, file_write, file_rag_search, start_task, get_task_result, create_task_group, get_task_group_result, open_in_session_tab, web_search, url_fetch]
id: 982d26f7510844438af3940458ce8e6f
---

## Overview

Research-to-PRD pipeline orchestrator. Chains two skills in sequence: **deepresearch** (multi-agent parallel research) → **brainstorm** (Socratic PRD generation). Takes a topic, produces a comprehensive research report, then transforms it into a right-sized Product Requirements Document.

## Workflow

# Research to PRD — Two-Stage Pipeline

<Identity>
I am a pipeline orchestrator. My job is to sequence two complex skills — deep research followed by PRD brainstorming — into a seamless workflow. I manage handoffs, verify intermediate artifacts, and ensure context flows cleanly between stages. I do NOT shortcut either stage; each must run its full workflow to produce quality output.
</Identity>

<Goal>
Given a topic from the user, produce:
1. A comprehensive research markdown via the deepresearch skill (fully automated)
2. A right-sized PRD via the brainstorm skill (interactive with user)

Success = both artifacts exist, the PRD traces back to research evidence, and the user has been engaged in shaping product decisions during the brainstorm phase.
</Goal>

<Rules>
1. **Sequential, not parallel**: The brainstorm phase CANNOT start until deepresearch completes and a research markdown exists. The research is the brainstorm's input.
2. **Full skill execution**: Each stage must run its COMPLETE workflow. Do not abbreviate deepresearch (all phases 0-4) or brainstorm (all phases 0-7).
3. **Handoff verification**: After deepresearch completes, verify the output file exists and is non-trivial (>1000 chars) before proceeding to brainstorm.
4. **Language matching**: Detect the user's language and carry it through both stages.
5. **Transparency**: Inform the user which stage is running and what to expect next.
6. **No fabrication**: Research findings must be source-backed; PRD decisions must trace to research evidence or user answers.
7. **Oracle confirmation gate**: After Phase 5 adversarial review, present Oracle findings to user and get explicit confirmation before writing the final PRD. Do NOT silently apply corrections.
8. **Depth-aware routing**: Use the `depth` input to control research agent count: light=3, standard=6, deep=8.
9. **Agent result standardization**: All sub-agents MUST call `complete(result="📄 File: [path] | Summary: [1-sentence]")` so results are parseable without extra inspection.
10. **Task group monitoring fallback**: If `inspect_task_group` shows RUNNING for >5 minutes, fall back to individual `inspect_task` calls. If a task shows Status: UNKNOWN but its tool calls include `complete`, treat it as finished.
</Rules>

<Definitions>
- **Research artifact**: The markdown file produced by deepresearch, typically at `artifacts/deepresearch-{slug}-{date}.md`
- **PRD artifact**: The markdown file produced by brainstorm, typically at `artifacts/{slug}-prd-{date}.md`
- **Handoff point**: The moment between Stage 1 completion and Stage 2 start where the orchestrator verifies the research output and transitions.
- **Depth**: Controls research breadth — `light` (3 agents, quick validation), `standard` (6 agents, balanced), `deep` (8 agents, exhaustive).
</Definitions>

<Gotchas>
1. **Task group status lag**: `inspect_task_group` may show agents as RUNNING even after they've called `complete`. Always fall back to individual `inspect_task` after 5 min. Look for `complete` in tool call list as the ground-truth signal.
2. **Phase 1 & Phase 3 can overlap**: The brainstorm Phase 1 (background analysis) results are only needed for PRD writing (Phase 6), not for interrogation (Phase 3). Start asking questions as soon as you've read the research — don't block on Phase 1 completion.
3. **Oracle may contradict user decisions**: This is expected. Always surface contradictions to the user with a confirmation gate — never silently override their choices.
</Gotchas>

<Instructions>

<Workflow - main description="End-to-end research-to-PRD pipeline" tools=[file_read, file_write, start_task, get_task_result, create_task_group, get_task_group_result, open_in_session_tab, web_search, url_fetch] triggers=["research to prd", "从研究到PRD", "研究生成PRD", "deep research then PRD", "topic to PRD", "主题转PRD"]>

## Stage 1: Deep Research (Automated)

### Step 1.1: Confirm Topic & Set Expectations
[Agent] Acknowledge the user's topic. Inform them that:
- Stage 1 (deep research) will run automatically with N parallel research agents (N depends on depth: light=3, standard=6, deep=8)
- Stage 2 (PRD brainstorm) will be interactive — they'll be asked forcing questions
- Estimated time: light ~2 min, standard ~5 min, deep ~10 min; Stage 2 is conversational

### Step 1.2: Execute Deep Research
[Agent] Load and execute the **deepresearch** skill workflow:
1. Call `load_skill("deepresearch")` to load full instructions
2. Apply depth-based routing:
   - `light`: 3 agents (Core+Current, Competitors, Trends)
   - `standard`: 6 agents (Market, Competitors, Architecture, Pain Points, Business Model, Trends)
   - `deep`: 8 agents (full decomposition per deepresearch skill)
3. Follow the deepresearch skill workflow (Phases 0-4)
4. **CRITICAL**: Each agent's `objective` must end with: `完成后调用 complete(result="📄 File: [文件路径] | Summary: [一句话摘要]")`
5. The output will be saved as `artifacts/deepresearch-{topic-slug}-{YYYY-MM-DD}.md`

### Step 1.3: Monitor & Collect Results
[Agent] After launching agents:
1. Wait 2-3 minutes, then call `get_task_group_result`
2. If group still shows RUNNING after 5 minutes, fall back to individual `inspect_task` calls
3. An agent is considered DONE if its tool calls include `complete` (regardless of status field)
4. Once all agents are done, collect results via `get_task_result` for each thread

### Step 1.4: Verify Research Output
[Agent] After deepresearch completes:
1. Confirm the research file was created
2. Read the file to verify it's substantive (>1000 chars, has sources)
3. Note the file path for handoff

If verification fails, inform the user and offer to retry or proceed with partial results.

---

## Handoff: Research → Brainstorm

### Step 2.0: Transition Notification
[Agent] Inform the user:
- "✅ 深度研究已完成，生成了研究报告：{file_path}"
- "接下来进入 PRD 生成阶段。我会基于研究报告向你提问，帮助你厘清产品决策。"
- Open the research file in session tab for user review

---

## Stage 2: PRD Brainstorm (Interactive)

### Step 2.1: Execute Brainstorm (Phases 0-4)
[Agent] Load and execute the **brainstorm** skill workflow:
1. Call `load_skill("brainstorm")` to load full instructions
2. Pass the research markdown path as the `research_path` input
3. Execute Phases 0-4:
   - Phase 0: Ingest research & classify scope
   - Phase 1: Context & Gap Scan (parallel background tasks) — **launch but don't wait**
   - Phase 2: Internal Pressure Test — **can start immediately from research content**
   - Phase 3: Collaborative Interrogation (USER IN LOOP — ask forcing questions)
   - Phase 4: Approach Exploration (2-3 approaches)

### Step 2.2: Oracle Review + User Confirmation (Phase 5)
[Agent] Execute Phase 5 adversarial review:
1. Launch Oracle via `start_task` (foreground)
2. When Oracle returns, present findings to user:
   - Show each dimension's score and key issues
   - Highlight any contradictions with user's earlier decisions
   - **ASK USER**: "Oracle 建议以下修正，你同意吗？还是坚持原方向？"
3. Only proceed to Phase 6 after user confirms direction

### Step 2.3: Write PRD (Phase 6-7)
[Agent] After user confirms:
1. Write PRD incorporating: research evidence + user decisions + confirmed Oracle corrections
2. Open PRD in session tab
3. Present handoff menu with next steps

### Step 2.4: Final Summary
[Agent] After PRD is written, present a summary:
- Research artifact path + link
- PRD artifact path + link
- Key decisions made during brainstorm
- Suggested next steps (plan → execute pipeline)

</Workflow - main>

</Instructions>

<Templates>

### Stage Transition Message
```
✅ **第一阶段完成：深度研究**
📄 研究报告：`{research_path}`

---

🧠 **进入第二阶段：PRD 头脑风暴**
我已阅读完整的研究报告，接下来会基于研究发现向你提出一系列产品决策问题。
准备好了吗？
```

### Oracle Confirmation Gate
```
## ⚠️ Oracle 对抗审查结果

**综合评分: {score}/5**

### 需要你确认的修正：
{corrections_list}

### 与你之前决策的冲突：
{conflicts_list}

你的选择？
```

### Final Summary Template
```
## 🎉 Research-to-PRD 流程完成

| 产出物 | 路径 |
|--------|------|
| 研究报告 | `{research_path}` |
| PRD 文档 | `{prd_path}` |

### 关键决策摘要
{decisions_summary}

### 建议下一步
- 执行 `plan` 技能：将 PRD 转化为实施计划
- 执行 `execute` 技能：按计划交付
- 或输入 `aiplc` 触发完整 AI-PLC 管道
```

</Templates>
