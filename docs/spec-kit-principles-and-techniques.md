# Spec Kit 使用原理与进阶技巧

> 适用仓库：`D:\studyProject\spec-kit`
>
> 阅读目标：不只会运行 `specify init` 和 `/speckit.*` 命令，而是理解 Spec Kit 的设计边界、执行链路、扩展机制和调试方法。
>
> 配套文档：[深度学习计划](learning-plan.md) 负责安排“怎么学”；本文负责解释“它为什么这样设计、内部怎么运转、遇到问题怎么判断”。

## 1. 总体心智模型

Spec Kit 可以理解为四层东西叠在一起：

1. **Spec-Driven Development 方法论层**

   它定义开发过程的阶段：先建立项目原则，再写需求规格，再做技术计划，再拆任务，最后实现。它关注的是“把模糊想法变成可执行的软件交付过程”。

2. **Specify CLI 脚手架层**

   `specify` 是把方法论落地到项目目录里的工具。它负责初始化 `.specify/`，复制模板和脚本，安装 AI agent 命令文件，记录集成状态，管理 preset、extension、workflow。

3. **模板与资产层**

   `templates/`、`scripts/`、`presets/`、`extensions/`、`workflows/` 是 Spec Kit 的可复制资产。CLI 本身不直接“生成需求内容”，它更多是把正确的模板、脚本和命令装进目标项目，让 AI agent 在项目中执行这些流程。

4. **AI agent 集成层**

   不同 agent 的命令格式不同：有的是 Markdown 命令，有的是 TOML，有的是 YAML recipe，有的是 `SKILL.md` 目录。Spec Kit 用 integration registry 和若干 base class 把这些差异统一起来。

一个简化的图是：

```text
用户命令
  |
  v
specify CLI
  |
  +-- 初始化共享基础设施：.specify/templates、.specify/scripts
  |
  +-- 安装 integration：.agents/skills、.gemini/commands、.github/agents 等
  |
  +-- 写入 agent context file：AGENTS.md、GEMINI.md、CLAUDE.md 等
  |
  +-- 记录状态：.specify/integration.json、manifest、registry
  |
  v
AI agent 读取命令/技能文件
  |
  v
执行 SDD 流程：constitution -> specify -> clarify -> plan -> tasks -> implement
```

学习时最重要的一点是：**Spec Kit 不是一个普通的代码生成器，而是一个把开发流程安装到项目里的流程框架。**

## 2. Spec-Driven Development 的核心逻辑

Spec-Driven Development 的关键不是“写更多文档”，而是把文档变成可执行的开发约束。

传统开发里，规格文档经常是一次性材料：写完之后，真正实现时靠开发者和 AI 临场判断。Spec Kit 试图把规格、计划和任务保留在项目结构里，让后续命令都能围绕同一组工件继续工作。

典型工件包括：

| 工件 | 典型位置 | 作用 |
|---|---|---|
| Constitution | `.specify/memory/constitution.md` | 项目原则、技术约束、团队规则 |
| Specification | `specs/<feature>/spec.md` | 需求、用户场景、验收条件 |
| Plan | `specs/<feature>/plan.md` | 技术方案、架构取舍、实施策略 |
| Tasks | `specs/<feature>/tasks.md` | 可执行任务列表 |
| Templates | `.specify/templates/*.md` | 生成上述工件时使用的模板 |
| Scripts | `.specify/scripts/<sh|powershell>/*` | 查找 feature、检查前置条件、设置计划和任务 |

这些工件之间有顺序关系：

```text
constitution
  -> specify
  -> clarify
  -> plan
  -> tasks
  -> analyze
  -> implement
```

这个顺序不是形式主义。每一步都把不确定性向后压缩：

- `constitution` 固化长期原则，减少每个功能都重复争论基础规则。
- `specify` 把自然语言想法变成结构化需求，先回答“要什么”。
- `clarify` 专门处理需求不清楚的问题，避免过早进入实现。
- `plan` 从规格出发选择技术路线，回答“怎么做”。
- `tasks` 把计划拆成可执行任务，回答“按什么顺序做”。
- `analyze` 检查规格、计划、任务之间是否矛盾。
- `implement` 让 agent 按任务清单执行，而不是凭一次性提示自由发挥。

深入学习时，不要只看命令名。要问三个问题：

1. 这个命令读取哪些已有工件？
2. 它写入或更新哪些工件？
3. 它把哪些判断留给 AI，哪些判断交给脚本或模板约束？

## 3. `specify init` 的执行链路

`specify init` 是理解整个项目的最佳入口，因为它把多数核心模块串起来了。

源码入口在：

- `src/specify_cli/__init__.py`
- `src/specify_cli/shared_infra.py`
- `src/specify_cli/integrations/__init__.py`
- `src/specify_cli/integrations/base.py`
- `src/specify_cli/integrations/manifest.py`

### 3.1 参数解析与入口

CLI 使用 Typer 定义命令。`src/specify_cli/__init__.py` 中的 `app = typer.Typer(...)` 是总入口，`init()` 是初始化命令。

关键参数可以按职责分组：

| 参数 | 作用 |
|---|---|
| `project_name` / `--here` | 决定初始化到新目录还是当前目录 |
| `--integration <key>` | 使用新的 integration 系统选择 agent |
| `--integration-options` | 传给具体 integration 的额外参数 |
| `--script sh|ps` | 选择 bash 或 PowerShell 脚本 |
| `--preset <id>` | 初始化时安装 preset |
| `--force` | 在已有目录或刷新 managed 文件时允许覆盖 |
| `--ignore-agent-tools` | 跳过 CLI agent 工具检查 |

重要设计点：

- `--integration` 和旧的 `--ai` 是互斥的。
- 新代码以 integration registry 为准，不再把 agent 元数据散落在多个地方。
- CLI-based integration 的 `key` 应当匹配真实可执行文件名，因为工具检查和非交互 dispatch 都依赖这个 key。

### 3.2 目录决策

`specify init my-project` 和 `specify init --here` 的行为不同：

- `my-project`：创建或使用一个目标目录。
- `--here` 或 `.`：在当前目录合并安装。

如果当前目录非空，CLI 默认会提醒，因为初始化会写入 `.specify/`、agent 命令目录、context file 等文件。`--force` 用来跳过确认或覆盖部分 managed 文件。

理解这个逻辑有助于判断为什么同一个命令在空目录和已有项目里表现不同。

### 3.3 共享基础设施安装

共享基础设施由 `shared_infra.py` 负责，核心目标是安装：

```text
.specify/
  templates/
  scripts/
    bash/
    powershell/
  integrations/
```

`install_shared_infra()` 会从两个位置找资源：

1. wheel 或打包后的 `core_pack`
2. 源码仓库里的 `templates/` 和 `scripts/`

这就是为什么本地开发和安装后的 CLI 都能工作：开发态读源码目录，发布态读打包资产。

共享基础设施有几个重要安全设计：

- 目标路径必须在 project root 内。
- 不跟随会逃逸项目根目录的 symlink。
- 已存在文件默认不覆盖。
- 有 manifest 记录的 managed 文件可以判断是否被用户改过。
- `force=True` 才会更激进地覆盖普通文件。

### 3.4 模板里的命令引用会随 integration 改变

共享模板中可能出现：

```text
__SPECKIT_COMMAND_PLAN__
__SPECKIT_COMMAND_TASKS__
__SPECKIT_COMMAND_GIT_COMMIT__
```

这些占位符会通过 `IntegrationBase.resolve_command_refs()` 替换成实际调用形式。

Markdown 命令类 agent 通常得到：

```text
/speckit.plan
/speckit.tasks
/speckit.git.commit
```

Skills 类 agent 通常得到：

```text
/speckit-plan
/speckit-tasks
/speckit-git-commit
```

这解释了一个常见现象：同一个 Spec Kit 项目切换默认 integration 后，共享模板可能需要刷新，因为模板中的命令调用风格要跟默认 agent 对齐。

## 4. Integration 架构原理

Integration 是 Spec Kit 最重要的架构模块之一。它解决的问题是：**同一组 SDD 命令如何安装到不同 AI agent 的不同文件格式里。**

### 4.1 Registry 是 Python 集成元数据的单一事实源

核心文件：

```text
src/specify_cli/integrations/__init__.py
```

里面有：

```python
INTEGRATION_REGISTRY: dict[str, IntegrationBase] = {}
```

每个内置 integration 都是一个子包，例如：

```text
src/specify_cli/integrations/
  codex/
  gemini/
  goose/
  copilot/
  windsurf/
```

`_register_builtins()` 负责导入并注册这些类。这个函数要求 import 和 `_register()` 基本保持字母顺序，这降低冲突和 review 成本。

关键理解：

- registry 存的是已经实例化的 integration 对象。
- CLI 命令通过 `get_integration(key)` 找到具体 integration。
- `CommandRegistrar.AGENT_CONFIGS` 也是从 `INTEGRATION_REGISTRY` 派生的。
- 添加新 integration 时，如果忘记注册，CLI 和 registrar 都看不到它。

### 4.2 Integration 类的必填字段

每个 integration 类通常至少包含：

```python
class GeminiIntegration(TomlIntegration):
    key = "gemini"
    config = {
        "name": "Gemini CLI",
        "folder": ".gemini/",
        "commands_subdir": "commands",
        "install_url": "...",
        "requires_cli": True,
    }
    registrar_config = {
        "dir": ".gemini/commands",
        "format": "toml",
        "args": "{{args}}",
        "extension": ".toml",
    }
    context_file = "GEMINI.md"
```

字段含义：

| 字段 | 用途 |
|---|---|
| `key` | CLI 使用的集成标识。CLI 工具型 agent 应匹配真实命令名 |
| `config["folder"]` | agent 根目录 |
| `config["commands_subdir"]` | 命令或 skills 子目录 |
| `config["requires_cli"]` | 是否需要检查本地 CLI 工具 |
| `registrar_config["dir"]` | extension/preset 注册命令时使用的目标目录 |
| `registrar_config["format"]` | 输出格式：markdown、toml、yaml 等 |
| `registrar_config["args"]` | agent 接收用户参数的占位符 |
| `registrar_config["extension"]` | 命令文件扩展名，skills 用 `/SKILL.md` |
| `context_file` | 写入 Spec Kit managed section 的 agent 指令文件 |

`config` 偏向“agent 元数据和 init 安装位置”，`registrar_config` 偏向“后续 extension/preset 生成命令时怎么渲染”。

### 4.3 Base class 的选择

Integration 的基类决定了命令模板如何转换。

| 基类 | 适合场景 | 输出 |
|---|---|---|
| `MarkdownIntegration` | 大多数 Markdown 命令 agent | `speckit.<name>.md` |
| `TomlIntegration` | TOML prompt agent | `speckit.<name>.toml` |
| `YamlIntegration` | YAML recipe agent | `speckit.<name>.yaml` |
| `SkillsIntegration` | skills 目录型 agent | `speckit-<name>/SKILL.md` |
| `IntegrationBase` | 完全自定义行为 | 自己决定 |

选择基类时不要先想“这个 agent 是谁”，而要想“这个 agent 的命令发现机制是什么格式”。

例如：

- Codex 使用 `SkillsIntegration`，因为它安装到 `.agents/skills/speckit-*/SKILL.md`。
- Gemini 使用 `TomlIntegration`，因为它读取 TOML 命令。
- Goose 使用 `YamlIntegration`，因为它读取 YAML recipe。
- Copilot 用 `IntegrationBase`，因为它默认需要 `.agent.md`、伴随 `.prompt.md`，还要合并 `.vscode/settings.json`；同时又支持 `--skills` 模式。

### 4.4 模板处理流水线

核心方法：

```text
IntegrationBase.process_template()
```

它会处理命令模板中的通用占位符：

| 模板内容 | 处理结果 |
|---|---|
| frontmatter 的 `scripts.sh` 或 `scripts.ps` | 根据 `--script` 选出脚本命令 |
| `{SCRIPT}` | 替换成对应脚本命令 |
| `{ARGS}` / `$ARGUMENTS` | 替换成当前 agent 的参数占位符 |
| `__AGENT__` | 替换成 integration key |
| `__CONTEXT_FILE__` | 替换成当前 agent 的 context file |
| `scripts/` / `templates/` 等路径 | 改写到 `.specify/scripts/`、`.specify/templates/` |
| `__SPECKIT_COMMAND_*__` | 改写成 agent 对应的 slash command 调用 |

这条流水线解释了为什么新 integration 多数时候不需要重写 `setup()`：只要 agent 的输出格式是标准 Markdown、TOML、YAML 或 skills，基类就能处理绝大多数差异。

### 4.5 Context file 是 agent 长期记忆入口

`context_file` 通常是：

| Agent | context file |
|---|---|
| Codex | `AGENTS.md` |
| Claude | `CLAUDE.md` |
| Gemini | `GEMINI.md` |
| Copilot | `.github/copilot-instructions.md` |
| Cursor | `.cursor/rules/specify-rules.mdc` |

`IntegrationBase.upsert_context_section()` 会写入：

```text
<!-- SPECKIT START -->
...
<!-- SPECKIT END -->
```

这是一段 managed section。再次安装或升级时，Spec Kit 只替换标记之间的内容，尽量保留用户在同一文件里的其他内容。

卸载时，`remove_context_section()` 只删除完整、顺序正确的 managed section。如果标记损坏，它会保守地保留文件，避免误删用户内容。

学习技巧：

- 如果 agent 行为不符合预期，先看 context file 是否存在。
- 再看 managed section 是否是当前 integration 生成的。
- 再看命令文件中 `__CONTEXT_FILE__` 是否已正确替换。

### 4.6 Manifest 保护用户改动

Integration 安装时会记录每个 managed 文件的 hash：

```text
.specify/integrations/<key>.manifest.json
```

它的作用是支持安全卸载和升级：

- 文件没改过：可以删除或覆盖。
- 文件被用户改过：默认保留，提示用户。
- 用户指定 `--force`：才允许强制删除或覆盖。

这也是 Spec Kit 很多命令看起来“保守”的原因。它不是不知道怎么覆盖，而是刻意不默认覆盖用户可能手改过的 agent 指令文件。

### 4.7 `multi_install_safe` 的含义

有些 integration 可以安装在同一个项目里，有些不适合。

一个 integration 想声明 `multi_install_safe = True`，通常要满足：

- agent 目录独立。
- context file 不和其他 safe integration 冲突。
- 命令调用风格稳定。
- 安装 manifest 独立。
- 不需要动态用户目录，例如任意 `--commands-dir`。

多 integration 安装不是默认推荐的工作方式，它更多用于团队里不同人使用不同 agent 的场景。项目仍然只有一个默认 integration，`.specify/integration.json` 会记录它。

## 5. CommandRegistrar 的角色

Integration 负责“初始化某个 agent”。`CommandRegistrar` 负责“后续 extension/preset 要把命令注册到已有 agent 目录时怎么渲染”。

核心文件：

```text
src/specify_cli/agents.py
```

它的关键设计是：

```python
CommandRegistrar.AGENT_CONFIGS
```

这个配置不是手写维护，而是从 `INTEGRATION_REGISTRY` 派生。

这样做的好处：

- integration 元数据只维护一份。
- preset/extension 生成命令时复用同样的格式规则。
- 新增 integration 后，只要 `registrar_config` 写对，后续命令注册能力也自然接入。

### 5.1 register_commands 的核心步骤

`register_commands()` 大致做这些事：

1. 根据 agent 找到目标命令目录。
2. 读取 extension/preset 的源命令 Markdown。
3. 解析 YAML frontmatter。
4. 根据策略处理 `wrap`、脚本路径、frontmatter 特殊字段。
5. 替换 `$ARGUMENTS` 为 agent 自己的参数占位符。
6. 替换 `__SPECKIT_COMMAND_*__`。
7. 按 agent 格式渲染 Markdown、TOML、YAML 或 `SKILL.md`。
8. 写入目标目录。
9. 如果是 Copilot，还写 companion `.prompt.md`。

所以，调试 extension/preset 命令生成问题时，不要只看 integration 类，也要看 `CommandRegistrar`。

### 5.2 参数占位符是跨 agent 的核心差异

常见参数占位符：

| Agent 类型 | 参数占位符 |
|---|---|
| 大多数 Markdown agent | `$ARGUMENTS` |
| TOML/YAML agent | `{{args}}` |
| Forge | `{{parameters}}` |
| Skills agent | 通常仍从模板进入，再由 skills 语义处理 |

如果一个命令在某个 agent 中没有收到用户输入，优先检查：

1. integration 的 `registrar_config["args"]`。
2. `process_template()` 是否执行。
3. `CommandRegistrar._convert_argument_placeholder()` 是否覆盖到该路径。
4. 输出文件里是否还残留错误占位符。

## 6. 模板、脚本和 `.specify/` 的关系

Spec Kit 的项目输出不是只靠 Python 代码。它很大一部分行为来自模板和脚本。

### 6.1 源码仓库里的资产

源码中重要资产：

```text
templates/
  commands/
    specify.md
    plan.md
    tasks.md
    implement.md
  spec-template.md
  plan-template.md
  tasks-template.md
  constitution-template.md

scripts/
  bash/
  powershell/

presets/
extensions/
workflows/
```

这些文件被安装进目标项目后，通常变成：

```text
.specify/
  templates/
  scripts/
  presets/
  extensions/
  workflows/
```

### 6.2 命令模板和工件模板不同

要区分两类模板：

| 类型 | 源码位置 | 目标位置 | 用途 |
|---|---|---|---|
| 命令模板 | `templates/commands/*.md` | agent 命令目录或 skills 目录 | 告诉 AI agent 如何执行 `/speckit.*` |
| 工件模板 | `templates/*-template.md` | `.specify/templates/*.md` | 生成 spec、plan、tasks 等项目工件 |

很多初学者会混淆这两者。

例如：

- `/speckit.plan` 的命令文件告诉 agent “如何执行 plan 工作流”。
- `.specify/templates/plan-template.md` 是 plan 工件的目标结构。

命令模板是“操作说明”，工件模板是“输出格式”。

### 6.3 脚本负责确定项目状态

脚本通常负责 AI 不应该凭感觉判断的事情，例如：

- 当前 feature 是哪个。
- spec、plan、tasks 文件在哪里。
- 是否满足前置条件。
- 新 feature 应该编号为多少。
- 当前项目是否在 git 仓库里。

这就是 `{SCRIPT}` 占位符的意义。命令文件通过 `{SCRIPT}` 调用 `.specify/scripts/<variant>/...`，把确定性信息交给脚本。

深入学习时要经常把命令模板和脚本一起看：

```powershell
Get-Content templates\commands\plan.md
Get-Content scripts\powershell\setup-plan.ps1
```

否则容易误以为所有逻辑都在 Python 里。

## 7. Preset 原理与技巧

Preset 的定位是：**改变 Spec Kit 的工作方式，但不新增 Python 功能。**

它适合做：

- 团队方法论定制。
- 行业合规模板。
- 精简版工作流。
- 特定组织的 spec/plan/tasks 格式。
- 对核心命令进行 prepend、append、wrap 或 replace。

核心文件：

```text
src/specify_cli/presets.py
presets/README.md
presets/lean/preset.yml
presets/self-test/preset.yml
```

### 7.1 Preset manifest

Preset 由 `preset.yml` 描述。核心字段包括：

```yaml
schema_version: "1.0"
preset:
  id: lean
  name: Lean
  version: 1.0.0
  description: ...
requires:
  speckit_version: "..."
provides:
  templates:
    - type: command
      name: speckit.specify
      file: commands/speckit.specify.md
      strategy: replace
```

模板类型：

| type | 作用 |
|---|---|
| `template` | 替换或组合 `.specify/templates/*.md` |
| `command` | 替换或组合 agent 命令 |
| `script` | 替换或包装脚本 |

策略：

| strategy | 含义 |
|---|---|
| `replace` | 完全替换 |
| `prepend` | 在核心模板前添加内容 |
| `append` | 在核心模板后添加内容 |
| `wrap` | 用 `{CORE_TEMPLATE}` 包住核心模板 |

脚本只支持 `replace` 和 `wrap`，因为脚本的 prepend/append 语义很容易破坏可执行逻辑。

### 7.2 Preset 是“层”，不是一次性复制

Preset 有 registry：

```text
.specify/presets/.registry
```

它记录安装了哪些 preset、版本、来源、优先级、是否启用。

优先级规则通常是：**数字越小，优先级越高。**

这很重要，因为多个 preset 可以叠加。排查模板最终来自哪里时，要看：

1. 项目本地 override。
2. 已安装 preset，按 priority 排序。
3. extension 提供的命令。
4. core 模板。

具体优先级要以 resolver 实现为准，但理解“层叠解析”这个模型就能避免很多误判。

### 7.3 什么时候用 preset

使用 preset 的判断标准：

- 你想改变“默认工作方式”。
- 你要替换核心 spec/plan/tasks 模板。
- 你要让整个团队遵循同一套输出格式。
- 你不需要新增新的 slash command，只是调整已有流程。

不适合用 preset 的情况：

- 你要新增 `/speckit.xxx.yyy` 命令。
- 你要接入外部工具。
- 你要提供独立可启停的能力。
- 你要发布一个可以被别人单独安装的功能包。

这些更适合 extension。

## 8. Extension 原理与技巧

Extension 的定位是：**给 Spec Kit 增加新能力。**

它适合做：

- 新的 slash command。
- 外部工具接入。
- 质量检查。
- Git/Jira/Linear/GitHub Issues 等流程增强。
- 项目内可选能力。

核心文件：

```text
src/specify_cli/extensions.py
extensions/README.md
extensions/EXTENSION-DEVELOPMENT-GUIDE.md
extensions/template/extension.yml
extensions/git/extension.yml
```

### 8.1 Extension manifest

Extension 由 `extension.yml` 描述。核心结构：

```yaml
schema_version: "1.0"
extension:
  id: git
  name: Git
  version: 1.0.0
  description: ...
requires:
  speckit_version: "..."
provides:
  commands:
    - name: speckit.git.commit
      file: commands/speckit.git.commit.md
      aliases: []
hooks:
  ...
```

命令命名有硬规则：

```text
speckit.{extension-id}.{command}
```

例如：

```text
speckit.git.commit
speckit.git.feature
speckit.git.remote
```

这个规则的意义是避免 extension 和 core 命令互相覆盖。

### 8.2 Extension 和 core 命令的命名边界

core 命令包括：

```text
analyze
checklist
clarify
constitution
implement
plan
specify
tasks
taskstoissues
```

Extension ID 不能和这些 core namespace 冲突。比如 extension id 不应该叫 `plan`，否则它的 `speckit.plan.xxx` 会和 core `/speckit.plan` 的概念空间混在一起。

### 8.3 Extension 安装时做什么

安装 extension 时，通常会：

1. 验证 `extension.yml`。
2. 检查版本兼容。
3. 检查命令名和别名是否冲突。
4. 复制 extension 文件到 `.specify/extensions/<id>/`。
5. 通过 `CommandRegistrar` 把 extension 命令注册到当前 agent 的命令目录。
6. 如果当前 agent 使用 skills，还可能生成 extension skills。
7. 更新 `.specify/extensions/.registry`。

卸载时则反向清理已注册命令和 registry 记录。

### 8.4 Extension 的调试方法

如果 extension 命令没有出现，按这个顺序查：

1. `.specify/extensions/<id>/extension.yml` 是否存在。
2. `.specify/extensions/.registry` 是否记录了该 extension。
3. `extension.yml` 的 `provides.commands` 是否使用 `speckit.<id>.<cmd>`。
4. 当前 agent 命令目录是否存在。
5. 输出命令文件是否写入 agent 目录。
6. 如果是 skills agent，看 `speckit-<id>-<cmd>/SKILL.md` 是否生成。
7. 如果是 Copilot，看 `.github/prompts/*.prompt.md` 是否生成。

## 9. Workflow Engine 原理

Workflow 的定位是：**把多个 SDD 操作编排成可重复、可暂停、可恢复的自动化流程。**

核心文件：

```text
src/specify_cli/workflows/base.py
src/specify_cli/workflows/__init__.py
src/specify_cli/workflows/engine.py
src/specify_cli/workflows/expressions.py
src/specify_cli/workflows/steps/*/__init__.py
workflows/README.md
workflows/ARCHITECTURE.md
workflows/speckit/workflow.yml
```

### 9.1 WorkflowDefinition

Workflow YAML 会被解析成 `WorkflowDefinition`。

它关心：

| 字段 | 作用 |
|---|---|
| `workflow.id` | workflow 唯一 ID |
| `workflow.name` | 展示名称 |
| `workflow.version` | semver 版本 |
| `workflow.integration` | 默认 integration |
| `workflow.model` | 默认模型 |
| `workflow.options` | 默认选项 |
| `inputs` | 用户输入定义 |
| `steps` | 步骤列表 |

`validate_workflow()` 会检查 schema version、ID 格式、版本格式、input 类型、step ID 唯一性、step type 是否存在。

### 9.2 Step registry

Workflow step 和 integration 一样，也有 registry：

```text
STEP_REGISTRY: dict[str, StepBase]
```

内置 step 类型包括：

| type | 作用 |
|---|---|
| `command` | 调用 Spec Kit command |
| `prompt` | 向 agent 发送 prompt |
| `shell` | 执行 shell 命令 |
| `gate` | 人工确认或暂停点 |
| `if` | 条件分支 |
| `switch` | 多分支 |
| `while` | 条件循环 |
| `do-while` | 先执行后判断的循环 |
| `fan-out` | 对集合逐项执行 |
| `fan-in` | 聚合 fan-out 结果 |

每个 step 继承 `StepBase`，实现：

```python
execute(config: dict[str, Any], context: StepContext) -> StepResult
```

### 9.3 StepContext 和 StepResult

`StepContext` 是 step 执行时共享的上下文：

| 字段 | 含义 |
|---|---|
| `inputs` | workflow 输入 |
| `steps` | 已执行 step 的结果 |
| `item` | fan-out 当前项 |
| `fan_in` | fan-in 聚合结果 |
| `default_integration` | 默认 integration |
| `default_model` | 默认模型 |
| `project_root` | 项目根目录 |
| `run_id` | 当前运行 ID |

`StepResult` 是 step 的返回值：

| 字段 | 含义 |
|---|---|
| `status` | completed、failed、paused 等 |
| `output` | step 输出数据 |
| `next_steps` | 控制流产生的嵌套步骤 |
| `error` | 失败信息 |

理解这两个对象，就能读懂 workflow engine 的大部分逻辑。

### 9.4 状态持久化

Workflow run 会写入：

```text
.specify/workflows/runs/<run_id>/
  state.json
  inputs.json
  workflow.yml
  log.jsonl
```

这样做有两个目的：

1. 支持暂停和恢复。
2. 保留当时运行的 workflow 定义，即使原始 YAML 后来被移动或删除，也能 resume。

`gate` 或用户中断可能让 workflow 进入 `paused`。`resume()` 会读取 run 目录里的状态和 workflow 副本，继续执行。

需要注意一个实现边界：嵌套 step 暂停时，当前实现会重新执行父 step 的嵌套体，而不是从精确嵌套路径恢复。源码里把更精确的 nested resume 标为未来增强方向。

### 9.5 integration = auto 的含义

Workflow 输入里常见：

```yaml
integration:
  default: auto
```

`WorkflowEngine._resolve_default()` 会尝试从：

```text
.specify/integration.json
```

读取当前项目默认 integration。这样 workflow 不需要硬编码 `copilot`、`codex` 或 `gemini`，而是跟随项目实际初始化状态。

如果读取失败，则保留默认值或按 fallback 处理。

## 10. Catalog 机制

Preset、extension、workflow 都有 catalog 概念。

它解决的问题是：**如何发现和安装官方或社区提供的包。**

典型 catalog 文件：

```text
presets/catalog.json
presets/catalog.community.json
extensions/catalog.json
extensions/catalog.community.json
workflows/catalog.json
workflows/catalog.community.json
```

常见命令：

```powershell
specify preset catalog list
specify extension catalog list
specify workflow catalog list
```

Catalog 通常分两类：

| 类型 | 含义 |
|---|---|
| official | 官方维护、默认可安装 |
| community | 社区贡献，通常需要用户自行判断可信度 |

学习 catalog 时要重点看：

- catalog entry 结构。
- `install_allowed` 的含义。
- cache 机制。
- project-level 和 user-level catalog stack。
- 环境变量是否允许覆盖默认 catalog。

Catalog 是发现机制，不是信任机制。安装社区 workflow 或 extension 前应查看来源内容。

## 11. Integration、Preset、Extension、Workflow 的边界

这是深入使用 Spec Kit 时最重要的判断题。

| 需求 | 应该用 |
|---|---|
| 支持一个新的 AI agent | Integration |
| 改变 spec/plan/tasks 的组织方式 | Preset |
| 新增一个 `/speckit.xxx.yyy` 能力 | Extension |
| 把多个步骤自动串起来 | Workflow |
| 改一个核心 SDD 命令的提示词 | Preset 或 core template |
| 接入 Git、Issue、CI、质量检查 | Extension |
| 让团队统一使用一套模板 | Preset |
| 让团队一键执行一串流程 | Workflow |

一个实用判断法：

1. **目标是支持新工具吗？** 是 -> integration。
2. **目标是改变默认方法论吗？** 是 -> preset。
3. **目标是新增能力吗？** 是 -> extension。
4. **目标是自动编排多个能力吗？** 是 -> workflow。

不要用 extension 去改一堆核心模板，也不要用 preset 去模拟一个新命令。能工作不代表边界清晰。

## 12. 新增 Integration 的实战路径

新增 integration 时，按这个顺序做最稳。

### 12.1 先确定 agent 类型

先回答：

1. 它是不是 CLI-based？
2. 它的可执行文件名是什么？
3. 它读什么格式的命令？
4. 命令目录在哪里？
5. 是否需要 context file？
6. 参数占位符是什么？
7. slash command 调用风格是什么？

如果 `requires_cli=True`，`key` 应该匹配真实可执行文件名。例如 `cursor-agent` 不应该简写成 `cursor`。

### 12.2 建子包

路径规则：

```text
src/specify_cli/integrations/<package_dir>/__init__.py
```

如果 key 有 hyphen，目录用 underscore：

| key | package dir |
|---|---|
| `kiro-cli` | `kiro_cli` |
| `cursor-agent` | `cursor_agent` |

### 12.3 填 class

最小 Markdown integration 类似：

```python
from ..base import MarkdownIntegration


class ExampleIntegration(MarkdownIntegration):
    key = "example"
    config = {
        "name": "Example",
        "folder": ".example/",
        "commands_subdir": "commands",
        "install_url": None,
        "requires_cli": False,
    }
    registrar_config = {
        "dir": ".example/commands",
        "format": "markdown",
        "args": "$ARGUMENTS",
        "extension": ".md",
    }
    context_file = "EXAMPLE.md"
```

### 12.4 注册

修改：

```text
src/specify_cli/integrations/__init__.py
```

在 `_register_builtins()` 中加 import 和 `_register()`，保持字母顺序。

### 12.5 写测试

测试文件：

```text
tests/integrations/test_integration_<key_with_underscores>.py
```

常跑：

```powershell
pytest tests/integrations/test_registry.py -v
pytest tests/integrations/test_integration_<name>.py -v
pytest tests/test_agent_config_consistency.py -v
```

测试重点：

- registry 能看到它。
- 输出目录正确。
- 命令文件格式正确。
- context file 写入正确。
- uninstall/manifest 行为正确。
- 如果是 skills，`SKILL.md` 结构正确。
- 如果是 CLI-based，`requires_cli` 和 key 逻辑正确。

## 13. 常见调试场景

### 13.1 `specify integration list` 看不到新 integration

检查：

1. 子包路径是否正确。
2. class 名是否能 import。
3. `_register_builtins()` 是否 import。
4. `_register_builtins()` 是否 `_register()`。
5. `key` 是否为空或重复。
6. 测试是否加载了源码版而不是旧安装版。

建议命令：

```powershell
python -m src.specify_cli integration list
pytest tests/integrations/test_registry.py -v
```

### 13.2 命令文件生成了，但 agent 调不起来

检查：

1. 文件是否在 agent 期望的目录。
2. 文件扩展名是否正确。
3. agent 是否要求额外 frontmatter。
4. slash command 名是否和文件名匹配。
5. 参数占位符是否是 agent 支持的格式。
6. context file 是否告诉 agent 使用 Spec Kit。

### 13.3 `{SCRIPT}` 没有替换

检查：

1. 命令模板 frontmatter 是否包含 `scripts:`。
2. `--script` 选择的是 `sh` 还是 `ps`。
3. 对应 `scripts.sh` 或 `scripts.ps` 是否存在。
4. integration 是否调用了 `process_template()`。
5. 自定义 `setup()` 是否绕过了基类处理。

### 13.4 `$ARGUMENTS` 没有替换

检查：

1. `registrar_config["args"]` 是否正确。
2. `process_template()` 或 `CommandRegistrar` 是否走到。
3. 输出格式是不是 TOML/YAML，需要 `{{args}}`。
4. 特殊 agent 是否有自定义占位符，例如 Forge。

### 13.5 卸载后文件没被删

这通常不是 bug。原因可能是：

- 文件 hash 和 manifest 不一致，说明用户改过。
- context section 标记不完整，出于安全保留。
- 文件不在 manifest 里，Spec Kit 不认为它是 managed 文件。
- 需要 `--force` 才删除。

先看：

```text
.specify/integrations/<key>.manifest.json
```

### 13.6 切换 integration 后模板命令风格不对

检查：

1. `.specify/integration.json` 的默认 integration。
2. 共享模板是否刷新。
3. 是否因为用户改过模板被保留。
4. 是否需要 `specify integration use <key> --force` 或 `upgrade --force`。

核心原因通常是：

- Markdown agent 用 `/speckit.plan`。
- Skills agent 用 `/speckit-plan`。

共享模板必须跟默认 integration 的调用风格一致。

### 13.7 Workflow 没有用当前项目的 agent

检查：

1. workflow YAML 是否写了固定 `workflow.integration`。
2. input 是否使用 `default: auto`。
3. `.specify/integration.json` 是否存在且可解析。
4. `default_integration` 是否被 step-level `integration` 覆盖。

## 14. 读源码的推荐顺序

如果目标是理解原理，而不是马上改 bug，建议按这个顺序：

1. `docs/concepts/sdd.md`
2. `docs/reference/core.md`
3. `src/specify_cli/__init__.py`
4. `src/specify_cli/shared_infra.py`
5. `src/specify_cli/integrations/__init__.py`
6. `src/specify_cli/integrations/base.py`
7. `src/specify_cli/integrations/codex/__init__.py`
8. `src/specify_cli/integrations/gemini/__init__.py`
9. `src/specify_cli/integrations/goose/__init__.py`
10. `src/specify_cli/integrations/copilot/__init__.py`
11. `src/specify_cli/agents.py`
12. `src/specify_cli/presets.py`
13. `src/specify_cli/extensions.py`
14. `src/specify_cli/workflows/base.py`
15. `src/specify_cli/workflows/engine.py`
16. `src/specify_cli/workflows/steps/*/__init__.py`
17. 对应测试文件

读每个模块时，用这张表做笔记：

| 问题 | 记录 |
|---|---|
| 这个模块负责什么？ | |
| 它读哪些文件？ | |
| 它写哪些文件？ | |
| 它依赖哪些 registry？ | |
| 它如何保护用户改动？ | |
| 它有哪些路径安全检查？ | |
| 它对应哪些测试？ | |

## 15. 修改不同模块时应该跑哪些测试

| 修改范围 | 建议测试 |
|---|---|
| CLI 参数或命令入口 | `pytest tests/test_cli_version.py tests/test_check_tool.py -v`，以及相关命令的专项测试 |
| integration registry | `pytest tests/integrations/test_registry.py -v` |
| integration base class | `pytest tests/integrations/test_integration_base_markdown.py tests/integrations/test_integration_base_toml.py tests/integrations/test_integration_base_yaml.py tests/integrations/test_integration_base_skills.py -v` |
| 某个 integration | `pytest tests/integrations/test_integration_<name>.py -v` |
| agent config 派生 | `pytest tests/test_agent_config_consistency.py -v` |
| manifest 安全 | `pytest tests/integrations/test_manifest.py -v` |
| preset | `pytest tests/test_presets.py -v` |
| extension | `pytest tests/test_extensions.py tests/test_extension_registration.py -v` |
| git extension | `pytest tests/extensions/git/test_git_extension.py -v` |
| workflow | `pytest tests/test_workflows.py -v` |
| 脚本行为 | `pytest tests/test_setup_plan_feature_json.py tests/test_setup_tasks.py tests/test_branch_numbering.py -v` |

如果不确定，先跑目标模块测试，再跑：

```powershell
pytest tests/integrations -v
pytest tests/test_agent_config_consistency.py -v
pytest tests/test_extensions.py -v
pytest tests/test_workflows.py -v
```

## 16. 深入使用技巧

### 16.1 把 Spec Kit 当成“可审查流程”

不要只让 AI 跑 `/speckit.implement`。高质量使用方式是：

1. 先检查 `spec.md` 是否有明确验收条件。
2. 再检查 `plan.md` 是否解释了关键取舍。
3. 再检查 `tasks.md` 是否能独立执行。
4. 最后才执行 implement。

如果前面工件质量不高，后面的实现会把模糊性变成随机性。

### 16.2 让 constitution 写具体约束

`constitution.md` 不应该只写“代码要高质量”。它应该写可执行约束：

- 使用哪种测试策略。
- 是否允许引入新依赖。
- 数据迁移如何处理。
- UI 变更是否需要截图验证。
- 安全和隐私边界。
- 提交前必须跑哪些命令。

AI agent 更容易遵守具体规则，而不是抽象口号。

### 16.3 用 preset 固化团队风格

如果你发现自己每个项目都要改同样的 `spec-template.md` 或 `plan-template.md`，不要长期手改 core 模板。更好的方式是做一个 preset。

Preset 的价值是可复用和可版本化：

- 新项目一条命令安装。
- 团队内模板一致。
- 后续可以升级。
- 不污染 upstream core。

### 16.4 用 extension 扩展项目能力

如果你想加“把 tasks 转成某个平台 issue”“检查 spec 是否满足公司规范”“生成发布说明”这类能力，用 extension。

Extension 的价值是可启停：

- 项目需要时安装。
- 不需要时卸载。
- 可以独立发布。
- 不和 core 命令混在一起。

### 16.5 用 workflow 固化重复流程

如果一组命令你会反复按同样顺序执行，就考虑 workflow。

例如：

```text
specify -> clarify -> plan -> gate -> tasks -> analyze
```

Workflow 的价值是可重复：

- 同一个流程每次执行一致。
- 可以加入 gate。
- 可以保存运行状态。
- 可以让团队共享一套流程。

## 17. 常见误区

### 误区 1：把 Spec Kit 当成普通代码生成器

Spec Kit 的重点不是一次性生成代码，而是维护从规格到实现的可追踪链路。只用最后一步 implement，会损失大部分价值。

### 误区 2：新增 integration 时随便取 key

CLI-based integration 的 key 要匹配可执行文件名。否则 `shutil.which(key)`、工具检查、dispatch 都可能出问题。

### 误区 3：把所有定制都做成 extension

如果只是改模板和流程风格，preset 更合适。Extension 应该用于新增能力。

### 误区 4：手改 generated command 文件后期待 upgrade 覆盖

Spec Kit 会通过 manifest 发现你改过文件，然后默认保留。需要明确使用 `--force` 才覆盖。

### 误区 5：只看 Python，不看模板和脚本

很多行为在 `templates/commands/*.md` 和 `scripts/<variant>/*` 里。只看 Python 会误解执行链路。

### 误区 6：以为所有 agent 都用 `/speckit.plan`

Skills agent 往往用 `/speckit-plan`。命令引用风格由 integration 的 `invoke_separator` 决定。

### 误区 7：忽略 context file

Agent 命令文件告诉 agent “执行某个命令时做什么”，context file 告诉 agent “这个项目长期遵循什么规则”。两者缺一不可。

## 18. 一张总览表

| 模块 | 关键文件 | 主要状态文件 | 核心问题 |
|---|---|---|---|
| CLI | `src/specify_cli/__init__.py` | `.specify/init-options.json` | 用户命令如何映射到模块 |
| Shared infra | `src/specify_cli/shared_infra.py` | `.specify/integrations/speckit.manifest.json` | 模板和脚本如何安全安装 |
| Integration | `src/specify_cli/integrations/*` | `.specify/integration.json`、`.specify/integrations/<key>.manifest.json` | agent 差异如何统一 |
| Registrar | `src/specify_cli/agents.py` | agent 命令目录 | extension/preset 命令如何渲染 |
| Preset | `src/specify_cli/presets.py` | `.specify/presets/.registry` | 工作方式如何层叠定制 |
| Extension | `src/specify_cli/extensions.py` | `.specify/extensions/.registry` | 新能力如何安装和卸载 |
| Workflow | `src/specify_cli/workflows/*` | `.specify/workflows/runs/<run_id>/` | 多步骤流程如何执行和恢复 |

## 19. 推荐练习

### 练习 1：追踪一次 init

运行：

```powershell
python -m src.specify_cli init ..\spec-kit-demo-principles --integration codex --ignore-agent-tools --script ps
```

然后检查：

```powershell
Get-ChildItem ..\spec-kit-demo-principles\.specify -Recurse
Get-ChildItem ..\spec-kit-demo-principles\.agents\skills -Recurse
Get-Content ..\spec-kit-demo-principles\AGENTS.md
Get-Content ..\spec-kit-demo-principles\.specify\integration.json
```

目标：

- 说清楚每个文件从哪里来。
- 说清楚哪些是共享基础设施，哪些是 Codex integration 生成的。
- 找出 `speckit-plan/SKILL.md` 中的脚本调用。

### 练习 2：对比两个 integration

分别初始化：

```powershell
python -m src.specify_cli init ..\spec-kit-demo-codex --integration codex --ignore-agent-tools --script ps
python -m src.specify_cli init ..\spec-kit-demo-gemini --integration gemini --ignore-agent-tools --script ps
```

对比：

```powershell
Get-ChildItem ..\spec-kit-demo-codex -Recurse | Select-String "speckit"
Get-ChildItem ..\spec-kit-demo-gemini -Recurse | Select-String "speckit"
```

目标：

- 解释为什么 Codex 是 `SKILL.md`。
- 解释为什么 Gemini 是 `.toml`。
- 解释参数占位符差异。

### 练习 3：安装 preset 并观察覆盖关系

运行：

```powershell
python -m src.specify_cli init ..\spec-kit-demo-lean --integration codex --preset lean --ignore-agent-tools --script ps
```

检查：

```powershell
Get-Content ..\spec-kit-demo-lean\.specify\presets\.registry
Get-ChildItem ..\spec-kit-demo-lean\.specify\templates
Get-ChildItem ..\spec-kit-demo-lean\.agents\skills -Recurse
```

目标：

- 找出哪些文件来自 core，哪些来自 preset。
- 解释 preset priority 的意义。

### 练习 4：安装 extension 并观察命令注册

在 demo 项目中运行：

```powershell
cd ..\spec-kit-demo-principles
python -m D:\studyProject\spec-kit\src.specify_cli extension add git
```

检查：

```powershell
Get-Content .specify\extensions\.registry
Get-ChildItem .specify\extensions\git -Recurse
Get-ChildItem .agents\skills -Recurse | Select-String "speckit-git"
```

目标：

- 解释 extension 文件和 agent 命令文件的区别。
- 解释为什么 extension 命令名要带 `speckit.git.*`。

### 练习 5：读 workflow run 状态

运行一个 workflow 后检查：

```powershell
Get-ChildItem .specify\workflows\runs -Recurse
```

目标：

- 找到 `state.json`、`inputs.json`、`workflow.yml`、`log.jsonl`。
- 解释 resume 为什么不依赖原始 workflow 文件。

## 20. 最后应该形成的能力

学完本文后，你应该能做到：

- 看 `specify init` 输出目录，就知道每类文件由哪个模块创建。
- 看到一个 integration 类，就能判断它会生成什么格式的命令文件。
- 遇到 agent 命令不可用，能按目录、格式、占位符、context file、manifest 逐项排查。
- 判断需求该用 integration、preset、extension 还是 workflow。
- 新增一个简单 Markdown integration，并写对应测试。
- 修改 preset 或 extension 时知道状态文件和 registry 在哪里。
- 读 workflow YAML 时能预判运行状态会怎么保存。
- 修改代码前能选出最小但有效的测试集合。

真正掌握 Spec Kit 的标志不是背出所有命令，而是能回答：

> 这个行为是 CLI 逻辑、模板内容、脚本逻辑、integration 渲染、preset 层叠、extension 注册，还是 workflow 编排造成的？

能把问题放回正确层次，调试和贡献就会快很多。
