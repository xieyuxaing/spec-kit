# 02. Feature Specification

**Feature**: Daily Run Status

**Source feature slug**: `2026-07-04-daily-run-status`

**Real implementation commit**: `9cc1b86 Add daily run status dashboard`

**Status**: Implemented in source project, reconstructed here for learning

**Input**: Add a read-only daily status center for the stock screener dashboard.

## 中文摘要

本 spec 定义一个只读的每日状态中心。
它按交易日和策略汇总行情数据、因子、候选股、候选复核、
人工计划、执行报告和 SQLite 备份状态。

本功能不新增数据库表，不触发任务，不执行交易，不改变策略模型。

## User Scenarios & Testing

### User Story 1: 查看每日流程是否可复核 (Priority: P1)

作为本地操作者，我希望在 dashboard 上看到当前交易日和策略的每日流程状态，
从而快速判断今天是否可以开始候选股复核。

**Why this priority**:

这是最小有价值功能。没有它，操作者仍然需要到多个页面和文件中拼判断。

**Independent Test**:

给定 seeded SQLite 中存在当日行情和因子，并且当前策略可以生成候选股，
访问 `GET /api/daily-status?trade_date=20260421&strategy=price_momentum`
应返回 `ready_for_review`，前端 dashboard 应显示“可复核”。

**Acceptance Scenarios**:

1. **Given** 当日行情和因子已存在，**When** 操作者打开 dashboard，
   **Then** 系统显示当前交易日、策略、Tushare 积分等级和每日状态。
2. **Given** 当前策略生成了候选股，**When** daily status API 返回，
   **Then** `candidate_selection` 步骤为 `done`，整体状态至少为 `ready_for_review`。
3. **Given** 候选股尚未复核，**When** 状态中心展示流程，
   **Then** `candidate_review` 步骤显示待完成，而不是伪装为完成。

---

### User Story 2: 识别策略数据缺失并阻塞错误使用 (Priority: P1)

作为操作者，我希望当所选策略缺少必要字段时，状态中心明确显示阻塞原因，
避免我把“没有候选股”误判为“策略运行成功但无机会”。

**Why this priority**:

策略缺字段是业务判断错误的高风险来源。特别是质量动量和低估值策略依赖财务字段，
在 Tushare 积分或数据准备不足时不能静默成功。

**Independent Test**:

给定 SQLite 中只有价格动量需要的字段，访问
`GET /api/daily-status?trade_date=20260421&strategy=quality_momentum`
应返回整体 `blocked`，并在 `missing_fields` 中包含 `roe`。

**Acceptance Scenarios**:

1. **Given** `quality_momentum` 缺少 `roe` 和 `pe_ttm`，**When**
   操作者选择该策略，**Then** daily status 返回 `blocked`。
2. **Given** 有其他策略不可用，**When** dashboard 展示状态，
   **Then** 前端显示不可用策略和缺失字段。

---

### User Story 3: 查看计划、执行和备份审计状态 (Priority: P2)

作为操作者，我希望同一个状态中心也展示人工计划、执行报告和备份状态，
从而知道每日流程是否已经进入执行和审计阶段。

**Why this priority**:

复核之后的计划、执行和备份决定了交易审计是否完整。
它们不影响最小可复核状态，但影响每日流程闭环。

**Independent Test**:

给定当前交易日已经有人工计划、执行报告或新备份，
daily status API 应分别把对应步骤标记为 `done`，
并把整体状态推进到 `ready_for_execution` 或 `executed`。

**Acceptance Scenarios**:

1. **Given** 当前交易日有 confirmed manual trade plans，**When**
   daily status API 返回，**Then** `manual_trade_plans` 步骤为 `done`。
2. **Given** 当前交易日有 manual execution reports，**When**
   daily status API 返回，**Then** 整体状态为 `executed`。
3. **Given** 最新 `backups/stocks-*.sqlite` 不早于数据库文件，
   **When** 状态中心展示备份步骤，**Then** `backup` 为 `done`。
4. **Given** 没有备份或备份早于数据库，**When** 状态中心展示备份步骤，
   **Then** `backup` 为 `warning`。

## Edge Cases

- 没有任何行情数据：返回 `not_run`，所有步骤为 missing 或 warning。
- 有行情但没有因子：整体为 `not_run`，`factor_values` 为 missing。
- 当前策略未知：候选步骤为 missing，并显示 unknown strategy message。
- 当前策略缺字段：整体为 `blocked`，候选步骤为 `blocked`。
- 策略运行成功但候选数量为 0：候选步骤为 `warning`，不是 `blocked`。
- 没有候选复核记录：`candidate_review` 为 missing。
- 有复核记录但 decision 全是 `none`：仍视为未复核完成。
- 有人工计划但没有执行报告：整体为 `ready_for_execution`。
- 有执行报告：整体为 `executed`。
- 没有备份目录：备份步骤为 `warning`，不能报错。
- 备份文件存在但早于数据库：备份步骤为 `warning`。

## Functional Requirements

- **FR-001**: System MUST expose `GET /api/daily-status`.
- **FR-002**: API MUST accept optional `trade_date` and `strategy` query parameters.
- **FR-003**: If `trade_date` is omitted, API MUST use the latest known market trade date.
- **FR-004**: API MUST return the selected trade date, selected strategy, and Tushare points level.
- **FR-005**: API MUST return an overall status from this set:
  `not_run`, `blocked`, `ready_for_review`, `ready_for_planning`,
  `ready_for_execution`, `executed`.
- **FR-006**: API MUST return ordered step summaries for:
  `market_data`, `factor_values`, `candidate_selection`, `candidate_review`,
  `manual_trade_plans`, `manual_execution_reports`, `backup`.
- **FR-007**: Each step MUST include `key`, `label`, `status`, `count`, and `message`.
- **FR-008**: Candidate selection MUST reuse existing candidate API semantics.
- **FR-009**: Strategy data insufficiency MUST be represented as `blocked`
  with `missing_fields`.
- **FR-010**: API MUST report availability for supported strategies.
- **FR-011**: API MUST report backup freshness without creating backups.
- **FR-012**: Dashboard MUST fetch daily status using the current trade date and strategy.
- **FR-013**: Dashboard MUST render the overall daily status and step statuses.
- **FR-014**: Dashboard MUST render unavailable strategies and missing fields.
- **FR-015**: The feature MUST NOT mutate SQLite, reports, backups, logs, broker state,
  candidate reviews, trade plans, or execution reports.

## Non-Functional Requirements

- **NFR-001**: Status calculation MUST be deterministic for a fixed database and file state.
- **NFR-002**: API response shape MUST be stable enough to document in `api-contracts.md`.
- **NFR-003**: Missing data MUST produce structured status output rather than 500 errors.
- **NFR-004**: Frontend layout MUST remain readable on desktop and mobile widths.
- **NFR-005**: Tests MUST cover both ready and blocked branches.

## Key Entities

### DailyStatus

Represents the complete read-only status for one trade date and selected strategy.

Key attributes:

- `status`
- `trade_date`
- `strategy`
- `points_level`
- `selected_strategy`
- `available_strategies`
- `steps`

### DailyStatusStep

Represents one workflow checkpoint.

Key attributes:

- `key`
- `label`
- `status`
- `count`
- `message`
- optional `missing_fields`
- optional `latest_backup`

### StrategyAvailability

Represents whether a strategy has enough stored data to run.

Key attributes:

- `strategy`
- `available`
- `status`
- `candidate_count`
- `missing_fields`
- `message`

### BackupFreshness

Represents audit readiness for SQLite backup.

Key rule:

- Latest `backups/stocks-*.sqlite` is fresh when its mtime is not older than the active
  SQLite database file mtime.

## Success Criteria

- **SC-001**: With seeded ready data, API returns `ready_for_review`.
- **SC-002**: With missing financial fields for `quality_momentum`, API returns `blocked`.
- **SC-003**: Dashboard displays “每日状态” and then waits for loaded status such as “可复核”.
- **SC-004**: Backend API tests cover step statuses, strategy availability, and backup status.
- **SC-005**: Frontend test covers overall status, trade date, Tushare points, step message,
  and unavailable strategy missing fields.
- **SC-006**: Full backend pytest suite passes.
- **SC-007**: Full frontend Vitest suite passes.
- **SC-008**: Frontend production build passes.
- **SC-009**: Project Doc Impact check reports matching project docs and feature spec updates.

## Assumptions

- Existing SQLite repository methods are sufficient.
- Existing candidate API behavior is authoritative for strategy availability.
- No new persistence is needed because status can be derived from existing facts.
- Dashboard is the right first UI surface because it is the operator's overview.
- The first version only needs read-only status, not automatic remediation.
- The project continues to run verification from WSL.

## Out of Scope

- New SQLite tables or migrations.
- New strategy model.
- New broker integration.
- Live trading.
- Automatic task execution.
- Creating or repairing backups.
- Editing candidate reviews or trade plans from the status panel.
- Changing export contracts for manual execution reports.

## Reader Check

After reading only this spec, a reader should be able to answer:

- What is the user trying to decide?
- What are the seven status steps?
- Which states are allowed?
- Why is the endpoint read-only?
- Which tests prove the important branches?
- Which project docs need updates?

If any answer is unclear, the spec is not ready for planning.
