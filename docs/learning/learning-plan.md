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
uv run specify init ../spec-kit-demo --integration codex --ignore-agent-tools --script sh
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
uv run specify init ../spec-kit-preset-demo --integration codex --preset lean --ignore-agent-tools --script sh
```

- 在临时项目中安装 bundled extension：

```bash
(
  cd ../spec-kit-preset-demo
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
uv run specify bundle build --path examples/bundles/developer --output ../spec-kit-bundle-demo
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

**目标**：把 Spec Kit 的 SDD 流程应用到同级 WSL 项目 `/home/xieyx/projects/tu-share-stock-screener`，先学习流程价值，再决定哪些部分长期固化。

建议选择一个低风险功能作为第一轮练习，例如：

- 为某个已有页面补一段查询或筛选能力。
- 为已有 API 增加一个只读导出或状态查看接口。
- 为手工执行历史增加一个小的展示字段或过滤条件。

实践顺序：

1. 先读目标项目结构：

   ```bash
   cd /home/xieyx/projects/tu-share-stock-screener
   git status --short --branch
   rg --files | head -80
   ```

2. 在目标项目里建立专门分支，不直接污染主分支：

   ```bash
   git checkout -b docs/spec-kit-first-practice
   ```

3. 先只产出 Spec Kit 工件，不急着实现：

   ```bash
   # 在目标项目中运行已安装的本地 specify
   /home/xieyx/projects/spec-kit/.venv/bin/specify init . --integration codex --ignore-agent-tools --script sh --force
   ```

   如果只是验证流程，可以在临时副本或临时目录里先跑，不要马上写入真实项目。

4. 用 Spec Kit 的顺序写一轮小功能：

   ```text
   constitution -> specify -> clarify -> plan -> tasks
   ```

   第一轮先停在 `tasks.md`。重点观察：

   - 生成的 spec 是否比原始需求更清楚。
   - plan 是否捕捉了项目现有架构。
   - tasks 是否能拆成可验证的小步。
   - 哪些模板或说明不适合 `tu-share-stock-screener`。

5. 对比目标项目现有验证方式：

   ```bash
   # 后续以目标项目自己的命令为准；这里先记录实际可用命令
   rg "pytest|npm run|uv|python -m" README.md pyproject.toml package.json .github -n
   ```

6. 复盘是否要固化：

   - 如果只是个人学习，保留 spec/plan/tasks 作为练习记录即可。
   - 如果对真实开发有帮助，再考虑为 `tu-share-stock-screener` 写项目级 constitution。
   - 如果发现模板不适配，再回到 Spec Kit 学习 preset/extension，而不是直接手改生成文件。

这一阶段的目标不是马上让目标项目完全“Spec Kit 化”，而是验证 SDD 工件是否能减少需求不清、任务过大、验证不完整的问题。

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
