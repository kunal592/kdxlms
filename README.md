# KDXLabsLLM — Learning Management Platform (LMS) Backend

An enterprise-grade, high-performance backend architecture for an online technology education platform (reverse-engineered and architected from the **TuteDude** feature scope).

Built as a **modular monolith** in Go with PostgreSQL, Redis, and asynchronous workers, designed for high concurrency, payment idempotency, secure video streaming, mentor doubt-resolution workflows, and automated course-completion refund systems.

**Developer:** [KDXLabs](https://www.kdxlabs.cloud) · [www.kdxlabs.cloud](https://www.kdxlabs.cloud)

---

## Table of Contents

- [1. Executive Summary](#1-executive-summary)
- [2. High-Level Design (HLD)](#2-high-level-design-hld)
  - [System Architecture](#system-architecture)
  - [Recommended Technology Stack](#recommended-technology-stack)
- [3. Domain Modules](#3-domain-modules)
- [4. Low-Level Design (SLD) & Data Models](#4-low-level-design-sld--data-models)
  - [Core Database Schema (PostgreSQL)](#core-database-schema-postgresql)
  - [Entity Relationship Diagram](#entity-relationship-diagram)
- [5. API Endpoint Catalog](#5-api-endpoint-catalog)
  - [A. Catalog & Course Discovery](#a-catalog--course-discovery)
  - [B. Authentication & User Accounts](#b-authentication--user-accounts)
  - [C. Enrollment, Bundles & Checkout](#c-enrollment-bundles--checkout)
  - [D. Student Learning Platform](#d-student-learning-platform)
  - [E. Assignments, Projects & Assessments](#e-assignments-projects--assessments)
  - [F. Certificates & Conditional Refunds](#f-certificates--conditional-refunds)
  - [G. Mentorship, Doubts & Live Sessions](#g-mentorship-doubts--live-sessions)
  - [H. Internship & Career Services](#h-internship--career-services)
  - [I. Reviews, Notifications & Support](#i-reviews-notifications--support)
  - [J. Mentor & Administration Operations](#j-mentor--administration-operations)
- [6. Critical Business Flows](#6-critical-business-flows)
  - [1. Order Checkout, Payment Webhook & Idempotent Enrollment](#1-order-checkout-payment-webhook--idempotent-enrollment)
  - [2. Authorized Video Streaming & Progress Tracking](#2-authorized-video-streaming--progress-tracking)
  - [3. Conditional Course-Completion Refund Pipeline](#3-conditional-course-completion-refund-pipeline)
- [7. Security & Reliability](#7-security--reliability)
- [8. Project Layout](#8-project-layout)
- [9. Getting Started](#9-getting-started)
- [10. Architecture Knowledge Graph](#10-architecture-knowledge-graph)
- [11. Developer & Maintainer](#11-developer--maintainer)

---

## 1. Executive Summary

Modern e-learning platforms balance content delivery, real-time mentorship, commerce, and progression-linked financial rewards (e.g. course-completion refund guarantees). 

This platform handles:
- **Course & Bundle Discovery**: Standalone courses and multi-course customizable bundles.
- **Transactional Commerce**: Checkout with idempotency keys, signed payment webhooks, and automatic duplicate-enrollment prevention.
- **Secure Media Delivery**: Expiring signed video URLs and chunked lesson progress persistence.
- **1-on-1 Mentorship & Doubts**: Live video signaling, doubt ticketing, chat, and attachment handling.
- **Assessments & Certifications**: Automated evaluation, verified certificate codes, and audit trails.
- **100% Course-Completion Refund Verification**: Multi-condition eligibility engines with asynchronous payout reconciliation.

---

## 2. High-Level Design (HLD)

### System Architecture

```
                                      [ Clients ]
                ┌───────────────────────────┼───────────────────────────┐
         Public Website               Student Portal               Mentor / Admin Portal
         (Next.js / SSR)             (SPA / Dashboard)             (Staff & Instructors)
                │                           │                               │
                └───────────────────────────┼───────────────────────────────┘
                                            ▼
                             [ Edge & Security Infrastructure ]
                             DNS · Cloudflare CDN · WAF · TLS
                                            │
                                            ▼
                               [ Load Balancer / Ingress ]
                               Rate Limiter · TLS Termination
                                            │
                                            ▼
                           [ API Gateway / Router (Gin) ]
                            Auth Middleware · Tracing (OTel)
                                            │
               ┌────────────────────────────┼────────────────────────────┐
               ▼                            ▼                            ▼
      [ Catalog Service ]          [ Identity Service ]        [ Commerce Service ]
      Courses, Modules,            JWT/Sessions, Roles,        Orders, Invoices, Webhooks,
      Search & Curriculum          OTP, Account Settings       Enrollments, Bundles
               │                            │                            │
               ▼                            ▼                            ▼
      [ Learning Service ]         [ Mentorship Service ]      [ Career & Refund Engine ]
      Progress Tracking,           1-on-1 Doubts, Chat,        Internships, Resumes,
      Videos, Assessments          Screen-share Signaling      Audited Refund Pipeline
               │                            │                            │
               └────────────────────────────┼────────────────────────────┘
                                            │
                     ┌──────────────────────┴──────────────────────┐
                     ▼                                             ▼
           [ Primary Database ]                           [ In-Memory Datastore ]
          PostgreSQL 16 (ACID)                             Redis 7 (Cache, Sessions,
         Orders, Enrollments, Users                         Rate Limits, Stream Queue)
                     │                                             │
                     ▼                                             ▼
          [ Object Storage (S3) ]                         [ Asynchronous Workers ]
          Encrypted Video Chunks,                         Video Transcoding, Emails,
          Submissions, Resumes                            Certificates, Refund Execution
```

### Recommended Technology Stack

| Layer | Component | Rationale |
|---|---|---|
| **API Server** | **Go 1.23+ / Gin** | High throughput, minimal memory footprint, strict compile-time types. |
| **Relational DB** | **PostgreSQL 16** | Strict ACID guarantees for commerce, enrollments, and progress ledgers. |
| **Cache & Queue** | **Redis 7+** | Distributed session locks, API rate-limiting tokens, fast cache lookups. |
| **Object Store** | **AWS S3 / Cloudflare R2** | Private media, assignments, resumes, and PDF certificate storage. |
| **Media Delivery** | **Cloudflare Stream / CDN** | Low-latency HLS/DASH video delivery using short-lived signed tokens. |
| **Observability** | **OpenTelemetry, Zap, Prometheus** | Structured JSON logs, latency tracing, and metrics instrumentation. |

---

## 3. Domain Modules

The platform is structured into clear business domains:

1. **`auth`**: User registration, password hashing (Argon2id/bcrypt), refresh token rotation, mobile OTP verification, and RBAC middleware.
2. **`catalog`**: Courses, categories, modules, lessons, SEO slugs, search autocomplete, and curriculum trees.
3. **`commerce`**: Price quotations, customizable bundles, checkout orders, payment intent lifecycle, signed webhook processing, and enrollment grants.
4. **`learning`**: Video playback authorization tokens, per-lesson progress persistence, and milestone tracking.
5. **`assessment`**: Assignment submissions, draft revisions, automated quizzes, and mentor review queues.
6. **`mentorship`**: 1-on-1 session scheduling, doubt ticket routing, message attachments, and WebRTC signaling.
7. **`refund`**: Course-completion eligibility calculations, fraud prevention checks, administrative approval workflows, and gateway payout execution.
8. **`career`**: Internship tracks, resumes, mock interviews, and partner placement opportunities.
9. **`admin`**: Audit logs, KPI aggregations, user role provisioning, and course publishing controls.

---

## 4. Low-Level Design (SLD) & Data Models

### Core Database Schema (PostgreSQL)

- **`users`**: UUID primary key, unique email, phone, hashed password, role (`student`, `mentor`, `admin`), active status.
- **`courses`**: Course metadata, slug, thumbnail, base price, refund eligibility flag, published status.
- **`course_modules`**: Structural grouping of modules per course with sequence ordering.
- **`lessons`**: Module lessons, video duration, asset reference key, preview availability flag.
- **`bundles` & `course_bundle_items`**: Pre-packaged and custom multi-course combinations.
- **`orders` & `order_items`**: Purchase orders, line items, currency, coupon discounts, order state machine (`pending`, `completed`, `failed`, `cancelled`).
- **`payments`**: Payment gateway transactions, provider transaction ID, idempotency key, webhook audit log.
- **`enrollments`**: Active course grants linking users to courses with enrollment lifecycle status.
- **`lesson_progress`**: High-frequency progress checkpoints (watched seconds, completion timestamp).
- **`assignments` & `submissions`**: Course assignments, student files, mentor scores, and feedback strings.
- **`certificates`**: Issued upon completion; includes unique verification hash and PDF download URL.
- **`refund_requests`**: State machine (`requested`, `under_review`, `approved`, `rejected`, `paid_out`).

### Entity Relationship Diagram

```mermaid
erDiagram
    USERS ||--o{ ORDERS : places
    USERS ||--o{ ENROLLMENTS : owns
    USERS ||--o{ LESSON_PROGRESS : records
    USERS ||--o{ SUBMISSIONS : submits
    COURSES ||--o{ COURSE_MODULES : contains
    COURSE_MODULES ||--o{ LESSONS : contains
    COURSES ||--o{ ENROLLMENTS : grants
    COURSES ||--o{ COURSE_BUNDLE_ITEMS : included_in
    BUNDLES ||--o{ COURSE_BUNDLE_ITEMS : contains
    ORDERS ||--o{ ORDER_ITEMS : contains
    ORDERS ||--o{ PAYMENTS : settles
    ENROLLMENTS ||--o{ ASSIGNMENTS : unlocks
    ENROLLMENTS ||--o{ CERTIFICATES : earns
    ENROLLMENTS ||--o{ REFUND_REQUESTS : may_request
    ASSIGNMENTS ||--o{ SUBMISSIONS : receives
```

---

## 5. API Endpoint Catalog

All routes assume the API version prefix `/api/v1`.

### Access Tiers
- **Public**: No authentication required.
- **Auth**: Valid Bearer Token (Authenticated Student/User).
- **Mentor**: Authenticated user with Mentor or Admin role.
- **Admin**: Privileged administrator.

---

### A. Catalog & Course Discovery

| Method | Endpoint | Description | Access |
|---|---|---|---|
| `GET` | `/home` | Featured courses, categories, and homepage metadata | Public |
| `GET` | `/courses` | Paginated course catalog with filters & sorting | Public |
| `GET` | `/courses/:slug` | Detailed course profile by slug | Public |
| `GET` | `/courses/:id/curriculum` | Course syllabus, modules, and public lesson outline | Public |
| `GET` | `/courses/:id/projects` | Hands-on projects included with course | Public |
| `GET` | `/courses/:id/reviews` | Verified student reviews and ratings | Public |
| `GET` | `/courses/:id/related` | Recommended related courses | Public |
| `GET` | `/categories` | List all active course categories | Public |
| `GET` | `/categories/:slug/courses` | Courses belonging to a specific category | Public |
| `GET` | `/search` | Full-text search across courses and bundles | Public |
| `GET` | `/search/suggestions` | Search query autocomplete suggestions | Public |
| `GET` | `/courses/:id/preview` | Preview/trial lesson streaming data | Public |
| `GET` | `/pages/:slug` | Static CMS pages (policies, terms, about) | Public |
| `GET` | `/faqs` | Platform-wide and category-specific FAQs | Public |

---

### B. Authentication & User Accounts

| Method | Endpoint | Description | Access |
|---|---|---|---|
| `POST` | `/auth/register` | Register new student account | Public |
| `POST` | `/auth/login` | Email/password login with JWT & refresh token issuance | Public |
| `POST` | `/auth/logout` | Invalidate current session/refresh token | Auth |
| `POST` | `/auth/refresh` | Rotate access token using valid refresh token | Auth |
| `POST` | `/auth/otp/send` | Send verification OTP via SMS/Email | Public |
| `POST` | `/auth/otp/verify` | Verify OTP and authenticate/confirm contact | Public |
| `POST` | `/auth/password/forgot` | Initiate password reset procedure | Public |
| `POST` | `/auth/password/reset` | Complete password reset via verification token | Public |
| `GET` | `/auth/me` | Fetch current authenticated user profile & roles | Auth |
| `PATCH` | `/users/me` | Update personal profile details | Auth |
| `GET` | `/users/me/sessions` | List active user device sessions | Auth |
| `DELETE` | `/users/me/sessions/:id` | Revoke specific device session | Auth |
| `DELETE` | `/users/me` | Request account deactivation / data deletion | Auth |

---

### C. Enrollment, Bundles & Checkout

| Method | Endpoint | Description | Access |
|---|---|---|---|
| `GET` | `/bundles` | List curated course combo packs | Public |
| `GET` | `/bundles/:slug` | Detailed bundle contents and pricing | Public |
| `POST` | `/bundles/custom` | Calculate pricing for user-selected custom bundle | Public |
| `POST` | `/checkout/quote` | Validate coupon codes and compute final order summary | Public |
| `POST` | `/checkout/orders` | Create pending order with idempotency check | Auth |
| `GET` | `/orders` | List user order history | Auth |
| `GET` | `/orders/:id` | Detailed order summary and invoice status | Auth |
| `POST` | `/payments/intent` | Create payment gateway intent/session | Auth |
| `GET` | `/payments/:id/status` | Poll payment confirmation status | Auth |
| `POST` | `/payments/webhooks/:provider` | Ingest signed payment provider webhooks | Signature |
| `GET` | `/enrollments` | List all active enrolled courses | Auth |
| `GET` | `/enrollments/:id` | Detailed enrollment state & expiry | Auth |
| `POST` | `/enrollments/:id/cancel` | Cancel enrollment within allowable window | Auth |
| `GET` | `/users/me/purchases` | Full transaction ledger for student | Auth |

---

### D. Student Learning Platform

| Method | Endpoint | Description | Access |
|---|---|---|---|
| `GET` | `/learning/dashboard` | Main student hub (recent courses, upcoming deadlines) | Auth |
| `GET` | `/learning/my-courses` | Active courses with completion percentage | Auth |
| `GET` | `/learning/courses/:id` | Course player navigation structure | Auth |
| `GET` | `/lessons/:id` | Verify access and generate signed video playback URL | Auth |
| `POST` | `/lessons/:id/progress` | Submit playback progress and heartbeat | Auth |
| `GET` | `/learning/progress` | Aggregate progress across modules | Auth |
| `GET` | `/modules/:id/lessons` | List all lessons in module | Auth |
| `GET` | `/resources/:id/download` | Generate signed download link for lesson attachment | Auth |
| `GET` | `/learning/continue` | Retrieve next uncompleted lesson in active course | Auth |

---

### E. Assignments, Projects & Assessments

| Method | Endpoint | Description | Access |
|---|---|---|---|
| `GET` | `/courses/:id/assignments` | List assignments for enrolled course | Auth |
| `GET` | `/assignments/:id` | Assignment instructions and starter files | Auth |
| `POST` | `/assignments/:id/submissions` | Upload assignment submission | Auth |
| `GET` | `/submissions/:id` | Check grading status and mentor comments | Auth |
| `PATCH` | `/submissions/:id` | Update draft submission prior to cutoff | Auth |
| `POST` | `/submissions/:id/resubmit` | Resubmit reworked solution if permitted | Auth |
| `GET` | `/courses/:id/assessments` | Quizzes and objective tests | Auth |
| `POST` | `/assessments/:id/attempts` | Start timed assessment attempt | Auth |
| `POST` | `/attempts/:id/answers` | Checkpoint intermediate answers | Auth |
| `POST` | `/attempts/:id/submit` | Finalize assessment attempt | Auth |
| `GET` | `/attempts/:id/results` | Assessment scorecard and breakdown | Auth |
| `GET` | `/projects/:id` | Capstone project guidelines | Auth |
| `POST` | `/projects/:id/submissions` | Submit capstone project code / repo URL | Auth |

---

### F. Certificates & Conditional Refunds

| Method | Endpoint | Description | Access |
|---|---|---|---|
| `GET` | `/certificates` | List earned credentials | Auth |
| `GET` | `/certificates/:id` | Certificate metadata and verification hash | Auth |
| `GET` | `/certificates/:id/download` | Download verifiable PDF certificate | Auth |
| `GET` | `/certificates/verify/:code` | Public certificate authenticity verification | Public |
| `GET` | `/refunds/eligibility/:enrollmentId` | Check 100% completion refund eligibility | Auth |
| `POST` | `/refunds/requests` | Submit course-completion refund application | Auth |
| `GET` | `/refunds/requests` | List user refund applications | Auth |
| `GET` | `/refunds/requests/:id` | Check refund review stage | Auth |
| `GET` | `/admin/refunds` | Staff queue of pending refund applications | Admin |
| `POST` | `/admin/refunds/:id/approve` | Approve refund request with audit reason | Admin |
| `POST` | `/admin/refunds/:id/reject` | Reject refund request with explanation | Admin |
| `POST` | `/admin/refunds/:id/execute` | Dispatch payout via payment gateway | Admin |

---

### G. Mentorship, Doubts & Live Sessions

| Method | Endpoint | Description | Access |
|---|---|---|---|
| `GET` | `/mentors` | Public mentor directory & domain expertise | Public |
| `GET` | `/mentors/:id` | Mentor profile and credentials | Public |
| `GET` | `/mentorship/sessions` | List scheduled 1-on-1 sessions | Auth |
| `POST` | `/mentorship/sessions` | Request/book mentor session | Auth |
| `GET` | `/mentorship/sessions/:id` | Session details and calendar invite | Auth |
| `POST` | `/mentorship/sessions/:id/cancel` | Cancel scheduled session | Auth |
| `GET` | `/mentorship/sessions/:id/join` | Authorize live video/audio meeting access | Auth |
| `POST` | `/doubts` | File instant doubt ticket | Auth |
| `GET` | `/doubts` | List active and resolved doubts | Auth |
| `GET` | `/doubts/:id` | Full thread history of doubt | Auth |
| `POST` | `/doubts/:id/messages` | Post question / follow-up message | Auth |
| `POST` | `/doubts/:id/attachments` | Upload code snippet or screenshot | Auth |
| `PATCH` | `/doubts/:id/status` | Mark doubt as resolved or escalated | Mentor/Admin |

---

### H. Internship & Career Services

| Method | Endpoint | Description | Access |
|---|---|---|---|
| `GET` | `/internships` | Browse affiliated internship programs | Public |
| `GET` | `/internships/:slug` | Internship prerequisites and details | Public |
| `POST` | `/internship-applications` | Apply for eligible internship track | Auth |
| `GET` | `/internship-applications/me` | Application status tracking | Auth |
| `GET` | `/career/profile` | View student career profile | Auth |
| `PATCH` | `/career/profile` | Update portfolio links and preferences | Auth |
| `POST` | `/career/resume` | Upload formatted resume | Auth |
| `GET` | `/career/opportunities` | Job board for certified students | Auth |
| `POST` | `/career/opportunities/:id/apply` | Direct application to placement partner | Auth |
| `POST` | `/career/mock-interviews` | Schedule mock technical interview | Auth |

---

### I. Reviews, Notifications & Support

| Method | Endpoint | Description | Access |
|---|---|---|---|
| `POST` | `/courses/:id/reviews` | Submit verified student review | Auth |
| `PATCH` | `/reviews/:id` | Update existing review | Auth |
| `DELETE` | `/reviews/:id` | Remove review | Auth |
| `GET` | `/notifications` | List user in-app notifications | Auth |
| `PATCH` | `/notifications/:id/read` | Mark notification as read | Auth |
| `POST` | `/notifications/read-all` | Mark all notifications as read | Auth |
| `POST` | `/support/tickets` | Open customer support inquiry | Auth |
| `GET` | `/support/tickets` | List support tickets | Auth |
| `GET` | `/support/tickets/:id` | View ticket correspondence | Auth |
| `POST` | `/support/tickets/:id/messages` | Reply to support ticket | Auth |
| `GET` | `/referrals/me` | Referral link, credits, and conversion stats | Auth |
| `GET` | `/promotions/validate` | Validate promo/discount code | Public |

---

### J. Mentor & Administration Operations

| Method | Endpoint | Description | Access |
|---|---|---|---|
| `GET` | `/mentor/dashboard` | Mentor metrics (assigned students, pending doubts) | Mentor |
| `GET` | `/mentor/students` | List assigned mentees | Mentor |
| `GET` | `/mentor/sessions` | Mentor session schedule | Mentor |
| `GET` | `/mentor/doubts` | Open doubt queue | Mentor |
| `POST` | `/mentor/doubts/:id/replies` | Reply and resolve doubt ticket | Mentor |
| `GET` | `/mentor/submissions` | Student assignment submission queue | Mentor |
| `POST` | `/mentor/submissions/:id/feedback` | Grade and provide review notes | Mentor |
| `GET` | `/admin/dashboard` | High-level platform KPIs | Admin |
| `GET` | `/admin/users` | User management & filtering | Admin |
| `PATCH` | `/admin/users/:id/status` | Ban, suspend, or activate user | Admin |
| `POST` | `/admin/courses` | Create new course draft | Admin |
| `PATCH` | `/admin/courses/:id` | Edit course metadata & pricing | Admin |
| `POST` | `/admin/courses/:id/publish` | Publish course to catalog | Admin |
| `POST` | `/admin/courses/:id/modules` | Add module to course | Admin |
| `POST` | `/admin/modules/:id/lessons` | Add lesson to module | Admin |
| `GET` | `/admin/orders` | Search all financial transactions | Admin |
| `GET` | `/admin/enrollments` | Manage student course access | Admin |
| `GET` | `/admin/reports/revenue` | Financial summaries and sales velocity | Admin |
| `GET` | `/admin/audit-logs` | Tamper-evident administrative action log | Admin |

---

## 6. Critical Business Flows

### 1. Order Checkout, Payment Webhook & Idempotent Enrollment

```
Student              API Server           PostgreSQL            Payment Gateway          Worker
   │                     │                    │                        │                   │
   │── POST /checkout ──>│                    │                        │                   │
   │   (Course/Bundle)   │── Create Order ───>│                        │                   │
   │                     │   (status=pending) │                        │                   │
   │<── Order & Intent ──│                    │                        │                   │
   │                     │                    │                        │                   │
   │────────── Complete Payment via Gateway ──────────────────────────>│                   │
   │                     │                    │                        │                   │
   │                     │<──────── POST /webhooks/provider ───────────│                   │
   │                     │         (HMAC Signature Validated)          │                   │
   │                     │                    │                        │                   │
   │                     │── Begin Tx ───────>│                        │                   │
   │                     │   Update Order     │                        │                   │
   │                     │   Insert Payment   │                        │                   │
   │                     │   Grant Enrollment │                        │                   │
   │                     │── Commit Tx ──────>│                        │                   │
   │                     │                    │                        │                   │
   │                     │── Publish 'order.completed' ───────────────────────────────────>│
   │                     │                    │                        │                   │ Send Welcome Email
   │<── 200 OK (Ack) ────│                    │                        │                   │ Invalidate Cache
```

### 2. Authorized Video Streaming & Progress Tracking
1. Student requests `GET /lessons/:id`.
2. Auth middleware verifies the session token.
3. Access policy checks whether an active enrollment exists for the parent course.
4. Server mints a short-lived signed video token (valid for 15 minutes).
5. Player streams video segments directly from CDN edge.
6. Progress heartbeat (`POST /lessons/:id/progress`) records watched offsets.
7. Background worker triggers course milestone evaluation when completion criteria are met.

### 3. Conditional Course-Completion Refund Pipeline
1. Student submits a refund application (`POST /refunds/requests`).
2. Engine computes eligibility:
   - Must have 100% lesson progress.
   - All required assignments and capstone projects graded and approved.
   - Request filed within the refund policy guarantee window.
3. Admin reviews the auditable log (`/admin/refunds`).
4. On approval, worker triggers payment gateway refund API with idempotency key.
5. Payout status is recorded, and enrollment status is transitioned to `refunded`.

---

## 7. Security & Reliability

- **Authentication**: JWT access tokens (15-min lifespan) paired with cryptographically secure, rotating refresh tokens stored in Redis with revocation capabilities.
- **Authorization**: Role-Based Access Control (RBAC) enforced at the middleware layer, plus resource-ownership validation (e.g. verifying `user_id` matches enrollment owner).
- **Payment Security**: Strict HMAC SHA-256 signature verification on all incoming gateway webhooks, with replay-attack defenses and database unique constraints on transaction IDs.
- **Media Protection**: Video storage buckets are fully private; playback is authorized exclusively through expiring HMAC signed tokens tied to student IP/subnet.
- **Rate Limiting**: Distributed token-bucket rate limiting via Redis on authentication, OTP generation, and checkout endpoints.
- **Audit Trails**: Non-repudiable audit logs recorded for administrative privileges, refund authorizations, and grade revisions.

---

## 8. Project Layout

The repository follows standard Go domain-driven package organization:

```
.
├── cmd/
│   ├── api/
│   │   └── main.go               # HTTP API server entry point
│   └── worker/
│       └── main.go               # Background queue consumer entry point
├── internal/
│   ├── admin/                    # Platform administration & reporting
│   ├── assessment/               # Quizzes, assignments, and capstone reviews
│   ├── auth/                     # Authentication, tokens, and password security
│   ├── career/                   # Internships, resumes, and opportunities
│   ├── catalog/                  # Courses, modules, lessons, and categories
│   ├── certificates/             # PDF generation and hash verification
│   ├── commerce/                 # Orders, bundles, checkout, and payments
│   ├── database/                 # PostgreSQL pool (pgx) and migration harness
│   ├── enrollment/               # Course grants and access policies
│   ├── learning/                 # Lesson progress and playback tokens
│   ├── mentorship/               # Doubts, 1-on-1 sessions, and signaling
│   ├── platform/                 # Shared configs, logging (Zap), and telemetry
│   └── refunds/                  # Completion verification and payout engine
├── migrations/                   # Sequential SQL schema migrations
├── files/                        # Incremental implementation files (f1.md - f43.md)
├── overview.md                   # System design & architecture blueprint
├── .gitignore                    # Production git ignore configuration
└── README.md                     # Comprehensive architecture documentation
```

---

## 9. Getting Started

### Prerequisites

- **Go**: `1.23.0` or later
- **PostgreSQL**: `16.0` or later
- **Redis**: `7.0` or later
- **Docker & Docker Compose** (optional, for local infrastructure)

### Local Environment Setup

1. **Clone the repository**:
   ```bash
   git clone https://github.com/kunal592/kdxlms.git
   cd kdxlms
   ```

2. **Configure Environment Variables**:
   Copy `.env.example` to `.env` and configure:
   ```bash
   APP_ENV=development
   HTTP_ADDR=:8080
   DATABASE_URL=postgres://postgres:postgres@localhost:5432/kdxlabsllm?sslmode=disable
   REDIS_URL=redis://localhost:6379/0
   JWT_SECRET=your-32-byte-secret-key-here-must-be-secure
   PAYMENT_WEBHOOK_SECRET=whsec_your_webhook_signing_secret
   STORAGE_BUCKET=kdxlabs-media-dev
   ```

3. **Start Backing Services (Docker Compose)**:
   ```bash
   docker compose up -d postgres redis
   ```

4. **Run Migrations & Start API**:
   ```bash
   go run ./cmd/api/main.go
   ```

5. **Run Test Suite**:
   ```bash
   go test -v -race ./...
   ```

---

## 10. Architecture Knowledge Graph

This repository is indexed into a persistent GraphRAG knowledge graph using **Graphify**.

- **Inspect Graph Metrics**:
  ```bash
  graphify_stats
  ```
- **Explore Architecture Relationships**:
  ```bash
  graphify query "How does checkout grant course enrollment?"
  graphify path "CommerceService" "EnrollmentService"
  ```
- **Interactive Visualizer**:
  Open `graphify-out/graph.html` in your browser to inspect community clustering, god nodes, and inter-service call graphs.

---

## 11. Developer & Maintainer

**Developed and engineered by [KDXLabs](https://www.kdxlabs.cloud)**
- **Website**: [www.kdxlabs.cloud](https://www.kdxlabs.cloud)
- **Organization**: KDXLabs Engineering & AI Backend Architecture
- **Inquiries**: [contact@kdxlabs.cloud](mailto:contact@kdxlabs.cloud) or visit [www.kdxlabs.cloud](https://www.kdxlabs.cloud)

