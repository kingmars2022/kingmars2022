# Siguang Zhao

Software developer in Montreal. I build backend services and the interfaces that sit on top of them.

**Open to new-grad and junior roles** — backend, full-stack, or QA.  
Montreal · Toronto · Ottawa · Vancouver, and open to relocating.

**Canadian permanent resident** — no sponsorship required.  
Working languages: English · **French (B2 certified)** · Mandarin (native)

📧 [siguangzhao@gmail.com](mailto:siguangzhao@gmail.com) · 💼 [LinkedIn](https://linkedin.com/in/siguangzhao)

---

## What I've built

### [Stockroom](https://github.com/kingmars2022/stockroom-warehouse-system) — warehouse operations system
`Python` `FastAPI` `PostgreSQL` `Redis` `Next.js` `AWS Cognito` `Kubernetes` `Terraform`

19 REST endpoints over an 8-table schema, where **stock cannot go negative under concurrency**.
Row-level `SELECT … FOR UPDATE` locking, proven by a test that races **20 threads for 10 units**
against a real PostgreSQL instance — exactly 10 succeed, 10 are rejected, the balance lands on zero.

A Redis read-through cache that **degrades to in-process memory during an outage** rather than
taking the API down, with hit/miss stats exposed on `/health`.
**159 tests at 94% statement coverage**, enforced by a coverage gate in CI.

### [Bienvenue à Blainville](https://github.com/kingmars2022/blainville-waste-sorting) — municipal waste sorting platform
`Vue 3` `TypeScript` `Spring Boot` `Java 21` `MyBatis` `MySQL` `Flyway` `Spring Security`

A trilingual (**French / English / Chinese**) app for Blainville residents, with a full admin back office.
**JWT authentication end to end** — a custom Spring Security filter, BCrypt hashing, ADMIN/USER roles
returning correct 401 versus 403 semantics.

Unit tests plus an integration suite that boots the full Spring context against a **real MySQL database**.
Manual end-to-end verification caught **six defects that mocked tests structurally could not** — an app
that crashed on startup, admin endpoints throwing `ClassCastException` on every write, and a 401/403 mix-up.

---

## Tools I reach for

**Languages** Java · Python · TypeScript · JavaScript · SQL  
**Backend** Spring Boot · Spring Security · FastAPI · Node.js · MyBatis · REST APIs  
**Frontend** React · Next.js · Vue 3 · Pinia · Vite  
**Data** PostgreSQL · MySQL · Redis · Flyway · Alembic · schema design  
**Delivery** Docker · Kubernetes · Terraform · GitHub Actions · pytest · JUnit · MockMvc

---

*Most recently at **ThinTrix**, building Java/Spring Boot and Node.js services behind a React admin
console for a condominium property-management platform, in a five-developer team.*
