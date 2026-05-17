# Spec Kit 深度学习计划

> 适用仓库：`D:\studyProject\spec-kit`
>
> 目标：从“会使用 Specify CLI”逐步推进到“能读懂核心实现、能添加集成、能修复问题并提交贡献”。

## 学习路线总览

这个项目可以按 5 条主线理解：

1. **Spec-Driven Development 方法论**：理解 Spec Kit 为什么要把需求、计划、任务、实现拆成阶段。
2. **Specify CLI**：理解 `specify init`、`integration`、`preset`、`extension`、`workflow` 等命令如何工作。
3. **Integration 架构**：理解不同 AI agent 如何通过统一 registry 和 base class 接入。
4. **模板、脚本和资产打包**：理解 `.specify/`、templates、scripts、core pack、wheel 打包之间的关系。
5. **测试和贡献流程**：能跑测试、定位问题、添加新集成或修复 bug。

建议节奏：每天 60-90 分钟，持续 6 周。时间紧的话可以压缩成 3 周，每天完成两个小节。

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

```powershell
uv venv
.\.venv\Scripts\Activate.ps1
uv pip install -e ".[test]"
```

- 查看 CLI：

```powershell
python -m src.specify_cli --help
python -m src.specify_cli init --help
python -m src.specify_cli integration --help
python -m src.specify_cli preset --help
python -m src.specify_cli extension --help
python -m src.specify_cli workflow --help
```

- 初始化一个临时项目：

```powershell
python -m src.specify_cli init ..\spec-kit-demo --integration codex --ignore-agent-tools --script ps
```

验收标准：

- 能解释 `specify init` 会生成哪些目录。
- 能说明 `.specify/`、agent command files、context file 的用途。
- 能成功跑通一个本地 `init` 示例。

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

```powershell
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
- `src/specify_cli/_console.py`
- `src/specify_cli/_utils.py`
- `src/specify_cli/_assets.py`
- `src/specify_cli/_version.py`

阅读顺序：

1. 从 `app = typer.Typer(...)` 开始看 CLI 注册。
2. 找 `@app.command()`，列出顶层命令。
3. 重点读 `init()`，理解项目初始化流程。
4. 再读 integration、preset、extension、workflow 相关命令。

动手任务：

- 给自己写一份命令映射表：

| CLI 命令 | 入口函数 | 主要职责 | 依赖模块 |
|---|---|---|---|
| `specify init` | `init()` | 初始化项目 | integrations, shared_infra, assets |
| `specify integration list` | `integration_list()` | 列出集成 | integrations |
| `specify preset add` | `preset_add()` | 安装 preset | presets |
| `specify extension add` | `extension_add()` | 安装 extension | extensions |
| `specify workflow run` | `workflow_run()` | 运行 workflow | workflows |

建议调试：

```powershell
python -m src.specify_cli --version
python -m src.specify_cli check
python -m src.specify_cli integration list
```

验收标准：

- 能说明 `Typer` 在这个项目里如何组织命令。
- 能从 CLI 参数追踪到对应的核心逻辑。
- 能解释 `--integration`、`--script`、`--ignore-agent-tools` 的作用。

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
- `context_file`
- `registrar_config`

动手任务：

- 选择 4 个 integration 做对比：

| Integration | Base class | 输出格式 | context file | 特殊逻辑 |
|---|---|---|---|---|
| Codex | `SkillsIntegration` | `SKILL.md` | `AGENTS.md` | skills mode |
| Gemini | `TomlIntegration` | `.toml` | `GEMINI.md` | `{{args}}` |
| Goose | `YamlIntegration` | `.yaml` | `AGENTS.md` | recipe format |
| Copilot | `IntegrationBase` | `.agent.md` + `.prompt.md` | `.github/copilot-instructions.md` | settings merge |

- 跑集成测试：

```powershell
pytest tests/integrations/test_registry.py -v
pytest tests/integrations/test_integration_base_markdown.py -v
pytest tests/integrations/test_integration_codex.py -v
pytest tests/integrations/test_integration_copilot.py -v
```

验收标准：

- 能新增一个简单 Markdown integration。
- 能解释为什么 CLI-based integration 的 `key` 要匹配可执行文件名。
- 能解释 manifest 如何支持 uninstall。

## 第 4 周：Presets、Extensions 和 Template Resolution

**目标**：理解 Spec Kit 如何被定制和扩展。

重点文件：

- `src/specify_cli/presets.py`
- `src/specify_cli/extensions.py`
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

```powershell
python -m src.specify_cli init ..\spec-kit-preset-demo --integration codex --preset lean --ignore-agent-tools --script ps
```

- 在临时项目中安装 bundled extension：

```powershell
cd ..\spec-kit-preset-demo
python -m D:\studyProject\spec-kit\src.specify_cli extension add git
```

- 观察 `.specify/` 和 agent 命令目录变化。

验收标准：

- 能解释“模板运行时解析”和“命令安装时写入”的区别。
- 能说明 preset 更适合改“工作方式”，extension 更适合加“新能力”。
- 能定位某个命令文件最终来自 core、preset 还是 extension。

## 第 5 周：Workflow Engine

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

```powershell
rg --files workflows
rg "type:|steps:|command|prompt|shell|gate" workflows src\specify_cli\workflows
```

- 跑 workflow 相关测试：

```powershell
pytest tests/test_workflows.py -v
```

验收标准：

- 能说明 workflow 如何暂停、恢复和保存状态。
- 能解释表达式系统负责什么。
- 能新增或修改一个简单 workflow step 的测试。

## 第 6 周：测试、调试和贡献

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

```powershell
pytest -v
pytest tests/integrations -v
pytest tests/test_agent_config_consistency.py -v
pytest tests/test_workflows.py -v
```

建议贡献练习：

1. **文档贡献**：修正一个文档中的不清晰段落。
2. **测试贡献**：为现有 integration 增加一个边界测试。
3. **小 bug 修复**：找一个失败场景，先写失败测试，再修复。
4. **新增 integration**：按 `AGENTS.md` 的流程新增一个最小 Markdown integration。

新增 integration 练习步骤：

- 创建 `src/specify_cli/integrations/<name>/__init__.py`
- 继承 `MarkdownIntegration`
- 填写 `key`、`config`、`registrar_config`、`context_file`
- 在 `src/specify_cli/integrations/__init__.py` 注册
- 新增 `tests/integrations/test_integration_<name>.py`
- 跑单测

验收标准：

- 能独立完成一个小 PR 级别修改。
- 能解释失败测试的原因。
- 能知道改 CLI、改集成、改模板分别应该跑哪些测试。

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
4. `src/specify_cli/integrations/__init__.py`
5. `src/specify_cli/integrations/base.py`
6. `src/specify_cli/integrations/codex/__init__.py`
7. `src/specify_cli/integrations/copilot/__init__.py`
8. `src/specify_cli/shared_infra.py`
9. `src/specify_cli/presets.py`
10. `src/specify_cli/extensions.py`
11. `src/specify_cli/workflows/engine.py`
12. `tests/integrations/test_registry.py`
13. `tests/integrations/test_integration_codex.py`
14. `tests/test_workflows.py`

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
- 跑相关测试并根据失败信息定位问题。
- 给 upstream 提交一个小而完整的贡献。
