# SplitFlow Technical Architecture (Backend & Frontend)

**Backend:** Laravel 10+ (RESTful API)  
**Frontend:** Nuxt 3 (SSR) separate application  
**Database:** MySQL 8.0+  
**Auth:** Token-based via Laravel Passport (Bearer JWT-like tokens)  
**Transport:** RESTful API with JSON payloads  
**Date:** January 2026  
**Status:** Draft

---

## 1. High-Level Architecture

- **Client (Nuxt3 SSR):** Renders pages server-side; hydrates on client. Communicates with API via HTTPS using Bearer tokens.  
- **API (Laravel 10+):** Stateless REST API, Passport for token issuance/validation, returns JSON with consistent envelope.  
- **Database (MySQL 8.0+):** InnoDB, utf8mb4. Strict mode on.  
- **Cache/Queue (optional/rec’d):** Redis for caching sessions/rate limits/queues; Horizon for queue monitoring.  
- **Object Storage (optional future):** For receipts/assets when enabled.  
- **Real-time (future):** WebSockets or SSE for live unread counts; for now, REST polling allowed.  
- **Environment:** `.env` per service; secrets not committed.  

---

## 2. Deployment Topology

- **Frontend:** Nuxt SSR runtime (Node 18+), served behind Nginx/ALB/Cloudflare.  
- **API:** Laravel app (PHP 8.2+), FPM behind Nginx/ALB/Cloudflare.  
- **DB:** Managed MySQL 8.0+. Automated backups, PITR, replicas for HA/reads.  
- **Cache/Queue:** Redis (managed if possible).  
- **Artifacts:** CI builds Docker images for FE/BE; tagged by commit SHA.  
- **Environments:** dev → staging → prod, isolated databases and credentials.  

---

## 3. Backend (Laravel) Design

### 3.1 Core Conventions
- **Routing:** `routes/api.php`; versioned prefixes (`/api/v1`).  
- **Controllers:** Thin; delegate to Services/Actions; type-hint DTOs/Requests.  
- **Requests:** FormRequest for validation/authorization; return 422 with errors.  
- **Resources:** `JsonResource` for response shaping; consistent envelope.  
- **Models:** Eloquent with guarded/fillable; use UUIDs for public IDs.  
- **Migrations:** Idempotent; `down` blocks optional but prefer reversible.  
- **Seeds:** Minimal baseline seeds; no prod data.  
- **Policies:** Authorization via Gates/Policies; roles: admin/member.  
- **Events/Listeners:** For side-effects (unread counts, emails).  
- **Jobs/Queues:** For email/notifications; use Redis queue + Horizon.  
- **Logging:** Monolog to stdout; JSON format.  
- **Env:** `.env` only; never commit.  

### 3.2 Authentication (Passport)
- Use **Passport Personal Access Tokens** or **Passwordless custom grant** for email+OTP.  
- Token transport: `Authorization: Bearer <token>`.  
- Token TTL: short-lived access token; optional refresh token if implemented.  
- OTP flows: endpoints for issue/verify; rate-limit by IP/email.  

### 3.3 Error Handling & Responses
- Envelope: `{ "data": ..., "meta": {...}, "errors": [...] }`  
- Errors: include `code`, `message`, optional `field`, optional `details`.  
- HTTP status: 2xx success; 4xx client; 5xx server. Use 401/403/404/409/422 appropriately.  
- Validation errors: 422 with field-level messages.  

### 3.4 Pagination & Filtering
- Cursor or page-based (standardize): `page`, `per_page` (cap e.g. 100).  
- Sorting: `sort=created_at,-amount`.  
- Filtering: query params, whitelist per endpoint.  

### 3.5 Security
- HTTPS only; HSTS.  
- Rate limiting: per-IP and per-user (Laravel RateLimiter).  
- Input sanitization: Request validation + casting.  
- SQL: Eloquent/Query Builder only.  
- Headers: `X-Content-Type-Options`, `X-Frame-Options`, `Content-Security-Policy` (frontend).  
- CORS: allow FE origin(s) only; allow Authorization header.  
- Secrets: env/secret manager.  
- Audit logs: admin role changes, approvals/rejections, auth events.  

### 3.6 Testing
- PHPUnit + Pest (optional) for unit/feature tests.  
- HTTP tests using `actingAs` with tokens.  
- Factories for seed data; DatabaseTransactions for isolation.  

### 3.7 Folder Structure (Backend)
```
app/
  Actions/         # Use-case specific actions
  Console/
  Events/
  Exceptions/
  Http/
    Controllers/   # Thin; delegate to Actions/Services
    Middleware/
    Requests/      # FormRequest validation
    Resources/     # API Resources (JsonResource)
  Jobs/            # Queue jobs
  Listeners/
  Models/
  Policies/
  Services/        # Integrations, domain services
bootstrap/
config/
database/
  factories/
  migrations/
  seeders/
routes/
  api.php
tests/
```

### 3.8 Coding Style (Backend)
- PSR-12; Laravel Pint for formatting.  
- Strict typing where feasible; scalar/return types on new code.  
- Avoid facades in domain logic; inject services.  
- DTOs for cross-layer data where helpful.  

---

## 4. Frontend (Nuxt 3 SSR) Design

### 4.1 Core Conventions
- **Pages/Routes:** File-based routing; middleware for auth/role.  
- **State:** Pinia stores for auth/user/groups/activities.  
- **API Client:** Axios (or ofetch) wrapper with base URL, auth interceptor for Bearer token, unified error handling.  
- **SSR Auth:** Store token in HttpOnly cookie if possible; fallback to secure local storage with CSRF mitigation.  
- **Env Config:** Runtime config for API base URL, feature flags.  
- **i18n:** If needed, Nuxt i18n.  

### 4.2 UI/Layout
- Mobile-first; **desktop constrained to mobile-width column centered horizontally** (per requirement).  
- Components for lists/cards/forms; skeleton loaders; toasts for feedback.  
- Accessibility: WCAG 2.1 AA; focus states; semantic HTML.  

### 4.3 Error Handling
- Global error boundary/page.  
- API errors normalized (code/message/fields).  
- 401 → logout + redirect to login; 403 → show “not allowed”; 404 → not found page.  

### 4.4 Performance
- Nuxt image optimization; route-level code splitting.  
- Cache per-page data where safe (SWR strategy).  
- Avoid blocking scripts; prefer async/defer.  

### 4.5 Testing
- Component tests: Vitest + Testing Library.  
- E2E: Playwright/Cypress hitting staging API.  

### 4.6 Folder Structure (Frontend)
```
app/              # Nuxt app entry
components/       # UI components
composables/      # Reusable logic (e.g., useAuth, useApi)
layouts/          # Base layouts
middleware/       # Route guards (auth, role)
pages/            # File-based routes
plugins/          # Axios/ofetch plugin, etc.
public/           # Static assets
server/           # Nitro server routes/middleware if needed
stores/           # Pinia stores
types/            # TS types
utils/            # Helpers (formatters, validators)
tests/            # Unit/e2e setup
```

### 4.7 Coding Style (Frontend)
- TypeScript for all code.  
- ESLint + Prettier + Stylelint; Nuxt recommended config.  
- Composition API with `<script setup>`.  
- Avoid implicit any; enable `strict` in tsconfig.  

---

## 5. API Design Conventions

- **Base URL:** `/api/v1`  
- **Auth Header:** `Authorization: Bearer <token>`  
- **Content-Type:** `application/json`  
- **Idempotency:** Use UUIDs; POST creates, PUT/PATCH updates, DELETE archives where applicable.  
- **Soft-delete/Archive:** Groups are archived, not deleted.  
- **Currency:** Default/only currency in this phase: Bangladeshi Taka (৳).  
- **No receipts in this phase:** Expense submission has no attachment field.  
- **Unread counts:** Derived per-group counts, no notification storage.  
- **OTP flows:** Endpoints for issue/verify login and registration OTPs. Rate-limit aggressively.  

Example response envelope:
```json
{
  "data": { /* resource */ },
  "meta": {
    "request_id": "uuid",
    "pagination": { "page": 1, "per_page": 20, "total": 50 }
  },
  "errors": []
}
```

---

## 6. Observability & Ops
- **Logging:** JSON to stdout; include request_id, user_id.  
- **Metrics:** HTTP latency, error rates, DB slow queries.  
- **Tracing:** If available (OpenTelemetry).  
- **Health Checks:** `/health` for liveness/readiness.  
- **Rate Limits:** Login/OTP issuance; general API throttling.  
- **Backups:** Automated MySQL backups; restore drills.  

---

## 7. CI/CD
- **CI:** Lint, test, build both FE and BE; run migrations in staging.  
- **CD:** Deploy via pipelines; run migrations before flipping traffic.  
- **Secrets:** Inject at deploy time from secret manager.  
- **Artifact Versioning:** Tag images by commit SHA.  

---

## 8. Things to Consider / Next Steps
- Real-time transport for unread counts (WebSockets or SSE) vs. short polling.  
- Refresh tokens/rotation for long-lived sessions if needed.  
- Feature flags (launch darkly/open-source alternative) for phased rollouts.  
- Rate limit/abuse protection around OTP endpoints.  
- Caching strategy for group lists and unread counts.  
- Monitoring dashboards and alerting (latency/error budgets).  
- Blue-green or canary deploys for safer releases.  
- Contract tests (API schemas) to keep FE/BE in sync.  
