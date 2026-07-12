<div align="center">
    <img src="./media/logo_large.webp" alt="Spec Kit Logo" width="200" height="200"/>
    <h1>🌱 Spec Kit</h1>
    <h3><em>更快地构建高质量软件。</em></h3>
</div>

<p align="center">
    <strong>一个开源工具包，让你专注于产品场景和可预测的结果，而不是从零开始凭感觉编写每一个部分。</strong>
</p>

<p align="center">
    <a href="https://github.com/github/spec-kit/releases/latest"><img src="https://img.shields.io/github/v/release/github/spec-kit" alt="Latest Release"/></a>
    <a href="https://github.com/github/spec-kit/stargazers"><img src="https://img.shields.io/github/stars/github/spec-kit?style=social" alt="GitHub stars"/></a>
    <a href="https://github.com/github/spec-kit/blob/main/LICENSE"><img src="https://img.shields.io/github/license/github/spec-kit" alt="License"/></a>
    <a href="https://github.github.io/spec-kit/"><img src="https://img.shields.io/badge/docs-GitHub_Pages-blue" alt="Documentation"/></a>
</p>

<p align="center">
    <a href="README.md">English</a> | 简体中文
</p>

---

## 目录

- [什么是规格驱动开发？](#what-is-sdd)
- [快速开始](#get-started)
- [视频概览](#video-overview)
- [社区](#community)
- [支持的 AI 编程 Agent Integrations](#supported-integrations)
- [Specify CLI 参考](#cli-reference)
- [定制 Spec Kit：Extensions 与 Presets](#extensions-presets)
- [Bundles：面向角色的配置](#bundles)
- [核心理念](#core-philosophy)
- [开发阶段](#development-phases)
- [实验目标](#experimental-goals)
- [前置条件](#prerequisites)
- [深入学习](#learn-more)
- [详细流程](#detailed-process)
- [支持](#support)
- [致谢](#acknowledgements)
- [许可证](#license)

<a id="what-is-sdd"></a>

## 🤔 什么是规格驱动开发？

规格驱动开发颠覆了传统软件开发的工作方式。几十年来，代码一直处于核心位置，而规格只是我们在“真正的”编码工作开始前搭建、随后丢弃的脚手架。规格驱动开发改变了这一点：**规格变得可执行**，不再只是指导实现，而是直接生成可工作的实现。

<a id="get-started"></a>

## ⚡ 快速开始

### 1. 安装 Specify CLI

需要 **[uv](https://docs.astral.sh/uv/)**（[安装 uv](./docs/install/uv.md)）。请把 `vX.Y.Z` 替换为 [Releases](https://github.com/github/spec-kit/releases) 中的最新标签：

```bash
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@vX.Y.Z
```

其他安装方式、验证、升级和故障排除请参阅[安装指南](./docs/installation.md)。

### 2. 初始化项目

```bash
specify init my-project --integration copilot
cd my-project
```

使用自管理命令检查更新或升级已安装的 CLI。详细场景和定制选项请参阅[升级指南](./docs/upgrade.md)。

```bash
# 检查是否有新版本（只读，不修改任何内容）
specify self check

# 预览将执行的操作，但不真正升级
specify self upgrade --dry-run

# 原地升级到最新稳定版本（自动识别 uv tool 或 pipx 安装）
specify self upgrade

# 或固定到指定发布标签（把 vX.Y.Z[suffix] 替换为目标标签）
specify self upgrade --tag vX.Y.Z[suffix]
```

直接运行 `specify self upgrade` 会立即执行，这与 `pip install -U` 和 `npm update` 等命令的无提示行为一致。对于 `uv tool` 安装，它会在内部运行 `uv tool install specify-cli --force --from <git ref>`，因此可以使用固定发布标签，包括 dev、alpha/beta/rc 或构建元数据后缀。系统会识别临时 `uvx` 运行和源码 Checkout，并给出针对该路径的说明，而不是运行安装程序。可以设置 `SPECIFY_UPGRADE_TIMEOUT_SECS` 限制安装子进程的最长运行时间；默认不超时，必要时使用 `Ctrl+C` 中断。

### 3. 建立项目原则

在项目目录中启动编程 Agent。大多数 Agent 以 `/speckit.*` Slash Command 形式提供 Spec Kit；Codex CLI 的 Skills 模式使用 `$speckit-*`；GitHub Copilot CLI 使用 `/agents` 选择 Agent，也可以在提示词中直接指定。

使用 **`/speckit.constitution`** 命令创建项目治理原则和开发指南，它们会指导后续所有开发工作。

```bash
/speckit.constitution Create principles focused on code quality, testing standards, user experience consistency, and performance requirements
```

### 4. 创建规格

使用 **`/speckit.specify`** 描述要构建什么。此时专注于**做什么**和**为什么做**，而不是技术栈。

```bash
/speckit.specify Build an application that can help me organize my photos in separate photo albums. Albums are grouped by date and can be re-organized by dragging and dropping on the main page. Albums are never in other nested albums. Within each album, photos are previewed in a tile-like interface.
```

### 5. 创建技术实施计划

使用 **`/speckit.plan`** 提供技术栈和架构选择。

```bash
/speckit.plan The application uses Vite with minimal number of libraries. Use vanilla HTML, CSS, and JavaScript as much as possible. Images are not uploaded anywhere and metadata is stored in a local SQLite database.
```

### 6. 拆分任务

使用 **`/speckit.tasks`** 根据实施计划生成可执行任务列表。

```bash
/speckit.tasks
```

### 7. 执行实现

使用 **`/speckit.implement`** 执行所有任务，并按照计划构建功能。

```bash
/speckit.implement
```

详细分步说明请参阅[完整指南](./spec-driven.md)。

<a id="video-overview"></a>

## 📽️ 视频概览

想看看 Spec Kit 的实际运行效果？请观看[视频概览](https://www.youtube.com/watch?v=a9eR1xsfvHg&pp=0gcJCckJAYcqIYzv)。

[![Spec Kit video header](/media/spec-kit-video-header.jpg)](https://www.youtube.com/watch?v=a9eR1xsfvHg&pp=0gcJCckJAYcqIYzv)

<a id="community"></a>

## 🌍 社区

在 [Spec Kit 文档站点](https://github.github.io/spec-kit/)浏览社区贡献资源：

- [Extensions](https://github.github.io/spec-kit/community/extensions.html)：命令、Hook 和能力。
- [Presets](https://github.github.io/spec-kit/community/presets.html)：模板和术语覆盖。
- [Bundles](https://github.github.io/spec-kit/community/bundles.html)：由现有组件组成的角色和团队工具栈。
- [Walkthroughs](https://github.github.io/spec-kit/community/walkthroughs.html)：端到端 SDD 场景。
- [Friends](https://github.github.io/spec-kit/community/friends.html)：扩展 Spec Kit 或基于它构建的项目。

> [!NOTE]
> 社区贡献由各自作者独立创建和维护。安装前请审查源码，并自行判断是否使用。

想参与贡献？请参阅 [Extension 发布指南](extensions/EXTENSION-PUBLISHING-GUIDE.md)、[Preset 发布指南](presets/PUBLISHING.md)或[社区 Bundle 指南](docs/community/bundles.md)。

<a id="supported-integrations"></a>

## 🤖 支持的 AI 编程 Agent Integrations

Spec Kit 支持 30 多种 AI 编程 Agent，包括 CLI 工具和基于 IDE 的助手。完整列表、说明和用法请参阅[支持的 AI 编程 Agent Integrations](https://github.github.io/spec-kit/reference/integrations.html) 指南。

运行 `specify integration list` 可以查看当前安装版本中的所有 Integration。

## 可用 Slash Commands

运行 `specify init` 后，AI 编程 Agent 将获得下面这些用于结构化开发的 Slash Command。对于支持 Skills 模式的 Integration，传入 `--integration <agent> --integration-options="--skills"` 会安装 Agent Skills，而不是 Slash Command 提示词文件。

### 核心命令

规格驱动开发工作流的必要命令：

| 命令 | Agent Skill | 说明 |
| --- | --- | --- |
| `/speckit.constitution` | `speckit-constitution` | 创建或更新项目治理原则和开发指南 |
| `/speckit.specify` | `speckit-specify` | 定义要构建什么，包括需求和用户故事 |
| `/speckit.plan` | `speckit-plan` | 使用所选技术栈创建技术实施计划 |
| `/speckit.tasks` | `speckit-tasks` | 生成可执行实施任务列表 |
| `/speckit.taskstoissues` | `speckit-taskstoissues` | 把生成的任务列表转换为 GitHub Issue，用于跟踪和执行 |
| `/speckit.implement` | `speckit-implement` | 按照计划执行所有任务并构建功能 |
| `/speckit.converge` | `speckit-converge` | 根据 spec/plan/tasks 评估代码库，并把剩余工作追加为新任务 |

### 可选命令

用于增强质量和验证的附加命令：

| 命令 | Agent Skill | 说明 |
| --- | --- | --- |
| `/speckit.clarify` | `speckit-clarify` | 澄清定义不足的部分，建议在 `/speckit.plan` 前运行，原名 `/quizme` |
| `/speckit.analyze` | `speckit-analyze` | 跨工件一致性和覆盖率分析，在 `/speckit.tasks` 后、`/speckit.implement` 前运行 |
| `/speckit.checklist` | `speckit-checklist` | 生成自定义质量检查表，验证需求完整性、清晰度和一致性，类似“针对自然语言的单元测试” |

<a id="cli-reference"></a>

## 🔧 Specify CLI 参考

完整命令说明、选项和示例请参阅 [CLI 参考](https://github.github.io/spec-kit/reference/overview.html)，也可以阅读仓库内的[中文 CLI 总览](./docs/reference/overview.zh-CN.md)。

<a id="extensions-presets"></a>

## 🧩 定制 Spec Kit：Extensions 与 Presets

Spec Kit 可以通过两个互补系统进行定制：**Extension** 和 **Preset**；此外，单次调整还可以使用项目本地覆盖。

| 优先级 | 组件类型 | 位置 |
| ---: | --- | --- |
| ⬆ 1 | 项目本地覆盖 | `.specify/templates/overrides/` |
| 2 | Preset：定制 Core 和 Extension | `.specify/presets/templates/` |
| 3 | Extension：增加新能力 | `.specify/extensions/templates/` |
| ⬇ 4 | Spec Kit Core：内置 SDD 命令和模板 | `.specify/templates/` |

- **模板**在**运行时**解析：Spec Kit 从上到下遍历优先级栈，使用第一个匹配项。
- 项目本地覆盖 `.specify/templates/overrides/` 允许只为单个项目进行一次性调整，无需创建完整 Preset。
- **Extension/Preset 命令**在**安装时**应用：运行 `specify extension add` 或 `specify preset add` 时，命令文件会写入 Agent 目录，例如 `.claude/commands/`。
- 如果多个 Preset 或 Extension 提供同名命令，优先级最高的版本生效。移除后，系统自动恢复下一个最高优先级版本。
- 如果不存在覆盖或定制，Spec Kit 使用 Core 默认值。

### Extensions：增加新能力

当需求超出 Spec Kit Core 时使用 **Extension**。Extension 引入新命令和模板，例如增加内置 SDD 命令没有覆盖的领域专用工作流、集成外部工具，或者加入全新的开发阶段。它扩展的是 Spec Kit **可以做什么**。

```bash
# 搜索可用 Extension
specify extension search

# 安装 Extension
specify extension add <extension-name>
```

例如，Extension 可以增加 Jira 集成、实现后代码审查、V-Model 测试可追踪性或项目健康诊断。

完整命令指南请参阅 [Extensions 参考](https://github.github.io/spec-kit/reference/extensions.html)，可用内容请浏览[社区 Extensions](https://github.github.io/spec-kit/community/extensions.html)。

### Presets：定制现有工作流

如果想改变 Spec Kit **如何工作**，而不是增加新能力，请使用 **Preset**。Preset 可以覆盖 Core 和已安装 Extension 自带的模板及命令，例如强制使用面向合规的规格格式、采用领域专用术语，或者把组织标准应用到计划和任务。它定制的是 Spec Kit 及其 Extension 生成的工件和指令。

```bash
# 搜索可用 Preset
specify preset search

# 安装 Preset
specify preset add <preset-name>
```

例如，Preset 可以重构规格模板以强制监管追踪，使工作流适配 Agile、Kanban、Waterfall、Jobs-to-be-Done 或领域驱动设计，为计划增加强制安全审查 Gate，强制测试优先任务顺序，或者把整个工作流本地化为其他语言。[pirate-speak Demo](https://github.com/mnriem/spec-kit-pirate-speak-preset-demo) 展示了定制可以深入到什么程度。多个 Preset 可以按优先级叠加。

完整命令指南、解析顺序和优先级叠加规则请参阅 [Presets 参考](https://github.github.io/spec-kit/reference/presets.html)。

<a id="bundles"></a>

## 📦 Bundles：面向角色的配置

Extension 和 Preset 是独立构建块。**Bundle** 把一组经过整理的 Extension、Preset、Step 和 Workflow 打包成一个带版本、面向角色的配置，使整个团队角色，例如产品经理、业务分析师、安全研究员或开发人员，可以用一条命令完成配置。

Bundle 由手写的 `bundle.yml` Manifest 描述。它为每个组件固定版本，并可以选择指定 Integration；不包含 `integration` 的 Bundle 是**无关 Integration 的**，会继承项目当前使用的 Integration。

```bash
# 在已激活 Catalog 栈中发现 Bundle
specify bundle search [<query>]

# 查看 Bundle 将添加的准确组件集合，与 install 行为一致
specify bundle info <bundle-id>

# 一次性安装 Bundle 的完整组件集合
specify bundle install <bundle-id>

# 查看已安装内容，然后进行非破坏性更新或移除
specify bundle list
specify bundle update <bundle-id>     # 或 --all
specify bundle remove <bundle-id>     # 只移除该 Bundle 的组件
```

Bundle 从**按优先级排序的 Catalog 栈**解析，顺序为项目、用户、内置。每个来源都有安装策略：`install-allowed` 来源允许安装；`discovery-only` 来源会显示在 `search` 和 `info` 中，但拒绝安装。使用 `specify bundle catalog list|add|remove` 管理该栈。

作者在本地验证和打包 Bundle。发布方式是托管构建产物并添加 Catalog 来源；社区 Bundle 使用 [Bundle Submission](https://github.com/github/spec-kit/issues/new?template=bundle_submission.yml) Issue 模板提交，以便检查所需组件 Catalog 和安装证据：

```bash
specify bundle validate --path ./my-bundle      # 结构和引用检查
specify bundle build --path ./my-bundle         # 生成带版本的 .zip 工件
```

[`examples/bundles/`](examples/bundles/) 中提供了四份可直接阅读的示例 Manifest：产品经理、业务分析师、安全研究员和开发人员。

核心保证包括：`info` 准确显示 `install` 将添加的内容，确保透明；安装具有幂等性并限制在项目根目录；`remove` 不会触碰其他已安装 Bundle 仍然需要的组件；所有使用和创作命令都可以针对本地或固定来源**离线**工作。

### 如何选择

| 目标 | 使用 |
| --- | --- |
| 增加全新命令或工作流 | Extension |
| 定制 spec、plan 或 tasks 格式 | Preset |
| 集成外部工具或服务 | Extension |
| 执行组织或监管标准 | Preset |
| 发布可复用的领域模板 | 两者都可以：覆盖模板使用 Preset，与新命令一起发布模板使用 Extension |
| 用一条命令配置完整的角色工具集 | Bundle |

<a id="core-philosophy"></a>

## 📚 核心理念

规格驱动开发是一种结构化过程，强调：

- **意图驱动开发**：规格先定义“做什么”，再考虑“怎么做”。
- **丰富的规格创建**：使用约束条件和组织原则提高规格质量。
- **多步骤细化**：不依赖提示词一次性生成代码，而是逐步完善。
- **高度依赖**高级 AI 模型解释规格的能力。

<a id="development-phases"></a>

## 🌟 开发阶段

| 阶段 | 重点 | 关键活动 |
| --- | --- | --- |
| **从 0 到 1 开发**（Greenfield） | 从零生成 | <ul><li>从高层需求开始</li><li>生成规格</li><li>规划实施步骤</li><li>构建可用于生产的应用</li></ul> |
| **创意探索** | 并行实现 | <ul><li>探索不同解决方案</li><li>支持多种技术栈和架构</li><li>试验 UX 模式</li></ul> |
| **迭代增强**（Brownfield） | 既有系统现代化 | <ul><li>迭代增加功能</li><li>现代化遗留系统</li><li>调整流程</li></ul> |

对于现有项目，应把 Spec Kit 工具更新与功能工件演进分开：升级时刷新受管理的项目文件，预期行为变化时更新 `specs/` 工件。[规格演进指南](./docs/guides/evolving-specs.md)介绍了推荐的既有项目迭代循环。

<a id="experimental-goals"></a>

## 🎯 实验目标

我们的研究和实验聚焦于以下方面。

### 技术无关性

- 使用不同技术栈创建应用。
- 验证规格驱动开发是一种不依赖特定技术、编程语言或框架的过程。

### 企业约束

- 展示关键任务应用开发。
- 纳入组织约束，例如云提供商、技术栈和工程实践。
- 支持企业设计系统和合规要求。

### 以用户为中心的开发

- 为不同用户群体和偏好构建应用。
- 支持不同开发方式，从 Vibe Coding 到 AI 原生开发。

### 创造性和迭代过程

- 验证并行探索实现方案的概念。
- 提供可靠的迭代式功能开发工作流。
- 扩展流程，使其能够处理升级和现代化任务。

<a id="prerequisites"></a>

## 🔧 前置条件

- **Linux/macOS/Windows**。
- [受支持](#supported-integrations)的 AI 编程 Agent。
- 用于包管理的 [uv](https://docs.astral.sh/uv/)（推荐），或用于持久安装的 [pipx](https://pipx.pypa.io/)。
- [Python 3.11+](https://www.python.org/downloads/)。
- [Git](https://git-scm.com/downloads)。

如果某个 Agent 出现问题，请提交 Issue，帮助我们改进 Integration。

<a id="learn-more"></a>

## 📖 深入学习

- **[完整规格驱动开发方法论](./spec-driven.md)**：深入理解完整过程。
- **[详细演练](#detailed-process)**：逐步实施指南。

---

<a id="detailed-process"></a>

## 📋 详细流程

<details>
<summary>点击展开详细分步演练</summary>

可以使用 Specify CLI 初始化项目，它会把所需工件安装到你的环境中：

```bash
specify init <project_name>
```

或者在当前目录初始化：

```bash
specify init .
# 或使用 --here
specify init --here
# 当前目录已经包含文件时跳过确认
specify init . --force
# 或者
specify init --here --force
```

![Specify CLI bootstrapping a new project in the terminal](./media/specify_cli.gif)

在交互式终端中，系统会提示你选择正在使用的编程 Agent Integration。在 CI 或管道运行等非交互式会话中，除非传入 `--integration`，否则 `specify init` 默认使用 GitHub Copilot。也可以直接在终端中指定 Integration：

```bash
specify init <project_name> --integration copilot
specify init <project_name> --integration gemini
specify init <project_name> --integration codex

# 或在当前目录中：
specify init . --integration copilot
specify init . --integration codex --integration-options="--skills"

# 或使用 --here
specify init --here --integration copilot
specify init --here --integration codex --integration-options="--skills"

# 强制合并到非空当前目录
specify init . --force --integration copilot

# 或者
specify init --here --force --integration copilot
```

如果所选 Integration 设置了 `requires_cli: True`，CLI 会检查机器上是否安装其所需的 CLI 工具。如果没有安装所需工具，或者只想获取模板而不检查工具，可以使用 `--ignore-agent-tools`：

```bash
specify init <project_name> --integration copilot --ignore-agent-tools
```

### **第 1 步：** 建立项目原则

进入项目目录并运行编程 Agent。下面的示例使用 `claude`。

![Bootstrapping Claude Code environment](./media/bootstrap-claude-code.gif)

如果可以看到 `/speckit.constitution`、`/speckit.specify`、`/speckit.plan`、`/speckit.tasks` 和 `/speckit.implement` 命令，就说明配置正确。

第一步应该使用 `/speckit.constitution` 建立项目治理原则。这有助于在后续所有开发阶段保持决策一致：

```text
/speckit.constitution Create principles focused on code quality, testing standards, user experience consistency, and performance requirements. Include governance for how these principles should guide technical decisions and implementation choices.
```

该步骤创建或更新 `.specify/memory/constitution.md`，其中包含项目基础指南，编程 Agent 会在规格、规划和实施阶段引用这些指南。

### **第 2 步：** 创建项目规格

建立项目原则后，可以开始创建功能规格。运行 `/speckit.specify`，然后提供要开发项目的具体需求。

> [!IMPORTANT]
> 尽可能明确地说明要构建**什么**以及**为什么**。此时**不要关注技术栈**。

示例提示词：

```text
Develop Taskify, a team productivity platform. It should allow users to create projects, add team members,
assign tasks, comment and move tasks between boards in Kanban style. In this initial phase for this feature,
let's call it "Create Taskify," let's have multiple users but the users will be declared ahead of time, predefined.
I want five users in two different categories, one product manager and four engineers. Let's create three
different sample projects. Let's have the standard Kanban columns for the status of each task, such as "To Do,"
"In Progress," "In Review," and "Done." There will be no login for this application as this is just the very
first testing thing to ensure that our basic features are set up. For each task in the UI for a task card,
you should be able to change the current status of the task between the different columns in the Kanban work board.
You should be able to leave an unlimited number of comments for a particular card. You should be able to, from that task
card, assign one of the valid users. When you first launch Taskify, it's going to give you a list of the five users to pick
from. There will be no password required. When you click on a user, you go into the main view, which displays the list of
projects. When you click on a project, you open the Kanban board for that project. You're going to see the columns.
You'll be able to drag and drop cards back and forth between different columns. You will see any cards that are
assigned to you, the currently logged in user, in a different color from all the other ones, so you can quickly
see yours. You can edit any comments that you make, but you can't edit comments that other people made. You can
delete any comments that you made, but you can't delete comments anybody else made.
```

输入提示词后，Claude Code 会启动规划和规格起草过程，并触发一些内置脚本来设置仓库。

该步骤完成后，应该会创建新分支，例如 `001-create-taskify`，并在 `specs/001-create-taskify` 目录中生成新规格。

生成的规格应按照模板包含一组用户故事和功能需求。

此时项目目录结构应类似：

```text
.
├── .specify
│   ├── memory
│   │   └── constitution.md
│   ├── scripts
│   │   └── bash
│   │       ├── check-prerequisites.sh
│   │       ├── common.sh
│   │       ├── create-new-feature.sh
│   │       ├── setup-plan.sh
│   │       └── setup-tasks.sh
│   └── templates
│       ├── plan-template.md
│       ├── spec-template.md
│       └── tasks-template.md
└── specs
    └── 001-create-taskify
        └── spec.md
```

### **第 3 步：** 澄清功能规格（规划前必须完成）

创建基础规格后，可以澄清首次生成时没有正确捕获的需求。

在创建技术计划**之前**运行结构化澄清工作流，以减少后续返工。

推荐顺序：

1. 使用 `/speckit.clarify` 进行结构化澄清。它按顺序、基于覆盖范围提问，并把答案记录到 Clarifications 章节。
2. 如果仍有模糊之处，可以继续进行临时的自由形式细化。

如果有意跳过澄清，例如进行 Spike 或探索性原型，请明确说明，避免 Agent 因缺少澄清而阻塞。

在 `/speckit.clarify` 后仍需要细化时，可以使用下面的自由形式提示词：

```text
For each sample project or project that you create there should be a variable number of tasks between 5 and 15
tasks for each one randomly distributed into different states of completion. Make sure that there's at least
one task in each stage of completion.
```

还应要求 Claude Code 验证 **Review & Acceptance Checklist**：通过验证或符合要求的项目打勾，不符合的项目保持未选中。可以使用：

```text
Read the review and acceptance checklist, and check off each item in the checklist if the feature spec meets the criteria. Leave it empty if it does not.
```

应把与 Claude Code 的交互当作澄清和追问规格的机会，**不要把第一次生成结果当作最终版本**。

### **第 4 步：** 生成计划

现在可以明确技术栈和其他技术要求。使用项目模板内置的 `/speckit.plan`，例如：

```text
We are going to generate this using .NET Aspire, using Postgres as the database. The frontend should use
Blazor server with drag-and-drop task boards, real-time updates. There should be a REST API created with a projects API,
tasks API, and a notifications API.
```

该步骤会生成若干实施细节文档，目录结构类似：

```text
.
├── CLAUDE.md
├── .specify
│   ├── memory
│   │   └── constitution.md
│   ├── scripts
│   │   └── bash
│   │       ├── check-prerequisites.sh
│   │       ├── common.sh
│   │       ├── create-new-feature.sh
│   │       ├── setup-plan.sh
│   │       └── setup-tasks.sh
│   └── templates
│       ├── CLAUDE-template.md
│       ├── plan-template.md
│       ├── spec-template.md
│       └── tasks-template.md
└── specs
    └── 001-create-taskify
        ├── contracts
        │   ├── api-spec.json
        │   └── signalr-spec.md
        ├── data-model.md
        ├── plan.md
        ├── quickstart.md
        ├── research.md
        └── spec.md
```

检查 `research.md`，确认根据指令使用了正确技术栈。如果某些组件有问题，可以要求 Claude Code 细化，也可以让它检查本地安装的平台或框架版本，例如 .NET。

如果所选技术栈变化很快，例如 .NET Aspire 或 JavaScript 框架，还可以要求 Claude Code 研究相关细节：

```text
I want you to go through the implementation plan and implementation details, looking for areas that could
benefit from additional research as .NET Aspire is a rapidly changing library. For those areas that you identify that
require further research, I want you to update the research document with additional details about the specific
versions that we are going to be using in this Taskify application and spawn parallel research tasks to clarify
any details using research from the web.
```

在此过程中，Claude Code 可能会研究错误的方向。可以用下面的提示词把它引导到正确范围：

```text
I think we need to break this down into a series of steps. First, identify a list of tasks
that you would need to do during implementation that you're not sure of or would benefit
from further research. Write down a list of those tasks. And then for each one of these tasks,
I want you to spin up a separate research task so that the net results is we are researching
all of those very specific tasks in parallel. What I saw you doing was it looks like you were
researching .NET Aspire in general and I don't think that's gonna do much for us in this case.
That's way too untargeted research. The research needs to help you solve a specific targeted question.
```

> [!NOTE]
> Claude Code 可能过于积极，加入你没有要求的组件。请要求它说明修改的理由和来源。

### **第 5 步：** 让 Claude Code 验证计划

计划完成后，应让 Claude Code 全面检查，确认没有遗漏。可以使用：

```text
Now I want you to go and audit the implementation plan and the implementation detail files.
Read through it with an eye on determining whether or not there is a sequence of tasks that you need
to be doing that are obvious from reading this. Because I don't know if there's enough here. For example,
when I look at the core implementation, it would be useful to reference the appropriate places in the implementation
details where it can find the information as it walks through each step in the core implementation or in the refinement.
```

这样可以细化实施计划，避免 Claude Code 在规划周期中遗漏盲点。完成第一轮细化后，再让 Claude Code 检查一次 Checklist，然后进入实施阶段。

如果已经安装 [GitHub CLI](https://docs.github.com/en/github-cli/github-cli)，还可以要求 Claude Code 从当前分支创建到 `main` 的 Pull Request，并提供详细说明，确保工作得到正确跟踪。

> [!NOTE]
> 在让 Agent 实施前，还应要求 Claude Code 交叉检查实施细节，确认是否存在过度工程设计。发现过度设计的组件或决策时，可以要求它解决。确保 Claude Code 遵循 `.specify/memory/constitution.md`，因为这是制定计划时必须遵守的基础原则。

### **第 6 步：** 使用 `/speckit.tasks` 生成任务拆分

实施计划验证完成后，可以把计划拆分为按正确顺序执行的具体任务。使用 `/speckit.tasks` 自动生成详细任务拆分：

```text
/speckit.tasks
```

该步骤在功能规格目录中创建 `tasks.md`，其中包含：

- **按用户故事组织的任务拆分**：每个用户故事成为独立实施阶段，并拥有自己的任务集合。
- **依赖管理**：任务按组件依赖排序，例如先 Model，再 Service，最后 Endpoint。
- **并行执行标记**：可以并行运行的任务标记为 `[P]`，用于优化开发流程。
- **文件路径说明**：每个任务包含实施应发生的准确文件路径。
- **测试驱动开发结构**：如果要求测试，测试任务会被加入并排在实现之前。
- **检查点验证**：每个用户故事阶段都包含检查点，用于验证独立功能。

生成的 `tasks.md` 为 `/speckit.implement` 提供清晰路线图，确保系统化实施、保持代码质量，并允许增量交付用户故事。

### **第 7 步：** 实施

准备完成后，使用 `/speckit.implement` 执行实施计划：

```text
/speckit.implement
```

`/speckit.implement` 会：

- 验证 Constitution、Spec、Plan 和 Tasks 等所有前置条件是否齐备。
- 解析 `tasks.md` 中的任务拆分。
- 按正确顺序执行任务，遵循依赖和并行执行标记。
- 遵循任务计划定义的 TDD 方法。
- 提供进度更新并正确处理错误。

> [!IMPORTANT]
> 编程 Agent 会执行本地 CLI 命令，例如 `dotnet`、`npm` 等，请确保机器上已安装所需工具。

实现完成后，测试应用并解决 CLI 日志中可能看不到的运行时错误，例如浏览器控制台错误。可以把这些错误复制给编程 Agent 进行修复。

</details>

---

<a id="support"></a>

## 💬 支持

如需支持，请提交 [GitHub Issue](https://github.com/github/spec-kit/issues/new)。欢迎报告 Bug、提出功能需求以及咨询规格驱动开发的使用问题。

<a id="acknowledgements"></a>

## 🙏 致谢

本项目深受 [John Lam](https://github.com/jflam) 的工作和研究影响，并以其成果为基础。

<a id="license"></a>

## 📄 许可证

本项目采用 MIT 开源许可证。完整条款请参阅 [LICENSE](./LICENSE)。
