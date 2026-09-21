# Siguang Zhao

Software developer in Montreal. I build backend services and the interfaces that sit on top of them.

**Open to new-grad and junior roles** — backend, full-stack, or QA.  
Montreal · Toronto · Ottawa · Vancouver, and open to relocating.

**Canadian permanent resident** — no sponsorship required.  
Working languages: English · **French (B2 certified)** · Mandarin (native)

📧 [siguangzhao@gmail.com](mailto:siguangzhao@gmail.com) · 💼 [LinkedIn](https://linkedin.com/in/siguangzhao)

---

## What I've built

### [Blainville Waste Sorting Platform](https://github.com/kingmars2022/blainville-waste-sorting) — municipal waste sorting and collection app
`Vue 3` `TypeScript` `Spring Boot` `Java 21` `MyBatis` `MySQL` `Redis` `Kafka` `MongoDB` `AWS S3/Lambda` `Flyway` `Spring Security`

A trilingual (**French / English / Chinese**) app for Blainville residents, with a full admin back office.
**JWT authentication end to end** — a custom Spring Security filter, BCrypt hashing, ADMIN/USER roles
returning correct 401 versus 403 semantics.

Residents ask questions in their own language and get **grounded answers or an explicit refusal** —
never an invented one. Retrieval is MySQL full-text with **two parsers**, the default word parser for
French and English and `ngram` for Chinese, because one parser made refusal impossible. Scored
**12/12 retrieval precision@1 and 6/6 refusal accuracy** over 18 questions. Telling someone to put
paint in the blue bin is a real-world error, so the guide decides the bin — the model only does the
wording, and answers nothing when retrieval returns nothing.

An **admin agent** proposes changes and cannot make them: Claude tool use where reads execute during
planning and every write is recorded as a plan a human approves. Plans live in Redis for 15 minutes,
are **single-use under concurrency** (`GETDEL`, with the owner in the key, after a test raced two
approvals and found both executing), and are bound to the administrator they were shown to.

Notice events go through a **transactional outbox** rather than a publish inside the request — the
event commits with the row, and a relay drains it to **Kafka**, where two consumer groups fan out a
resident inbox and an audit trail. Delivery is at-least-once, so both consumers are idempotent.
The audit trail is implemented **twice**, over **MongoDB** and a MySQL JSON column, with one contract
test run against both — and the honest conclusion is that MySQL wins at this scale. What MongoDB
earns is the other half: an aggregation over anonymous resident questions that reports **which
materials residents ask about that the guide cannot answer**, with retention as a TTL index.

Photo questions upload **straight to S3 on a presigned URL** with size and content type signed in, so
the bytes never pass through the application, and a **Lambda strips EXIF** before anything is served —
a photo of a bin on a driveway carries the GPS coordinates of the house.

**180 tests** — 81 of them against real MySQL, Redis, Kafka and S3 rather than mocks, which is how
most of the bugs in this repository's history were found: a dead Redis costing 4 seconds a request
until the client was told to fail fast, a calendar that would have silently run out on a fixed date,
an edit form that would have erased every card's examples, and a deployment jar that could never
have cold-started. Load tested at **200 concurrent users with zero failed requests**.

**Live at [blainville-waste-sorting.onrender.com](https://blainville-waste-sorting.onrender.com/)** —
one Docker image serving the API and the pages, on a free tier that sleeps when idle, so the first
request after a quiet spell takes about a minute. The photo pipeline is still exercised against a
real S3 API and a real Kafka broker locally rather than on AWS, and the repository says so rather
than implying otherwise.

### [Stockroom](https://github.com/kingmars2022/stockroom-warehouse-system) — warehouse operations system
`Python` `FastAPI` `PostgreSQL` `MongoDB` `Redis` `AWS Lambda` `API Gateway` `Cognito` `Next.js` `Kubernetes` `Terraform`

22 REST endpoints over an 8-table schema, where **stock cannot go negative under concurrency**.
Row-level `SELECT … FOR UPDATE` locking, proven by a test that races **20 threads for 10 units**
against a real PostgreSQL instance — exactly 10 succeed, 10 are rejected, the balance lands on zero.

Every audit entry is also published as a **MongoDB** document carrying the fields that action
actually has, so "every purchase that rose more than 20%" is a query instead of a prose scan.
A tool-using **agent** proposes reorders but cannot place one — every tool but the final
proposal is read-only, and the numbers come from the same replenishment engine the console
uses, not the model. Two **AWS Lambdas** close gaps a presigned-upload API can't reach on its
own: one checks a receipt's real file signature against what the client claimed, the other lets
suppliers push price quotes through a signed **API Gateway** webhook.

**289 tests at 95% statement coverage**, including passes in CI against real Redis, PostgreSQL,
and MongoDB instances rather than in-memory stand-ins, enforced by a coverage gate.

---

## Tools I reach for

**Languages** Java · Python · TypeScript · JavaScript · SQL  
**Backend** Spring Boot · Spring Security · FastAPI · Node.js · MyBatis · REST APIs  
**Frontend** React · Next.js · Vue 3 · Pinia · Vite  
**Data** PostgreSQL · MySQL · MongoDB · Redis · Flyway · Alembic · schema design  
**Cloud** AWS (Lambda · API Gateway · Cognito · S3) · Docker · Kubernetes · Terraform  
**Delivery** GitHub Actions · pytest · JUnit · MockMvc

---

*Most recently at **ThinTrix**, building Java/Spring Boot and Node.js services behind a React admin
console for a condominium property-management platform, in a five-developer team.*
