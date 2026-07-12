# 04. Tasks

## Purpose

This file shows how the plan should become executable work.
It is intentionally more granular than a normal summary so you can learn what a useful
`tasks.md` feels like.

Each task should be small enough for an agent to complete and verify without inventing scope.

## Task Rules

- Tests come before implementation.
- Backend and frontend tests are separate.
- Documentation impact is not a final afterthought.
- No task may require modifying Windows mirror code.
- No task may enable live trading or broker behavior.
- Every task should name files or verification evidence.

## Phase 1: Feature Spec and Context

- [X] T001 Read `AGENTS.md` in the source project.
- [X] T002 Read `docs/project/README.md`.
- [X] T003 Read `docs/project/invariants.md`.
- [X] T004 Read `docs/project/doc-map.yml`.
- [X] T005 Read relevant project memory:
  `workflows.md`, `api-contracts.md`, `data-model.md`, `verification.md`.
- [X] T006 Create `docs/features/2026-07-04-daily-run-status/spec.md`.
- [X] T007 Record scope, out of scope, business rules, API impact, test plan, and Doc Impact.

Acceptance:

- Feature spec clearly says the status center is read-only.
- Feature spec says no SQLite schema change.
- Feature spec names `workflows.md` and `api-contracts.md` as Project Doc Impact.

## Phase 2: Backend Tests First

- [X] T008 Add backend test for ready workflow status in `tests/test_web_api.py`.
- [X] T009 Seed market data and factor values using existing `_seed_basic_state`.
- [X] T010 Create a backup file under test workspace `backups/`.
- [X] T011 Assert API response includes:
  `status`, `trade_date`, `strategy`, `points_level`, `steps`, and `available_strategies`.
- [X] T012 Assert ready path:
  `market_data`, `factor_values`, and `candidate_selection` are `done`.
- [X] T013 Assert incomplete downstream steps:
  `candidate_review`, `manual_trade_plans`, and `manual_execution_reports` are `missing`.
- [X] T014 Assert backup is `done` when latest backup is fresh.
- [X] T015 Add backend test for blocked strategy due to missing fields.
- [X] T016 Assert `quality_momentum` returns `blocked` and includes missing `roe`.

Acceptance:

- Running the new backend tests before implementation fails because `/api/daily-status`
  or expected fields do not exist.
- After implementation, both tests pass.

## Phase 3: Frontend Test First

- [X] T017 Add daily status mock payload in `webapp/src/App.test.jsx`.
- [X] T018 Add dashboard test for daily status rendering.
- [X] T019 Assert visible loaded state:
  `可复核`, `交易日 20260421`, `Tushare 120 点`.
- [X] T020 Assert at least one step message such as `2 条日线记录`.
- [X] T021 Assert unavailable strategy message such as
  `质量动量 缺少 roe, pe_ttm`.
- [X] T022 Use `waitFor` on loaded status text, not only on static panel title.

Acceptance:

- Test fails before implementation because dashboard does not fetch or render daily status.
- Test passes after frontend implementation.

## Phase 4: Backend Implementation

- [X] T023 Add `get_daily_status(settings, strategy_name, trade_date=None)`.
- [X] T024 Resolve latest trade date when query omits `trade_date`.
- [X] T025 Read provider `points_level`.
- [X] T026 Read market data with `repo.load_daily_bars`.
- [X] T027 Read factor values with `repo.load_factor_values`.
- [X] T028 Call `get_candidates` for each supported strategy.
- [X] T029 Read candidate reviews.
- [X] T030 Read manual trade plans.
- [X] T031 Read manual execution reports.
- [X] T032 Add helper for empty daily steps.
- [X] T033 Add helper for count-based steps.
- [X] T034 Add helper for candidate step, including `insufficient_data`.
- [X] T035 Add helper for reviewed count excluding `none`.
- [X] T036 Add helper for backup freshness.
- [X] T037 Add helper for strategy availability.
- [X] T038 Add helper for deriving overall status.
- [X] T039 Add route `GET /api/daily-status`.

Acceptance:

- No write method is called.
- No broker adapter is imported for daily status.
- Unknown or missing data returns structured payload rather than crashing.

## Phase 5: Frontend Implementation

- [X] T040 Add `dailyStatus` state to `App.jsx`.
- [X] T041 Fetch `/api/daily-status${query}` inside `refreshConsole`.
- [X] T042 Pass `dailyStatus` to `DashboardView`.
- [X] T043 Add `DailyStatusPanel`.
- [X] T044 Add display helper for overall daily status.
- [X] T045 Add display helper for step status.
- [X] T046 Add tone helper for overall status.
- [X] T047 Add tone helper for step status.
- [X] T048 Render unavailable strategies and missing fields.
- [X] T049 Add CSS for panel, status head, step grid, steps, availability tags.
- [X] T050 Add responsive CSS for narrower screens.

Acceptance:

- Dashboard still renders existing metrics.
- Daily status panel appears below summary metrics.
- The panel displays initial empty state safely before API data loads.
- The test waits for loaded data, preventing false success.

## Phase 6: Project Memory Updates

- [X] T051 Update `docs/project/workflows.md`.
- [X] T052 Add Daily Status Review section.
- [X] T053 State that dashboard status is read-only.
- [X] T054 List summarized workflow steps.
- [X] T055 Update `docs/project/api-contracts.md`.
- [X] T056 Document `/api/daily-status` endpoint.
- [X] T057 Document stable response fields.
- [X] T058 Document stable step keys.
- [X] T059 State endpoint must not mutate state.

Acceptance:

- `scripts/check_project_docs.py --base origin/master` reports matching project docs.
- Final report can name both Feature Spec Impact and Project Doc Impact.

## Phase 7: Verification

- [X] T060 Run targeted backend tests:
  `pytest tests/test_web_api.py::test_daily_status_api_summarizes_workflow_readiness
  tests/test_web_api.py::test_daily_status_api_reports_blocked_strategy_when_required_fields_are_missing -q`.
- [X] T061 Run web API tests:
  `pytest tests/test_web_api.py -q`.
- [X] T062 Run full backend suite:
  `pytest`.
- [X] T063 Run full frontend tests:
  `cd webapp && npm run test -- --run`.
- [X] T064 Run frontend build:
  `cd webapp && npm run build`.
- [X] T065 Run doc impact check:
  `python scripts/check_project_docs.py --base origin/master`.
- [X] T066 Run whitespace check:
  `git diff --check`.

Acceptance:

- All verification commands exit 0.
- Any failure is debugged from root cause before declaring completion.

## Phase 8: Commit, Push, and Mirror Sync

- [X] T067 Inspect `git status -sb`.
- [X] T068 Stage only the 9 intended files.
- [X] T069 Commit with message `Add daily run status dashboard`.
- [X] T070 Push WSL `master` to `origin/master`.
- [X] T071 Check Windows mirror status.
- [X] T072 If Windows mirror is clean, run `git pull --ff-only origin master`.
- [X] T073 Confirm WSL and Windows mirror both point to `9cc1b86`.

Acceptance:

- WSL repo is clean and aligned with remote.
- Windows mirror is fast-forwarded.
- No unrelated files are staged or committed.

## Parallelization Notes

Safe to run in parallel:

- Full backend pytest and frontend Vitest.
- Frontend build, doc impact check, and `git diff --check` after tests pass.

Not safe to run in parallel:

- Editing backend service and tests if both touch same file.
- Commit/push before verification finishes.
- Windows mirror pull before WSL push succeeds.

## What This Task List Teaches

A useful task list is not just a checklist of files.
It encodes the order of confidence:

1. Understand project memory.
2. Write tests.
3. Implement the narrow behavior.
4. Update project memory.
5. Verify broadly.
6. Publish safely.

If a task list starts with “implement service” before tests and context, it is probably too shallow.
