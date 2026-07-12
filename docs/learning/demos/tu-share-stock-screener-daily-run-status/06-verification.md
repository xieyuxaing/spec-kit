# 06. Verification

## Purpose

This file explains what evidence proved the feature was complete.
Verification is not “I looked at the code and it seems right”.
Verification is a set of commands and assertions that cover the requirements.

## Commands Run

All commands were run from WSL source repository:

```text
/home/xieyx/projects/tu-share-stock-screener
```

### Targeted backend tests

```bash
. .venv/bin/activate && pytest \
  tests/test_web_api.py::test_daily_status_api_summarizes_workflow_readiness \
  tests/test_web_api.py::test_daily_status_api_reports_blocked_strategy_when_required_fields_are_missing \
  -q
```

Result:

```text
2 passed
```

What this proves:

- Ready path returns expected workflow steps.
- Blocked strategy path returns missing fields.
- API route and service integrate correctly.

What this does not prove:

- It does not prove the full backend suite still passes.
- It does not prove frontend rendering.

### Web API tests

```bash
. .venv/bin/activate && pytest tests/test_web_api.py -q
```

Result:

```text
49 passed
```

What this proves:

- New API behavior did not break existing web API tests.
- Route ordering still works for nearby API surfaces.

### Full backend suite

```bash
. .venv/bin/activate && pytest
```

Result:

```text
151 passed
```

What this proves:

- Backend, storage, trading, operations, docs check tests, and web API tests pass together.
- The new feature did not regress unrelated backend behavior covered by the test suite.

### Full frontend test suite

```bash
cd webapp && npm run test -- --run
```

Result:

```text
23 passed
```

What this proves:

- Dashboard daily status rendering works with mocked API data.
- Existing frontend behavior still passes.

### Frontend production build

```bash
cd webapp && npm run build
```

Result:

```text
vite build completed successfully
```

What this proves:

- The React code compiles for production.
- CSS and imports do not break the build.

### Project doc impact check

```bash
. .venv/bin/activate && python scripts/check_project_docs.py --base origin/master
```

Result:

```text
Project-doc updates in this change:
- docs/project/api-contracts.md
- docs/project/workflows.md

Feature-spec updates in this change:
- docs/features/2026-07-04-daily-run-status/spec.md
```

What this proves:

- Mapped backend and frontend code changes triggered relevant project memory docs.
- The change included both feature spec and project doc updates.

### Whitespace check

```bash
git diff --check
```

Result:

```text
exit 0
```

What this proves:

- No obvious trailing whitespace or diff whitespace errors.

## Debugging Evidence

During verification, `tests/test_web_api.py -q` initially failed:

```text
assert steps["backup"]["status"] == "done"
E AssertionError: assert 'warning' == 'done'
```

Root cause:

- The test created a backup file immediately after SQLite writes.
- In a larger test run, filesystem timestamp precision and scheduling made the backup mtime
  not reliably newer than the database mtime.
- The production rule was correct, but the test fixture did not prove it deterministically.

Fix:

```python
backup_mtime = settings.database_path.stat().st_mtime + 5
os.utime(backup_path, (backup_mtime, backup_mtime))
```

Why this is the right fix:

- It preserves the business rule.
- It makes the test explicitly model a fresh backup.
- It removes test timing flakiness.

## Requirement Coverage Matrix

| Requirement | Evidence |
| --- | --- |
| API exists | backend tests call `/api/daily-status` |
| Readiness steps exist | backend ready test asserts step keys and statuses |
| Strategy missing fields visible | backend blocked test asserts `roe` |
| Dashboard renders status | frontend test waits for `可复核` |
| Tushare points visible | frontend test asserts `Tushare 120 点` |
| Unavailable strategy visible | frontend test asserts missing fields text |
| No schema change | commit has no migration or data model file change |
| API contract documented | `docs/project/api-contracts.md` updated |
| Workflow documented | `docs/project/workflows.md` updated |
| Full backend not regressed | `pytest`: 151 passed |
| Full frontend not regressed | Vitest: 23 passed |
| Production frontend compiles | `npm run build` passed |
| Doc Impact checked | `check_project_docs.py` passed |

## Verification Gaps

Known gaps that are acceptable for this phase:

- No browser screenshot was captured.
- No live dev server manual inspection was recorded.
- No visual regression test exists.
- No performance benchmark exists for daily status aggregation.

Why acceptable:

- This is a local operational dashboard feature.
- The main behavioral risk is API semantics and correct status rendering.
- Those are covered by backend and frontend automated tests.
- The feature does not introduce heavy computation or new persistence.

## Completion Evidence

Final publish evidence:

```text
Commit: 9cc1b86 Add daily run status dashboard
Remote: origin/master
WSL HEAD: 9cc1b86
Windows mirror HEAD after fast-forward pull: 9cc1b86
```

This proves the implementation was not just local. It was pushed and mirrored according to
the project workflow.

## Reader Exercise

Before looking at `answer-key.md`, answer:

1. Which command proves the backend route works?
2. Which command proves the frontend builds?
3. Which test originally failed and why?
4. Why is `git diff --check` not enough for this feature?
5. What evidence proves the docs were updated for the correct code areas?
