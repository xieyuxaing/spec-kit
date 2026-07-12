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

## 今日练习结论

1. `specify init` 已经成功完成，不能只根据普通 `ls` 的输出判断是否失败。
2. WSL 中以 `.` 开头的目录默认隐藏，需要使用 `ls -la`、`ls -A` 或 `find` 查看。
3. Codex 集成内容位于 `.agents/skills/`，Spec Kit 核心内容位于 `.specify/`。
4. `specify init` 不会生成具体功能的 `specs/`，它们将在正式开始功能规格时创建。
5. WSL 是当前 Spec Kit 学习仓库的主环境，Windows 仓库只作为后续同步镜像。
