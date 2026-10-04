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

*A website that tells residents of a Quebec town which bin goes out and when, in French, English and Chinese — and says "I don't know" when the city's guide doesn't cover the question. Built with AI assistance (Claude Code); details in the repository README.*

`Vue 3` `TypeScript` `Spring Boot` `Java 21` `MyBatis` `MySQL` `Redis` `Kafka` `MongoDB` `AWS S3/Lambda` `Flyway` `Spring Security` `Web Push (RFC 8291)`

A trilingual (**French / English / Chinese**) app for Blainville residents, with a full admin back office.
**JWT authentication end to end** — a custom Spring Security filter, BCrypt hashing, ADMIN/USER roles
returning correct 401 versus 403 semantics.

Residents ask questions in their own language and get **grounded answers or an explicit refusal** —
never an invented one. Retrieval is MySQL full-text with **two parsers**, the default word parser for
French and English and `ngram` for Chinese, because one parser made refusal impossible. Scored
**19/19 retrieval precision@1 and 6/6 refusal accuracy** over 25 questions — and expanding that
set is what caught a bug nothing else would have: a question about chicken bones answered in
English and refused in French, because `os` is two letters and InnoDB's full-text index skips
tokens shorter than three. Telling someone to put
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

The same notice can also reach a resident as a **browser push notification**, encrypted to
**RFC 8291** against the JDK rather than a library — and the reason that is defensible is that it
is checked against somebody else's implementation instead of its own. A test encrypts a fixed
input and compares it byte for byte with output captured from `http_ece`, the JavaScript library
`web-push` uses. That comparison found two defects: the JDK's `KeyFactory` will happily accept a
public key that is **not a point on the P-256 curve** — an invalid-curve attack any API caller can
choose — so the point is now validated explicitly; and `jjwt` serialises a single `aud` claim as a
one-element array where every reference implementation emits a string. Push is a *second* delivery,
never the record: the notice row is written first, so a revoked permission costs a resident a buzz
rather than the notice.

Photo questions upload **straight to S3 on a presigned URL** with size and content type signed in, so
the bytes never pass through the application, and a **Lambda strips EXIF** before anything is served —
a photo of a bin on a driveway carries the GPS coordinates of the house.

**248 tests** — 101 of them against real MySQL, Redis, Kafka and S3 rather than mocks, which is how
most of the bugs in this repository's history were found: a dead Redis costing 4 seconds a request
until the client was told to fail fast, a calendar that would have silently run out on a fixed date,
an edit form that would have erased every card's examples, and a deployment jar that could never
have cold-started.

The one I'd rather be asked about passed every test for weeks. The collection calendar printed the
wrong bin, and no test caught it because every test compared the calendar to the database it was
generated from. Comparing it to the city's own published document instead showed that two collection
patterns had been labelled recycling where Blainville prints household waste — residents would have
put out the blue bin on a black-bin week all year. **A test that only checks a system against itself
cannot find a system that is confidently wrong**; that test now reads the municipal document.
Load tested at **200 concurrent users with zero failed requests**.

**Live at [blainville-waste-sorting.onrender.com](https://blainville-waste-sorting.onrender.com/)** —
one Docker image serving the API and the pages, on a free tier that sleeps when idle, so the first
request after a quiet spell takes about a minute. The photo pipeline is still exercised against a
real S3 API and a real Kafka broker locally rather than on AWS, and the repository says so rather
than implying otherwise.

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
**Backend** Spring Boot · Spring Security · FastAPI · Node.js · MyBatis · REST APIs  
**Frontend** React · Next.js · Vue 3 · Pinia · Vite  
**Data** PostgreSQL · MySQL · MongoDB · Redis · Flyway · Alembic · schema design  
**Cloud** AWS (Lambda · API Gateway · Cognito · S3) · Docker · Kubernetes · Terraform  
**Delivery** GitHub Actions · pytest · JUnit · MockMvc · Playwright

---

*Most recently at **ThinTrix**, building Java/Spring Boot and Node.js services behind a React admin
console for a condominium property-management platform, in a five-developer team.*
