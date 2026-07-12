# 05. Implementation Notes

## Purpose

This file explains what the real commit changed and why those changes satisfy the spec.
It is not a full code listing. It is a reading guide for the implementation.

Real commit:

```text
9cc1b86 Add daily run status dashboard
```

Commit summary:

```text
9 files changed, 604 insertions(+), 1 deletion(-)
```

## Changed Files

| File | Purpose |
| --- | --- |
| `docs/features/2026-07-04-daily-run-status/spec.md` | Feature spec |
| `docs/project/api-contracts.md` | Long-lived API contract |
| `docs/project/workflows.md` | Long-lived workflow memory |
| `screener/web/api.py` | FastAPI route |
| `screener/web/service.py` | Status aggregation logic |
| `tests/test_web_api.py` | Backend tests |
| `webapp/src/App.jsx` | Frontend state and component |
| `webapp/src/App.test.jsx` | Frontend test |
| `webapp/src/styles.css` | Frontend layout and responsive styles |

## Backend Route

`screener/web/api.py` adds:

```python
@app.get("/api/daily-status")
def daily_status(trade_date: str | None = None, strategy: str = "price_momentum") -> dict[str, Any]:
    return get_daily_status(app_settings, strategy, trade_date)
```

Why this is correct:

- It is explicit and placed before the frontend catch-all route.
- It accepts the two query parameters defined in the spec.
- It delegates all business logic to the service layer.
- It returns JSON without side effects.

## Backend Service

`screener/web/service.py` adds `get_daily_status`.

The function does four kinds of work:

1. Resolve context:
   - selected or latest trade date.
   - Tushare points level.
   - selected strategy.
2. Load facts:
   - daily bars.
   - factor values.
   - candidate states.
   - candidate reviews.
   - manual trade plans.
   - manual execution reports.
   - backup file metadata.
3. Convert facts into step summaries.
4. Derive overall status from step summaries.

The service intentionally reuses `get_candidates`.
That decision prevents a second implementation of strategy availability logic.

## Helper Functions

The commit adds small helper functions near the bottom of `service.py`.

| Helper | Responsibility |
| --- | --- |
| `_empty_daily_steps` | Return stable empty payload when no trade date exists |
| `_load_frame` | Convert repository read errors into empty data frames |
| `_count_step` | Convert count-based facts into done or missing steps |
| `_candidate_step` | Convert candidate API state into daily status step |
| `_reviewed_count` | Count non-`none` candidate decisions |
| `_backup_step` | Evaluate backup freshness |
| `_strategy_availability` | Normalize strategy availability response |
| `_daily_status` | Derive overall status |

This keeps `get_daily_status` readable.

## Backup Freshness Decision

Backup freshness is evaluated by comparing mtimes:

```text
latest backups/stocks-*.sqlite mtime >= active SQLite database mtime
```

Why this is acceptable for v1:

- No new table is needed.
- It uses the existing backup artifact convention.
- It answers the operational question: is there a backup at least as new as the current DB?

Why the test uses `os.utime`:

- Filesystem timestamp precision can vary.
- In a full test run, a backup created immediately after DB writes may not reliably appear
  newer on every filesystem.
- The test should prove the rule, not depend on scheduling timing.

## Overall Status Derivation

The implementation uses ordered precedence:

1. Missing data or factors -> `not_run`.
2. Candidate selection blocked -> `blocked`.
3. Execution reports exist -> `executed`.
4. Manual trade plans exist -> `ready_for_execution`.
5. Candidate reviews exist -> `ready_for_planning`.
6. Candidate selection done or warning -> `ready_for_review`.
7. Otherwise -> `not_run`.

This order prevents a later workflow step from being hidden by an earlier incomplete optional step.

## Frontend State

`webapp/src/App.jsx` adds:

- `dailyStatus` state.
- API fetch inside `refreshConsole`.
- `DailyStatusPanel`.
- display helpers for overall status and step status.
- tone helpers for CSS classes.

Important detail:

The dashboard can render before daily status has loaded.
Therefore the component must safely handle:

```text
{ status: "not_run", steps: [], available_strategies: [] }
```

## Frontend Test Lesson

The first version of the frontend test waited for the text `每日状态`.
That was insufficient because the panel title exists even in the initial empty state.

The corrected test waits for `可复核`, which only appears after the mocked
daily status response is fetched and rendered.

This is an important testing lesson:

- Static shell text proves the component mounted.
- Loaded data text proves the behavior worked.

## Styling

`webapp/src/styles.css` adds:

- `.daily-status-panel`
- `.daily-status-head`
- `.daily-step-grid`
- `.daily-step`
- `.daily-availability`
- tone border classes.

The first grid used seven columns on desktop.
Responsive adjustments were added:

- 4 columns under 1200px.
- 1 column under 900px.
- status head switches to grid layout on narrow screens.

This matters because status labels and Chinese text can become cramped on smaller screens.

## Documentation Implementation

The feature spec was added before implementation.
Long-term docs were updated after implementation:

- `workflows.md` records Daily Status Review as a stable operator workflow.
- `api-contracts.md` records endpoint fields, status values, step keys, and read-only rule.

This separation teaches a key rule:

- Feature spec records this feature's decision history.
- Project docs record stable rules future features must know.

## What Was Not Changed

The commit does not change:

- SQLite schema.
- strategy YAML files.
- factor construction.
- manual execution report persistence.
- broker adapters.
- live trading behavior.
- export payloads or filenames.

These non-changes are part of the implementation quality.
They show the feature stayed inside its intended scope.

## How to Read the Real Commit

Suggested order:

1. Read `docs/features/2026-07-04-daily-run-status/spec.md`.
2. Read the new tests in `tests/test_web_api.py`.
3. Read `get_daily_status` in `screener/web/service.py`.
4. Read the new route in `screener/web/api.py`.
5. Read the new frontend test in `webapp/src/App.test.jsx`.
6. Read `DailyStatusPanel` in `webapp/src/App.jsx`.
7. Read the doc updates.

Do not start with CSS. CSS is useful but it does not explain the business behavior.
