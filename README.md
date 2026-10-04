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

*A back-office system for a warehouse: track stock, receive goods and plan reorders, without ever selling stock that isn't there.*

`Python` `FastAPI` `PostgreSQL` `MongoDB` `Redis` `AWS Lambda` `API Gateway` `Cognito` `Next.js` `Playwright` `Kubernetes` `Terraform`

24 REST endpoints over a 9-table schema, where **stock cannot go negative under concurrency**.
Row-level `SELECT … FOR UPDATE` locking, proven by a test that races **20 threads for 10 units**
against a real PostgreSQL instance — exactly 10 succeed, 10 are rejected, the balance lands on zero.

Every audit entry is also published as a **MongoDB** document carrying the fields that action
actually has, so "every purchase that rose more than 20%" is a query instead of a prose scan.
A tool-using **agent** proposes reorders but cannot place one — every tool but the final
proposal is read-only, and the numbers come from the same replenishment engine the console
uses, not the model. Two **AWS Lambdas** close gaps a presigned-upload API can't reach on its
own: one checks a receipt's real file signature against what the client claimed, the other lets
suppliers push price quotes through a signed **API Gateway** webhook.

The part I'd rather be asked about is what happened when I stopped assuming it scaled and
measured it at **10,000 items and 80,000 rows of history**. Signing in downloaded **33 MB** —
the whole warehouse — and the inventory screen spent **7.4 seconds** laying out 130,287 DOM
nodes for a page that shows fifty. Working out what to reorder took 3.3 s and 161 MB, and a
profile put four fifths of that inside SQLAlchemy's row hydration: the engine read every
movement and purchase ever recorded to produce two numbers per item, so its cost grew with how
long the warehouse had been running rather than with how much it stocks. Rewritten as SQL
aggregates and a `ROW_NUMBER` window it is **0.97 s and 29 MB**, and with every list read a
page at a time, signing in is **151 KB**.

Paging the purchase history is where it got interesting, because it took an answer away from
the console: the price alerts were derived by filtering every purchase the browser had been
sent. In that dataset there are **10,817 alerts in the database against 19 visible in the fifty
most recent purchases** — a page would have quietly redefined "price alerts" as "price alerts
among recent purchases", on a figure the dashboard prints on a card. They are a server-side
query now, and so are the alternative supplier prices quoted beside each one.

I also published the wrong number first. "33 MB to 107 KB" left out the ranked purchase plan,
another 1.45 MB the console fetches on sign-in: I had measured the endpoints I changed rather
than the whole path. Re-measuring all of it is what caught it, and the commit that fixes it
says so.

**334 backend tests at 95% statement coverage** and **34 driving the console in a real
browser**, including passes in CI against real Redis, PostgreSQL and MongoDB instances rather
than in-memory stand-ins, enforced by a coverage gate.

**Live at [stockroom-warehouse-system.vercel.app](https://stockroom-warehouse-system.vercel.app)** —
the console with the warehouse seeded into the browser, so there is nothing to wake up: sign in
as `admin@stockroom.test` / `Stockroom!2026` and each of the three roles sees a different
system. The API, Cognito and the Lambdas are exercised in CI and locally rather than deployed
to AWS, and the repository says so rather than implying otherwise.

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
