---
name: executing-plans
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints
---

# Executing Plans

## Overview

Load plan, review critically, execute all tasks, report when complete.

**Announce at start:** "I'm using the executing-plans skill to implement this plan."

**Note:** The quality of work will be significantly higher with subagent support. If subagents are available, use `run_skill({name: "subagent-driven-development"})` instead of this skill.

## The Process

### Step 1: Load and Review Plan
1. Read plan file (via `read_file`)
2. Review critically — identify any questions or concerns about the plan
3. If concerns: Raise them with your human partner before starting. **End your turn** and wait for their response — do not create the todo list yet.
4. If no concerns: Create `todo_write` with all plan tasks and proceed to Step 2

### Step 2: Execute Tasks

For each task:
1. Mark as in_progress via `todo_write`
2. Follow each step exactly (plan has bite-sized steps)
3. Run verifications as specified (via `bash`)
4. Mark as completed via `todo_write`

**Important:** The todo list tracks execution progress. All tasks are executed by you autonomously — if a task requires user input, mark the current task completed (or remove it from the list), end your turn to get the user's response, then resume with a fresh todo list. Never leave a todo item in_progress across turns.

### Step 3: Complete Development

After all tasks complete and verified:
- Announce: "I'm using the finishing-a-development-branch skill to complete this work."
- **REQUIRED SUB-SKILL:** Use `run_skill({name: "finishing-a-development-branch"})`
- Follow that skill to verify tests, present options, execute choice

## When to Stop and Ask for Help

**STOP executing immediately when:**
- Hit a blocker (missing dependency, test fails, instruction unclear)
- Plan has critical gaps preventing starting
- You don't understand an instruction
- Verification fails repeatedly

**Ask for clarification rather than guessing.**

## When to Revisit Earlier Steps

**Return to Review (Step 1) when:**
- Partner updates the plan based on your feedback
- Fundamental approach needs rethinking

**Don't force through blockers** — stop and ask.

## Remember
- Review plan critically first
- Follow plan steps exactly
- Don't skip verifications
- Reference skills when plan says to (via `run_skill`)
- Stop when blocked, don't guess
- Never start implementation on main/master branch without explicit user consent

## Integration

**Required workflow skills:**
- **using-git-worktrees** — Ensures isolated workspace. Invoke via `run_skill({name: "using-git-worktrees"})`
- **writing-plans** — Creates the plan this skill executes. Invoke via `run_skill({name: "writing-plans"})`
- **finishing-a-development-branch** — Complete development after all tasks. Invoke via `run_skill({name: "finishing-a-development-branch"})`
