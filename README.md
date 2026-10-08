# Siguang Zhao

Software developer in Montreal. I build backend services and the interfaces that sit on top of them.

**Open to new-grad and junior roles** — backend, full-stack, or QA.  
Montreal · Toronto · Ottawa · Vancouver, and open to relocating.

**Canadian permanent resident** — no sponsorship required.  
Working languages: English · **French (B2 certified)** · Mandarin (native)

📧 [siguangzhao@gmail.com](mailto:siguangzhao@gmail.com) · 💼 [LinkedIn](https://linkedin.com/in/siguangzhao)

---

## What I've built

### [Blainville Waste Sorting](https://github.com/kingmars2022/blainville-waste-sorting) — municipal waste sorting and collection app

*A website for residents of a Quebec town: which bin goes out tonight, where any household item belongs, and what the city has announced. In French, English and Chinese. Built with AI assistance (Claude Code).*

`Java` `Spring Boot` `Spring Security` `MyBatis` `MySQL` `Redis` `Kafka` `MongoDB` `AWS S3/Lambda` `Flyway` `Web Push` `Vue` `TypeScript` `Docker`

A Spring Boot backend over a 14-table MySQL schema with 39 REST endpoints, and a Vue console on
top of it. Residents and staff see different systems, enforced by JWT through a custom Spring
Security filter with BCrypt hashing and ADMIN/USER roles that return 401 and 403 for the right
reasons. Schema and reference data both live in Flyway migrations (V1–V18), so an empty database
comes back fully populated on first boot.

What it does:

- **Collection schedules as recurring rules plus generated events,** so one statutory holiday
  shifts every affected pickup, including dates already generated months ahead, instead of
  editing each date by hand.
- **A sorting guide of 35 categories** taken from the city's printed calendar, served from the
  database to residents, the assistant, the photo lookup and the admin console alike rather than
  duplicated in the frontend.
- **An assistant that answers only from that guide** and refuses when it has nothing: 19/19
  retrieval precision@1 and 6/6 refusals over 25 questions. Retrieval is MySQL full-text with two
  parsers, the default word parser for French and English and `ngram` for Chinese, because one
  parser made refusal impossible.
- **An admin agent that proposes and cannot execute.** Every write is a plan a human approves.
  Plans live in Redis for 15 minutes, keyed to the administrator they were shown to, and are
  single-use under concurrency (`GETDEL`, after a test raced two approvals and found both running).
- **Notices through a transactional outbox,** so the event commits with the row instead of being
  published inside the request. A relay drains it to Kafka, where two idempotent consumer groups
  fan out a resident inbox and an audit trail.
- **An audit trail built twice,** over MongoDB and a MySQL JSON column, with one contract test run
  against both. MySQL wins at this scale. What MongoDB earns is the other half: an aggregation
  over anonymous resident questions reporting which materials the guide cannot answer, with
  retention as a TTL index.
- **Browser push encrypted to RFC 8291** against the JDK rather than a library, checked byte for
  byte against output from `http_ece`, the implementation `web-push` uses. That comparison found
  two defects: the JDK's `KeyFactory` accepts public keys that are not points on the P-256 curve,
  and `jjwt` serialises a single `aud` claim as a one-element array. Push is a second delivery;
  the notice row stays the record.
- **Photo questions uploaded straight to S3** on a presigned URL with size and content type signed
  in, so the bytes never reach the application, and a Lambda strips EXIF before anything is served.

The collection calendar is checked against the city's published document, not against the database
it was generated from. Every test passed for weeks while the calendar printed the wrong bin: two
patterns were labelled recycling where Blainville prints household waste.

**248 tests** — 112 unit, 101 against real infrastructure, 35 in the browser. CI runs the
integration suite against real MySQL and Redis; Kafka, MongoDB and S3 are exercised against real
local instances rather than mocks. Load tested at **200 concurrent users with zero failed requests**.

**Live at [blainville-waste-sorting.onrender.com](https://blainville-waste-sorting.onrender.com/)**
— one Docker image serving the API and the pages, on a free tier that sleeps when idle, so the
first request after a quiet spell takes about a minute. Redis, Kafka, MongoDB and the photo
pipeline are switched off there and the application degrades to MySQL by design.


### [Stockroom](https://github.com/kingmars2022/stockroom-warehouse-system) — warehouse operations system

*A back-office system for a warehouse: track what gets taken, what comes in, what needs reordering, and who approved the spending.*

`Python` `FastAPI` `PostgreSQL` `MongoDB` `Redis` `AWS Lambda` `API Gateway` `Cognito` `S3` `Next.js` `TypeScript` `Playwright` `Docker` `Kubernetes` `Terraform`

A FastAPI backend over a 9-table PostgreSQL schema with 24 REST endpoints, and a Next.js
console on top of it. Three roles — employee, supervisor, manager — each see a different
system, enforced on the server rather than by hiding buttons.

What it does:

- **Stock issues and receipts,** with row-level `SELECT … FOR UPDATE` locking so stock cannot
  go negative when several people take the same item at once. Tested by racing 20 threads for
  10 units against a real PostgreSQL: exactly 10 succeed, 10 are rejected.
- **Replenishment planning** from the last 90 days of usage — days of cover, how much to order,
  and suppliers ranked by price, lead time and rating. Cached in Redis.
- **Purchase and price history,** with an alert when a supplier raises a price past a configured
  threshold, shown next to what the other suppliers last charged for the same item.
- **Reimbursements** with receipts uploaded straight to S3 on a presigned URL, so the files never
  pass through the API. A Lambda checks each file's real signature against what the client
  claimed before it can be attached, and the approved version is pinned so it cannot be swapped.
- **An audit trail in two forms:** relational rows, plus a MongoDB document per event carrying
  the fields that action actually has, so "every purchase that rose more than 20%" is a query
  rather than a text search.
- **A tool-using agent** that proposes reorders and cannot place one. Every tool but the final
  proposal is read-only, and the quantities come from the same replenishment engine the console
  uses, not from the model.
- **A signed API Gateway webhook** suppliers push price quotes into, drained by a second Lambda.

Built to hold up at **10,000 items and 80,000 rows of history**: the replenishment engine
aggregates in SQL instead of loading every row (3.3 s → 0.97 s), and every list is read a page
at a time, which took signing in from **33 MB to 151 KB**.

**334 backend tests at 95% statement coverage** and **34 browser tests**, with CI runs against
real Redis, PostgreSQL and MongoDB instances rather than in-memory stand-ins. Infrastructure as
Terraform, with Kubernetes manifests as an alternative deployment path.

**Live demo: [stockroom-warehouse-system.vercel.app](https://stockroom-warehouse-system.vercel.app)**
— sign in as `admin@stockroom.test` / `Stockroom!2026`. The console runs with the warehouse
seeded into the browser; the API, Cognito and the Lambdas run in CI and locally, not on AWS.

---

## Tools I reach for

**Languages** Java · Python · TypeScript · JavaScript · SQL  
**Backend** Spring Boot · Spring Security · JWT · FastAPI · Node.js · MyBatis · REST APIs  
**Frontend** React · Next.js · Vue · Pinia · Vite  
**Data & messaging** PostgreSQL · MySQL · MongoDB · Redis · Kafka · Flyway · Alembic · schema design  
**Cloud** AWS (Lambda · API Gateway · Cognito · S3) · Docker · Kubernetes · Terraform  
**Delivery** GitHub Actions · pytest · JUnit · MockMvc · Vitest · Playwright

---

*Most recently at **Dianqing** **Company**, building Java/Spring Boot and Node.js services behind a React admin
console for a condominium property-management platform, in a five-developer team.*
