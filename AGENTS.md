# AGENTS.md

## 1. Purpose
This document defines how Codex and engineers must build, run, deploy, and evolve this project.

This file is authoritative for:
- implementation stack choices
- runtime architecture
- deployment model
- environment strategy
- coding and integration rules
- cloud portability principles
- operational guardrails

This file is **not** the source of truth for product behavior, user journeys, business rules, workflow states, or API/domain requirements. Those belong in the pack specs.

Use this file together with:
- PRD for product vision and constraints
- pack specs for functional and technical behavior
- task breakdowns for implementation sequencing

---

## 2. Architecture Intent
The system must be built so that:
- local development and VPS production follow the same core runtime shape
- early production runs cost-effectively on a single VPS
- future migration to cloud infrastructure is possible without major rewrites
- the application remains cloud-agnostic at the code and service-boundary level
- business logic remains independent of infrastructure vendors
- uploaded files are stored outside the application server
- AI is used as a controlled subsystem, not as the source of workflow truth

### Frozen architecture intent
- Start with a low-cost VPS deployment
- Keep architecture portable to public cloud later
- Keep local and VPS production runtime patterns as similar as practical
- Use externalized configuration for all environment-specific behavior
- Favor a modular monolith over microservices for v1 and early growth

---

## 3. Authoritative Stack Decisions

### 3.1 Backend
- Language: Java 21
- Framework: Spring Boot 4.x
- Security: Spring Security
- Persistence: Spring Data JPA + Hibernate
- Database migrations: Flyway
- Validation: Jakarta Bean Validation
- Build tool: Maven
- Testing: JUnit 5

### 3.2 Frontend
- Framework: React
- Language: TypeScript
- Build tool: Vite
- Routing: React Router
- Server state / API fetching: TanStack Query
- Forms: React Hook Form
- Validation: Zod
- Styling: Tailwind CSS
- Component base: shadcn/ui

### 3.3 Data and Storage
- Primary transactional database: PostgreSQL
- File and image storage: OCI Object Storage
- Do not store user-uploaded images as the primary durable store on VPS disk
- Redis is not required for initial production unless justified by traffic, rate limiting, or caching needs

### 3.4 Infra and Deployment
- Initial production compute: OVHcloud VPS
- Initial production topology: single VPS
- Reverse proxy / TLS termination / static serving: Nginx
- Container runtime: Docker
- Multi-service orchestration for local and VPS: Docker Compose
- CI/CD target pattern: build artifact/image once, promote via config

### 3.5 AI
- Primary AI provider: Google Gemini
- Gemini is used for multimodal understanding, structured extraction, rubric drafting, and analysis
- AI outputs must be treated as controlled machine outputs subject to schema validation and workflow approval

### 3.6 Notifications and External Delivery
- Email provider: configurable adapter, production provider to be finalized separately
- SMS provider: configurable adapter, production provider to be finalized separately
- Email and SMS delivery providers must never be hardcoded into business logic

---

## 4. System Shape
The system must be implemented as a **modular monolith** with clear internal module boundaries.

### Why modular monolith
This product is workflow-heavy, state-heavy, and still early-stage. It benefits from:
- low operational complexity
- strong transactional consistency
- lower hosting cost
- easier local development
- easier debugging
- simpler deployment on a single VPS

The application must not be split into microservices unless there is a proven operational or scaling reason.

### Expected high-level modules
The exact module list can evolve pack by pack, but the architectural direction should support modules such as:
- auth
- verification / OTP
- user management
- admin operations
- course catalog
- enrollments
- tutor-student mapping
- access control
- file ingestion / storage integration
- AI orchestration
- feedback / evaluation workflows
- audit and observability

These modules may live in one deployable backend application but must be separated by package boundaries and service contracts.

---

## 5. Runtime Topology

### 5.1 Local development runtime shape
Local development should mirror VPS production runtime shape as closely as practical.

Standard local stack:
- nginx
- frontend
- backend
- postgres
- optional worker later

All of the above should run through Docker Compose unless there is a strong temporary reason not to.

### 5.2 VPS production runtime shape
Initial production runtime on VPS should follow the same logical pattern:
- nginx (Docker container)
- frontend (served via nginx)
- backend (Docker container)
- postgres (host OS)

Later additions may include:
- worker service for async/AI tasks
- redis if needed
- monitoring/agent containers if needed


Later additions may include:
- worker service for async/AI tasks
- redis if needed
- monitoring/agent containers if needed

### 5.3 Future cloud migration shape
Cloud deployment may change the hosting topology while preserving service boundaries.
Examples:
- postgres may move to managed DB
- frontend may move to CDN/static hosting
- nginx may be replaced or fronted by managed ingress/load balancer
- worker may scale separately
- backend may scale to multiple instances

The codebase must be structured so that migration changes infrastructure wiring more than application logic.

---

## 6. Reverse Proxy and Frontend Serving
Nginx is part of the standard deployment shape.

### Nginx responsibilities
- terminate TLS
- serve the built React frontend as static files
- reverse proxy `/api` traffic to the Spring Boot backend
- provide basic request routing and buffering
- optionally enforce body size limits and simple request hardening

### Routing model
- `/` and frontend routes -> React static app via Nginx
- `/api` -> Spring Boot backend

React must be treated as a browser-side application, not as a server-rendered JSP/ASP-style UI.
The frontend build output is static HTML/CSS/JS served by Nginx.

---

## 7. Containerization Policy

### 7.1 Why containers are standard
Containers are the standard packaging and runtime model because they improve:
- reproducibility across machines
- local/prod parity
- deployment portability
- rollback safety
- version pinning
- provider migration flexibility

### 7.2 Required containerization stance
- Backend must be containerized
- Frontend must be built and served in a containerized deployment path
- Nginx should be part of the containerized deployment path unless there is a deliberate operational reason to run it on host
- PostgreSQL may run in a container for parity if backup and volume discipline are implemented correctly

### 7.3 Production guidance for PostgreSQL
For initial VPS production, PostgreSQL will run on the host OS.

Rationale:
- simpler operational model for backups, restores, and upgrades
- avoids common container volume misconfiguration risks
- better alignment with single-node VPS simplicity for v1

Containerized PostgreSQL remains acceptable for local development for parity, but production will use host-installed PostgreSQL.

For initial VPS production, PostgreSQL may be run either:
- in Docker with named volumes and disciplined backup procedures, or
- on the host OS if operational simplicity is preferred

If PostgreSQL is containerized in production:
- data must live on persistent mounted volumes
- backup and restore procedures must be documented and tested
- upgrades must be explicit and reversible

---

## 8. Configuration and Environment Strategy
The same codebase must support local, VPS production, and future cloud environments through configuration.

### 8.1 Rules
- Never hardcode environment-specific values in business logic
- Use environment variables and Spring configuration files
- Use profiles only for environment wiring, not for feature behavior divergence unless explicitly intended
- Frontend environment values must be externalized appropriately for build/runtime usage

### 8.2 Standard environments
- local
- production-vps
- future cloud environments as needed

### 8.3 Configuration responsibilities
Configuration may vary by environment for:
- database connection details
- object storage bucket names and endpoints
- AI provider credentials and model defaults
- email/SMS provider credentials
- hostnames and domains
- TLS and security settings
- logging levels
- rate limiting thresholds
- CORS origins

But the business workflow and domain behavior must remain consistent across environments unless explicitly specified otherwise.

---

## 9. Cloud-Agnostic Design Principles
The application must be portable across infrastructure providers.

### 9.1 Code must depend on abstractions
External integrations must be expressed behind interfaces/adapters, not directly embedded everywhere.
Examples include:
- `FileStorageService`
- `EmailSender`
- `SmsSender`
- `AiProviderClient`
- `Clock` where deterministic time handling matters

### 9.2 Avoid vendor lock-in in core logic
Core business services must not directly depend on:
- OVH-specific APIs
- OCI-specific storage code in domain services
- cloud-provider-specific auth/session assumptions
- local filesystem as durable truth

### 9.3 External durable state
Durable state belongs in:
- PostgreSQL
- OCI Object Storage
- other explicit external systems if added later

Durable state must not rely on:
- container-local filesystem
- server-local temp folders as the source of truth
- in-memory server state

### 9.4 Stateless backend direction
Backend services should be designed to be stateless from an application-node perspective.
This enables future scaling to multiple instances.

---

## 10. Phone Number Policy

### 10.1 Purpose
- Phone numbers are collected strictly as **contact information**.
- Phone numbers are **not used for identity verification in v1**.

### 10.2 Verification
- No OTP (SMS or external messaging (optional future)) is used for phone verification in v1.
- No outbound verification messages should be triggered during signup.
- Phone numbers are stored with status = `UNVERIFIED`.

### 10.3 Future extensibility
- Phone collection mechanisms (SMS OTP, external messaging (optional future) confirmation) must be implemented via adapters if added later.
- Current system must not assume phone verification as a prerequisite for any workflow.

---

## 11. File and Image Storage Policy

### 10.1 Storage model
- Durable file and image storage must use OCI Object Storage
- The application server may use temporary local disk only for transient processing where necessary
- Local/VPS disk must not be the long-term source of truth for uploaded user images

### 10.2 Upload strategy direction
The system should be written so upload flow can support direct or semi-direct object storage upload patterns later.
If the initial implementation proxies uploads through the backend, the code must still preserve object storage as the final durable store.

### 10.3 Storage abstractions
The backend must use a storage abstraction layer rather than embedding object-storage-specific logic throughout business modules.

---

## 12. AI Architecture Rules
Google Gemini is the primary AI provider, but the application owns workflow truth.

### 11.1 AI responsibilities
Gemini may be used for:
- image/document understanding
- structured content extraction
- rubric draft generation
- answer analysis
- feedback draft generation

### 11.2 AI non-responsibilities
Gemini must not be the sole authority for:
- workflow state transitions
- access control decisions
- approval state truth
- business policy enforcement
- user/account state persistence

### 11.3 AI integration rules
- AI calls must originate from backend-controlled services, not directly from the frontend
- Prompts must be versioned where practical
- Prefer structured JSON outputs over free-form text for machine-consumed steps
- All AI outputs must be validated before entering downstream workflow state
- AI metadata should be stored where traceability is needed: model, prompt version, timestamps, request context, and confidence/notes if available
- v1 student-visible feedback remains subject to tutor approval as defined by the specs

### 11.4 Model strategy
Use a lower-cost default model path for routine workloads and reserve stronger/more expensive model paths for escalation cases.

---

## 13. API and Backend Design Rules

### 12.1 Backend style
- Backend is API-first
- Frontend consumes backend APIs over HTTP/JSON
- Backend must not be written as a server-side page rendering application

### 12.2 Layering
Standard backend layering should be followed:
- controller / transport layer
- application/service layer
- domain/model layer
- repository/persistence layer
- integration/adapters layer

Controllers must remain thin.
Business decisions belong in services/domain logic, not in controllers.

### 12.3 DTO discipline
- Never expose persistence entities directly as external API contracts
- Use request/response DTOs
- Validate incoming payloads explicitly
- Keep API contracts stable and intentional

### 12.4 Transactions
Use transactions thoughtfully around workflow-critical mutations.
Do not spread workflow-critical state changes across uncontrolled service calls.

---

## 14. Frontend Design Rules

### 13.1 Frontend stance
The frontend is a SPA-style React application.
It is responsible for:
- user interaction
- client-side routing
- forms and validation UX
- API interaction
- role/state-aware UI rendering

The frontend is not the source of truth for business policy.

### 13.2 Frontend structure expectations
The frontend should be structured around:
- route modules/screens
- reusable UI components
- API client layer
- form schemas
- query/mutation hooks
- state derived from backend truth

### 13.3 Frontend runtime rules
- Treat backend as the source of workflow truth
- Do not encode sensitive business rules only in the client
- Handle pending/rejected/approval states based on backend responses and route guards

---

## 15. Database and Migration Rules

### 14.1 Database authority
PostgreSQL is the source of truth for transactional data.

### 14.2 Migration discipline
- All schema changes must be managed via Flyway migrations
- Do not mutate production schemas manually except in carefully controlled emergency situations
- Migrations must be committed to version control
- Migrations must be forward-only in normal workflow

### 14.3 Database safety
- Use explicit indexes where justified by queries and domain rules
- Use database constraints where they protect core invariants
- Application logic and DB constraints should complement each other

### 14.4 Backups
Database backups are mandatory in production.
Production backups must be automated, retained, and stored off-server.
Restore procedures must be documented and periodically tested.

---

## 16. Security Rules

### 15.1 Core rules
- Use HTTPS in production
- Terminate TLS via Nginx
- Use secure secret management practices
- Never commit secrets to the repository
- Store passwords only as strong hashes
- Never store plain OTP values long term
- Use least-privilege credentials for external integrations where practical

### 15.2 Access and auth rules
- Security-sensitive access control must be enforced on backend APIs
- Frontend route protection is UX support, not the security boundary
- Account states and role checks must be enforced server-side

### 15.3 Upload and request hardening
- enforce upload size limits
- validate content types where relevant
- protect public endpoints against abuse where practical

---

## 17. Observability and Operations

### 16.1 Logging
- Use structured application logs where practical
- Include request correlation and key workflow identifiers where useful
- Do not log secrets, raw passwords, or sensitive tokens

### 16.2 Metrics
The system should be able to track meaningful application metrics such as:
- signup initiation counts
- OTP send/verify success/failure
- approval turnaround times
- enrollment request volumes
- mapping completion times
- AI processing counts and latencies
- AI cost-related usage metrics where practical

### 16.3 Error tracking
Application errors should be observable through centralized logs and/or an error tracking tool.

---

## 18. Cost and Scalability Posture
The initial system must optimize for low cost while preserving a path to scale.

### 17.1 Cost posture
- Start on a single VPS
- Keep the backend as a modular monolith
- Avoid unnecessary managed services early
- Use object storage for files rather than over-sizing VPS disk usage
- Add Redis only when justified
- Treat AI and SMS as variable-cost centers that must be measured explicitly

### 17.2 Scale posture
When scaling later, prefer these steps in order:
1. optimize queries, prompts, and workflows
2. add worker separation for async/AI work
3. split DB from app host if not already split
4. move infrastructure components to managed services where justified
5. scale backend instances only when needed

Do not prematurely adopt Kubernetes or microservices.

---

## 19. Repository and Deployment Structure Guidance
A clear repo structure is required so the deployment model is reproducible.

Recommended high-level shape:
- `backend/`
- `frontend/`
- `deploy/`
  - `compose/`
  - `nginx/`
  - `scripts/`
- `docs/`
- `.env.example`

### Deployment config principles
- Deployment config must live in version control
- Environment-specific values must be injected, not hardcoded
- Production setup should be reproducible from repo-managed assets and documented commands

---

## 20. Rules for SpecKit and Codex

### 19.1 Separation of responsibility
- Pack specs define WHAT the system must do
- This AGENT.md defines HOW the system must be built and operated
- Tasks define the implementation sequence

### 19.2 Task generation expectations
When generating tasks, use:
- specs for workflow/business requirements
- AGENT.md for stack, architecture, operational constraints, and implementation style

Tasks should not redefine the stack if already frozen here.

### 19.3 Implementation expectations for Codex
Codex should:
- follow the frozen stack and runtime model in this file
- preserve cloud-agnostic abstractions
- avoid embedding vendor-specific assumptions in core business logic
- keep backend modules cohesive and explicit
- favor maintainability and portability over clever shortcuts

## 20A. Task Completion Testing and Clarification Policy

After implementing each task, developers must run the full relevant test suite before considering the task complete.

Minimum required coverage per task:
- backend tests (unit + integration)
- frontend tests (unit/component)
- cross-layer integration tests where the task impacts API, state transitions, auth, workflows, or persistence

Rules:
- The exact test cases and command variants may be decided by the developer per task based on scope and impact.
- Do not mark a task as complete unless all selected tests pass.
- For every task, generate a concise test report and attach it to task notes/PR.
- The test report must include: tests executed, command(s) used, pass/fail status, and any skipped tests with reason and risk.
- If implementation details are unclear or ambiguous, ask for clarification before coding.
- Do not assume missing behavior, business rules, API contracts, or edge-case handling without confirmation.

---

## 21. Non-Negotiable Principles
The following principles are mandatory unless explicitly overridden by a later architecture decision:
- modular monolith for v1 and early growth
- Docker Compose as the standard runtime model for local and VPS production
- Nginx in the deployment path
- React frontend served as static assets, not JSP-style server-rendered pages
- Spring Boot backend as API-first application
- PostgreSQL as primary transactional store
- OCI Object Storage as durable file/image store
- Google Gemini as primary AI provider for AI-assisted flows
- externalized configuration for all environment-specific behavior
- adapter-based integration boundaries for external services
- no business-critical durable state on local/container filesystem
- backend remains stateless from the application-node perspective
- AI outputs must be validated and governed by workflow rules

---

## 22. Approval Checkpoints
The following items should be explicitly confirmed before freezing this AGENT.md:
1. whether PostgreSQL in VPS production should run in Docker or on host OS
2. whether Nginx should run in Docker or on host OS in VPS production
3. which email provider is frozen for production
4. which SMS provider is frozen for production
5. whether a worker container is included from day 1 or added later
6. whether direct browser-to-object-storage uploads are part of v1 or a later optimization
