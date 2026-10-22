# TuteDude — API Endpoints, HLD, SLD & System Design

Website: TuteDude — [tutedude.com](https://www.tutedude.com/?utm_source=chatgpt.com)

I'll approach this as a reverse-engineering and system-design document for an online learning platform, covering public pages, backend API endpoints, high-level design (HLD), low-level design (SLD), database models, authentication, and deployment.

One important distinction: a website's public pages do not reveal every backend API. To produce a truly exhaustive endpoint list, we would need to inspect its publicly accessible pages, browser network requests, and any published API documentation. I'll distinguish endpoints that are verified from proposed endpoints that represent how a platform like TuteDude could be designed, rather than inventing routes and claiming they exist.

## 1. What the public website reveals

The current public search results show several functional areas, not just a course catalog:

[Tutedude.in: ¿es seguro? (Confianza 70/100)](https://images.openai.com/static-rsc-4/UQEkktNgpcRHVDUkWSmO88oArStUoc9Gn2EwosqlDas9z9W1vxWydTbAkYU_ppw8YTxrfr4YoG2fxQ24JxuRxyxtZOP2xaymOYBreorrFHfhhyv7ONCZTqiOIBCMQ1rHgualWmwrLxmO7ru_RVCY4phlk4QkwX-ugYcbvkzRASs?purpose=inline)

Course discovery and sales

Browse courses, filter by category, view curriculum and projects, and explore course bundles.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.tutedude.com\&sz=32)

tutedude

+2



[tutedude.com - Tutedude: Online Tech Courses ... - Tutedude](https://images.openai.com/static-rsc-4/C7h8giXGqj_YY4B6A8Owb88K43KySSk5X7Y_-oS6DzfCXDRISO93aLXzjWUal85z25x7A8FuMP2bdZOKzziRTHtVWiMQUUYGqBL_rF_kh-O-XVzjlO2vANgI8IZDbAFryyKhPLG8Wil1jjlYbTUorDeQFU4HCWaCAHXTeXU4BfA?purpose=inline)

Learning and student services

Recorded lectures, assignments, projects, certificates, mentorship, and course-completion refund offers.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.tutedude.com\&sz=32)

tutedude

+1



[Is tutedude.com Safe? Security Scan Report | PCrisk](https://images.openai.com/static-rsc-4/zXkuQHqBeIm0YkaH_DEzhYJ3x_xBueCATPsCSGmB3iK4yIkDWu9-TaaOuPYTTJGhH9n5Dm032tsI2lNktEDabbZIPFvmhTVgCYIwyzTVsUpfH8oTg_d9MN-fuOcjQ_DOMuyTSC0W9Gc7ELgDW8nd4v6IO3pgCG-lBNKE42vu--o?purpose=inline)

Mentor portal

A separate mentor login interface is publicly indexed at [chat.tutedude.com](https://chat.tutedude.com/?utm_source=chatgpt.com).&#x20;

[image](https://www.google.com/s2/favicons?domain=https://chat.tutedude.com\&sz=32)

chat.tutedude.com



### Public page routes identified

These are page URLs, not backend API endpoints.

| Route                               | Purpose                          |
| ----------------------------------- | -------------------------------- |
| `/`                                 | Homepage and course discovery    |
| `/category/python`                  | Python course details            |
| `/category/datastructure`           | Data structures course           |
| `/category/customizepack`           | Custom course bundle             |
| `/category/upgrade`                 | Course upgrade options           |
| `/category/registration`            | Course registration/landing page |
| `/categoryV2/videoediting`          | Video editing course page        |
| `/combopack/fullstackdev`           | Full-stack developer bundle      |
| `/login/` on `webmail.tutedude.com` | Login-related public route       |
| `/` on `chat.tutedude.com`          | Mentor login                     |

Sources: [Official homepage](https://www.tutedude.com/?utm_source=chatgpt.com) · [Course catalog example](https://www.tutedude.com/category/python?utm_source=chatgpt.com) · [Bundle example](https://www.tutedude.com/category/customizepack?utm_source=chatgpt.com) · [Mentor portal](https://chat.tutedude.com/?utm_source=chatgpt.com).&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.tutedude.com\&sz=32)

tutedude

+3



This is an initial list of discovered routes, not a claim that every route on the website has been enumerated.

## 2. Complete proposed backend API endpoint inventory

The website's internal API routes have not been independently verified. The following is a comprehensive proposed API specification for recreating the publicly visible features and the likely supporting student, mentor, and administration workflows.

Assumed API prefix: `/api/v1`

Legend:

-

  Public

  &#x20;No user login required.


-

  Auth

  &#x20;Authenticated user required.


-

  Admin

  &#x20;Privileged staff or administrator required.



### A. Homepage, search and course catalog

| Method | Endpoint                    | Purpose                             | Access |
| ------ | --------------------------- | ----------------------------------- | ------ |
| GET    | `/home`                     | Homepage content and featured items | Public |
| GET    | `/courses`                  | Paginated course catalog            | Public |
| GET    | `/courses/:slug`            | Course detail                       | Public |
| GET    | `/courses/:id/curriculum`   | Course syllabus and modules         | Public |
| GET    | `/courses/:id/projects`     | Course projects                     | Public |
| GET    | `/courses/:id/reviews`      | Course reviews                      | Public |
| GET    | `/courses/:id/related`      | Related courses                     | Public |
| GET    | `/categories`               | List course categories              | Public |
| GET    | `/categories/:slug/courses` | Courses by category                 | Public |
| GET    | `/search`                   | Search courses and bundles          | Public |
| GET    | `/search/suggestions`       | Search autocomplete                 | Public |
| GET    | `/courses/:id/preview`      | Preview course content              | Public |
| GET    | `/pages/:slug`              | Static pages, policies and FAQs     | Public |
| GET    | `/faqs`                     | Frequently asked questions          | Public |

Query parameters for `GET /courses` could include `page`, `limit`, `category`, `search`, `sort`, `minPrice`, and `maxPrice`.

### B. Authentication and user accounts

| Method | Endpoint                 | Purpose                              | Access |
| ------ | ------------------------ | ------------------------------------ | ------ |
| POST   | `/auth/register`         | Create account                       | Public |
| POST   | `/auth/login`            | Login                                | Public |
| POST   | `/auth/logout`           | End session                          | Auth   |
| POST   | `/auth/refresh`          | Refresh access token/session         | Auth   |
| POST   | `/auth/otp/send`         | Send verification OTP                | Public |
| POST   | `/auth/otp/verify`       | Verify OTP                           | Public |
| POST   | `/auth/password/forgot`  | Request password reset               | Public |
| POST   | `/auth/password/reset`   | Reset password                       | Public |
| GET    | `/auth/me`               | Current user profile and permissions | Auth   |
| PATCH  | `/users/me`              | Update profile                       | Auth   |
| GET    | `/users/me/sessions`     | List active sessions                 | Auth   |
| DELETE | `/users/me/sessions/:id` | Revoke a session                     | Auth   |
| DELETE | `/users/me`              | Request account deletion             | Auth   |

If social sign-in is supported, add OAuth authorization and callback routes. Their actual providers need verification.

### C. Enrollment, course bundles and checkout

TuteDude publicly presents individual courses and multi-course packages, including customizable bundles.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.tutedude.com\&sz=32)

tutedude

+1



| Method | Endpoint                       | Purpose                           | Access             |
| ------ | ------------------------------ | --------------------------------- | ------------------ |
| GET    | `/bundles`                     | List available bundles            | Public             |
| GET    | `/bundles/:slug`               | Bundle details                    | Public             |
| POST   | `/bundles/custom`              | Calculate a custom bundle         | Public             |
| POST   | `/checkout/quote`              | Calculate final price             | Public             |
| POST   | `/checkout/orders`             | Create pending order              | Auth               |
| GET    | `/orders`                      | List user's orders                | Auth               |
| GET    | `/orders/:id`                  | Order details                     | Auth               |
| POST   | `/payments/intent`             | Create payment attempt            | Auth               |
| GET    | `/payments/:id/status`         | Check payment status              | Auth               |
| POST   | `/payments/webhooks/:provider` | Receive payment-provider events   | Provider signature |
| GET    | `/enrollments`                 | List enrolled courses             | Auth               |
| GET    | `/enrollments/:id`             | Enrollment details                | Auth               |
| POST   | `/enrollments/:id/cancel`      | Request cancellation, if eligible | Auth               |
| GET    | `/users/me/purchases`          | Purchase history                  | Auth               |

Payment rule: Create the enrollment only after the payment provider confirms successful payment. Use idempotency keys and signed webhook validation to prevent duplicate orders or enrollments.

### D. Student learning platform

| Method | Endpoint                  | Purpose                          | Access |
| ------ | ------------------------- | -------------------------------- | ------ |
| GET    | `/learning/dashboard`     | Student dashboard                | Auth   |
| GET    | `/learning/my-courses`    | Enrolled courses                 | Auth   |
| GET    | `/learning/courses/:id`   | Course learning view             | Auth   |
| GET    | `/lessons/:id`            | Lesson metadata and access check | Auth   |
| POST   | `/lessons/:id/progress`   | Save lesson progress             | Auth   |
| GET    | `/learning/progress`      | Overall learning progress        | Auth   |
| GET    | `/modules/:id/lessons`    | Lessons in a module              | Auth   |
| GET    | `/resources/:id/download` | Download authorized resource     | Auth   |
| GET    | `/learning/continue`      | Resume last lesson               | Auth   |

For video delivery, the backend should issue short-lived playback URLs or tokens rather than exposing private video-storage credentials.

### E. Assignments, projects and assessments

| Method | Endpoint                       | Purpose                        | Access |
| ------ | ------------------------------ | ------------------------------ | ------ |
| GET    | `/courses/:id/assignments`     | List assignments               | Auth   |
| GET    | `/assignments/:id`             | Assignment details             | Auth   |
| POST   | `/assignments/:id/submissions` | Submit assignment              | Auth   |
| GET    | `/submissions/:id`             | Submission status and feedback | Auth   |
| PATCH  | `/submissions/:id`             | Update a draft submission      | Auth   |
| POST   | `/submissions/:id/resubmit`    | Resubmit if permitted          | Auth   |
| GET    | `/courses/:id/assessments`     | List assessments               | Auth   |
| POST   | `/assessments/:id/attempts`    | Start an attempt               | Auth   |
| POST   | `/attempts/:id/answers`        | Save answers                   | Auth   |
| POST   | `/attempts/:id/submit`         | Submit assessment              | Auth   |
| GET    | `/attempts/:id/results`        | Retrieve result                | Auth   |
| GET    | `/projects/:id`                | Project requirements           | Auth   |
| POST   | `/projects/:id/submissions`    | Submit project                 | Auth   |

### F. Certificates and refunds

The refund and certification workflows are important because the public site advertises course completion, assignment completion, certificates, and conditional refunds. Exact eligibility rules should be taken from the applicable offer and policy.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.tutedude.com\&sz=32)

tutedude

+2



| Method | Endpoint                             | Purpose                         | Access |
| ------ | ------------------------------------ | ------------------------------- | ------ |
| GET    | `/certificates`                      | List certificates               | Auth   |
| GET    | `/certificates/:id`                  | Certificate details             | Auth   |
| GET    | `/certificates/:id/download`         | Download certificate            | Auth   |
| GET    | `/certificates/verify/:code`         | Public certificate verification | Public |
| GET    | `/refunds/eligibility/:enrollmentId` | Check refund eligibility        | Auth   |
| POST   | `/refunds/requests`                  | Request a refund                | Auth   |
| GET    | `/refunds/requests`                  | List refund requests            | Auth   |
| GET    | `/refunds/requests/:id`              | Refund status                   | Auth   |
| GET    | `/admin/refunds`                     | Review refund queue             | Admin  |
| POST   | `/admin/refunds/:id/approve`         | Approve refund                  | Admin  |
| POST   | `/admin/refunds/:id/reject`          | Reject refund with reason       | Admin  |
| POST   | `/admin/refunds/:id/execute`         | Execute an approved refund      | Admin  |

Refund execution should be asynchronous, auditable and idempotent. Approval alone must not be treated as proof that money has been returned.

### G. Mentorship, doubts and live sessions

The official website describes one-to-one mentorship and doubt-solving services, including chat, audio and screen sharing.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.tutedude.com\&sz=32)

tutedude

+1



| Method | Endpoint                          | Purpose                          | Access       |
| ------ | --------------------------------- | -------------------------------- | ------------ |
| GET    | `/mentors`                        | Browse mentors                   | Public       |
| GET    | `/mentors/:id`                    | Mentor profile                   | Public       |
| GET    | `/mentorship/sessions`            | List a user's sessions           | Auth         |
| POST   | `/mentorship/sessions`            | Request a session                | Auth         |
| GET    | `/mentorship/sessions/:id`        | Session details                  | Auth         |
| POST   | `/mentorship/sessions/:id/cancel` | Cancel a session                 | Auth         |
| GET    | `/mentorship/sessions/:id/join`   | Obtain authorized meeting access | Auth         |
| POST   | `/doubts`                         | Create a doubt request           | Auth         |
| GET    | `/doubts`                         | List user's doubts               | Auth         |
| GET    | `/doubts/:id`                     | Doubt details                    | Auth         |
| POST   | `/doubts/:id/messages`            | Send a doubt message             | Auth         |
| POST   | `/doubts/:id/attachments`         | Upload an attachment             | Auth         |
| PATCH  | `/doubts/:id/status`              | Update doubt status              | Mentor/Admin |

Real-time chat and session signaling can use WebSockets or a managed real-time service, with REST APIs for persistence and authorization.

### H. Internship and career services

| Method | Endpoint                          | Purpose                  | Access |
| ------ | --------------------------------- | ------------------------ | ------ |
| GET    | `/internships`                    | Browse internship tracks | Public |
| GET    | `/internships/:slug`              | Internship details       | Public |
| POST   | `/internship-applications`        | Apply to a program       | Auth   |
| GET    | `/internship-applications/me`     | Track applications       | Auth   |
| GET    | `/career/profile`                 | Career profile           | Auth   |
| PATCH  | `/career/profile`                 | Update career profile    | Auth   |
| POST   | `/career/resume`                  | Upload resume            | Auth   |
| GET    | `/career/opportunities`           | Available opportunities  | Auth   |
| POST   | `/career/opportunities/:id/apply` | Apply for an opportunity | Auth   |
| POST   | `/career/mock-interviews`         | Request mock interview   | Auth   |

### I. Reviews, notifications, support and referrals

| Method | Endpoint                        | Purpose                   | Access |
| ------ | ------------------------------- | ------------------------- | ------ |
| POST   | `/courses/:id/reviews`          | Submit a review           | Auth   |
| PATCH  | `/reviews/:id`                  | Edit own review           | Auth   |
| DELETE | `/reviews/:id`                  | Remove own review         | Auth   |
| GET    | `/notifications`                | List notifications        | Auth   |
| PATCH  | `/notifications/:id/read`       | Mark notification as read | Auth   |
| POST   | `/notifications/read-all`       | Mark all as read          | Auth   |
| POST   | `/support/tickets`              | Create support ticket     | Auth   |
| GET    | `/support/tickets`              | List support tickets      | Auth   |
| GET    | `/support/tickets/:id`          | Ticket details            | Auth   |
| POST   | `/support/tickets/:id/messages` | Reply to ticket           | Auth   |
| GET    | `/referrals/me`                 | Referral status           | Auth   |
| GET    | `/promotions/validate`          | Validate promotional code | Public |

### J. Mentor and administrator APIs

| Method | Endpoint                           | Purpose               | Access |
| ------ | ---------------------------------- | --------------------- | ------ |
| GET    | `/mentor/dashboard`                | Mentor dashboard      | Mentor |
| GET    | `/mentor/students`                 | Assigned students     | Mentor |
| GET    | `/mentor/sessions`                 | Assigned sessions     | Mentor |
| GET    | `/mentor/doubts`                   | Assigned doubt queue  | Mentor |
| POST   | `/mentor/doubts/:id/replies`       | Respond to a doubt    | Mentor |
| GET    | `/mentor/submissions`              | Review queue          | Mentor |
| POST   | `/mentor/submissions/:id/feedback` | Submit feedback       | Mentor |
| GET    | `/admin/dashboard`                 | Admin KPIs            | Admin  |
| GET    | `/admin/users`                     | Search users          | Admin  |
| PATCH  | `/admin/users/:id/status`          | Change account status | Admin  |
| POST   | `/admin/courses`                   | Create course         | Admin  |
| PATCH  | `/admin/courses/:id`               | Update course         | Admin  |
| POST   | `/admin/courses/:id/publish`       | Publish course        | Admin  |
| POST   | `/admin/courses/:id/modules`       | Create module         | Admin  |
| POST   | `/admin/modules/:id/lessons`       | Create lesson         | Admin  |
| GET    | `/admin/orders`                    | Search orders         | Admin  |
| GET    | `/admin/enrollments`               | Search enrollments    | Admin  |
| GET    | `/admin/reports/revenue`           | Revenue reporting     | Admin  |
| GET    | `/admin/audit-logs`                | Audit events          | Admin  |

This endpoint inventory covers the major domains needed for an implementation. It is not evidence that TuteDude has these exact routes or supports every proposed operation.

## 3. HLD — High-Level Design

The HLD explains the main components and how data flows between them.

Client applications

Public website · Student portal · Mentor portal · Admin portal

Edge and security layer

DNS · CDN · WAF · TLS · Rate limiting

Load balancer / API gateway

Routing · Authentication enforcement · Request limits

Catalog service

Courses, categories, search, curriculum

Identity service

Login, sessions, roles, permissions

Commerce service

Orders, payments, enrollments, refunds

Learning service

Lessons, progress, assignments, certificates

Mentor service

Doubts, sessions, feedback

Career service

Internships, resumes, opportunities

Data and asynchronous infrastructure

PostgreSQL · Redis · Object storage · Search index · Message queue · Workers

External integrations

Payment gateway · Email/SMS · Video delivery · Meeting provider

This is a recommended architecture, not a diagram of TuteDude's confirmed internal infrastructure.

### Recommended technology stack

| Layer           | Recommendation                               | Reason                                          |
| --------------- | -------------------------------------------- | ----------------------------------------------- |
| Frontend        | Next.js + TypeScript                         | SEO and interactive learning interfaces         |
| Backend         | Go or NestJS                                 | Typed API and business logic                    |
| Database        | PostgreSQL                                   | Orders, users, enrollments, progress            |
| Cache           | Redis                                        | Sessions, rate limits, frequently accessed data |
| File storage    | S3-compatible object storage                 | Videos, assignments, resumes, certificates      |
| CDN             | Cloudflare or equivalent                     | Static assets and video delivery                |
| Search          | PostgreSQL initially; dedicated search later | Course discovery                                |
| Background jobs | Redis-backed queue or managed queue          | Email, refunds, certificates                    |
| Payments        | Payment provider with signed webhooks        | Payment processing                              |
| Observability   | OpenTelemetry, Prometheus and Grafana        | Monitoring and troubleshooting                  |
| Deployment      | Docker, managed containers and CI/CD         | Repeatable releases and scaling                 |

Architecture decision: Start with a modular monolith rather than immediately building numerous microservices. Split out video processing, notifications, search or other independently scaling workloads when actual demand justifies it.

## 4. SLD — Low-Level Design

The SLD defines the internal modules, data structures, authorization rules, and transaction behavior.

### Core database schema

\#chatgpt-mermaid-\_r_pb\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(237, 237, 237);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_pb\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_pb\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_pb\_ .error-icon{fill:rgb(48, 48, 48);}#chatgpt-mermaid-\_r_pb\_ .error-text{fill:rgb(237, 237, 237);stroke:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_pb\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_pb\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_pb\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_pb\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_pb\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_pb\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_pb\_ .marker{fill:rgb(175, 175, 175);stroke:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_pb\_ .marker.cross{stroke:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_pb\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_pb\_ p{margin:0;}#chatgpt-mermaid-\_r_pb\_ .entityBox{fill:rgb(9, 23, 44);stroke:rgb(31, 78, 148);}#chatgpt-mermaid-\_r_pb\_ .relationshipLabelBox{fill:rgb(48, 48, 48);opacity:0.7;background-color:rgb(48, 48, 48);}#chatgpt-mermaid-\_r_pb\_ .relationshipLabelBox rect{opacity:0.5;}#chatgpt-mermaid-\_r_pb\_ .labelBkg{background-color:rgba(48, 48, 48, 0.5);}#chatgpt-mermaid-\_r_pb\_ .edgeLabel{background-color:rgb(0, 0, 0);}#chatgpt-mermaid-\_r_pb\_ .edgeLabel .label rect{fill:rgb(0, 0, 0);}#chatgpt-mermaid-\_r_pb\_ .edgeLabel .label text{fill:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_pb\_ .edgeLabel .label{fill:rgb(31, 78, 148);font-size:14px;}#chatgpt-mermaid-\_r_pb\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_pb\_ .edge-pattern-dashed{stroke-dasharray:8,8;}#chatgpt-mermaid-\_r_pb\_ .node rect,#chatgpt-mermaid-\_r_pb\_ .node circle,#chatgpt-mermaid-\_r_pb\_ .node ellipse,#chatgpt-mermaid-\_r_pb\_ .node polygon{fill:rgb(9, 23, 44);stroke:rgb(31, 78, 148);stroke-width:1px;}#chatgpt-mermaid-\_r_pb\_ .relationshipLine{stroke:rgb(175, 175, 175);stroke-width:1px;fill:none;}#chatgpt-mermaid-\_r_pb\_ .marker{fill:none!important;stroke:rgb(175, 175, 175)!important;stroke-width:1;}#chatgpt-mermaid-\_r_pb\_ [data-look=neo].labelBkg{background-color:rgba(48, 48, 48, 0.5);}#chatgpt-mermaid-\_r_pb\_ .node .neo-node{stroke:rgb(31, 78, 148);}#chatgpt-mermaid-\_r_pb\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_pb\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_pb\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_pb\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_pb\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_pb\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_pb\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_pb\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_pb\_ [data-look="neo"].node .neo-line path{stroke:rgb(31, 78, 148);filter:none;}#chatgpt-mermaid-\_r_pb\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_pb\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_pb\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_pb\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_pb\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_pb\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_pb\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_pb\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}USERSuuididPKstringemailUKstringpassword_hashstringstatustimestampcreated_atORDERSuuididPKuuiduser_idFKinttotal_minorstringcurrencystringstatusENROLLMENTSuuididPKuuiduser_idFKuuidcourse_idFKstringstatusLESSON_PROGRESSuuididPKuuiduser_idFKuuidlesson_idFKintprogress_secondsCOURSESuuididPKstringslugUKstringtitlestringstatusintprice_minorCOURSE_MODULESuuididPKuuidcourse_idFKstringtitleintpositionLESSONSuuididPKuuidmodule_idFKstringtitlestringasset_keyCOURSE_BUNDLE_ITEMSuuidbundle_idFKuuidcourse_idFKBUNDLESuuididPKstringnameintprice_minorORDER_ITEMSuuididPKuuidorder_idFKuuidcourse_idFKintprice_minorPAYMENTSuuididPKuuidorder_idFKstringprovider_referencestringstatusASSIGNMENTSuuididPKuuidenrollment_idFKstringtitleSUBMISSIONSuuididPKuuidassignment_idFKuuiduser_idFKstringstatusstringfeedbackCERTIFICATESuuididPKuuidenrollment_idFKstringverification_codeUKREFUND_REQUESTSuuididPKuuidenrollment_idFKstringstatusintamount_minorplacesownstrackscontainscontainsincluded_incontainscontainspurchasedhasunlocksreceivessubmitsearnsmay_request

This is a starting schema. A production implementation would also need roles, sessions, OTP attempts, audit logs, payment webhook records, discounts, mentor assignments, support tickets and more. Bundle purchases should create the appropriate enrollments without creating duplicate course access.

### Core backend module structure

For a Go implementation, one possible repository layout is:

```
tutedude-platform/
├── cmd/
│   ├── api/
│   └── worker/
├── internal/
│   ├── auth/
│   │   ├── handler.go
│   │   ├── service.go
│   │   ├── repository.go
│   │   └── middleware.go
│   ├── catalog/
│   ├── commerce/
│   ├── enrollment/
│   ├── learning/
│   ├── assignments/
│   ├── certificates/
│   ├── refunds/
│   ├── mentorship/
│   ├── career/
│   ├── notifications/
│   ├── admin/
│   └── platform/
├── migrations/
├── api/
│   └── openapi.yaml
├── deployments/
├── Dockerfile
└── go.mod
```

Each business module should expose handlers, services and repository interfaces. Handlers validate HTTP input, services enforce business rules, and repositories perform database operations.

### Example request flow: purchase and enrollment

Job QueuePayment ProviderPostgreSQLCommerce ServiceAPI ServerJob QueuePayment ProviderPostgreSQLCommerce ServiceAPI Server#chatgpt-mermaid-\_r_pq\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(237, 237, 237);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_pq\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_pq\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_pq\_ .error-icon{fill:rgb(48, 48, 48);}#chatgpt-mermaid-\_r_pq\_ .error-text{fill:rgb(237, 237, 237);stroke:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_pq\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_pq\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_pq\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_pq\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_pq\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_pq\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_pq\_ .marker{fill:rgb(175, 175, 175);stroke:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_pq\_ .marker.cross{stroke:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_pq\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_pq\_ p{margin:0;}#chatgpt-mermaid-\_r_pq\_ .actor{stroke:rgb(31, 78, 148);fill:rgb(9, 23, 44);stroke-width:1;}#chatgpt-mermaid-\_r_pq\_ rect.actor.outer-path[data-look="neo"]{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_pq\_ rect.note[data-look="neo"]{stroke:rgb(58, 132, 63);fill:rgb(48, 48, 48);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_pq\_ text.actor>tspan{fill:rgb(237, 237, 237);stroke:none;}#chatgpt-mermaid-\_r_pq\_ .actor-line{stroke:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_pq\_ .innerArc{stroke-width:1.5;stroke-dasharray:none;}#chatgpt-mermaid-\_r_pq\_ .messageLine0{stroke-width:1.5;stroke-dasharray:none;stroke:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_pq\_ .messageLine1{stroke-width:1.5;stroke-dasharray:2,2;stroke:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_pq\_ [id$="-arrowhead"] path{fill:rgb(175, 175, 175);stroke:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_pq\_ .sequenceNumber{fill:#505050;}#chatgpt-mermaid-\_r_pq\_ [id$="-sequencenumber"]{fill:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_pq\_ [id$="-crosshead"] path{fill:rgb(175, 175, 175);stroke:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_pq\_ .messageText{fill:rgb(237, 237, 237);stroke:none;}#chatgpt-mermaid-\_r_pq\_ .labelBox{stroke:rgba(255, 255, 255, 0.15);fill:rgb(0, 0, 0);filter:none;}#chatgpt-mermaid-\_r_pq\_ .labelText,#chatgpt-mermaid-\_r_pq\_ .labelText>tspan{fill:rgb(237, 237, 237);stroke:none;}#chatgpt-mermaid-\_r_pq\_ .loopText,#chatgpt-mermaid-\_r_pq\_ .loopText>tspan{fill:rgb(237, 237, 237);stroke:none;}#chatgpt-mermaid-\_r_pq\_ .sectionTitle,#chatgpt-mermaid-\_r_pq\_ .sectionTitle>tspan{fill:rgb(237, 237, 237);stroke:none;}#chatgpt-mermaid-\_r_pq\_ .loopLine{stroke-width:2px;stroke-dasharray:2,2;stroke:rgba(255, 255, 255, 0.15);fill:rgba(255, 255, 255, 0.15);}#chatgpt-mermaid-\_r_pq\_ .note{stroke:rgb(58, 132, 63);fill:rgb(48, 48, 48);}#chatgpt-mermaid-\_r_pq\_ .noteText,#chatgpt-mermaid-\_r_pq\_ .noteText>tspan{fill:rgb(237, 237, 237);stroke:none;font-weight:normal;}#chatgpt-mermaid-\_r_pq\_ .activation0{fill:rgb(48, 48, 48);stroke:hsl(0, 0%, 8.8235294118%);}#chatgpt-mermaid-\_r_pq\_ .activation1{fill:rgb(48, 48, 48);stroke:hsl(0, 0%, 8.8235294118%);}#chatgpt-mermaid-\_r_pq\_ .activation2{fill:rgb(48, 48, 48);stroke:hsl(0, 0%, 8.8235294118%);}#chatgpt-mermaid-\_r_pq\_ .actorPopupMenu{position:absolute;}#chatgpt-mermaid-\_r_pq\_ .actorPopupMenuPanel{position:absolute;fill:rgb(9, 23, 44);box-shadow:0px 8px 16px 0px rgba(0,0,0,0.2);filter:drop-shadow(3px 5px 2px rgb(0 0 0 / 0.4));}#chatgpt-mermaid-\_r_pq\_ .actor-man circle,#chatgpt-mermaid-\_r_pq\_ line{fill:rgb(9, 23, 44);stroke-width:2px;}#chatgpt-mermaid-\_r_pq\_ g rect.rect{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));stroke:rgb(31, 78, 148);}#chatgpt-mermaid-\_r_pq\_ .node .neo-node{stroke:rgb(31, 78, 148);}#chatgpt-mermaid-\_r_pq\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_pq\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_pq\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_pq\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_pq\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_pq\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_pq\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_pq\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_pq\_ [data-look="neo"].node .neo-line path{stroke:rgb(31, 78, 148);filter:none;}#chatgpt-mermaid-\_r_pq\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_pq\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_pq\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_pq\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_pq\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_pq\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_pq\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_pq\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}StudentPOST /checkout/ordersValidate cart and priceCreate pending orderOrder IDOrder and payment detailsCheckout responseComplete paymentSigned payment webhookVerify event and amountTransaction: payment + enrollmentsPublish purchase eventAcknowledge webhookTrigger confirmation workflowStudent

The webhook may be retried, so the payment event must be processed idempotently. The enrollment transaction should commit only after the payment is validated.

### Example request flow: watch a lesson

1. The student requests `GET /lessons/:id`.
2. The API authenticates the student.
3. The learning service checks active enrollment and lesson availability.
4. The backend generates a short-lived media playback authorization.
5. The CDN delivers the video.
6. Progress updates are recorded through `POST /lessons/:id/progress`.
7. A background worker recalculates completion eligibility when required.

Never trust a frontend-supplied `userId`, completed percentage or refund eligibility decision without validating it on the server.

## 5. Security and reliability requirements

| Area            | Required design                                                           |
| --------------- | ------------------------------------------------------------------------- |
| Authentication  | Secure sessions or short-lived access tokens with rotation and revocation |
| Authorization   | RBAC plus resource-level ownership checks                                 |
| Payments        | Signed webhooks, idempotency and transaction-safe order updates           |
| Refunds         | Explicit eligibility rules, audit trails and provider reconciliation      |
| Media           | Private storage, signed URLs and expiring access                          |
| API abuse       | Rate limits, input validation, payload limits and abuse detection         |
| Data protection | TLS, encrypted secrets, restricted database access                        |
| Privacy         | Minimize personal data, enforce retention and deletion policies           |
| Reliability     | Retries, dead-letter queues, timeouts and structured logs                 |
| Monitoring      | API latency, payment failures, queue lag and enrollment errors            |

For the admin portal, separate privileges for course management, support, payment reconciliation and refund approval. Do not give every staff member unrestricted access to financial or student data.

## 6. Deployment and scaling

A practical initial deployment would use a CDN and WAF in front of a load balancer, two or more stateless API instances, managed PostgreSQL, Redis, object storage and a separate background worker. Add database backups, point-in-time recovery and monitoring before scaling up.

As demand grows:

- Cache public course pages and category responses.
- Use a CDN for videos and static assets.
- Index course slugs, categories, enrollment lookups and order references.
- Keep progress updates lightweight and batch them where appropriate.
- Partition or archive high-volume event and progress tables when needed.
- Scale workers separately from API instances.
- Use a transactional outbox when reliable event publication becomes necessary.

## 7. What is needed to produce the actual endpoint list?

The public pages are only the first part of the investigation. To identify real API routes, collect browser Network-tab evidence while using the website normally, or provide an authorized HAR file with sensitive data removed.

The evidence should include:

- Request URL and HTTP method
- Request and response content types
- Sanitized request and response bodies
- Status codes and pagination parameters
- Whether the route requires authentication
- Any API schema or OpenAPI document you are authorized to access

A HAR can contain session cookies, bearer tokens, personal information and payment data, so remove these before sharing it.

Recommended next step: Treat the endpoint tables above as the proposed API contract and validate each route against actual browser traffic. That produces a defensible endpoint inventory rather than mistaking a system-design proposal for TuteDude's private implementation.