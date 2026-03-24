---
name: pad-mode
description: |
  PAD Mode (Plan → Act → Deliver). Structured planning and execution workflow for complex or multi-step tasks. Breaks down user requirements into a trackable plan document, iterates with the user until approved, then executes with live progress updates and completion confirmation.

  Triggers:
  1. Slash command: "/pad" in conversation
  2. Explicit keywords: "pad mode", "plan mode", "做个计划", "帮我规划", "做计划", "enter plan mode", "make a plan"
  3. Auto-detect: When the user's request is complex (3+ distinct tasks, multi-file changes, architectural decisions, or ambiguous requirements), proactively suggest entering PAD mode.

  Use when: user wants structured execution tracking for non-trivial tasks, not for simple one-shot questions or commands.
---

# PAD Mode (Plan → Act → Deliver)

## Overview

PAD Mode transforms ambiguous requests into structured, trackable execution plans. Four phases: **Plan → Discuss → Act → Deliver**.

## Phase 1: Decompose

When triggered, analyze the user's request and create a plan document.

1. Create the plan file: `plans/YYYY-MM-DD-<short-slug>.md`
   - Use the template at `assets/plan-template.md`
   - Slug = 2-4 word summary of the task, hyphenated, lowercase
2. Fill in:
   - Title, status (`🟡 讨论中`), timestamp
   - Original requirement (user's words verbatim)
   - Understanding (your interpretation — confirm this is correct)
   - Initial task breakdown with tentative deliverables
3. **Do NOT do extensive research first.** Present a concise summary with the main tasks. Deep research happens after approval.
4. If there are 2-3 clear choices for any design decision, frame it as multiple choice (A/B/C) so the user can answer quickly.
5. Ask up to 4 clarifying questions max. Don't overwhelm.

Present the plan summary to the user and wait for feedback.

## Phase 2: Discuss

Iterate on the plan based on user feedback:

1. Update the plan document with each round of changes
2. Add entries to the change log section
3. Refine task breakdown and deliverables
4. Confirm scope boundaries — what's IN and what's OUT
5. Each task MUST have a concrete, verifiable deliverable
   - ❌ "优化代码" (vague)
   - ✅ "重构 auth 模块，将 token 验证逻辑提取到 `auth/validator.js`，原 login 路由调用新模块，测试通过" (specific)

Continue until the user says the plan is good / looks good / approved.

## Phase 3: Approve

When the user confirms the plan:

1. Update status to `🔵 已确认`
2. Lock the plan — no more scope changes without explicit user request
3. Summarize what will be executed: task list + expected deliverables
4. Move to Phase 4

## Phase 4: Execute

Execute tasks with live tracking:

1. Update status to `🟢 执行中`
2. **Before starting execution**, assess the estimated complexity:
   - If the plan has 3+ tasks or any task involves significant work (multi-file changes, deployment, etc.), **ask the user**:
     > 这个计划有 N 个任务，预计需要一些时间。你想前台执行（每步实时更新）还是转后台执行（完成后通知你）？
     - Use Telegram buttons: `前台执行` / `后台执行`
   - If the plan is simple (1-2 quick tasks), execute directly in the foreground.
3. **前台执行模式**: Work through tasks directly, notifying after each one.
4. **后台执行模式**: Spawn a sub-agent with the plan context. The sub-agent:
   - Reads the plan document
   - Executes tasks sequentially
   - Sends progress updates to the main session after each task via `sessions_send`
   - Main agent forwards updates to the user
5. Work through tasks **in order of dependencies** (independent tasks may run in parallel via sub-agents)
6. For each task:
   - Update task status to `🔄 进行中` in the plan doc
   - Execute the work
   - On success: mark `✅ 完成`, fill in notes with what was done
   - On failure: mark `❌ 失败`, document the issue, propose a fix or skip
   - **Notify the user immediately** after each task completes
7. If a task reveals that the plan needs adjustment:
   - Pause execution
   - Update the plan doc and change log
   - Ask the user before continuing

## Phase 5: Completion Confirmation

After all tasks are marked complete, **do NOT automatically close the plan**. Instead:

1. Update status to `⏳ 待确认`
2. Send a completion summary to the user:
   > 📋 计划「XXX」所有任务已完成。交付物清单：
   > - T1.1 ✅ xxx
   > - T2.1 ✅ xxx
   > ...
   > 
   > 你确认一下是否都符合预期？有需要修改的地方吗？
3. Use Telegram buttons: `✅ 确认归档` / `🔧 还有修改`
4. If user clicks **确认归档**:
   - Update status to `✅ 已完成`
   - Add archive timestamp to the plan doc
   - Send final confirmation
5. If user clicks **还有修改**:
   - Go back to Phase 2 (Discuss) to refine
   - Add new tasks if needed
   - Resume Phase 4 execution

## Parallel Execution

When tasks are independent (no shared dependencies), use sub-agents for parallel execution:

```
Task A (independent) ── sub-agent 1 ──┐
Task B (independent) ── sub-agent 2 ──┤── merge results ── update plan doc
Task C (depends on A) ── wait for A ──┘
```

Always update the plan doc from the main agent, not from sub-agents.

## Plan Document Location

All plans live in: `~/.openclaw/workspace/plans/`

Create the directory if it doesn't exist. Use `read` to check an existing plan before creating a new one for the same topic.

## Resuming a Plan

If the user references an existing plan (e.g., "继续上次的计划"), search for it in `plans/`, read the doc, identify the last completed task, and resume from there.
