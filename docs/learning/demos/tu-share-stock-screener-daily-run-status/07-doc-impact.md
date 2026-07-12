# 07. Doc Impact

## Purpose

This file explains how to decide which documentation to update.
It is the part most likely to be skipped when using an AI agent, and that is exactly why
it matters.

## Two Kinds of Documentation

This project uses two documentation layers.

### Feature Spec Impact

Feature spec documents the current feature:

```text
docs/features/2026-07-04-daily-run-status/spec.md
```

It records:

- goal.
- scope.
- out of scope.
- business rules.
- API impact.
- test plan.
- expected project memory updates.

This is short-term feature memory. It answers:

> What did we decide for this feature?

### Project Doc Impact

Project docs record stable long-term memory:

```text
docs/project/workflows.md
docs/project/api-contracts.md
```

They answer:

> What should future agents know before changing related behavior?

## Why `workflows.md` Changed

The feature added a new stable operator workflow:

> Dashboard daily status review.

That belongs in `workflows.md` because it changes how the operator understands the daily
screening flow.

The update records:

- Daily status is read-only.
- It does not run jobs or mutate SQLite.
- It summarizes market data, factors, candidates, reviews, trade plans, execution reports,
  backup freshness, Tushare points, and strategy availability.
- Operators still use existing review, planning, execution, and export workflows to change state.

Without this update, a future agent might treat the status panel as a control surface and add
mutation behavior in the wrong place.

## Why `api-contracts.md` Changed

The feature added a stable endpoint:

```text
GET /api/daily-status?trade_date=YYYYMMDD&strategy=price_momentum
```

That belongs in `api-contracts.md` because response shape and status semantics are now
part of the user-facing frontend/backend contract.

The update records:

- endpoint purpose.
- omitted `trade_date` behavior.
- stable response fields.
- allowed overall statuses.
- stable step keys.
- read-only invariant.

Without this update, future API changes could rename fields, reorder step semantics, or
accidentally add side effects without noticing.

## Why `data-model.md` Did Not Change

No SQLite table or stored field meaning changed.

The feature reads existing state:

- `daily_bars`
- `factor_values`
- `candidate_reviews`
- `manual_trade_plans`
- `manual_execution_reports`

It does not introduce new source-of-truth semantics.
Therefore `data-model.md` does not need an update.

If a future version persisted a `daily_status_runs` table, then `data-model.md` would need
an update.

## Why `invariants.md` Did Not Change

The feature obeys existing invariants but does not create a new invariant.

Existing rules already cover:

- strategy and broker decoupling.
- live trading disabled.
- execution history auditability.
- WSL source of truth.

If a future version added a stronger rule such as “daily status endpoint must always remain
read-only”, it could be promoted to `invariants.md`. In this phase, documenting it in
`api-contracts.md` and `workflows.md` is enough.

## Why `verification.md` Did Not Change

The verification commands did not change.

The feature used existing commands:

- `pytest`
- `npm run test -- --run`
- `npm run build`
- `scripts/check_project_docs.py`
- `git diff --check`

No new validation tool or standard command was introduced.

## How `doc-map.yml` Helped

The code changed these areas:

- `screener/web/**`
- `webapp/**`

The doc map points those areas to:

- `docs/project/api-contracts.md`
- `docs/project/workflows.md`

The script output confirmed matching docs were updated.

This is why local reminders are useful: they do not replace judgment, but they keep
obvious documentation targets visible.

## Final Doc Impact Report

The final report should say:

```text
Feature Spec Impact:
- Added docs/features/2026-07-04-daily-run-status/spec.md.

Project Doc Impact:
- Updated docs/project/workflows.md for the Daily Status Review workflow.
- Updated docs/project/api-contracts.md for GET /api/daily-status.
```

It should not say only:

```text
Docs updated.
```

That is too vague. A future reader needs to know which memory layer changed and why.

## Future Update Rules

Update `docs/features/.../spec.md` when:

- the current feature scope changes.
- an implementation decision changes during the feature.
- the test plan changes before completion.

Update `docs/project/workflows.md` when:

- the daily operating sequence changes.
- the user-facing meaning of a workflow step changes.
- the status panel gains mutation behavior.

Update `docs/project/api-contracts.md` when:

- endpoint path changes.
- query parameters change.
- response fields change.
- status enum changes.
- step keys change.
- read-only behavior changes.

Update `docs/project/data-model.md` when:

- new tables are added.
- stored field meanings change.
- source-of-truth rules change.

Update `docs/project/invariants.md` when:

- a rule becomes globally binding across future features.
- violating the rule would create trading, audit, or safety risk.

## Reader Exercise

Given a future request:

> Add a button to rerun missing daily status steps.

Which docs should change?

Expected reasoning:

- New feature spec is required.
- `workflows.md` must change because status panel becomes an action surface.
- `api-contracts.md` must change if new API routes are added.
- `invariants.md` may need review because mutation and trading boundaries become riskier.
- `data-model.md` changes only if new persisted state is introduced.
