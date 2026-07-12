# 03. Implementation Plan

## Summary

Implement a read-only daily run status flow for the stock screener console.
The backend adds `GET /api/daily-status`, derives workflow readiness from existing
SQLite tables, existing candidate service behavior, provider config, and backup files.
The frontend fetches the endpoint on dashboard refresh and renders a compact status panel.

No schema migration, strategy change, broker change, or live trading behavior is allowed.

## Technical Context

| Area | Existing technology |
| --- | --- |
| Backend API | FastAPI |
| Backend service | Python service helpers in `screener/web/service.py` |
| Storage | SQLite via `StockRepository` |
| Frontend | React in `webapp/src/App.jsx` |
| Frontend tests | Vitest + Testing Library |
| Backend tests | pytest + FastAPI `TestClient` |
| Documentation discipline | `docs/features/`, `docs/project/`, `scripts/check_project_docs.py` |

## Constitution and Invariant Check

This feature is not inside the Spec Kit codebase, but the same planning idea applies:
evaluate long-lived rules before implementation.

| Rule | Plan decision |
| --- | --- |
| WSL main repo is source of truth | Implement and verify in WSL |
| SQLite remains local state source | Derive status from repository reads |
| Strategy and broker stay decoupled | Do not import broker adapters for status |
| Live trading disabled | No live execution path added |
| Manual execution auditability preserved | Read `manual_execution_reports`, do not overwrite |
| API routes must not be shadowed | Add explicit `/api/daily-status` before frontend catch-all |
| Doc Impact required | Update `workflows.md` and `api-contracts.md` |

## Architecture

The feature has four pieces.

### 1. Backend service aggregation

Add `get_daily_status(settings, strategy_name, trade_date=None)` to
`screener/web/service.py`.

Responsibilities:

- Resolve trade date.
- Read Tushare provider points level.
- Read daily bars for market data readiness.
- Read factor values for factor readiness.
- Call existing `get_candidates()` for every supported strategy.
- Read candidate reviews for current trade date and strategy.
- Read manual trade plans for current trade date and strategy.
- Read manual execution reports for current trade date and strategy.
- Inspect latest `backups/stocks-*.sqlite`.
- Return a stable JSON-friendly dictionary.

This function must not mutate any state.

### 2. Backend route

Add route to `screener/web/api.py`:

```text
GET /api/daily-status?trade_date=YYYYMMDD&strategy=price_momentum
```

Route responsibilities:

- Accept query parameters.
- Delegate to service.
- Return service payload directly.

No custom response class is needed because this is JSON.

### 3. Frontend dashboard state

Add `dailyStatus` state to `webapp/src/App.jsx`.

During `refreshConsole()`:

- Build the same query used by candidates and plans.
- Fetch `/api/daily-status`.
- Store result in `dailyStatus`.
- Pass it to `DashboardView`.

Render a `DailyStatusPanel` inside dashboard.

### 4. Documentation updates

Update:

- `docs/features/2026-07-04-daily-run-status/spec.md`
- `docs/project/workflows.md`
- `docs/project/api-contracts.md`

## Data Flow

```text
Dashboard refresh
  |
  v
GET /api/daily-status?trade_date=...&strategy=...
  |
  v
get_daily_status(settings, strategy, trade_date)
  |
  +-- StockRepository.load_daily_bars()
  +-- StockRepository.load_factor_values()
  +-- get_candidates() for supported strategies
  +-- StockRepository.load_candidate_reviews()
  +-- StockRepository.load_manual_trade_plans()
  +-- StockRepository.load_manual_execution_reports()
  +-- backups/stocks-*.sqlite mtime check
  |
  v
DailyStatus JSON
  |
  v
DailyStatusPanel
```

## Status Derivation

Overall status is derived from steps:

| Condition | Overall status |
| --- | --- |
| Missing market data or factors | `not_run` |
| Selected candidate step blocked | `blocked` |
| Execution reports exist | `executed` |
| Manual trade plans exist | `ready_for_execution` |
| Reviewed candidate decisions exist | `ready_for_planning` |
| Candidate selection done or warning | `ready_for_review` |
| None of the above | `not_run` |

This order matters. Execution is later than planning, and planning is later than review.

## Step Semantics

| Step | Source | Done condition |
| --- | --- | --- |
| `market_data` | `daily_bars` | Count > 0 |
| `factor_values` | `factor_values` | Count > 0 |
| `candidate_selection` | existing candidate service | `succeeded` |
| `candidate_review` | `candidate_reviews` | decision count excluding `none` > 0 |
| `manual_trade_plans` | `manual_trade_plans` | Count > 0 |
| `manual_execution_reports` | `manual_execution_reports` | Count > 0 |
| `backup` | `backups/stocks-*.sqlite` | latest backup mtime >= database mtime |

## Test Strategy

Use TDD.

Backend tests first:

1. Ready path:
   - seed market data and factor values.
   - create a fresh backup file.
   - call `/api/daily-status` for `price_momentum`.
   - assert `ready_for_review`, step statuses, counts, and strategy availability.
2. Blocked path:
   - use same seed but request `quality_momentum`.
   - assert `blocked`, selected strategy unavailable, and missing `roe`.

Frontend test:

- mock `/api/daily-status?trade_date=20260421&strategy=price_momentum`.
- render dashboard.
- wait for loaded status text, not just panel title.
- assert status, trade date, Tushare points, one step message, and unavailable strategy text.

## Implementation Order

1. Create feature spec.
2. Add failing backend tests.
3. Add failing frontend test.
4. Implement backend service helper.
5. Add API route.
6. Add frontend state fetch.
7. Add dashboard panel.
8. Add responsive CSS.
9. Update project memory docs.
10. Run verification.
11. Commit and push.
12. Sync Windows mirror.

## Risks and Mitigations

| Risk | Mitigation |
| --- | --- |
| Endpoint accidentally mutates state | Keep implementation read-only and test through GET |
| Strategy missing fields hidden | Assert `missing_fields` in API tests |
| Frontend test passes too early on static title | Wait for loaded status text |
| Backup freshness flaky in tests | Explicitly set backup mtime with `os.utime` |
| New API contract undocumented | Update `docs/project/api-contracts.md` |
| Workflow change undocumented | Update `docs/project/workflows.md` |
| Mobile layout cramped | Add responsive grid rules |

## Alternatives Considered

### Alternative A: Frontend computes status from existing APIs

Rejected.

Reason:

- Duplicates business logic in frontend.
- Makes status semantics harder to test.
- Forces frontend to know table-level concepts.
- Makes API contract weaker.

### Alternative B: Persist daily status in a new SQLite table

Rejected.

Reason:

- The status is derivable from existing facts.
- A new table introduces cache invalidation questions.
- It violates the first-version constraint of no schema change.

### Alternative C: Add buttons to run missing steps

Rejected for this phase.

Reason:

- It changes the feature from read-only review to orchestration.
- It increases risk around trading and execution flows.
- It needs a separate spec.

## Expected File Changes

| File | Change |
| --- | --- |
| `docs/features/2026-07-04-daily-run-status/spec.md` | New feature spec |
| `screener/web/service.py` | Add daily status aggregation |
| `screener/web/api.py` | Add API route |
| `tests/test_web_api.py` | Add backend API tests |
| `webapp/src/App.jsx` | Add state, fetch, panel |
| `webapp/src/App.test.jsx` | Add frontend test |
| `webapp/src/styles.css` | Add status panel styles |
| `docs/project/workflows.md` | Document daily status workflow |
| `docs/project/api-contracts.md` | Document API contract |

## Completion Definition

The feature is complete only when:

- API returns stable status payload.
- Dashboard renders loaded status.
- Tests and build pass.
- Project docs and feature spec are updated.
- WSL commit is pushed.
- Windows mirror is fast-forwarded after push.
