# Specify Init 练习记录

日期：2026-07-12

## 练习目标

在 WSL 环境中初始化一个使用 Codex 集成的 Spec Kit Demo，确认初始化结果，并理解生成目录的用途。

练习目录：

```text
/home/xieyx/projects/spec-kit/docs/learning/demos/spec-kit-demo
```

## 1. 初始化 Demo

### 执行命令

```bash
cd /home/xieyx/projects/spec-kit

uv run specify init docs/learning/demos/spec-kit-demo \
  --integration codex \
  --ignore-agent-tools \
  --script sh
```

### 参数说明

- `docs/learning/demos/spec-kit-demo`：指定 Demo 的生成位置。
- `--integration codex`：安装适用于 Codex 的 Spec Kit Skills。
- `--ignore-agent-tools`：跳过 Codex 等 Agent 工具是否已经安装的检查，不影响文件生成。
- `--script sh`：生成适用于 WSL、Linux 和 Bash 的 Shell 脚本。

## 2. 问题：运行后没有看到目录变化，是运行失败了吗？

### 答案

没有失败，初始化已经成功。

没有立即看到变化，是因为 `specify init` 生成的两个主要目录都是以 `.` 开头的 Linux 隐藏目录：

```text
.agents/
.specify/
```

另外，命令是在 WSL 主仓库中运行的，生成结果位于 WSL 文件系统，不会自动出现在 Windows 镜像仓库 `D:\studyProject\spec-kit` 中。

### 验证方法

```bash
cd /home/xieyx/projects/spec-kit

ls -la docs/learning/demos/spec-kit-demo
find docs/learning/demos/spec-kit-demo
git status --short -- docs/learning/demos/spec-kit-demo
```

本次检查结果显示：

- `spec-kit-demo` 目录存在。
- `.agents/` 和 `.specify/` 已经生成。
- 初始化共生成 27 个文件。
- Git 将整个目录识别为新增且尚未跟踪的内容。

## 3. 问题：如何在 WSL 中查看隐藏文件？

### 临时查看

```bash
# 显示所有内容，包括隐藏项、当前目录和父目录
ls -la

# 显示隐藏项，但不显示 . 和 ..
ls -A

# 递归查看指定目录中的全部内容
find docs/learning/demos/spec-kit-demo
```

Linux 中，以 `.` 开头的文件和目录默认属于隐藏项，普通 `ls` 不会显示。

## 4. 问题：如何让 WSL 中的 ls 永久显示隐藏文件？

已经在下面的文件中添加永久配置：

```text
/home/xieyx/.bash_aliases
```

配置内容：

```bash
alias ls='ls --color=auto -A'
```

这个别名会让交互式 Bash 中的普通 `ls` 默认显示隐藏项，同时保留彩色输出。

当前终端可以执行下面的命令立即加载配置：

```bash
source ~/.bashrc
```

重新打开 WSL 终端时，配置会自动生效。它只影响 WSL 的交互式 Bash，不影响 Windows 文件资源管理器。

## 5. Specify Init 生成了哪些目录？

本次实际生成的主要结构如下：

```text
spec-kit-demo/
├── .agents/
│   └── skills/
│       ├── speckit-analyze/
│       ├── speckit-checklist/
│       ├── speckit-clarify/
│       ├── speckit-constitution/
│       ├── speckit-converge/
│       ├── speckit-implement/
│       ├── speckit-plan/
│       ├── speckit-specify/
│       ├── speckit-tasks/
│       └── speckit-taskstoissues/
└── .specify/
    ├── integrations/
    ├── memory/
    ├── scripts/
    │   └── bash/
    ├── templates/
    └── workflows/
        └── speckit/
```

### `.agents/skills/`

由 `--integration codex` 生成。每个 `speckit-*` 子目录都包含对应的 `SKILL.md`，供 Codex 执行规格编写、澄清、规划、任务拆分、实现和检查等工作流。

### `.specify/scripts/bash/`

由 `--script sh` 决定，包含创建功能、检查前置条件、初始化计划和初始化任务等 Bash 脚本。

### `.specify/templates/`

保存功能规格、实施计划、任务列表、检查清单和项目原则等 Markdown 模板。

### `.specify/memory/`

保存项目长期遵循的原则，目前包含 `constitution.md`。

### `.specify/integrations/`

保存集成清单，用于记录 Codex 和 Spec Kit 安装了哪些文件，支持后续更新和卸载。

### `.specify/workflows/`

保存 Spec Kit 工作流定义和工作流注册信息。

### `.specify/` 根目录配置

- `init-options.json`：记录本次初始化选项。
- `integration.json`：记录当前启用的 Agent 集成信息。

## 6. 为什么现在还没有 specs 目录？

`specify init` 只负责初始化 Spec Kit 工作环境，不会自动创建具体功能规格。

后续开始第一个功能练习并执行 `speckit-specify` 后，才会生成类似下面的结构：

```text
specs/
└── 001-feature-name/
    └── spec.md
```

这体现了 Spec-Driven Development 的基本顺序：先初始化工作环境，再针对具体需求创建规格、计划和任务。

## 7. `.specify/`、Agent command files 和 context file 的原理是什么？

可以把 Spec Kit 分成三个层次：

```text
用户需求
   |
   v
Agent command / Skill             AI 的操作说明
   |
   v
.specify/                         模板、脚本、状态和项目原则
   |
   v
specs/<feature>/                  具体功能的 spec、plan 和 tasks
   |
   v
agent-context extension（可选）  把当前 plan 入口写入 context file
```

### `.specify/` 是项目的 SDD 工具箱

`.specify/` 保存所有功能共同使用的机器可读配置和固定工具：

- `scripts/`：执行创建功能、检查前置条件、初始化计划等确定性操作。
- `templates/`：规定 `spec.md`、`plan.md`、`tasks.md` 等工件的结构。
- `memory/constitution.md`：保存项目长期遵循的原则。
- `integrations/`：保存 Core 和 Agent Integration 安装文件的 manifest。
- `workflows/`：保存工作流定义和注册状态。
- `init-options.json`：保存初始化时选择的 Agent、脚本类型和编号方式。
- `integration.json`：保存当前启用的 Agent Integration。

`.specify/` 不保存某一个具体功能的全部业务内容。具体功能工件位于：

```text
specs/
└── 001-feature-name/
    ├── spec.md
    ├── plan.md
    └── tasks.md
```

### Agent command files 是 AI 的操作说明

Agent command files 告诉 AI：执行某个 SDD 阶段时应该按什么顺序工作、读取哪些文件、调用哪些脚本，以及生成什么结果。

Codex 使用 Skills 格式：

```text
.agents/skills/
├── speckit-specify/SKILL.md
├── speckit-clarify/SKILL.md
├── speckit-plan/SKILL.md
├── speckit-tasks/SKILL.md
└── speckit-implement/SKILL.md
```

以 `speckit-specify` 为例，Codex 读取对应 `SKILL.md` 后会：

1. 读取用户输入的功能需求。
2. 检查是否存在 Extension Hook。
3. 计算功能编号和短名称。
4. 使用 `.specify/scripts/` 中的脚本。
5. 使用 `.specify/templates/spec-template.md`。
6. 生成 `specs/<feature>/spec.md`。

Skill 通常不保存业务结果，它保存的是可重复执行的操作流程。

### Context file 是 Agent 的长期入口

不同 Agent 使用不同的 context file：

| Agent | 常见 context file |
| --- | --- |
| Codex | `AGENTS.md` |
| Claude | `CLAUDE.md` |
| Gemini | `GEMINI.md` |
| Copilot | `.github/copilot-instructions.md` |

Context file 与 Skill 的加载时机不同：

- Context file：Agent 进入项目或处理文件时读取，适合保存长期项目规则。
- `SKILL.md`：执行特定工作流时按需读取。
- `.specify/`：工作流执行过程中使用的工具、模板和状态。

### 为什么由 opt-in 的 `agent-context` Extension 管理？

`specify init` 默认不会修改 `AGENTS.md`。原因是：

1. Context file 可能已经包含用户自己的规则，自动改写存在覆盖风险。
2. 不同 Agent 使用不同的 context file。
3. 并非所有用户都希望 Spec Kit 管理 Agent 的长期上下文。

因此 context file 管理被独立成 `agent-context` Extension。只有用户明确安装或启用后，它才会管理 context file：

```bash
specify extension add agent-context
```

Extension 使用自己的配置：

```text
.specify/extensions/agent-context/agent-context-config.yml
```

配置示例：

```yaml
context_file: AGENTS.md

context_files: []

context_markers:
  start: "<!-- SPECKIT START -->"
  end: "<!-- SPECKIT END -->"
```

如果没有显式设置 `context_file`，Extension 会读取 `.specify/init-options.json` 中的 Integration key，再查询自己的 `agent-context-defaults.json`。例如 Codex 对应 `AGENTS.md`。

更新时，Extension 会查找当前功能的 `plan.md`，然后只插入或替换 marker 包围的区域：

```markdown
<!-- SPECKIT START -->
For additional context about technologies to be used, project structure,
shell commands, and other important information, read the current plan
at specs/001-feature-name/plan.md
<!-- SPECKIT END -->
```

Marker 之外的用户内容会保留。Extension 未安装或被禁用时，Spec Kit 不会创建、更新或删除 context file。

当前 `spec-kit-demo` 只安装了 Codex Integration，还没有安装 `agent-context` Extension，因此不会自动创建或修改 Demo 中的 `AGENTS.md`。

## 8. 每类文件来自 Core、Integration、Extension 还是用户输入？

### 来源标记

```text
spec-kit-demo/
├── .agents/                                      [Integration 输出]
│   └── skills/
│       └── speckit-*/SKILL.md                    [Core 内容 + Codex 转换]
│
├── .specify/
│   ├── init-options.json                         [Core 状态 + 用户 init 参数]
│   ├── integration.json                          [Core 状态 + Integration 信息]
│   ├── integrations/
│   │   ├── speckit.manifest.json                 [Core 文件清单]
│   │   └── codex.manifest.json                   [Integration 文件清单]
│   ├── scripts/bash/*                            [Core]
│   ├── templates/*                               [Core]
│   ├── workflows/*                               [Core]
│   └── memory/constitution.md                    [Core 初始化，用户维护]
│
└── 2026-07-12-specify-init-practice.md           [用户学习内容]
```

### 文件来源说明

| 文件 | 来源 | 实现方式 |
| --- | --- | --- |
| `.specify/scripts/bash/*` | Core | 根据 `--script sh` 复制 Spec Kit 自带的 Bash 脚本 |
| `.specify/templates/*` | Core | 复制 Core 模板，定义 spec、plan、tasks 等结构 |
| `.specify/workflows/speckit/workflow.yml` | Core | `init` 安装内置 `speckit` 工作流 |
| `workflow-registry.json` | Core 状态 | CLI 安装工作流时自动登记 |
| `.specify/memory/constitution.md` | Core + 用户 | Core 从模板初始化，之后由用户维护项目原则 |
| `.specify/init-options.json` | Core + 用户参数 | CLI 保存用户选择的 Agent、脚本类型和编号方式 |
| `.specify/integration.json` | Core + Integration | CLI 保存当前 Integration 及相关设置 |
| `speckit.manifest.json` | Core 管理状态 | 保存 Core 安装文件的路径和 SHA-256 |
| `codex.manifest.json` | Integration 管理状态 | 保存 Codex Integration 输出文件的路径和 SHA-256 |
| `.agents/skills/*/SKILL.md` | Core + Integration | Core 提供命令模板，Codex Integration 转换为 Skill |
| `specs/*/spec.md` | Core + 用户输入 + Agent | Agent 根据用户需求和 Core 模板生成 |
| `specs/*/plan.md` | Core + Agent | Agent 根据代码和 spec 生成技术计划 |
| `specs/*/tasks.md` | Core + Agent | Agent 根据 plan 拆分可执行任务 |
| Context file 普通区域 | 用户 | 用户维护项目长期规则 |
| Context file marker 区域 | Extension | `agent-context` 只维护 marker 包围的内容 |

### `specify init` 的实现顺序

```text
用户执行 specify init
        |
        ├─ 1. 解析用户参数
        |     integration = codex
        |     script = sh
        |
        ├─ 2. 调用 CodexIntegration.setup()
        |     生成 .agents/skills/*
        |
        ├─ 3. 写 integration.json
        ├─ 4. 安装 Core scripts 和 templates
        ├─ 5. 从模板初始化 constitution.md
        ├─ 6. 安装内置 workflow
        └─ 7. 写 init-options.json
```

### Codex Skill 的生成链路

```text
templates/commands/specify.md                  [Core 原始模板]
                 |
                 v
SkillsIntegration.process_template()           [Integration 通用转换]
                 |
                 ├─ 选择 sh 脚本命令
                 ├─ 替换 {SCRIPT}
                 ├─ 保留运行时 $ARGUMENTS
                 ├─ 重写 .specify 路径
                 └─ 生成 Skill frontmatter
                 |
                 v
.agents/skills/speckit-specify/SKILL.md        [Codex 输出]
```

初始化时，具体功能需求不会写进 `SKILL.md`。用户后续输入：

```text
speckit-specify 增加用户登录功能
```

运行时才会发生：

```text
“增加用户登录功能”                       [用户输入]
          |
          v
speckit-specify/SKILL.md 中的 $ARGUMENTS
          |
          v
Agent 执行脚本并使用模板
          |
          v
specs/001-user-login/spec.md             [用户需求派生产物]
```

### 如何快速判断文件归属？

查看两个 manifest：

- `speckit.manifest.json` 中列出的文件由 Core 管理。
- `codex.manifest.json` 中列出的文件由 Codex Integration 管理。

Manifest 保存文件路径和 SHA-256。升级时会比较当前文件与原始哈希：

- 哈希未改变：仍是受管理的原始文件，可以安全更新。
- 哈希已改变：说明用户修改过，默认保留，避免升级覆盖。
- 显式使用 `--force`：允许覆盖用户修改。

`agent-context` 的边界更严格：Integration 不生成 context file，Core CLI 也不修改它；只有用户主动安装的 `agent-context` Extension 能更新 marker 包围的区域。

## 今日练习结论

1. `specify init` 已经成功完成，不能只根据普通 `ls` 的输出判断是否失败。
2. WSL 中以 `.` 开头的目录默认隐藏，需要使用 `ls -la`、`ls -A` 或 `find` 查看。
3. Codex 集成内容位于 `.agents/skills/`，Spec Kit 核心内容位于 `.specify/`。
4. `specify init` 不会生成具体功能的 `specs/`，它们将在正式开始功能规格时创建。
5. WSL 是当前 Spec Kit 学习仓库的主环境，Windows 仓库只作为后续同步镜像。
6. `.specify/` 保存工具和状态，Agent Skill 保存操作流程，`specs/` 保存具体功能工件。
7. Context file 由 opt-in 的 `agent-context` Extension 管理，未安装时 Spec Kit 不会修改它。
8. Manifest 是判断文件归属和保护用户修改的主要依据。
