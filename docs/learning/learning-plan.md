# Spec Kit 深度学习计划

> 适用仓库：`/home/xieyx/projects/spec-kit`
>
> 目标：从“会使用 Specify CLI”逐步推进到“能读懂核心实现、能添加集成、能修复问题并提交贡献”。

> 配套深入阅读：[Spec Kit 使用原理与进阶技巧](spec-kit-principles-and-techniques.md) 负责解释架构心智模型、执行链路、模块边界和调试方法。建议在执行本计划的第 1 周后开始同步阅读。
>
> 本目录入口：[Spec Kit 学习文档](README.md)。

## 学习路线总览

这个项目可以按 7 条主线理解：

1. **Spec-Driven Development 方法论**：理解 Spec Kit 为什么要把需求、计划、任务、实现拆成阶段。
2. **Specify CLI**：理解 `specify init`、`integration`、`preset`、`extension`、`workflow`、`bundle` 等命令如何工作。
3. **Integration 架构**：理解不同 AI agent 如何通过统一 registry 和 base class 接入。
4. **模板、脚本和资产打包**：理解 `.specify/`、templates、scripts、core pack、wheel 打包之间的关系。
5. **Preset / Extension / Bundle 生态**：理解工作方式定制、新能力扩展和角色化组合安装的边界。
6. **Workflow Engine**：理解自动化、多步骤、可暂停恢复的 SDD 执行模型。
7. **测试和贡献流程**：能跑测试、定位问题、添加新集成或修复 bug。

建议节奏：每天 60-90 分钟，持续 7 周。时间紧的话可以压缩成 4 周，每天完成两个小节。

## 第 0 周：环境和基本体验

**目标**：先跑起来，不急着读源码。

阅读：

- `README.md`
- `docs/installation.md`
- `docs/quickstart.md`
- `docs/local-development.md`
- `docs/concepts/sdd.md`

动手：

- 创建虚拟环境并安装开发依赖：

```bash
cd /home/xieyx/projects/spec-kit
uv sync --extra test
```

- 查看 CLI：

```bash
uv run specify --help
uv run specify init --help
uv run specify integration --help
uv run specify preset --help
uv run specify extension --help
uv run specify workflow --help
uv run specify bundle --help
```

- 初始化一个临时项目：

```bash
uv run specify init docs/learning/demos/spec-kit-demo --integration codex --ignore-agent-tools --script sh
```

验收标准：

- 能解释 `specify init` 会生成哪些目录。
- 能说明 `.specify/`、agent command files 的用途，并知道 context file 由 opt-in 的 `agent-context` extension 管理。
- 能成功跑通一个本地 `init` 示例。
- 本周产出物：一份 demo 项目目录树笔记，标出每类文件来自 core、integration 还是用户输入。

## 第 1 周：理解 SDD 工作流和用户视角

**目标**：从用户角度理解 Spec Kit 提供什么价值。

重点阅读：

- `README.md`
- `docs/reference/overview.md`
- `docs/reference/core.md`
- `docs/reference/workflows.md`
- `workflows/README.md`

重点理解：

- `/speckit.constitution`
- `/speckit.specify`
- `/speckit.clarify`
- `/speckit.plan`
- `/speckit.tasks`
- `/speckit.analyze`
- `/speckit.implement`

动手任务：

- 在临时项目中查看生成的命令文件。
- 找到这些命令对应的模板来源。
- 对比不同 integration 生成的命令格式差异，例如 `codex`、`gemini`、`goose`。

建议命令：

```bash
rg "speckit" templates src tests docs
rg "constitution|specify|plan|tasks|implement" templates src tests
```

验收标准：

- 能画出一条完整 SDD 流程：constitution -> specify -> plan -> tasks -> implement。
- 能说明 Spec Kit 中“模板”和“CLI 逻辑”的边界。
- 能解释为什么不同 AI agent 需要不同 command format。

## 第 2 周：Specify CLI 主入口

**目标**：读懂 CLI 命令如何组织。

重点文件：

- `src/specify_cli/__init__.py`
- `src/specify_cli/commands/init.py`
- `src/specify_cli/integrations/_commands.py`
- `src/specify_cli/presets/_commands.py`
- `src/specify_cli/extensions/_commands.py`
- `src/specify_cli/workflows/_commands.py`
- `src/specify_cli/commands/bundle/__init__.py`
- `src/specify_cli/_console.py`
- `src/specify_cli/_utils.py`
- `src/specify_cli/_assets.py`
- `src/specify_cli/_version.py`

阅读顺序：

1. 从 `src/specify_cli/__init__.py` 的 `app = typer.Typer(...)` 开始看根命令。
2. 找 `register(app)` 和 `app.add_typer(...)`，列出顶层命令组。
3. 重点读 `src/specify_cli/commands/init.py`，理解项目初始化流程。
4. 再读 integration、preset、extension、workflow、bundle 的命令模块。

动手任务：

- 给自己写一份命令映射表：

| CLI 命令 | 入口模块 | 主要职责 | 依赖模块 |
|---|---|---|---|
| `specify init` | `commands/init.py` | 初始化项目 | integrations, shared_infra, assets |
| `specify integration *` | `integrations/_commands.py` 等 | 管理 AI agent 集成 | integrations, integration_state |
| `specify preset *` | `presets/_commands.py` | 安装和管理 preset | presets, catalogs |
| `specify extension *` | `extensions/_commands.py` | 安装和管理 extension | extensions, catalogs, agents |
| `specify workflow *` | `workflows/_commands.py` | 运行和管理 workflow | workflows |
| `specify bundle *` | `commands/bundle/__init__.py` | 安装和管理组件组合包 | bundler, presets, extensions, workflows |

建议调试：

```bash
uv run specify --version
uv run specify check
uv run specify integration list
uv run specify bundle search --help
```

验收标准：

- 能说明 `Typer` 在这个项目里如何组织命令。
- 能从 CLI 参数追踪到对应的核心逻辑。
- 能解释 `--integration`、`--script`、`--ignore-agent-tools` 的作用。
- 本周产出物：一张命令映射表，至少覆盖 `init`、`integration`、`preset`、`extension`、`workflow`、`bundle`。

## 第 3 周：Integration 架构

**目标**：掌握这个项目最重要、最容易贡献的模块。

重点文件：

- `src/specify_cli/integrations/__init__.py`
- `src/specify_cli/integrations/base.py`
- `src/specify_cli/integrations/manifest.py`
- `src/specify_cli/integration_runtime.py`
- `src/specify_cli/integration_state.py`
- `src/specify_cli/integrations/codex/__init__.py`
- `src/specify_cli/integrations/copilot/__init__.py`
- `src/specify_cli/integrations/gemini/__init__.py`
- `src/specify_cli/integrations/goose/__init__.py`

核心概念：

- `INTEGRATION_REGISTRY`
- `_register_builtins()`
- `IntegrationBase`
- `MarkdownIntegration`
- `TomlIntegration`
- `YamlIntegration`
- `SkillsIntegration`
- `IntegrationManifest`
- `registrar_config`

动手任务：

- 选择 4 个 integration 做对比：

| Integration | Base class | 输出格式 | 参数占位符 | 特殊逻辑 |
|---|---|---|---|---|
| Codex | `SkillsIntegration` | `SKILL.md` | `$ARGUMENTS` | skills mode |
| Gemini | `TomlIntegration` | `.toml` | `{{args}}` | TOML prompt |
| Goose | `YamlIntegration` | `.yaml` | `{{args}}` | recipe format |
| Copilot | `IntegrationBase` | `.agent.md` + `.prompt.md` | `$ARGUMENTS` | settings merge；可选 skills mode |

- 跑集成测试：

```bash
uv run python -m pytest tests/integrations/test_registry.py -v
uv run python -m pytest tests/integrations/test_integration_base_markdown.py -v
uv run python -m pytest tests/integrations/test_integration_codex.py -v
uv run python -m pytest tests/integrations/test_integration_copilot.py -v
```

验收标准：

- 能新增一个简单 Markdown integration。
- 能解释为什么 CLI-based integration 的 `key` 要匹配可执行文件名。
- 能解释 manifest 如何支持 uninstall。
- 本周产出物：一张 4 个 integration 的对比表，记录 base class、输出目录、文件格式、参数占位符和特殊 setup 逻辑。

## 第 4 周：Presets、Extensions 和 Template Resolution

**目标**：理解 Spec Kit 如何被定制和扩展。

重点文件：

- `src/specify_cli/presets/__init__.py`
- `src/specify_cli/presets/_commands.py`
- `src/specify_cli/extensions/__init__.py`
- `src/specify_cli/extensions/_commands.py`
- `src/specify_cli/catalogs.py`
- `src/specify_cli/shared_infra.py`
- `presets/README.md`
- `presets/lean/README.md`
- `extensions/README.md`
- `extensions/git/README.md`
- `docs/reference/presets.md`
- `docs/reference/extensions.md`

核心问题：

- preset 和 extension 的区别是什么？
- project-local override、preset、extension、core template 的优先级是什么？
- 安装 extension 时，哪些文件被写入项目？
- 移除 extension 后，如何恢复下一个优先级的命令？

动手任务：

- 在临时项目中安装 bundled preset：

```bash
uv run specify init docs/learning/demos/spec-kit-preset-demo --integration codex --preset lean --ignore-agent-tools --script sh
```

- 在临时项目中安装 bundled extension：

```bash
(
  cd docs/learning/demos/spec-kit-preset-demo
  /home/xieyx/projects/spec-kit/.venv/bin/specify extension add git
)
```

- 观察 `.specify/` 和 agent 命令目录变化。

验收标准：

- 能解释“模板运行时解析”和“命令安装时写入”的区别。
- 能说明 preset 更适合改“工作方式”，extension 更适合加“新能力”。
- 能定位某个命令文件最终来自 core、preset 还是 extension。
- 本周产出物：一张 resolution stack 图，标出 project-local、preset、extension、core 的优先级。

## 第 5 周：Bundles 和组件组合

**目标**：理解 bundle 如何把 extensions、presets、workflows 和 steps 组合成角色或团队可复用的安装单元。

重点文件：

- `src/specify_cli/commands/bundle/__init__.py`
- `src/specify_cli/bundler/models/manifest.py`
- `src/specify_cli/bundler/models/catalog.py`
- `src/specify_cli/bundler/services/resolver.py`
- `src/specify_cli/bundler/services/installer.py`
- `src/specify_cli/bundler/services/packager.py`
- `docs/reference/bundles.md`
- `docs/community/bundles.md`
- `examples/bundles/*/bundle.yml`

核心问题：

- bundle 和 preset / extension / workflow 的边界是什么？
- `bundle.yml` 如何声明组件、版本、优先级和目标 integration？
- 安装 bundle 时，哪些逻辑委托给 primitive installer，哪些逻辑由 bundler 自己负责？
- 为什么 bundle 是分发和组合层，而不是新的运行时能力？

动手任务：

```bash
uv run specify bundle search --help
uv run specify bundle validate --path examples/bundles/developer
uv run specify bundle build --path examples/bundles/developer --output docs/learning/demos/spec-kit-bundle-demo
```

验收标准：

- 能解释 bundle 安装和单独安装 preset / extension 的区别。
- 能读懂一个 `bundle.yml`，说清楚它会安装哪些组件。
- 能说明 bundle 的 catalog、manifest、resolver、installer 各自负责什么。
- 本周产出物：一张 bundle install 数据流图，从 catalog/local path 到 primitive install。

## 第 6 周：Workflow Engine

**目标**：理解自动化工作流的执行模型。

重点文件：

- `src/specify_cli/workflows/base.py`
- `src/specify_cli/workflows/catalog.py`
- `src/specify_cli/workflows/engine.py`
- `src/specify_cli/workflows/expressions.py`
- `src/specify_cli/workflows/steps/*/__init__.py`
- `docs/reference/workflows.md`
- `workflows/README.md`

重点 step：

- `command`
- `prompt`
- `shell`
- `if_then`
- `switch`
- `while_loop`
- `do_while`
- `fan_out`
- `fan_in`
- `gate`

动手任务：

- 阅读 bundled workflow：

```bash
rg --files workflows
rg "type:|steps:|command|prompt|shell|gate" workflows src/specify_cli/workflows
```

- 跑 workflow 相关测试：

```bash
uv run python -m pytest tests/test_workflows.py -v
```

验收标准：

- 能说明 workflow 如何暂停、恢复和保存状态。
- 能解释表达式系统负责什么。
- 能新增或修改一个简单 workflow step 的测试。
- 本周产出物：一份 workflow run 状态目录说明，解释 `state.json`、`inputs.json`、`workflow.yml`、`log.jsonl`。

## 第 7 周：测试、调试和贡献

**目标**：具备真实贡献能力。

重点文件：

- `tests/conftest.py`
- `tests/integrations/conftest.py`
- `tests/integrations/test_registry.py`
- `tests/test_agent_config_consistency.py`
- `tests/test_setup_tasks.py`
- `tests/test_upgrade.py`
- `tests/test_extension_registration.py`

常用测试命令：

```bash
uv run python -m pytest -v
uv run python -m pytest tests/integrations -v
uv run python -m pytest tests/test_agent_config_consistency.py -v
uv run python -m pytest tests/test_workflows.py -v
```

建议贡献练习：

1. **文档贡献**：修正一个文档中的不清晰段落。
2. **测试贡献**：为现有 integration 增加一个边界测试。
3. **小 bug 修复**：找一个失败场景，先写失败测试，再修复。
4. **新增 integration**：按本仓库 `AGENTS.md` 的流程新增一个最小 Markdown integration。

新增 integration 练习步骤：

- 创建 `src/specify_cli/integrations/<name>/__init__.py`
- 继承 `MarkdownIntegration`
- 填写 `key`、`config`、`registrar_config`
- 在 `src/specify_cli/integrations/__init__.py` 注册
- 新增 `tests/integrations/test_integration_<name>.py`
- 跑单测

验收标准：

- 能独立完成一个小 PR 级别修改。
- 能解释失败测试的原因。
- 能知道改 CLI、改集成、改模板分别应该跑哪些测试。
- 本周产出物：一份“改动范围 -> 测试命令 -> 手工验证”的 checklist。

## 长期保持 upstream 最新

**目标**：让本地学习分支长期跟随官方 `github/spec-kit`，同时把个人学习资料控制在最小冲突面内。

当前约定：

- 主工作区是 WSL：`/home/xieyx/projects/spec-kit`。
- `origin` 是个人 fork：`https://github.com/xieyuxaing/spec-kit.git`。
- `upstream` 是官方仓库：`https://github.com/github/spec-kit.git`。
- 个人学习资料只放在 `docs/learning/`，不修改 `docs/toc.yml`、`docs/index.md`、`docs/README.md`。

每次同步前先检查：

```bash
cd /home/xieyx/projects/spec-kit
git status --short --branch
git fetch upstream --prune
git fetch origin --prune
git rev-list --left-right --count main...upstream/main
git rev-list --left-right --count main...origin/main
```

判断方式：

- `main...origin/main = 0 N`：本地只落后个人 fork，可以 `git merge --ff-only origin/main`。
- `main...upstream/main = 0 N`：本地只落后官方，可以 `git merge --ff-only upstream/main`，再推到 fork。
- 左侧不是 `0`：说明本地有个人提交。先确认这些提交是否只在 `docs/learning/`；如果是，通常应该 rebase 或等待 fork 合并 upstream 后再处理。

同步后的基础验证：

```bash
uv sync --extra test
uv run specify --version
uv run python -m pytest tests/test_agent_config_consistency.py -q
git diff --name-status upstream/main..HEAD
```

最后一条应该只显示 `docs/learning/` 下的学习资料变更。若出现官方入口文件或源码文件，先确认是不是有意修改。

## 应用到 tu-share-stock-screener 的实践路线

**目标**：把 Spec Kit 的 SDD 流程应用到同级 WSL 项目 `/home/xieyx/projects/tu-share-stock-screener`，先通过可控 demo 学习流程价值，再决定哪些内容进入真实项目。

### 实战边界

先区分两种模式：

| 模式 | 放在哪里 | 会改变什么 | 什么时候用 |
| --- | --- | --- | --- |
| 学习 demo | `/home/xieyx/projects/spec-kit/docs/learning/demos/<demo-name>/` | 只改变 `spec-kit` 的学习目录 | 第一次演练、验证模板、比较方案、记录踩坑 |
| 真实项目落地 | `/home/xieyx/projects/tu-share-stock-screener` | 改目标项目的代码、测试、项目文档或 Spec Kit 工件 | 已确认需求要真正实现时 |

默认先走学习 demo。只有当 demo 证明流程有帮助，并且明确要把结果应用到真实项目时，才进入 `tu-share-stock-screener` 工作树。

### 适合第一轮实战的场景

优先选低风险、边界清楚、容易验证的小功能：

| 场景 | 为什么适合 | 可能产物 |
| --- | --- | --- |
| 给已有页面增加一个筛选项 | 涉及前端状态、API 参数、测试，但业务风险低 | spec、plan、前端测试、API 测试 |
| 给已有 API 增加只读导出 | 不改核心交易逻辑，验收标准明确 | spec、API 响应约定、导出文件名规则 |
| 给手工执行历史增加展示字段 | 能练习数据来源、接口、页面展示的完整链路 | spec、tasks、后端和前端测试 |
| 给报表增加一个只读状态查看 | 适合练习“用户场景 -> 验收标准 -> 任务拆解” | spec、plan、轻量实现 checklist |

第一轮不建议选择：

- live trading 或真实券商操作。
- 数据库结构大改。
- 多个页面和多个 API 同时改。
- 需要新增复杂调度、权限或长期运行服务的功能。

### Step 1：创建学习 demo 容器

操作：

```bash
cd /home/xieyx/projects/spec-kit
mkdir -p docs/learning/demos/tu-share-stock-screener-first-practice
uv run specify init docs/learning/demos/tu-share-stock-screener-first-practice --integration codex --ignore-agent-tools --script sh --force
```

怎么做：

- 在 `spec-kit` 仓库内创建一个专门 demo 目录。
- 用当前源码里的 `specify` 初始化这个 demo。
- 使用 `codex` integration，生成 Codex 使用的 `speckit-*` skills。
- 使用 `--script sh`，让生成命令引用 Bash 脚本，匹配 WSL 环境。

会发生的改变：

- `docs/learning/demos/tu-share-stock-screener-first-practice/.specify/` 会出现模板、脚本、集成状态等文件。
- `docs/learning/demos/tu-share-stock-screener-first-practice/.agents/skills/` 会出现 Spec Kit 命令技能。
- 这些变化都留在 `docs/learning/` 下，不污染仓库根目录和官方文档入口。

主要目的：

- 先验证 Spec Kit 的目录、模板、命令和产物形态。
- 给后续实战留下可复盘的工件，而不是只靠聊天记录。

应用场景：

- 第一次学习 Spec Kit。
- 试一个新模板或新流程。
- 想比较“直接写代码”和“先写 spec/plan/tasks”的差异。

注意：

- 如果只是学习，不要在这一步对 `/home/xieyx/projects/tu-share-stock-screener` 运行 `specify init .`。
- demo 目录可以长期保留，但每个 demo 应该有清楚的名字和 README。

### Step 2：读取目标项目事实

操作：

```bash
cd /home/xieyx/projects/tu-share-stock-screener
git status --short --branch
sed -n '1,180p' README.md
sed -n '1,220p' AGENTS.md
find . -maxdepth 2 -type d | sort | sed -n '1,120p'
```

怎么做：

- 先确认目标项目工作树是否干净。
- 读取项目 README，了解当前系统能力。
- 读取目标项目自己的 AGENTS 指南，确认项目约定。
- 快速扫目录结构，建立后端、前端、文档、测试的地图。

会发生的改变：

- 这一步不应该改任何文件。
- 只产生观察结论，可以记录到 demo 的 README 或 notes 文件。

主要目的：

- 防止 Spec Kit 生成的计划脱离真实项目。
- 避免重复发明目标项目已经有的文档流程。

当前已确认的目标项目事实：

- WSL 仓库 `/home/xieyx/projects/tu-share-stock-screener` 是事实源。
- 主分支名是 `master`。
- 后端是 FastAPI，代码主要在 `screener/`。
- 前端是 React/Vite，目录是 `webapp/`。
- 测试目录是 `tests/`，前端测试在 `webapp` 内。
- 项目已有 `docs/project/`、`docs/features/` 和 AGENTS 里的 Feature Spec Impact / Doc Impact 要求。

应用场景：

- 每次准备让 Spec Kit 参与真实项目开发前都要做。
- 目标项目结构或测试命令变过时，先重新确认。

### Step 3：记录验证基线

操作：

```bash
cd /home/xieyx/projects/tu-share-stock-screener
. .venv/bin/activate
python scripts/check_project_docs.py --base origin/master
python -m pytest -q

cd /home/xieyx/projects/tu-share-stock-screener/webapp
npm run test -- --run
npm run build
```

怎么做：

- 后端使用项目虚拟环境。
- 先跑项目文档影响检查，再跑后端测试。
- 前端测试和构建在 `webapp/` 目录内执行。

会发生的改变：

- 理想情况下不改变源码。
- `npm run build` 可能刷新 `webapp/dist/`，跑完要回到项目根目录执行 `git status --short` 检查是否有生成文件变化。
- 如果测试失败，不要继续进入实现；先记录失败原因。

主要目的：

- 建立“改动前是健康的还是已经坏了”的基线。
- 后续判断 Spec Kit 拆出来的任务是否真的可验证。

应用场景：

- 真正实现前。
- demo 结束后准备迁移到真实项目前。
- 任何涉及 API、前端页面、交易模拟、手工执行流程的功能前。

### Step 4：选择一个实战题目

操作：

在 demo README 中先写清楚题目，推荐格式：

```markdown
# tu-share-stock-screener first practice

## 题目

为手工执行历史增加一个按 symbol 过滤的只读查看能力。

## 非目标

- 不改交易执行逻辑。
- 不改真实券商接口。
- 不新增数据库迁移。

## 初始验证命令

- `. .venv/bin/activate && python -m pytest -q`
- `cd webapp && npm run test -- --run`
- `cd webapp && npm run build`
```

怎么做：

- 用一句话定义题目。
- 明确不做什么。
- 先写验证命令，而不是最后才想怎么验收。

会发生的改变：

- `docs/learning/demos/<demo-name>/README.md` 会记录本次实战目标。
- 真实项目暂时不发生变化。

主要目的：

- 控制范围。
- 给 Spec Kit 的 `/speckit-specify` 提供更清楚的输入。

应用场景：

- 需求模糊时。
- 容易越做越大的功能。
- 想训练“需求边界”和“验收条件”的写法。

### Step 5：用 Spec Kit 生成 spec

操作方式不是 shell 命令，而是在以 demo 目录为工作区的 agent 会话里运行：

```text
/speckit-specify 为手工执行历史增加按 symbol 过滤的只读查看能力。不要修改交易执行逻辑，不新增数据库迁移，第一轮只要求生成清晰 spec。
```

怎么做：

- 把 Step 4 的题目和非目标喂给 Spec Kit。
- 让 agent 按模板生成 `spec.md`。
- 如果 agent 提问，优先回答业务边界、用户场景、验收条件。

会发生的改变：

- demo 目录下会出现 `specs/<feature-slug>/spec.md`。
- spec 会把自然语言需求拆成用户场景、功能需求、验收标准和边界条件。

主要目的：

- 把“我想加个功能”变成“用户要完成什么、系统必须满足什么、哪些不做”。
- 让后续 plan/tasks 不靠临场猜测。

应用场景：

- 用户需求还比较口语化。
- 需要跟 AI 或未来的自己对齐验收标准。
- 想判断一个需求是否适合拆成小任务。

检查重点：

- spec 是否能被不了解聊天上下文的人读懂。
- 是否写清楚输入、输出和错误情况。
- 是否出现了未确认的技术实现细节；如果有，先标成待澄清，不要直接当结论。

### Step 6：用 clarify 压缩不确定性

操作方式：

```text
/speckit-clarify
```

怎么做：

- 让 agent 基于当前 spec 找出不明确之处。
- 对每个问题给出具体选择。
- 只回答会影响实现或验收的问题；纯偏好问题不要过度展开。

会发生的改变：

- `spec.md` 会增加或修正澄清后的需求。
- 一些模糊句子会变成可测试的条件。

主要目的：

- 防止过早进入技术方案。
- 避免实现时才发现“其实需求不是这个意思”。

应用场景：

- API 参数含义不清。
- 页面筛选的空值、默认值、异常态没有定义。
- 导出文件名、排序、分页、权限边界没有定义。

### Step 7：用 plan 对齐目标项目架构

操作方式：

```text
/speckit-plan 请基于 tu-share-stock-screener 当前架构规划实现。后端是 FastAPI，前端是 React/Vite，项目要求保留 Feature Spec Impact 和 Doc Impact 报告。
```

怎么做：

- 把 Step 2 读取到的目标项目事实写进 plan 输入。
- 明确告诉 agent 不要绕开目标项目现有约定。
- 要求 plan 说明会影响哪些代码、测试和项目文档。

会发生的改变：

- demo 目录下的 `specs/<feature-slug>/plan.md` 会出现技术路线。
- 可能还会生成 data model、contracts、quickstart 等辅助工件，具体取决于模板和 agent 输出。

主要目的：

- 从“需求是什么”进入“在这个项目里怎么做”。
- 让计划尊重目标项目已有结构，而不是生成泛化方案。

应用场景：

- 需求涉及前后端联动。
- 需要决定 API 放在哪里、测试写在哪里。
- 需要判断是否更新 `docs/project/` 或 `docs/features/`。

检查重点：

- 是否引用了真实目录：`screener/`、`tests/`、`webapp/`、`docs/project/`、`docs/features/`。
- 是否把真实交易、券商接口、数据库迁移等高风险内容排除在第一轮之外。
- 是否列出验证命令。

### Step 8：用 tasks 拆成可执行任务

操作方式：

```text
/speckit-tasks
```

怎么做：

- 让 agent 基于 spec 和 plan 生成 `tasks.md`。
- 检查任务是否能独立验证。
- 把过大的任务拆小，把没有验收方式的任务退回修改。

会发生的改变：

- demo 目录下的 `specs/<feature-slug>/tasks.md` 会出现任务列表。
- 每个任务应该能映射到文件、测试或文档产物。

主要目的：

- 把计划拆成后续可以逐项实现的工作单元。
- 训练自己识别“任务是否太大、是否可验证、是否有依赖顺序”。

应用场景：

- 准备让 agent 进入实现前。
- 需要把一个小需求拆成后端、前端、测试、文档影响几类任务时。

### Step 9：对照目标项目自己的文档纪律

操作：

```bash
cd /home/xieyx/projects/tu-share-stock-screener
sed -n '1,220p' docs/project/feature-workflow.md
find docs/features -maxdepth 3 -type f | sort | sed -n '1,120p'
. .venv/bin/activate && python scripts/check_project_docs.py --base origin/master
```

怎么做：

- 把 Spec Kit 生成的 spec/plan/tasks 和目标项目现有 feature workflow 对照。
- 判断是否需要在目标项目里创建或更新 `docs/features/YYYY-MM-DD-feature-slug/spec.md`。
- 判断是否需要更新 `docs/project/` 下的长期项目记忆。

会发生的改变：

- 在 demo 阶段，只把判断记录到 `docs/learning/demos/<demo-name>/README.md`。
- 真正落地阶段，才在目标项目里创建或更新 `docs/features/`、`docs/project/`。

主要目的：

- 避免 Spec Kit 产物和目标项目已有文档体系冲突。
- 保留目标项目要求的 Feature Spec Impact 和 Doc Impact 报告习惯。

应用场景：

- 目标项目已经有自己的 feature spec、project memory 或 AGENTS 规则。
- 需要把外部 SDD 流程引入一个已有项目，而不是从零开始。

### Step 10：决定是否进入真实项目实现

进入真实项目前回答：

1. spec 是否已经清楚到可以独立阅读？
2. plan 是否引用了真实项目结构？
3. tasks 是否每条都有验收方式？
4. 是否知道需要更新哪些目标项目文档？
5. 是否知道要跑哪些测试？

如果任一答案是否，继续在 demo 里修改 spec/plan/tasks，不进入实现。

如果全部是，可以在目标项目创建工作分支：

```bash
cd /home/xieyx/projects/tu-share-stock-screener
git status --short --branch
git checkout -b feat/spec-kit-first-practice
```

会发生的改变：

- 目标项目只新增一个分支，不应该立刻改文件。
- 后续实现才会改代码、测试和项目文档。

主要目的：

- 把“学习推演”和“真实实现”分开。
- 保证目标项目主分支不被实验性产物污染。

应用场景：

- demo 已经证明需求、计划和任务清晰。
- 用户明确要把这轮实战变成目标项目真实改动。

### Step 11：真实实现时按任务执行

操作原则：

```text
一次只做 tasks.md 中一小组相关任务：
1. 先写或调整测试。
2. 再改实现。
3. 跑对应验证命令。
4. 更新 Feature Spec Impact 和 Doc Impact。
5. 回到 tasks.md 标记或记录完成情况。
```

怎么做：

- 后端任务优先跑相关 `pytest`。
- 前端任务优先跑 `npm run test -- --run`。
- 涉及页面构建再跑 `npm run build`。
- 涉及项目记忆或 feature spec 时跑 `python scripts/check_project_docs.py --base origin/master`。

会发生的改变：

- 目标项目代码、测试和文档开始发生真实变化。
- 每个任务完成后都应该能在 git diff 中解释清楚。

主要目的：

- 防止 agent 一次性改太多。
- 保持每步都有验证证据。

应用场景：

- 从 demo 过渡到真实开发。
- 需要让 AI 辅助实现，但仍保留人工可审查边界。

### Step 12：收尾、同步和复盘

目标项目真实实现完成后，按目标项目规则收尾：

```bash
cd /home/xieyx/projects/tu-share-stock-screener
git status --short --branch
. .venv/bin/activate && python scripts/check_project_docs.py --base origin/master
python -m pytest -q
(cd webapp && npm run test -- --run)
(cd webapp && npm run build)
```

然后回到学习目录记录复盘：

```bash
cd /home/xieyx/projects/spec-kit
sed -n '1,220p' docs/learning/demos/tu-share-stock-screener-first-practice/README.md
```

复盘内容建议包含：

- 本轮 feature 题目。
- Spec Kit 哪一步最有帮助。
- 哪一步生成内容不适合目标项目。
- 最终采用了哪些目标项目验证命令。
- 是否需要为 `tu-share-stock-screener` 固化 constitution、preset 或 extension。

会发生的改变：

- 目标项目产生真实提交时，按目标项目 WSL-first 流程推送并同步 Windows 镜像。
- `spec-kit` 的 `docs/learning/demos/` 保留学习记录。

主要目的：

- 把一次实战沉淀成下一次可复用的经验。
- 判断 Spec Kit 是只作为学习工具，还是要成为目标项目长期流程的一部分。

应用场景：

- 每完成一轮 demo 或真实功能后。
- 准备把 Spec Kit 用法长期固化到 `tu-share-stock-screener` 前。

### 可以补充的长期方向

如果第一轮实战效果好，可以继续做这些改进：

1. 为 `tu-share-stock-screener` 写项目级 constitution 草稿，先放在 `docs/learning/demos/` 评审，不直接写入目标项目。
2. 总结目标项目专用的 feature 模板差异，判断是否需要 Spec Kit preset。
3. 把目标项目的 Feature Spec Impact / Doc Impact 要求整理成 Spec Kit extension 或 agent-context 说明。
4. 设计一个“只读 API 变更”的固定任务模板，用于筛选、导出、报表类功能。
5. 每轮实战后维护一个 `docs/learning/demos/<demo-name>/retrospective.md`，记录哪些步骤真正节省了返工。

这一阶段的目标不是马上让目标项目完全“Spec Kit 化”，而是验证 SDD 工件是否能减少需求不清、任务过大、验证不完整的问题。只有验证有效的部分才值得固化到真实项目。

## 每周复盘问题

每周结束时回答这些问题：

1. 这周我能用一句话解释哪个模块？
2. 这个模块最关键的 3 个文件是什么？
3. 这个模块最容易出 bug 的边界在哪里？
4. 我能用哪个测试证明自己理解了它？
5. 如果要给项目提一个小 PR，我会改哪里？

## 推荐阅读顺序

按这个顺序读源码，阻力最小：

1. `README.md`
2. `docs/local-development.md`
3. `src/specify_cli/__init__.py`
4. `src/specify_cli/commands/init.py`
5. `src/specify_cli/integrations/__init__.py`
6. `src/specify_cli/integrations/base.py`
7. `src/specify_cli/integrations/codex/__init__.py`
8. `src/specify_cli/integrations/copilot/__init__.py`
9. `src/specify_cli/shared_infra.py`
10. `src/specify_cli/presets/_commands.py`
11. `src/specify_cli/extensions/_commands.py`
12. `src/specify_cli/commands/bundle/__init__.py`
13. `src/specify_cli/bundler/services/installer.py`
14. `src/specify_cli/workflows/engine.py`
15. `tests/integrations/test_registry.py`
16. `tests/integrations/test_integration_codex.py`
17. `tests/test_workflows.py`

## 学习笔记模板

建议在本地新建 `notes/` 或使用你自己的笔记工具。每个模块一页：

```markdown
# 模块名

## 一句话理解

## 入口文件

## 关键类和函数

## 数据流

## 典型命令

## 相关测试

## 我还没理解的问题

## 可以尝试的贡献点
```

## 最终能力清单

完成这份计划后，你应该能做到：

- 独立本地运行和调试 `specify` CLI。
- 解释 Spec Kit 的 SDD 流程和核心命令。
- 读懂 integration registry 和 base classes。
- 添加一个新的 AI agent integration。
- 判断 preset、extension、local override 的使用场景。
- 解释 bundle 如何组合 preset、extension、workflow 和 step。
- 跑相关测试并根据失败信息定位问题。
- 给 upstream 提交一个小而完整的贡献。
