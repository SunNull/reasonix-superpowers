# Reasonix Tool Mapping

Skills in this pack use generic tool names adapted from Claude Code. When you encounter these in a skill, use the Reasonix equivalent:

## Tool Name Mapping

| Skill references | Reasonix equivalent |
|-----------------|---------------------|
| `Task` tool (dispatch subagent) | `task` tool with `prompt` parameter |
| Multiple `Task` calls (parallel) | Multiple `task` calls with `run_in_background: true` |
| Task status/output | Collect via `wait` tool; notified when done |
| `Skill` tool (invoke a skill) | `run_skill` tool |
| `TodoWrite` (task tracking) | `todo_write` tool |
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
