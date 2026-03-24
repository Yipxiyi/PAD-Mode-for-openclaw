# PAD Mode for OpenClaw

[中文文档](./README_zh.md)

**Plan → Act → Deliver** — A structured task execution skill for OpenClaw.

## Why PAD Mode?

Large language models are powerful but fragile in long execution chains. Without structure, they tend to:

- **Lose track** of what's been done and what's left
- **Drift off-topic** during multi-step tasks
- **Skip deliverables** or leave work half-finished
- **Forget context** after long conversations

PAD Mode was inspired by the `plan mode` patterns in **Codex** and **Claude Code** — where structured planning dramatically improves execution reliability. The idea is simple: **don't start coding until the plan is clear, and don't call it done until the user confirms.**

This brings that same discipline to OpenClaw: decompose first, agree on scope, execute with tracking, and verify completion together.

## How It Works

Four phases that enforce discipline at every step:

```
  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
  │   PLAN   │───▶│ DISCUSS  │───▶│   ACT    │───▶│ DELIVER  │
  │ Decompose│    │ Iterate  │    │ Execute  │    │ Confirm  │
  └──────────┘    └──────────┘    └──────────┘    └──────────┘
       │               │               │               │
       ▼               ▼               ▼               ▼
  Create plan    Refine scope    Track progress   Review & archive
  doc + tasks    + deliverables  per task         or iterate more
```

## Triggers

| Method | Example |
|--------|---------|
| Slash command | `/pad` |
| Keywords | "pad mode", "plan mode", "make a plan" |
| Auto-detect | Complex requests (3+ tasks, multi-file, architectural) |

## Key Features

- 📋 **Plan Document** — Every task has a concrete, verifiable deliverable. No vague "improve the code".
- 🔄 **Foreground / Background** — Complex plans can run in background (sub-agent) with real-time progress updates pushed to you.
- ✅ **Completion Gate** — Plans never auto-close. You must review and confirm before archiving.
- 📦 **Parallel Execution** — Independent tasks run concurrently via sub-agents.
- 🔁 **Resumable** — Interrupted plans can be picked up from the last checkpoint.
- 📝 **Change Log** — Every modification is tracked in the plan document.

## Status Flow

```
🟡 Discussing → 🔵 Confirmed → 🟢 Executing → ⏳ Pending Review → ✅ Completed → 📦 Archived
```

## Installation

### Method 1: From source

```bash
git clone https://github.com/Yipxiyi/PAD-Mode-for-openclaw.git
cp -r PAD-Mode-for-openclaw ~/.openclaw/workspace/skills/pad-mode/
```

### Method 2: .skill file

Download `PAD_mode.skill` from the [releases](https://github.com/Yipxiyi/PAD-Mode-for-openclaw/releases) page.

## Plan Document

Each plan generates a Markdown file in `plans/`:

```markdown
# 📋 Project Name

**Status:** 🟢 Executing
**Created:** 2026-03-24 18:00
**Deliverables:** Specific verifiable outputs

## Task Breakdown
- [ ] **T1.1** Task description
  - Deliverable: What exactly will be produced
  - Dependencies: none
  - Status: ✅ Done

## Execution Log
| Time | Task | Result | Notes |
|------|------|--------|-------|
```

## Background

Inspired by the structured planning modes in **OpenAI Codex** and **Anthropic Claude Code**, which proved that LLMs perform significantly better when forced to plan before executing. PAD Mode brings this pattern to OpenClaw with additional features:

- **User confirmation gates** (plan approval + completion review)
- **Foreground/background execution** choice
- **Real-time progress tracking** via plan documents
- **Sub-agent parallelism** for independent tasks

The goal: make OpenClaw as reliable for long-running tasks as it is for quick one-shots.

## License

MIT
