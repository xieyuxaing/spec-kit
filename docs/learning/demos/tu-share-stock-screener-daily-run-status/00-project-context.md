# 00. Project Context

## 目的

本文件回答一个问题：在写 `daily-run-status` 的 spec 之前，agent
应该先知道哪些长期背景。

Spec Kit 里的 `constitution` 和项目记忆承担类似职责：它们不是单个功能的需求，
而是所有功能都必须遵守的长期约束。对于这个案例，长期背景来自主项目的
`AGENTS.md` 和 `docs/project/`。

## 主项目事实源

| 项目 | 路径或值 |
| --- | --- |
| WSL 主仓库 | `/home/xieyx/projects/tu-share-stock-screener` |
| Windows 镜像 | `D:\study_docs\06-trade_stocks\tu-share-stock-screener` |
| 真实提交 | `9cc1b86 Add daily run status dashboard` |
| 功能 spec | `docs/features/2026-07-04-daily-run-status/spec.md` |
| 相关长期文档 | `docs/project/workflows.md`, `docs/project/api-contracts.md` |

主仓库是唯一事实源。Windows 目录只是镜像，不作为主开发区。
这个约束对实现没有业务影响，但对开发流程有影响：代码提交推送后，
需要让 Windows 镜像快进同步。

## 主项目已有项目记忆

在做这个功能前，主项目已经建立了三层记忆。

第一层是 `AGENTS.md`：

- 规定 WSL 主仓库优先。
- 规定开发前先读项目记忆。
- 规定完成前报告 Doc Impact。
- 规定验证默认在 WSL 执行。
- 规定不得清理、回滚或暂存无关脏文件。

第二层是 `docs/project/`：

- `README.md`：项目记忆目录入口。
- `invariants.md`：不可破坏规则。
- `workflows.md`：稳定业务流程。
- `api-contracts.md`：稳定 API 和导出契约。
- `data-model.md`：SQLite 表和 source-of-truth 语义。
- `domain.md`：业务术语。
- `verification.md`：验证命令。
- `doc-map.yml`：代码区域到项目记忆文档的映射。

第三层是 `docs/features/`：

- 单个功能的短期事实源。
- 记录本次功能的目标、边界、测试计划和 Doc Impact。
- 稳定规则才沉淀到 `docs/project/`。

## 当前业务背景

`tu-share-stock-screener` 是一个本地股票筛选和交易辅助项目。
它围绕 Tushare 数据源、本地 SQLite 状态、策略选股、候选股复核、
交易计划、手工执行和审计报告运行。

在 `daily-run-status` 之前，用户可以分别查看候选股、研究回测、组合计划和运维信息。
但每日流程是否完整，需要从多个地方拼判断：

- 数据是否更新。
- 因子是否构建。
- 当前策略是否有足够字段运行。
- 候选股是否生成。
- 候选股是否复核。
- 人工交易计划是否确认。
- 人工执行报告是否生成。
- SQLite 是否有足够新的备份。

这个判断如果只靠聊天上下文或操作者记忆，很容易漏掉步骤。

## 核心业务约束

这个功能必须遵守以下约束：

| 约束 | 含义 |
| --- | --- |
| SQLite 是本地状态源 | 不能绕过已有 repository 直接假造状态 |
| 状态中心只读 | 不能触发任务、修改复核、生成计划或执行交易 |
| 策略和 broker 解耦 | 状态检查不能引入 broker SDK 或 live trading |
| live trading 默认关闭 | 不能因为状态中心新增任何真实交易能力 |
| 执行历史可审计 | `manual_execution_reports` 仍是执行历史事实源 |
| API 契约要稳定 | 新端点状态枚举和字段语义需要写入长期文档 |
| 文档影响必须报告 | 行为、API、流程变化要更新 `docs/project/` |

这些约束不是实现细节，而是 spec 的输入。
如果 spec 没有体现这些约束，后续 plan 和 tasks 就容易偏离目标。

## 已有系统能力

实现前可复用的能力包括：

- 后端 FastAPI 应用在 `screener/web/api.py`。
- Web service 汇总逻辑在 `screener/web/service.py`。
- SQLite 访问封装在 `screener/storage/repository.py`。
- 已有候选股 API 可以判断策略是否成功运行或缺字段。
- 前端控制台主入口在 `webapp/src/App.jsx`。
- 前端测试使用 Vitest 和 Testing Library。
- 后端测试使用 pytest 和 FastAPI `TestClient`。
- 项目文档影响检查脚本是 `scripts/check_project_docs.py`。

## 不应该假设的事情

写 spec 前不能假设：

- 用户想自动修复缺失步骤。
- 用户想让状态中心运行每日流程。
- 用户想新增数据库表。
- 用户想接入真实 broker。
- 用户只关心候选股，不关心备份和审计。
- 前端只要显示一个静态说明就够了。

这些都要么超出范围，要么会破坏已有 invariants。

## 读者练习

阅读本文件后，先不要看后面的 spec。请自己回答：

1. 这个功能的最小有价值版本是什么？
2. 哪些信息必须来自已有 SQLite 或已有 service？
3. 哪些行为必须明确写成 out of scope？
4. 哪些长期文档会被影响？

如果回答不出来，说明还没有准备好写 `02-spec.md`。
