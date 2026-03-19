# CLAUDE.md
**Customer Risk API · Execution Contract v1.0**
*Frozen. Do not edit inline. New version required for any change.*

---

## System Intent

A read-only authenticated HTTP service that wraps a pre-populated Postgres database so operations staff can look up customer risk tiers and contributing factors without direct database access. A single FastAPI container serves both the JSON API and a vanilla HTML/JS browser UI. A separate Postgres container holds the data. Nothing is computed — the API is a controlled read surface only.

---

## Hard Invariants

These conditions must never be violated. Each task that touches one requires a code review, not just a passing test.

- **INV-01** Every HTTP 200 response contains exactly three fields: `customer_id` (string), `risk_tier` (string), `risk_factors` (array of strings). No more, no fewer.
- **INV-02** A missing customer always returns HTTP 404 with a static message. Never a 200 with empty data. Never a 500.
- **INV-03** The system never executes INSERT, UPDATE, DELETE, or DDL under any circumstance, including malformed or injected input.
- **INV-04** Authentication is checked before any database call is made. Requests without a valid `X-API-Key` header are rejected at the dependency layer.
- **INV-05** HTTP 401 responses contain a static literal detail string only. The submitted key value never appears in any response body.
- **INV-06** No stack trace, psycopg2 error, connection string, or internal hostname ever appears in any HTTP response under any failure condition.
- **INV-07** The API key never appears in any response, error detail, or log output surfaced to a caller.
- **INV-08** The API returns data exactly as stored in the database. No transformation, enrichment, defaulting, or hardcoding at any layer.
- **INV-09** All database queries use psycopg2 parameterised statements. User input is never interpolated into SQL via f-strings or concatenation.
- **INV-10** The system makes no external network calls at runtime. All dependencies are local to the Compose stack.
- **INV-11** The UI renders `risk_tier` and `risk_factors` exactly as received from the API. No mapping, substitution, or reformatting in JavaScript.
- **INV-12** The API container does not serve requests until Postgres has passed its healthcheck (`depends_on: condition: service_healthy`).
- **INV-13** The UI displays distinct messages for 401, 404, and 5xx. A single generic fallback is not sufficient.
- **INV-14** `/health` returns 200 with no customer data and requires no API key.
- **INV-15** The database initialises with at least 9 records — minimum 3 per tier (LOW, MEDIUM, HIGH).
- **INV-16** `index.html` loads no external assets. No CDN scripts, remote fonts, or third-party resources.
- **INV-17** `risk_tier` values are strictly LOW, MEDIUM, or HIGH. No other values may appear in any response.

---

## Scope Boundary

**Claude Code is permitted to:**
- Create and modify files listed in the Appendix B file manifest only
- Write SQL (SELECT only), Python, HTML, JavaScript, YAML, and shell verification commands
- Add logging at ERROR or WARNING level using Python's `logging` module

**Claude Code must never:**
- Modify this file (CLAUDE.md)
- Add dependencies not in `requirements.txt` as specified in S2.T1
- Write any SQL other than SELECT in application code
- Introduce connection pooling, an ORM, or any database abstraction layer
- Add routes, endpoints, or middleware not specified in the execution plan
- Substitute default values for missing or null database fields
- Log request headers, the API key value, or any credential

---

## Fixed Stack

| Component | Technology | Version |
|---|---|---|
| Orchestration | Docker Compose | Current stable |
| Database | Postgres | 15 |
| API runtime | Python | 3.11 |
| API framework | FastAPI | 0.111.0 |
| ASGI server | uvicorn[standard] | 0.29.0 |
| DB driver | psycopg2-binary | 2.9.9 |
| UI | Vanilla HTML + JavaScript | No framework |

No additions to this stack are permitted. Any task that requires a component not listed here cannot proceed without a new version of this document.