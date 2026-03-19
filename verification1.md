# VERIFICATION_RECORD.md
**Session:** S1 — Infrastructure: Docker Compose, Postgres, Schema, Seed Data  
**Date:**  
**Engineer:**

---

## Task 1.1 — Project scaffold and `.env.example`

**Test Cases Applied:**

| Case | Scenario | Expected | Result |
|---|---|---|---|
| TC1 | All required files and directories exist | `ls` shows `docker-compose.yml`, `.env.example`, `.gitignore`, `api/`, `db/` | |
| TC2 | `.env.example` contains all six variable names | `grep -c "=" .env.example` returns 6 | |
| TC3 | `.env` is listed in `.gitignore` | `grep "^\.env$" .gitignore` returns a match | |
| TC4 | `docker-compose.yml` defines both `postgres` and `api` services | `grep -c "^\s\{2\}[a-z]" docker-compose.yml` returns at least 2 | |

**Prediction Statement:**

- TC1: [ENGINEER: predicted output]
- TC2: [ENGINEER: predicted output]
- TC3: [ENGINEER: predicted output]
- TC4: [ENGINEER: predicted output]

**CD Challenge Output:**

> Prompt: *What did you not test in Task 1.1?*

[ENGINEER: record CC response here]

**Code Review:**

[ENGINEER: no invariant touch — structural scaffolding only. Record any observations.]

**Test Cases Added During Session:**

| Case | Scenario | Expected | Result |
|---|---|---|---|
| | | | |

**Scope Decisions:**

[ENGINEER: record any scope decisions made during this task — accepted deviations, rejected additions, clarifications issued to Claude Code.]

**Verification Verdict:**

- [ ] All test cases passed
- [ ] CC challenge reviewed and logged
- [ ] Code review complete (if invariant-touching)
- [ ] Commit made with correct message format

**Status:** In Progress  
**Engineer sign-off:**

---

## Task 1.2 — Docker Compose: Postgres service with health check

> ⚠️ **INVARIANT TOUCH: INV-12** — healthcheck is the enforcement mechanism for startup dependency.

**Test Cases Applied:**

| Case | Scenario | Expected | Result |
|---|---|---|---|
| TC1 | `docker compose up -d postgres` starts without error | Exit code 0 | |
| TC2 | Postgres container reaches `healthy` within 60s | `docker compose ps` shows `(healthy)` | |
| TC3 | Can connect and run a query | `docker exec <container> psql -U riskuser -d riskdb -c "SELECT 1;"` returns `1` | |
| TC4 | Named volume `pgdata` is created | `docker volume ls` shows `customer-risk-api_pgdata` | |

**Prediction Statement:**

- TC1: [ENGINEER: predicted output]
- TC2: [ENGINEER: predicted output]
- TC3: [ENGINEER: predicted output]
- TC4: [ENGINEER: predicted output]

**CD Challenge Output:**

> Prompt: *What did you not test in Task 1.2?*

[ENGINEER: record CC response here]

**Code Review:**

[ENGINEER: confirm `interval`, `retries`, and `start_period` are sufficient. Confirm `pg_isready` checks the correct user and database — not just the server process. Record findings.]

**Test Cases Added During Session:**

| Case | Scenario | Expected | Result |
|---|---|---|---|
| | | | |

**Scope Decisions:**

[ENGINEER: record any scope decisions made during this task.]

**Verification Verdict:**

- [ ] All test cases passed
- [ ] CC challenge reviewed and logged
- [ ] Code review complete (INV-12 touched)
- [ ] Commit made with correct message format

**Status:** In Progress  
**Engineer sign-off:**

---

## Task 1.3 — Database schema and seed data init script

> ⚠️ **INVARIANT TOUCH: INV-15** — seed data completeness (minimum 3 per tier).  
> ⚠️ **INVARIANT TOUCH: INV-03** — schema must have no write triggers; init.sql is the only write path.  
> ⚠️ **INVARIANT TOUCH: INV-17** — CHECK constraint enforces closed risk tier set.

**Test Cases Applied:**

| Case | Scenario | Expected | Result |
|---|---|---|---|
| TC1 | Script runs without SQL errors | `psql` exit code 0 | |
| TC2 | Table exists with correct columns | `\d customers` shows customer_id, risk_tier, risk_factors | |
| TC3 | Exactly 15 rows inserted | `SELECT COUNT(*) FROM customers;` returns 15 | |
| TC4 | All three tiers present with count 5 each | `SELECT risk_tier, COUNT(*) FROM customers GROUP BY risk_tier;` returns LOW=5, MEDIUM=5, HIGH=5 | |
| TC5 | risk_factors is a non-empty array for every row | `SELECT COUNT(*) FROM customers WHERE risk_factors = '{}';` returns 0 | |
| TC6 | CHECK constraint rejects invalid tier | `INSERT INTO customers VALUES ('X', 'INVALID', '{}');` raises constraint violation | |

**Prediction Statement:**

- TC1: [ENGINEER: predicted output]
- TC2: [ENGINEER: predicted output]
- TC3: [ENGINEER: predicted output]
- TC4: [ENGINEER: predicted output]
- TC5: [ENGINEER: predicted output]
- TC6: [ENGINEER: predicted output]

**CD Challenge Output:**

> Prompt: *What did you not test in Task 1.3?*

[ENGINEER: record CC response here]

**Code Review:**

[ENGINEER: confirm `CHECK (risk_tier IN ('LOW', 'MEDIUM', 'HIGH'))` is present. Confirm `NOT NULL` on both `risk_tier` and `risk_factors`. Confirm `CREATE TABLE IF NOT EXISTS` is used. Confirm `risk_factors` uses `TEXT[]` not `JSONB`. Confirm no UPDATE, DELETE, or trigger definitions appear anywhere in the file. Record findings.]

**Test Cases Added During Session:**

| Case | Scenario | Expected | Result |
|---|---|---|---|
| | | | |

**Scope Decisions:**

[ENGINEER: record any scope decisions made during this task.]

**Verification Verdict:**

- [ ] All test cases passed
- [ ] CC challenge reviewed and logged
- [ ] Code review complete (INV-15, INV-03, INV-17 touched)
- [ ] Commit made with correct message format

**Status:** In Progress  
**Engineer sign-off:**

---

## Task 1.4 — Mount init script into Docker Compose

> ⚠️ **INVARIANT TOUCH: INV-15** — seed data must be present after fresh initialisation.

**Test Cases Applied:**

| Case | Scenario | Expected | Result |
|---|---|---|---|
| TC1 | Fresh stack start seeds the database | After `docker compose down -v && docker compose up -d postgres`, all 15 rows present | |
| TC2 | Init script is mounted read-only | `docker compose exec postgres touch /docker-entrypoint-initdb.d/init.sql` fails with permission error | |
| TC3 | Second start with existing volume skips init script (expected behaviour) | After `docker compose restart postgres` (without `-v`), data still present and count still 15 | |

**Prediction Statement:**

- TC1: [ENGINEER: predicted output]
- TC2: [ENGINEER: predicted output]
- TC3: [ENGINEER: predicted output]

**CD Challenge Output:**

> Prompt: *What did you not test in Task 1.4?*

[ENGINEER: record CC response here]

**Code Review:**

[ENGINEER: confirm `:ro` flag is present on the bind mount. Confirm `pgdata` named volume mount is still present and unchanged. Record findings.]

**Test Cases Added During Session:**

| Case | Scenario | Expected | Result |
|---|---|---|---|
| | | | |

**Scope Decisions:**

[ENGINEER: record any scope decisions made during this task.]

**Verification Verdict:**

- [ ] All test cases passed
- [ ] CC challenge reviewed and logged
- [ ] Code review complete (INV-15 touched)
- [ ] Commit made with correct message format

**Status:** In Progress  
**Engineer sign-off:**