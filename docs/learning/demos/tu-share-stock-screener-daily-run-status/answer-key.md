# Answer Key

## How to Use This File

Read this only after you have worked through the demo.
The goal is not memorization. The goal is to check whether your reasoning matches the
spec-driven flow.

## Core Questions

### 1. Why did this feature need a feature spec?

Because it changed more than implementation details.

It affected:

- user-facing dashboard workflow.
- backend API contract.
- frontend rendering.
- test plan.
- long-term project memory.
- audit-related status interpretation.

A small bugfix might not need a feature spec. This feature does, because future agents need
a stable explanation of why the endpoint is read-only, which steps it summarizes, and what
it must not do.

### 2. What belongs in the feature spec?

Feature-specific decisions:

- the goal: one trusted place to see daily readiness.
- the included steps.
- the out-of-scope boundaries.
- the API path.
- response shape summary.
- test plan.
- expected project memory updates.

The feature spec answers:

> What did we decide for this change?

### 3. What belongs in project memory?

Stable future rules:

- dashboard daily status is a read-only workflow view.
- `/api/daily-status` has stable response fields and step keys.
- the endpoint must not mutate state.

Project memory answers:

> What should the next agent know before changing this area?

### 4. Why must the endpoint be read-only?

Because the feature is an observation surface, not an orchestration surface.

If a GET status endpoint starts mutating reviews, plans, execution reports, backups, or broker
state, it becomes unsafe and surprising. It would also cross trading and audit boundaries.

### 5. Why was no new SQLite table added?

The status can be derived from existing facts:

- market data from `daily_bars`.
- factors from `factor_values`.
- strategy readiness from existing candidate logic.
- reviews from `candidate_reviews`.
- plans from `manual_trade_plans`.
- execution from `manual_execution_reports`.
- backup freshness from backup files.

A persisted status table would introduce cache invalidation and source-of-truth questions
without solving a v1 need.

### 6. Why reuse `get_candidates` for strategy availability?

Because candidate API already defines strategy readiness semantics.

Reimplementing strategy availability inside daily status would create two sources of truth.
If strategy requirements changed later, one implementation could drift.

### 7. Why is `quality_momentum` blocked in the test?

The seed data includes price momentum fields but not the financial fields that
`quality_momentum` requires, such as `roe` and `pe_ttm`.

The correct behavior is not “success with no candidates”.
The correct behavior is “blocked because required fields are missing”.

### 8. Why did the frontend test wait for `可复核`?

Because `每日状态` is static panel title text.
It appears before the API response is loaded.

Waiting for `可复核` proves that:

- the daily status API mock was requested.
- the response reached component state.
- loaded status was rendered.

### 9. Why did backup test need `os.utime`?

The business rule is mtime-based:

```text
latest backup mtime >= database mtime
```

If the test creates the backup immediately after database writes, file timestamp precision
can make the ordering unreliable. `os.utime` makes the fixture deterministic.

### 10. Why did `workflows.md` change?

Because the feature added a stable operator workflow view:

```text
Daily Status Review
```

This affects how future agents understand daily operations.

### 11. Why did `api-contracts.md` change?

Because the feature added a stable API:

```text
GET /api/daily-status
```

The response fields, statuses, step keys, and read-only rule are now contract-level facts.

### 12. Why did `data-model.md` not change?

No table, stored field, or source-of-truth meaning changed.
The feature reads existing state and derives a response.

### 13. Why did `invariants.md` not change?

The feature obeyed existing invariants but did not create a new global invariant.
The read-only rule was important, but documenting it in workflow and API contract docs was
sufficient for this phase.

### 14. What proves completion?

Completion is proven by evidence, not by confidence.

Evidence included:

- targeted backend tests passed.
- `tests/test_web_api.py` passed.
- full pytest passed.
- frontend Vitest passed.
- frontend build passed.
- doc impact check passed.
- `git diff --check` passed.
- commit pushed to remote.
- Windows mirror fast-forwarded.

### 15. What would be different in a formal Spec Kit feature directory?

This learning demo would likely become:

```text
specs/001-daily-run-status/
  spec.md
  plan.md
  tasks.md
  quickstart.md
  contracts/
    daily-status.md
```

The current files are educational reconstructions, not a real Specify-generated feature.

## Common Mistakes

### Mistake 1: Treating the panel as the feature

The feature is not the visual card.
The feature is the operational status contract.

The card is just one consumer.

### Mistake 2: Writing the API before defining status semantics

If you do not define status precedence first, implementation can become inconsistent:

- Is no candidate `blocked` or `warning`?
- Does execution override missing review?
- Does stale backup block status?

The spec and plan should answer these before coding.

### Mistake 3: Updating only feature spec

Feature spec is not enough.
Because this introduced a stable workflow and API, long-term project memory also needed
updates.

### Mistake 4: Using tests that only check existence

A test that only sees “每日状态” is weak.
It proves the shell mounted, not that the data loaded.

Good tests assert behavior:

- overall status.
- trade date.
- points level.
- step message.
- missing fields.

### Mistake 5: Making status mutating because it is convenient

A status endpoint should not create missing data, run jobs, or update audit rows.
Those actions need explicit commands, permissions, tests, and probably a separate spec.

## Self-Review Rubric

When you write your own spec, grade it against this rubric.

| Area | Strong answer |
| --- | --- |
| Goal | Names the user decision, not only the UI |
| Scope | Includes exact workflow steps |
| Out of scope | Explicitly excludes mutation and live trading |
| API | Names endpoint, query parameters, status enum |
| Data | Explains existing sources of truth |
| Tests | Covers ready and blocked paths |
| Frontend | Verifies loaded data, not static shell |
| Docs | Separates feature spec from project memory |
| Verification | Lists commands and what each proves |

## Practice Prompt

Use this prompt after studying:

```text
I want to add a button that reruns missing daily status steps.
Write a feature spec, plan, and task list.
Do not implement yet.
Respect the existing daily status read-only contract unless your spec explicitly changes it.
```

Expected conclusion:

- This is a new feature, not a small extension.
- It changes the read-only contract.
- It probably needs new API routes.
- It may interact with scheduler or action endpoints.
- It requires careful invariants review.
- It should not be bundled into the original daily status spec silently.
