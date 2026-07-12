# 核心命令

> 本文是 [英文原文](core.md) 的中文翻译。命令、选项名称、环境变量和文件路径保持原样。

`specify` 的核心命令负责项目初始化、系统检查和版本信息查询。

## 初始化项目

```bash
specify init [<project_name>]
```

| 选项 | 说明 |
| --- | --- |
| `--integration <key>` | 要使用的 AI 编程 Agent Integration，例如 `copilot`、`claude`、`gemini`。所有可用 key 请参阅 [Integrations 参考](integrations.md) |
| `--integration-options` | 传递给 Integration 的选项，例如 `--integration-options="--commands-dir .myagent/cmds"` |
| `--script sh\|ps` | 脚本类型：`sh`（bash/zsh）或 `ps`（PowerShell） |
| `--here` | 在当前目录初始化，而不是创建新目录 |
| `--force` | 在已有目录中初始化时强制合并或覆盖 |
| `--ignore-agent-tools` | 跳过 AI 编程 Agent CLI 工具检查 |
| `--preset <id>` | 初始化时安装 Preset |

该命令创建一个新的 Spec Kit 项目，其中包含必要的目录结构、模板、脚本和 AI 编程 Agent Integration 文件。

> [!NOTE]
> Git 仓库初始化和分支管理由 **git Extension** 负责，而该 Extension 默认不会安装。初始化完成后运行 `specify extension add git`，即可启用 Git 工作流。

使用 `<project_name>` 创建新目录；使用 `--here`（或 `.`）在当前目录初始化。如果目录已经包含文件，使用 `--force` 可以不经确认直接合并。

省略 `--integration` 时，交互式终端会提示你选择 Integration。CI 或管道执行等非交互式会话默认使用 GitHub Copilot；如果需要其他 Integration，请显式传入 `--integration <key>`。

### 示例

```bash
# 使用指定 Integration 创建新项目
specify init my-project --integration copilot

# 在当前目录初始化
specify init --here --integration copilot

# 强制合并到非空目录
specify init --here --force --integration copilot

# 使用 PowerShell 脚本（Windows/跨平台）
specify init my-project --integration copilot --script ps

# 初始化时安装 Preset
specify init my-project --integration copilot --preset compliance
```

### 环境变量

| 变量 | 说明 |
| --- | --- |
| `SPECIFY_INIT_DIR` | 在目标项目目录之外（例如 monorepo 根目录）定位成员项目，无需先执行 `cd`，适合非交互式或 CI 场景。它必须指向**项目根目录**，也就是包含 `.specify/` 的目录；相对路径基于当前目录解析。路径必须存在并包含 `.specify/`，否则命令会报错，并且**不会**回退到当前目录。该值只在核心根目录辅助函数中解析一次（Bash 使用 `get_repo_root`，PowerShell 使用 `Get-RepoRoot`），因此核心功能脚本（`/speckit.plan`、`/speckit.tasks` 等）和继承该变量的 Git Extension 功能分支创建逻辑都会遵循它。`specify` CLI 对所有项目级子命令采用**相同**校验规则，包括 `specify integration …`、`specify extension …`、`specify workflow …`、`specify preset …` 以及其他操作 `.specify/` 项目的命令，因此这些命令也可以定位成员项目。未设置时，Bash/PowerShell 辅助函数继续向上查找；CLI 的项目级解析器默认只检查当前工作目录，除非某个命令明确实现了更广泛的检测，例如 Bundle 命令。 |
| `SPECIFY_FEATURE_DIRECTORY` | 覆盖已解析项目中的当前功能目录，优先级高于 `.specify/feature.json`。相对路径基于项目根目录解析。可以和 `SPECIFY_INIT_DIR` 一起使用，以非交互方式同时选择项目和功能。 |
| `SPECIFY_FEATURE` | 为非 Git 仓库覆盖功能检测。将其设置为功能目录名称，例如 `001-photo-albums`，即可在不使用 Git 分支时处理指定功能。使用 `/speckit.plan` 或后续命令前，必须先在 Agent 上下文中设置该变量。 |

> **两个独立的解析维度。** `SPECIFY_INIT_DIR` 选择**项目**，即哪个目录包含 `.specify/`；`SPECIFY_FEATURE_DIRECTORY` 或 `.specify/feature.json` 选择该项目中的**功能**。两者彼此独立，先解析项目，再解析功能。
>
> **符号链接项目根目录。** `SPECIFY_INIT_DIR` 改变的是项目所在位置，不会改变命令处理符号链接的方式；每个命令仍保持原有的当前目录路径策略。通过宽泛输入路径遍历和写入项目文件的命令，例如 `bundle` 和 `workflow run <file>`，会拒绝符号链接形式的 `.specify/`，以确保写入被限制在项目内。其他项目级命令在 `SPECIFY_INIT_DIR` 指向项目根目录时保持现有行为，其中可能包括跟随符号链接形式的 `.specify/`。

## 检查已安装工具

```bash
specify check
```

检查基于 CLI 的 AI 编程 Agent 在系统上是否可用。基于 IDE 的 Agent 不需要 CLI 工具，因此会被跳过。

该命令始终离线运行。如果某个命令的行为像旧版 Spec Kit，或者缺少预期的 CLI 功能，请运行 `specify self check`，检查本地 CLI 是否落后于最新版本。

## 版本信息

```bash
specify version
```

显示 Spec Kit CLI 版本、Python 版本、平台和架构。

不访问网络即可检查本地 CLI 支持的能力：

```bash
specify version --features
specify version --features --json
```

JSON 格式适合脚本和编程 Agent 使用，使其可以根据已安装 CLI 支持的功能选择工作流。

也可以使用下面的快捷命令检查版本：

```bash
specify --version
specify -V
```
