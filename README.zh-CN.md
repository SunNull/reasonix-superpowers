# Superpowers for Reasonix（中文说明）

> 为你的 [Reasonix](https://github.com/esengine/DeepSeek-Reasonix) 编码 agent 打造的完整软件开发方法论 —— 改编自 [obra/superpowers](https://github.com/obra/superpowers)（228k⭐）。

[English](./README.md) · **简体中文**

## 这是什么

[Superpowers](https://github.com/obra/superpowers) 是一个面向编码 agent 的技能框架和软件开发方法论，最初为 Claude Code 打造。它强制执行结构化工作流：**先头脑风暴再写代码 → 编写计划 → 通过子 agent + TDD 执行 → 审查 → 交付**。

本仓库将其 **14 个核心 skill** 适配到 Reasonix 的 skill 系统中 —— 保留了完整的方法论，同时将所有 Claude Code 工具引用映射为 Reasonix 的等价工具。

## Skill 列表

### 🧭 核心工作流

| Skill | 触发时机 | 作用 |
|-------|---------|------|
| `using-superpowers` | 每次对话开始时 | 引导 skill —— 让所有其他 skill 自动触发 |
| `brainstorming` | 任何创造性工作之前 | 苏格拉底式设计精炼；设计未经批准不写代码 |
| `writing-plans` | 设计通过后 | 将工作拆分为 2-5 分钟的 TDD 任务，含精确文件路径和代码 |
| `subagent-driven-development` | 执行计划时 | 每个任务派发独立子 agent + 两阶段审查（规格 → 质量） |
| `executing-plans` | 上者的替代方案 | 批量执行，带人工检查点 |
| `finishing-a-development-branch` | 任务完成时 | 验证测试 → 提供合并/PR/保留/丢弃选项 |

### 🔬 质量与调试

| Skill | 触发时机 | 作用 |
|-------|---------|------|
| `test-driven-development` | 编写实现代码之前 | 强制 RED-GREEN-REFACTOR 循环，无例外 |
| `systematic-debugging` | 遇到任何 bug、测试失败、异常行为时 | 4 阶段根因调查；禁止猜测式修复 |
| `verification-before-completion` | 声称工作完成之前 | 先证据后结论 —— 运行验证、读取输出、然后才能声称通过 |
| `requesting-code-review` | 任务之间 / 合并之前 | 派发代码审查子 agent，提供精心构造的上下文 |
| `receiving-code-review` | 收到审查反馈时 | 技术性评估，拒绝表演式认同 |

### 🛠 元技能与协作

| Skill | 触发时机 | 作用 |
|-------|---------|------|
| `using-git-worktrees` | 需要隔离的功能开发之前 | 通过 `git worktree` 确保隔离工作区 |
| `dispatching-parallel-agents` | 2+ 个独立问题时 | 通过 `run_in_background` 并发派发子 agent |
| `writing-skills` | 创建/编辑 skill 时 | 将 TDD 应用于流程文档 |

## 安装方式

### 方式一：通过 Reasonix 安装（推荐）

在 Reasonix 会话中运行：

```
install skills from https://github.com/SunNull/reasonix-superpowers
```

或使用 install-capability skill：

```
/install-capability https://github.com/SunNull/reasonix-superpowers
```

### 方式二：手动 clone

```sh
# 项目级（仅当前项目）
git clone https://github.com/SunNull/reasonix-superpowers.git \
  .reasonix/skills/superpowers

# 全局级（所有项目）
git clone https://github.com/SunNull/reasonix-superpowers.git \
  ~/.reasonix/skills/superpowers
```

Reasonix 会自动发现 `.reasonix/skills/`（项目）和 `~/.reasonix/skills/`（全局）下的 skill。每个 `<name>/SKILL.md` 会被自动识别；`references/*.md` 文件会自动追加到 skill body 中。

### 验证安装

安装后重启 Reasonix 会话，检查：

```
/skill list
```

应该能看到全部 14 个 skill 出现在索引中。

## 工作流程

```
用户: "我们来做一个 X 功能"
  │
  ▼
  brainstorming ──── 设计文档保存到 docs/specs/
  │
  ▼
  writing-plans ───── 实现计划保存到 docs/plans/
  │
  ▼
  subagent-driven-development
  ├── 任务 1: 实现者 → 规格审查 → 代码质量审查
  ├── 任务 2: 实现者 → 规格审查 → 代码质量审查
  └── ...
  │
  ▼
  finishing-a-development-branch ──── 合并 / PR / 保留 / 丢弃
```

过程中：
- **test-driven-development** 在每个实现步骤触发
- **systematic-debugging** 在测试失败或 bug 出现时触发
- **verification-before-completion** 在任何"完成"声明之前触发

## 从 Claude Code 的关键适配

| Claude Code（原始） | Reasonix（本包） |
|---------------------|------------------|
| `Task` 工具（派发子 agent） | `task` 工具，通过 `prompt` 参数传递任务 |
| `Skill` 工具（调用 skill） | `run_skill` 工具 |
| `TodoWrite` | `todo_write` |
| `Read` / `Write` / `Edit` / `Bash` | `read_file` / `write_file` / `edit_file` / `bash` |
| `EnterPlanMode` 工具 | 以文本呈现计划；用户批准后才实施 |
| `EnterWorktree` 工具 | `git worktree add`（Reasonix 无原生等价物） |
| `@file.md` 内联引用 | `references/*.md` 自动追加到 skill body |
| `superpowers:skill-name` 跨引用 | `run_skill({name: "skill-name"})` |
| Visual Companion（HTTP 服务器） | ASCII 图表 + `ask` 工具做结构化选择 |
| 并行 `Task()` 调用 | `task` + `run_in_background: true` + `wait` 收集 |

完整映射表：[`using-superpowers/references/reasonix-tools.md`](using-superpowers/references/reasonix-tools.md)

## Reasonix 独有增强

- **`continue_from` 迭代审查循环**：Reasonix 的 `task` 工具支持会话续接（`continue_from: "sa_..."`）—— 当审查者发现问题时，续接实现者的会话而非重新开始。比原始 Claude Code 工作流更高效。
- **`run_in_background` 并行 agent**：并发派发多个独立子 agent，通过 `wait` 收集结果。
- **`ask` 工具做结构化选择**：用多选题替代开放式提问（终端体验更好）。

## 文件结构

```
reasonix-superpowers/
├── brainstorming/
│   ├── SKILL.md
│   └── references/
│       └── spec-document-reviewer-prompt.md
├── subagent-driven-development/
│   ├── SKILL.md
│   └── references/
│       ├── implementer-prompt.md          ← 实现者子 agent 提示模板
│       ├── spec-reviewer-prompt.md        ← 规格审查者提示模板
│       └── code-quality-reviewer-prompt.md ← 代码质量审查者提示模板
├── systematic-debugging/
│   ├── SKILL.md
│   └── references/
│       ├── root-cause-tracing.md          ← 根因追溯技术
│       ├── defense-in-depth.md            ← 纵深防御验证
│       ├── condition-based-waiting.md     ← 条件等待替代硬延迟
│       ├── condition-based-waiting-example.ts
│       └── find-polluter.sh              ← 测试污染二分查找脚本
├── test-driven-development/
│   ├── SKILL.md
│   └── references/
│       └── testing-anti-patterns.md       ← 测试反模式参考
├── ... （还有 10 个 skill）
├── using-superpowers/
│   ├── SKILL.md
│   └── references/
│       └── reasonix-tools.md             ← 工具映射参考
├── README.md                              ← 英文说明
├── README.zh-CN.md                        ← 中文说明（本文件）
└── LICENSE
```

## 致谢

- 原始 [Superpowers](https://github.com/obra/superpowers) 由 [Jesse Vincent](https://github.com/obra) 及 Prime Radiant 团队创建。MIT 协议。
- 适配 for [Reasonix](https://github.com/esengine/DeepSeek-Reasonix)。

## 协议

MIT
