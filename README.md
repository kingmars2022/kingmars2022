# Siguang Zhao

Software developer in Montreal. I build backend services and the interfaces that sit on top of them.

**Open to new-grad and junior roles** — backend, full-stack, or QA.  
Montreal · Toronto · Ottawa · Vancouver, and open to relocating.

**Canadian permanent resident** — no sponsorship required.  
Working languages: English · **French (B2 certified)** · Mandarin (native)

📧 [siguangzhao@gmail.com](mailto:siguangzhao@gmail.com) · 💼 [LinkedIn](https://linkedin.com/in/siguangzhao)

---

## Experience

**Software Development Intern** · Shanghai Cosmosstars Information Technology Co., Ltd.  
*Sep 2025 – present · part-time, remote from Montreal*

On a five-developer team building a condominium property-management platform: a React admin console over Java/Spring Boot and Node.js services.

- Built React admin screens — paginated tables, multi-condition filters, sorting, and validated create/edit forms.
- Built a hierarchical property-record view that takes users from buildings down to units and resident records.
- Traced a pagination-count mismatch between the application and MySQL, then automated pre-release validation with a Python script.
- Debugged across the React front end, Spring Boot services and Node.js services, closing 15+ development and debugging tickets in two-week Scrum sprints.

## Tools I reach for

**Languages** Java · Python · TypeScript · JavaScript · SQL  
**Backend** Spring Boot · Spring Security · Spring Data JPA · JWT · FastAPI · Node.js · MyBatis · REST APIs  
**Frontend** React · Next.js · Vue · Pinia · Vite  
**Data & messaging** PostgreSQL · MySQL · MongoDB · Redis · Kafka · pgvector · Flyway · Alembic · schema design  
**AI** RAG · Ollama · Amazon Bedrock · PyTorch · LoRA  
**Cloud** AWS (Lambda · API Gateway · Cognito · S3 · SQS · ECS Fargate) · Docker · Kubernetes · Terraform  
**Delivery** GitHub Actions · pytest · JUnit · MockMvc · Vitest · Playwright

## Education

**Concordia University** — Bachelor of Computer Science (BCompSc) · Montreal · 2023 – 2026

---

## What I've built

### [tri-rive-nord](https://github.com/kingmars2022/tri-rive-nord) — waste collection and sorting for four Quebec towns

*Tells residents of Blainville, Sainte-Thérèse, Boisbriand and Rosemère which bin goes out tonight and where any household item belongs, in French, English and Chinese. Built with AI assistance (Claude Code).*

`Java` `Spring Boot` `Spring Security` `MyBatis` `PostgreSQL` `Redis` `Kafka` `MongoDB` `AWS S3/Lambda` `Flyway` `Web Push` `Vue` `TypeScript` `Docker`

- **Each city's rules come from that city's own printed calendar and guide,** and are tested
  against those documents rather than against the database they were loaded into.
- **An assistant that answers only from the selected city's guide** and says so when the guide
  has nothing, plus an admin agent that proposes changes and cannot apply one until a person approves it.
- **366 tests** (137 unit, 174 integration, 55 frontend), and load tested at 200 concurrent users
  with no failed requests.

**Live at [blainville-waste-sorting.onrender.com](https://blainville-waste-sorting.onrender.com/)**
(free tier, so the first visit after a quiet spell takes about a minute). Architecture and design
decisions are in the [project README](https://github.com/kingmars2022/tri-rive-nord).


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


### [ClaimPilot](https://github.com/kingmars2022/claimpilot) — group insurance claims assistant

*Upload your group benefits booklet, ask what it covers in English, French or Chinese, see what a claim needs, and get the claim form filled in from your own documents. You check every field, sign and submit it yourself.*

`Java` `Spring Boot` `PostgreSQL + pgvector` `MongoDB` `Kafka` `Redis` `AWS S3/SQS` `Bedrock` `Ollama` `Tesseract OCR` `PDFBox` `React` `TypeScript` `Docker`

A Spring Boot API over PostgreSQL with pgvector, a processing worker that reads PDFs (with OCR),
splits them into chunks and extracts key facts, and a React console with three modules: ask the
policy, a claim guide, and a pre-filled claim form.

What it does:

- **Every answer cites the policy page,** and has one of three outcomes: answered, unclear, or not in
  the policy. When the policy is silent, it prepares a call kit instead: the insurer's number, your
  policy and certificate numbers, and a script for the call.
- **The model reads, the code checks.** Each extracted fact comes with the exact quote it came from;
  code finds that quote in the document, confirms the value sits inside it, and marks anything it
  cannot confirm as unverified.
- **Hard rules in code, not in the prompt.** Signature and consent fields always stay blank, an
  "answered" reply that cites nothing is downgraded to "unclear", and an amount against a clause's
  threshold is settled by comparing the numbers.
- **Fills any insurer's fillable PDF** by matching field labels rather than field names; the mapping is
  stored per form version (SHA-256) so later claims need no model call. Six built-in forms ship with a
  reviewed mapping.
- **Decides which plan pays first** in code, following the Canadian (CLHIA) coordination of benefits
  guidelines, including custody per child for separated or divorced parents.
- **Your data stays yours:** uploads encrypted at rest with AES-256-GCM, every search scoped to the
  signed-in user, and *Delete my data* removes everything, cached model replies included.
- **Scales by Spring profile, not by code change:** two containers on a laptop by default, or S3 plus
  Kafka (or S3 notifications through SQS) to a separate worker, with Redis for shared cache and rate limits.

On an evaluation set of **41 questions in three languages over four fictional documents**, run with a
local 8B model on CPU, it gets the answer status right on 41/41, cites the expected page on 35/35, and
extracts 18/18 key facts. Every miss from earlier runs was fixed in retrieval or in code, not with a
larger model. Integration tests run over HTTP against real PostgreSQL/pgvector and MongoDB in Docker
(Testcontainers).


### [Enterprise Workflow Platform](https://github.com/kingmars2022/enterprise-workflow-platform) — approval workflow system

*An internal tool for requests that need sign-off — IT access, purchases, invoice exceptions: submit, review, approve or reject, and see who moved each request and when.*

`Java 17` `Spring Boot` `Spring Data JPA` `PostgreSQL` `React` `TypeScript` `Vite` `Docker Compose` `GitHub Actions` `Terraform` `AWS ECS Fargate`

A Spring Boot REST API over PostgreSQL and a React console with three panels: request intake, a
board grouped by status, and an inspector for the selected request.

- **Transitions enforced on the server.** A request moves SUBMITTED → IN_REVIEW → APPROVED →
  COMPLETED, or to REJECTED with a comment, and the rules live in the service layer, so the UI
  cannot push a request into a state the workflow does not allow.
- **An audit event for every creation and transition,** shown as a timeline on each request.
- **Search and filters** by status, title, system, priority and type, with live metrics on the intake panel.
- **Delivery:** Docker Compose locally, GitHub Actions for backend tests and the frontend build, and
  Terraform for an AWS deployment (ALB, ECS Fargate, ECR, RDS) released through GitHub OIDC.


### [EuroSAT Foundation Models](https://github.com/kingmars2022/eurosat-foundation-models) — label-efficient satellite image classification

*How many labelled satellite images do you really need? A benchmark of five generations of computer-vision methods on EuroSAT land-cover classification (Sentinel-2, 10 classes), from 0 to 500 labels per class.*

`Python` `PyTorch` `timm` `open_clip` `DINOv3` `CLIP / SigLIP 2 / RemoteCLIP` `Qwen3.5` `LoRA` `pytest` `GitHub Actions`

A pipeline that caches each frozen encoder's features once, then runs linear probes, zero-shot
prompting, in-context prompting and LoRA fine-tuning over the same fixed, stratified 2,000-image test
set. **170 runs, all on free Tesla T4 GPUs** (Colab and Kaggle), every number logged per run and
summarised by script.

What it found:

- **10 labels per class on a frozen encoder ≈ 500 per class from scratch.** A frozen DINOv3 ViT-L plus
  a linear layer reaches 91.7% with 10 per class; a ResNet-18 trained from scratch needs 500 per class
  for 92.0%.
- **Satellite pre-training did not beat web pre-training here.** With identical architecture, the
  ViT-L trained on 493M satellite tiles is below the web-trained one at every budget.
- **LoRA beats the linear probe at every budget while training 0.8% of the weights,** reaching
  98.5% at 500 per class, the best result in the project, and ahead of a linear probe trained on all
  25,000 images.
- **Chat VLMs do not replace a small labelled set.** The best zero-shot VLM (Qwen3.5-9B, 54.4%) is
  below a 1-shot linear probe (63.4%) and about 36x slower per image; LoRA on Qwen3.5-2B lifts it to
  88.0% at 50 per class, still under the vision encoders.

Unit tests cover the split, k-shot sampling, augmentation and answer parsing, and a smoke test runs
every pipeline stage end to end on CPU, offline, with tiny randomly initialised models. CI runs both
on every push.
