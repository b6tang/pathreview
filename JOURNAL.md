## Week 7 — Issue selection

**Issue link:** [Issue #154](https://github.com/ascherj/pathreview/issues/154)

**Issue title:** Health check DB probe passes a raw SQL string, which fails under SQLAlchemy 2.x

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Selection reasoning:**
This Tier 1 issue is a good fit for me because I already ran the application and reproduced the health endpoint error. The issue has a clear and small scope because it only affects the PostgreSQL check in `api/routes/health.py`. I also searched the codebase and found that the raw `"SELECT 1"` query related to this issue is only used in this health check. I can focus on one specific bug without changing many files or services.

**Problem summary:**
The health endpoint directly passes the string `"SELECT 1"` to `AsyncSession.execute()` when checking the PostgreSQL database in `api/routes/health.py`. In SQLAlchemy 2.x, `AsyncSession.execute()` cannot directly run a normal SQL string, so the database check fails and reports PostgreSQL as unhealthy. The fix is to wrap `"SELECT 1"` with `sqlalchemy.text()` before executing it. After the fix, the PostgreSQL check should report healthy when the database is available.

**Branch name:** `fix/154-health-check-db-probe`

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger


## Week 8 — Reproduction & solution planning

**Reproduction commit link:** [c0f2fed — Document Issue #154 reproduction](https://github.com/b6tang/pathreview/commit/c0f2fedc368696bfc6d72013043465c75890c5a7)

**Reproduction summary:**
With the Docker services `db`, `redis`, and `vector-db` running, and the FastAPI application running with Uvicorn, I ran `curl.exe -i http://localhost:8000/health` in PowerShell. The endpoint returned HTTP 503 and showed PostgreSQL as unhealthy, while the backend log showed `Textual SQL expression 'SELECT 1' should be explicitly declared as text('SELECT 1')`, which matches the problem in Issue #154.

**PLAN.md link:** [Solution plan for Issue #154](https://github.com/b6tang/pathreview/blob/30fc8b90cc103113f8e94e2bf3477e265704ebf2/PLAN.md)

**Walkthrough video (recommended):** None.

**Blockers or open questions:** None.


## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
I completed all four implementation steps from my PLAN.md. I updated `api/routes/health.py` to wrap `SELECT 1` with SQLAlchemy `text()`, and I added a focused regression test in `tests/unit/test_health.py`. The test checks that `db.execute()` receives a `TextClause` containing `SELECT 1` and that PostgreSQL is reported as healthy after the database probe succeeds.

The focused test passes, and the modified files pass Ruff, Black, and mypy. I also ran `make test-unit`. The baseline commit had 53 failed and 375 passed tests, while my fix commit had the same 53 failed tests and 376 passed tests. This confirms that the new health test passes and my change did not introduce additional failures.

I manually called `GET /health` with the local application running. The endpoint returned HTTP 503 because the separate Redis issue is still present, but PostgreSQL changed from `unhealthy` to `healthy`, while Redis remained `unhealthy` and the vector database remained `healthy`.

**Next steps:**
I will run `make check`, review `docs/CONTRIBUTING.md`, and confirm that my branch name, commit message, docstrings, and final diff follow the project conventions. Then I will push my branch, open a draft pull request for Issue #154, and request feedback from a classmate or mentor. After reviewing any feedback, I will mark the pull request as ready for review and complete Check-in 2.

**Blockers:**
No blocker for Issue #154. The full unit suite still contains 53 pre-existing failures that are unrelated to this change, but the baseline comparison confirms that my fix added no new failures.

---

### Check-in 2 (end of week)

**PR link:** [link to your submitted pull request]

**Branch:** [the branch name you worked on, e.g. `fix/123-short-description`]

**What you built:**
[1–3 sentences summarizing what your fix does and how it works]

**Tests added or updated:**
[Which test files did you touch? What do they cover?]

**Self-review confirmation:** [ ] make check passes  [ ] make test-unit passes

**Draft PR feedback received from:** [name or Slack handle, or "none"]