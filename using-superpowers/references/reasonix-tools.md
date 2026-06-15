# Reasonix Tool Mapping

Skills in this pack use generic tool names adapted from Claude Code. When you encounter these in a skill, use the Reasonix equivalent:

## Tool Name Mapping

| Skill references | Reasonix equivalent |
|-----------------|---------------------|
| `Task` tool (dispatch subagent) | `task` tool with `prompt` parameter |
| Multiple `Task` calls (parallel) | Multiple `task` calls with `run_in_background: true` |
| Task status/output | Collect via `wait` tool; notified when done |
| `Skill` tool (invoke a skill) | `run_skill` tool |
| `TodoWrite` (task tracking) | `todo_write` tool (planning) / `complete_step` tool (sign off a step WITH evidence — command/diff/files; empty evidence is rejected) |
| `Read` (file reading) | `read_file` tool |
| `Write` (file creation) | `write_file` tool |
| `Edit` (file editing) | `edit_file` tool |
| `MultiEdit` (multi-edit) | `multi_edit` tool |
| `Bash` (run commands) | `bash` tool |
| `Grep` (search file content) | `grep` tool |
| `Glob` (search files by name) | `glob` tool |
| `WebFetch` | `web_fetch` tool |
| `WebSearch` | No direct equivalent — use `web_fetch` with a search engine URL |
| `Ask` (structured user input) | `ask` tool (supports multiple-choice questions) |
| `EnterPlanMode` / `ExitPlanMode` | No tool — present your plan as text; the user approves before changes |

## Subagent Dispatch in Reasonix

Reasonix's `task` tool spawns a sub-agent in its own session with a filtered tool list. Only its final answer returns to you — its tool calls and reasoning never enter your context.

### Dispatching an implementer/reviewer subagent

When a skill says "dispatch a subagent using the implementer-prompt template", do this:

```
task tool:
  description: "Implement Task N: [task name]"  (or "Review spec compliance for Task N")
  prompt: |
    [Fill in the FULL prompt template from the skill's references/ section.
     Replace ALL placeholders like {DESCRIPTION} or [FULL TEXT of task].
     The subagent does not see this conversation — give it everything it needs.]
  tools: [optional whitelist, e.g. ["read_file", "write_file", "edit_file", "bash", "grep"]]
  model: [optional — use a cheaper model for mechanical tasks, stronger for judgment]
```

### Parallel dispatch (for dispatching-parallel-agents)

When dispatching multiple independent subagents:

```
task tool with run_in_background: true for each independent task.
Then use wait to collect all results.
```

### Iterative review loops (Reasonix advantage)

Reasonix supports `continue_from` — you can resume a prior subagent's session for iterative fix→review cycles:

```
1. Dispatch implementer subagent → get result + "Subagent reference: sa_..."
2. Dispatch spec reviewer as a NEW task
3. If issues found, dispatch implementer again with continue_from: "sa_..." (same session, retains context)
```

## Environment Detection

Skills that create worktrees or finish branches detect their environment with read-only git commands:

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

- `GIT_DIR != GIT_COMMON` → already in a linked worktree (skip creation)
- `BRANCH` empty → detached HEAD (cannot branch/push/PR from sandbox)

## Platform Notes

- **No native worktree tool**: Reasonix has no `EnterWorktree` equivalent. Always use the `git worktree add` fallback path (Step 1b in using-git-worktrees).
- **No EnterPlanMode tool**: Present designs and plans as text in your reply. The user reviews and approves before you proceed to implementation.
- **Visual companion**: The browser-based visual companion from brainstorming is not available. Use terminal-based presentation (text descriptions, ASCII diagrams) or the `ask` tool for structured choices instead.

## Reasonix Skill Triggering

This is the most important platform difference. **Reasonix does NOT auto-load skill bodies at session start.** Claude Code and Gemini CLI both preload skill metadata (or auto-inject a bootstrap skill) so the framework's workflows engage automatically. Reasonix does not — it only pins each skill's one-line `description` into the system prompt as an index, and loads the full body on demand when YOU call `run_skill`.

Consequences for this skill pack:

- **The `using-superpowers` bootstrap is not auto-injected.** In Claude Code it runs at every session start; in Reasonix it is just another index line unless you choose to `run_skill` it. The same applies to every downstream skill (brainstorming, test-driven-development, etc.) — they only fire if you invoke them.
- **You are the trigger.** Before responding to any user message, scan the `# Skills` index block in your system prompt. If even one skill is plausibly relevant, call `run_skill({ name: "<skill-name>" })` to load it, THEN follow it. Loading one imperfect skill is cheap; skipping a skill that applied is the failure mode.
- **Prefer `ask` over prose questions.** Reasonix's system prompt explicitly says: for any real fork (approach, library, scope), call the `ask` tool with 2-4 structured options instead of asking in prose. When a skill says "ask the user", use the `ask` tool — not an open-ended prose question — unless the question is genuinely open-ended.
- **Sign off steps with evidence.** Reasonix has `complete_step`, a stronger primitive than `todo_write`: finishing a sub-task requires evidence (a command that ran, a diff, files touched); empty evidence is rejected. When a skill's checklist says "mark this todo completed", prefer `complete_step` with concrete evidence over a bare `todo_write` flip.

### Trigger self-check (every turn)

Before you write any code, answer, or clarifying question, run this checklist:

1. What kind of task is this? (new feature → brainstorming; bug → systematic-debugging; plan to execute → executing-plans / subagent-driven-development; claiming done → verification-before-completion; etc.)
2. Scan the `# Skills` index for a matching skill.
3. If one matches even plausibly → `run_skill` it NOW, before anything else.
4. Follow the loaded skill's workflow exactly.

If you skip this check and act directly, you are breaking the methodology this pack exists to enforce.
