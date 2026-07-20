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