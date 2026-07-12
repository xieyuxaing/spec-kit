# Workflows

> 本文是 [英文原文](README.md) 的中文翻译。命令、YAML 键名、表达式和代码示例保持原样。

Workflow 是使用 YAML 定义的多步骤、可恢复自动化流水线。它跨 Integration 编排 Spec Kit 命令、计算控制流，并在人工审查 Gate 处暂停，从而无需手动逐步调用，就能完成端到端规格驱动开发周期。

## 工作原理

Workflow 定义声明一系列 Step。引擎按顺序执行这些 Step，把命令分发给 AI Integration、运行 Shell 命令、计算分支条件，并在 Gate 处暂停等待人工审查。每个 Step 完成后都会持久化状态，因此 Workflow 可以在中断后恢复。

```yaml
steps:
  - id: specify
    command: speckit.specify
    input:
      args: "{{ inputs.spec }}"

  - id: review
    type: gate
    message: "Review the spec before planning."
    options: [approve, reject]
    on_reject: abort

  - id: plan
    command: speckit.plan
```

详细架构和内部实现请参阅 [ARCHITECTURE.md](ARCHITECTURE.md)。

## 快速开始

```bash
# 搜索可用 Workflow
specify workflow search

# 安装内置 SDD Workflow
specify workflow add speckit

# 或直接从本地 YAML 文件运行
specify workflow run ./workflow.yml --input spec="Build a user authentication system with OAuth support"

# 带输入运行已安装的 Workflow
specify workflow run speckit --input spec="Build a user authentication system with OAuth support"

# 检查运行状态
specify workflow status

# 从 Gate 暂停处恢复
specify workflow resume <run_id>

# 获取 Workflow 详细信息
specify workflow info speckit

# 移除 Workflow
specify workflow remove speckit
```

## 运行 Workflow

### 运行已安装的 Workflow

```bash
specify workflow add speckit
specify workflow run speckit --input spec="Build a user authentication system with OAuth support"
```

### 从本地 YAML 文件运行

```bash
specify workflow run ./my-workflow.yml --input spec="Build a user authentication system with OAuth support"
```

### 多个输入

```bash
specify workflow run speckit \
  --input spec="Build a user authentication system with OAuth support" \
  --input scope="backend-only"
```

## Step 类型

Workflow 支持 11 种内置 Step 类型。

### Command Step（默认）

通过 Integration CLI 按名称调用已经安装的 Spec Kit 命令：

```yaml
- id: specify
  command: speckit.specify
  input:
    args: "{{ inputs.spec }}"
  integration: claude        # 可选：覆盖 Workflow 默认值
  model: "claude-sonnet-4-20250514"   # 可选：覆盖模型
```

### Prompt Step

向 Integration CLI 发送任意内联提示词，不需要命令文件：

```yaml
- id: security-review
  type: prompt
  prompt: "Review {{ inputs.file }} for security vulnerabilities"
  integration: claude
```

### Shell Step

运行 Shell 命令并捕获输出：

```yaml
- id: run-tests
  type: shell
  run: "cd {{ inputs.project_dir }} && npm test"
```

### Init Step

以与 `specify init` 相同的方式初始化项目，包括生成模板、脚本、共享基础设施和选定的编程 Agent Integration。它以非交互方式运行，默认启用 `--ignore-agent-tools`，并从 Step 配置或 Workflow 默认值解析 Integration：

```yaml
- id: bootstrap
  type: init
  here: true                 # 或：project: my-project
  integration: copilot       # 可选：默认为 Workflow Integration
  integration_options: "--skills"  # 可选：传给 Integration 的额外选项
  script: sh                 # 可选：sh 或 ps
  force: true                # 可选：目标目录已存在时必须设置
  preset: healthcare-compliance   # 可选 Preset ID
```

### Gate Step

暂停并等待人工审查。调用 `specify workflow resume` 后，Workflow 继续执行：

```yaml
- id: review-spec
  type: gate
  message: "Review the generated spec before planning."
  options: [approve, edit, reject]
  on_reject: abort
```

### If/Then/Else Step

根据表达式执行条件分支：

```yaml
- id: check-scope
  type: if
  condition: "{{ inputs.scope == 'full' }}"
  then:
    - id: full-plan
      command: speckit.plan
  else:
    - id: quick-plan
      command: speckit.plan
      options:
        quick: true
```

### Switch Step

根据表达式值进行多分支调度：

```yaml
- id: route
  type: switch
  expression: "{{ steps.review.output.choice }}"
  cases:
    approve:
      - id: plan
        command: speckit.plan
    reject:
      - id: log
        type: shell
        run: "echo 'Rejected'"
  default:
    - id: fallback
      type: gate
      message: "Unexpected choice"
```

### While Loop Step

条件为真时重复执行 Step：

```yaml
- id: retry
  type: while
  condition: "{{ steps.run-tests.output.exit_code != 0 }}"
  max_iterations: 5
  steps:
    - id: fix
      command: speckit.implement
```

### Do-While Loop Step

至少执行一次 Step，然后在条件成立时重复：

```yaml
- id: refine
  type: do-while
  condition: "{{ steps.review.output.choice == 'edit' }}"
  max_iterations: 3
  steps:
    - id: revise
      command: speckit.specify
```

### Fan-Out Step

对集合中的每个元素依次调度 Step 模板：

```yaml
- id: parallel-impl
  type: fan-out
  items: "{{ steps.tasks.output.task_list }}"
  max_concurrency: 3
  step:
    id: impl
    command: speckit.implement
```

### Fan-In Step

聚合 Fan-Out Step 的结果：

```yaml
- id: collect
  type: fan-in
  wait_for: [parallel-impl]
  output: {}
```

## 错误处理

默认情况下，任何在运行时返回 `StepResult(status=StepStatus.FAILED, ...)` 的 Step 都会停止整个运行，最常见的情况是 `shell` 或 `command` Step 以非零状态退出。在 Step 上设置 `continue_on_error: true`，可以记录失败结果并继续执行下一个同级 Step。如果失败来自非零退出，退出码仍可通过 `steps.<id>.output.exit_code` 访问，后续 `if` 或 `switch` 可以据此分支；`gate` 也可以通过 `message` 中的 `{{ }}` 插值把退出码显示给操作者：

```yaml
- id: heavy-thing
  type: command
  integration: claude
  command: speckit.heavy-thing
  continue_on_error: true

- id: check-result
  type: if
  condition: "{{ steps.heavy-thing.output.exit_code != 0 }}"
  then:
    - id: review
      type: gate
      message: "Step failed (exit {{ steps.heavy-thing.output.exit_code }}). Approve to run the recovery path, or reject to leave the failure recorded and move on."
      on_reject: skip
    - id: recover
      type: if
      condition: "{{ steps.review.output.choice == 'approve' }}"
      then:
        - id: rerun
          command: speckit.recovery
  else:
    - id: next-thing
      command: speckit.next-thing
```

理解该示例时需要注意：

- Gate 的两个选项 `approve` 和 `reject` 都会返回 `StepStatus.COMPLETED`。`on_reject: skip` 只控制引擎是否因 reject 而中止；设置为 `skip` 时不会中止，但它**不会**自动跳过 `then:` 列表中后续的同级 Step。Workflow 作者必须负责后续分支：在后续的 `if`、`switch` 或表达式中读取 `{{ steps.<gate-id>.output.choice }}`，就像上面的 `recover` Step。
- `on_reject` 有三个值：`abort` 是默认值，reject 会产生 `StepStatus.FAILED` 和 `output.aborted = True` 并停止运行；`skip` 让 reject 返回 `StepStatus.COMPLETED`，由作者自行处理分支；`retry` 让 reject 返回 `StepStatus.PAUSED`，下一次 `specify workflow resume` 会重新运行 Gate。
- Gate 不会自动重新运行失败的 Step。如果需要表达重试路径，可以定义自定义 Gate 选项并在后续根据选择分支，或者把失败 Step 包装在自定义循环中。

**注意：**

- 该字段必须是字面量布尔值 `true` 或 `false`；校验会拒绝类似 `"true"` 的强制转换字符串。
- **范围仅限返回的失败。** 该标志适用于 `status=StepStatus.FAILED` 的 Step 结果。Step 的 `execute()` 方法抛出且未处理的异常，会在上一层被 `WorkflowEngine.execute()` 捕获，记录为 `workflow_failed`，并且无论 `continue_on_error` 如何设置都会中止运行。如果 Step 作者希望该标志覆盖异常路径，Step 必须在内部捕获异常，并返回 `StepResult(status=StepStatus.FAILED, ...)`，同时在 `output` 中编码失败信息，例如 `exit_code`、`stderr` 或自定义字段。
- 操作者选择 `on_reject: abort` 产生的 Gate 中止始终会停止运行，`continue_on_error` 不会覆盖它。该标志用于暂时性或预期内的 Step 失败，不用于覆盖操作者的明确决定。
- 结构校验会预先执行。`specify workflow run` 在创建运行前就会拒绝无效 Workflow 定义，因此校验失败不会进入这条错误处理路径。
- 省略该标志时，行为与引入此功能之前逐字节一致。

## 表达式

Workflow 定义使用 `{{ expression }}` 语法表示动态值：

```yaml
# 访问输入
args: "{{ inputs.spec }}"

# 访问之前 Step 的输出
args: "{{ steps.specify.output.file }}"

# 比较
condition: "{{ steps.run-tests.output.exit_code != 0 }}"

# 过滤器
message: "{{ status | default('pending') }}"
```

支持的过滤器：`default`、`join`、`contains`、`map`、`from_json`。

### 运行时上下文

`{{ context.* }}` 暴露当前运行中由引擎管理的运行时元数据：

| 变量 | 说明 |
| --- | --- |
| `context.run_id` | 当前 Workflow 运行 ID，与 `workflow run` 结束时 Spec Kit 输出的 `Run ID:` 相同。自动生成的 ID 是来自 `uuid4` 的 8 位十六进制字符串；操作者提供的 ID 可以是包含连字符或下划线的任意字母数字字符串。不在运行上下文中时为空字符串。 |

```yaml
# 使用运行 ID 标记遥测事件，以便跨系统关联。
- id: emit-event
  type: shell
  run: 'echo "{\"run_id\":\"{{ context.run_id }}\",\"event\":\"started\"}" >> events.jsonl'

# 每次运行独立的临时目录。
- id: prep-scratch
  type: shell
  run: 'mkdir -p /tmp/run-{{ context.run_id }}'

# 把运行 ID 传给命令，用作工件元数据。
- id: tag-artifact
  command: speckit.specify
  input:
    args: "{{ context.run_id }}"
```

## 输入类型

Workflow 输入会执行类型检查，并从 CLI 字符串值进行转换：

```yaml
inputs:
  spec:
    type: string
    required: true
    prompt: "Describe what you want to build"
  task_count:
    type: number
    default: 5
  dry_run:
    type: boolean
    default: false
  scope:
    type: string
    default: "full"
    enum: ["full", "backend-only", "frontend-only"]
```

| 类型 | 接受的值 | 示例 |
| --- | --- | --- |
| `string` | 任意字符串 | `"user-auth"` |
| `number` | 数字字符串转换为 int/float | `"42"` → `42` |
| `boolean` | `true`/`1`/`yes` → `True`，`false`/`0`/`no` → `False` | `"true"` → `True` |

## 状态和恢复

每次 Workflow 运行都会把状态持久化到 `.specify/workflows/runs/<run_id>/`：

```bash
# 列出所有运行及其状态
specify workflow status

# 检查指定运行
specify workflow status <run_id>

# 恢复暂停的运行（批准 Gate 后）
specify workflow resume <run_id>

# 恢复失败的运行（从失败 Step 重试）
specify workflow resume <run_id>
```

运行状态：`created` → `running` → `completed` | `paused` | `failed` | `aborted`

## Catalog 管理

Workflow 通过 Catalog 发现。Spec Kit 默认使用官方 Catalog 和社区 Catalog：

> [!NOTE]
> 社区 Workflow 由各自作者独立创建和维护。GitHub 和 Spec Kit 维护者可能会检查向社区 Catalog 添加条目的 Pull Request 格式和结构，但他们**不会审查、审核、认可或支持 Workflow 定义本身**。安装前请检查 Workflow 源码，并自行判断是否使用。

```bash
# 列出已激活 Catalog
specify workflow catalog list

# 添加自定义 Catalog
specify workflow catalog add https://example.com/catalog.json --name my-org

# 移除 Catalog
specify workflow catalog remove <index>
```

## 创建 Workflow

1. 按照上面的 Schema 创建 `workflow.yml`。
2. 使用 `specify workflow run ./workflow.yml --input key=value` 在本地测试。
3. 使用 `specify workflow info ./workflow.yml` 验证。
4. 参阅 [PUBLISHING.md](PUBLISHING.md)，提交到 Catalog。

## 环境变量

| 变量 | 说明 |
| --- | --- |
| `SPECKIT_WORKFLOW_CATALOG_URL` | 覆盖 Catalog URL，并替换所有默认值 |

## 配置文件

| 文件 | 作用域 | 说明 |
| --- | --- | --- |
| `.specify/workflow-catalogs.yml` | 项目 | 当前项目的自定义 Catalog 栈 |
| `~/.specify/workflow-catalogs.yml` | 用户 | 所有项目的自定义 Catalog 栈 |

## 仓库结构

```text
workflows/
├── ARCHITECTURE.md                         # 内部架构文档
├── PUBLISHING.md                           # 向 Catalog 提交 Workflow 的指南
├── README.md                               # 英文原文
├── README.zh-CN.md                         # 中文翻译
├── catalog.json                            # 官方 Workflow Catalog
├── catalog.community.json                  # 社区 Workflow Catalog
└── speckit/                                # 内置 SDD 周期 Workflow
    └── workflow.yml
```
