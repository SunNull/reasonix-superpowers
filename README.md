# Superpowers for Reasonix

> A complete software development methodology for your [Reasonix](https://github.com/esengine/DeepSeek-Reasonix) coding agent — adapted from [obra/superpowers](https://github.com/obra/superpowers) (228k⭐).

## What This Is

[Superpowers](https://github.com/obra/superpowers) is an agentic skills framework and software development methodology originally built for Claude Code. It enforces structured workflows: **brainstorm before code → write a plan → execute via subagents with TDD → review → ship**.

This repo adapts its **14 core skills** to Reasonix's skill system — preserving the full methodology while mapping every Claude Code tool reference to its Reasonix equivalent.

## Skills

### 🧭 Core Workflow

| Skill | When it triggers | What it does |
|-------|-----------------|--------------|
| `using-superpowers` | Start of every conversation | Bootstrap — makes all other skills auto-trigger |
| `brainstorming` | Before any creative work | Socratic design refinement; no code until design is approved |
| `writing-plans` | After design approval | Break work into 2-5 min TDD tasks with exact file paths and code |
| `subagent-driven-development` | When executing a plan | Fresh subagent per task + two-stage review (spec → quality) |
| `executing-plans` | Alternative to above | Batch execution with human checkpoints |
| `finishing-a-development-branch` | When tasks complete | Verify tests → present merge/PR/keep/discard options |

### 🔬 Quality & Debugging

| Skill | When it triggers | What it does |
|-------|-----------------|--------------|
| `test-driven-development` | Before writing implementation code | RED-GREEN-REFACTOR enforcement, no exceptions |
| `systematic-debugging` | On any bug, test failure, unexpected behavior | 4-phase root cause investigation; no guessing |
| `verification-before-completion` | Before claiming work is done | Evidence before claims — run it, read it, then say it |
| `requesting-code-review` | Between tasks / before merge | Dispatch code reviewer subagent with crafted context |
| `receiving-code-review` | When receiving feedback | Technical evaluation, not performative agreement |

### 🛠 Meta & Collaboration

| Skill | When it triggers | What it does |
|-------|-----------------|--------------|
| `using-git-worktrees` | Before feature work needing isolation | Ensure isolated workspace via `git worktree` |
| `dispatching-parallel-agents` | 2+ independent problems | Concurrent subagent dispatch via `run_in_background` |
| `writing-skills` | Creating/editing skills | TDD applied to process documentation |

## Installation

### Option 1: Install via Reasonix (recommended)

In a Reasonix session, run:

```
install skills from https://github.com/SunNull/reasonix-superpowers
```

Or use the install-capability skill:

```
/install-capability https://github.com/SunNull/reasonix-superpowers
```

### Option 2: Manual clone

```sh
# Project scope (this project only)
git clone https://github.com/SunNull/reasonix-superpowers.git \
  .reasonix/skills/superpowers

# Global scope (all projects)
git clone https://github.com/SunNull/reasonix-superpowers.git \
  ~/.reasonix/skills/superpowers
```

Reasonix auto-discovers skills under `.reasonix/skills/` (project) and `~/.reasonix/skills/` (global). Each `<name>/SKILL.md` is picked up automatically; `references/*.md` files are auto-appended to the skill body.

### Verify installation

After install, restart your Reasonix session and check:

```
/skill list
```

You should see all 14 skills in the index.

## The Workflow

```
User: "Let's build X"
  │
  ▼
  brainstorming ──── design doc saved to docs/specs/
  │
  ▼
  writing-plans ───── plan saved to docs/plans/
  │
  ▼
  subagent-driven-development
  ├── Task 1: implementer → spec-reviewer → code-quality-reviewer
  ├── Task 2: implementer → spec-reviewer → code-quality-reviewer
  └── ...
  │
  ▼
  finishing-a-development-branch ──── merge / PR / keep / discard
```

Along the way:
- **test-driven-development** fires on every implementation step
- **systematic-debugging** fires when a test fails or a bug appears
- **verification-before-completion** fires before any "done" claim

## Key Adaptations from Claude Code

| Claude Code (original) | Reasonix (this pack) |
|------------------------|----------------------|
| `Task` tool (dispatch subagent) | `task` tool with `prompt` parameter |
| `Skill` tool (invoke skill) | `run_skill` tool |
| `TodoWrite` | `todo_write` |
| `Read` / `Write` / `Edit` / `Bash` | `read_file` / `write_file` / `edit_file` / `bash` |
| `EnterPlanMode` tool | Present plan as text; user approves before changes |
| `EnterWorktree` tool | `git worktree add` (no native equivalent in Reasonix) |
| `@file.md` inline references | `references/*.md` auto-appended to skill body |
| `superpowers:skill-name` cross-refs | `run_skill({name: "skill-name"})` |
| Visual Companion (HTTP server) | ASCII diagrams + `ask` tool for structured choices |
| Parallel `Task()` calls | `task` with `run_in_background: true` + `wait` |

Full mapping: [`using-superpowers/references/reasonix-tools.md`](using-superpowers/references/reasonix-tools.md)

## Reasonix-Specific Enhancements

- **`continue_from` for iterative review loops**: Reasonix's `task` tool supports session continuation (`continue_from: "sa_..."`) — when a reviewer finds issues, resume the implementer's session instead of starting fresh. More efficient than the original Claude Code workflow.
- **`run_in_background` for parallel agents**: Dispatch multiple independent subagents concurrently, collect results via `wait`.
- **`ask` tool for structured choices**: Replaces open-ended questions with multiple-choice (better UX in the terminal).

## File Structure

```
reasonix-superpowers/
├── brainstorming/
│   ├── SKILL.md
│   └── references/
│       └── spec-document-reviewer-prompt.md
├── subagent-driven-development/
│   ├── SKILL.md
│   └── references/
│       ├── implementer-prompt.md
│       ├── spec-reviewer-prompt.md
│       └── code-quality-reviewer-prompt.md
├── systematic-debugging/
│   ├── SKILL.md
│   └── references/
│       ├── root-cause-tracing.md
│       ├── defense-in-depth.md
│       ├── condition-based-waiting.md
│       ├── condition-based-waiting-example.ts
│       └── find-polluter.sh
├── test-driven-development/
│   ├── SKILL.md
│   └── references/
│       └── testing-anti-patterns.md
├── ... (10 more skills)
├── using-superpowers/
│   ├── SKILL.md
│   └── references/
│       └── reasonix-tools.md    ← tool mapping reference
├── README.md
└── LICENSE
```

## Credits

- Original [Superpowers](https://github.com/obra/superpowers) by [Jesse Vincent](https://github.com/obra) and the Prime Radiant team. MIT License.
- Adapted for [Reasonix](https://github.com/esengine/DeepSeek-Reasonix).

## License

MIT
