# Job Posting Feature — Functional Design (JOBPOSTINGFEATURE.md)

## Overview
This document describes the Client flow for posting jobs, the multi-step job form, business rules (escrow, budget minimums, expiry), recommended database models, API endpoints, payments/escrow flow (Razorpay), attachments handling, and validations.

Purpose: enable clients to post jobs, fund the required escrow (25% of budget), and publish jobs so freelancers can discover and apply.

---

## 1) Client flow summary

### Preconditions (Step 0)
Before posting a job, the system MUST ensure the client:
- has **completed onboarding** (User.isOnboardingCompleted === true)
- has **verified email & mobile** (isEmailVerified && isMobileVerified)
- has **wallet balance >= 25% of job budget** (or is able to pay via payment gateway)

If any precondition fails, the client is redirected to the appropriate flow: onboarding, verification, or wallet top-up.

### Step 1 — Click "Post a Job"
From dashboard, client clicks "Post a Job". System checks Profile completed and Wallet funded (or ready to pay via Razorpay).

### Step 2 — Multi-step Job Creation Wizard
Wizard sections (recommended):
1. Job Basics
2. Budget & Timeline
3. Attachments
4. Visibility & Preferences
5. Escrow Confirmation (Fund & Publish)

(See Section 2 for detailed fields.)

### Step 3 — Job Published
When escrow is funded and publish action confirmed, job status becomes `OPEN` and is visible to freelancers.

---

## 2) Job form & required data

### Section 1 — Job Basics
| Field | Required | Example |
|---|---:|---|
| title | ✅ | "Build e-commerce website" |
| category | ✅ | "Web Development" |
| jobType | ✅ | `fixed_price` / `hourly` |
| description | ✅ | "Need MERN developer to build..." |
| skills | ✅ | ["React","Node","MongoDB"] |

### Section 2 — Budget & Timeline
| Field | Required | Example |
|---|---:|---|
| budgetAmount (INR) | ✅ | 5000 |
| deadline (date) | ❌ | 2026-03-01 |
| experienceLevel | ❌ | `entry` / `intermediate` / `expert` |

Business rule: budgetAmount >= 500 INR. Escrow required = ceil(25% * budgetAmount).

### Section 3 — Attachments
Optional files: PDF, images, docs, ZIP. Store as signed URLs or via `/uploads` endpoint. Save metadata: { url, filename, mimeType, size }.

### Section 4 — Visibility & Preferences
| Field | Required | Notes |
|---|---:|---|
| visibility | ✅ | `public` / `invite_only` |
| allowNegotiation | ✅ | boolean (default true) |
| requireVerifiedFreelancer | ❌ | boolean, filters applicants |

### Section 5 — Escrow Confirmation
System displays:
- Job Budget: ₹X
- Required escrow (25%): ₹Y

Client clicks **Fund & Publish** and completes payment via Razorpay (UPI or other methods). On successful capture, the job becomes `OPEN` and escrow record is created.

---

## 3) Business rules & lifecycle
- Minimum budget: **₹500**
- **25% escrow mandatory** to publish
- Optional: `maxOpenJobs` configuration per client (enforced on create)
- Job auto-expires after **30 days** (set `expiresAt = createdAt + 30d`)
- Client can **edit** job only if `proposalsCount === 0` and job is not `OPEN`/`FILLED`/`CLOSED`
- Deleting a job triggers **refund** of escrow (if captured) and sets status `DELETED` or `CANCELLED` with `deletedAt`
- On job being filled and milestones released, escrow is released to freelancer (handled by separate workflow)

---

## 4) Recommended statuses
- `DRAFT` — job created but not funded/published
- `PENDING_FUNDS` — waiting for payment capture
- `OPEN` — published and visible to freelancers
- `FILLED` — client accepted a proposal / hired
- `CLOSED` — job completed and closed
- `EXPIRED` — auto-expired after 30 days without filling
- `CANCELLED` — client cancelled; may trigger refund
- `DELETED` — removed by client (after refund)

---

## 5) DB Models (Mongoose) — suggested schemas

### Job
```javascript
const jobSchema = new mongoose.Schema({
  client: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true, index: true }, // client user
  clientProfileSnapshot: { // denormalized snapshot for quick display / historical integrity
    firstName: String,
    lastName: String,
    companyName: String
  },

  title: { type: String, required: true, trim: true },
  category: { type: String, required: true, index: true },
  jobType: { type: String, enum: ['fixed_price','hourly'], required: true },
  description: { type: String, required: true },
  skills: [{ type: String }],

  budgetAmount: { type: Number, required: true, min: 500 },
  currency: { type: String, default: 'INR' },
  escrowAmount: { type: Number, required: true },
  escrowPaid: { type: Boolean, default: false },

  deadline: { type: Date },
  experienceLevel: { type: String, enum: ['entry','intermediate','expert'] },

  attachments: [{ url: String, filename: String, mimeType: String, size: Number }],

  visibility: { type: String, enum: ['public','invite_only'], default: 'public' },
  allowNegotiation: { type: Boolean, default: true },
  requireVerifiedFreelancer: { type: Boolean, default: false },

  status: { type: String, enum: ['DRAFT','PENDING_FUNDS','OPEN','FILLED','CLOSED','EXPIRED','CANCELLED','DELETED'], default: 'DRAFT', index: true },

  proposalsCount: { type: Number, default: 0 },

  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now },
  expiresAt: { type: Date, index: true },
  deletedAt: { type: Date }
});

// Pre-save compute escrowAmount = Math.ceil(0.25 * budgetAmount)
```

Notes:
- `escrowAmount` computed on create; store explicitly for audit.
- Index `client`, `status`, `expiresAt`, `category` for queries.


### Escrow / Payment record
```javascript
const escrowSchema = new mongoose.Schema({
  job: { type: mongoose.Schema.Types.ObjectId, ref: 'Job', required: true, index: true },
  client: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  amount: { type: Number, required: true },

  paymentProvider: { type: String, enum: ['RAZORPAY','RAZORPAY_UPI'], default: 'RAZORPAY' },
  providerOrderId: String, // order id returned by gateway
  providerPaymentId: String, // payment id when captured
  status: { type: String, enum: ['PENDING','PAID','REFUNDED','FAILED'], default: 'PENDING' },
  meta: { type: Object },

  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now }
});
```

Notes:
- Keep payment provider ids to allow refunds and ledger mapping.
- On successful capture, set `status = PAID` and mark `job.escrowPaid = true`, `job.status = OPEN`, set `job.expiresAt`.

---

## 6) API Endpoints (examples)
All endpoints should be under `/api/v1` and require authentication. Only clients may create jobs.

### POST /api/v1/jobs
Create a job draft or request to publish with payment.

Request (create draft):
```json
POST /api/v1/jobs
{
  "title": "Build e-commerce website",
  "category": "Web Development",
  "jobType": "fixed_price",
  "description": "Need MERN developer",
  "skills": ["React","Node"],
  "budgetAmount": 5000,
  "deadline": "2026-03-01",
  "visibility": "public",
  "allowNegotiation": true
}
```

Response (201):
```json
{ "success": true, "data": { "job": { /* job doc */ } } }
```

Option: to publish and request payment in single call send `publish: true` and backend responds with a payment order

Request (publish & fund):
```json
POST /api/v1/jobs
{
  /* job fields */, 
  "publish": true
}
```

Response (202 - Payment required):
```json
{ "success": true, "data": { "jobId": "<id>", "paymentOrder": { "provider": "RAZORPAY", "orderId": "order_abc", "amount": 125000 } } }
```

(amount in paise when using Razorpay)

---

### POST /api/v1/jobs/:id/fund
Create a payment order or mark a captured payment (used when separate flow used)

Request:
```json
POST /api/v1/jobs/:id/fund
{ "paymentMethod": "upi", "returnUrl": "https://app.example.com/payments/return" }
```

Response:
```json
{ "success": true, "data": { "orderId": "order_abc", "paymentUrl": "https://checkout.razorpay.com/xyz" } }
```

Frontend completes payment and Razorpay triggers webhook.

---

### POST /api/v1/payments/webhook
Payment gateway (Razorpay) will POST webhooks to this endpoint.
On `payment.captured` events validate signature, find escrow record by `providerOrderId` or `providerPaymentId`, set `status = PAID`, update `job.escrowPaid = true`, set `job.status = OPEN`, set `job.expiresAt = now + 30d`, and notify client / post job to feed.

Response: HTTP 200 to acknowledge.

---

### PATCH /api/v1/jobs/:id
Update a job draft or make minor edits. Allowed only if `proposalsCount === 0` (otherwise 403).

Request (example):
```json
PATCH /api/v1/jobs/:id
{ "title": "Updated title", "budgetAmount": 6000 }
```

Notes: If `budgetAmount` changes and escrow was already paid, require additional capture or refund flow to rebalance escrow.

---

### DELETE /api/v1/jobs/:id
Delete (cancel) a job. If escrow captured, initiate refund via payment provider and set job status `CANCELLED` or `DELETED`.

Response (200):
```json
{ "success": true, "message": "Job cancelled and refund initiated" }
```

---

## 7) Payments & Escrow flow (Razorpay) — high level
1. Client clicks **Fund & Publish** → backend creates a Razorpay Order with amount = escrowAmount (INR*100 paise) via Razorpay Orders API.
2. Backend returns order id and payment details to frontend.
3. Frontend completes UPI/checkout; on success Razorpay emits `payment.captured` webhook.
4. Webhook handler verifies signature, marks escrow record `PAID`, updates job `escrowPaid = true`, `status = OPEN`, sets `expiresAt`.
5. For refunds (job deletion/cancellation) call Razorpay Refunds API with `payment_id` and mark `escrow.status = REFUNDED` once processed.

Important: Don't trust frontend to mark payments. Always rely on webhook verification.

---

## 8) Attachments & file handling
- Prefer direct uploads to storage provider (S3/GCS) using signed URLs.
- Save metadata in `job.attachments` (url, filename, mimeType, size).
- If backend receives files, validate MIME and size, scan for malware if possible.

---

## 9) Indexes, performance & queries
- Index: `client`, `status`, `expiresAt`, `category` and `skills` for discovery.
- Use text index on `title` and `description` for search.
- Paginate `/jobs` list and add filters: category, budget range, experienceLevel, visibility.

---

## 10) Notifications & webhooks
- Send notifications on: job published, payment captured, refund processed, job expired, proposal received.
- Provide webhook for partner integrations if required.

---

## 11) Background tasks & expiry
- Scheduler (cron or queue-based) runs daily to mark jobs with `expiresAt < now` as `EXPIRED` and trigger notifications.
- On expiry evaluate business rules for refund (the default behaviour: do not auto-refund; refund only on explicit delete/cancel to avoid gaming the system). Decide based on product policy.

---

## 12) Security & validations
- Authz: only the job `client` may edit/cancel the job.
- Validate budget >= 500; reject otherwise.
- Sanitize all text inputs to avoid XSS.
- Verify file uploads for allowed MIME types and size limits.
- Rate limit job creation to prevent abuse.

---

## 13) Error codes & responses
- 400 Bad Request — validation errors (e.g. budget too low)
- 401 Unauthorized — not authenticated
- 403 Forbidden — not client or not allowed to edit
- 404 Not Found — job not found
- 409 Conflict — publish call when funds insufficient or concurrent publish
- 500 Server Error — unexpected failures

---

## 14) Next steps & implementation checklist
- [ ] Add `Job` and `Escrow` Mongoose models (`models/Job.js`, `models/Escrow.js`)
- [ ] Add job routes & controllers (`routes/jobRoutes.js`, `controllers/jobController.js`)
- [ ] Implement payment creation (`/api/v1/jobs/:id/fund`) and webhook (`/api/v1/payments/webhook`)
- [ ] Implement cron job to expire jobs
- [ ] Add background refund handling and ledger entries for wallet
- [ ] Add tests (unit + integration for payment webhooks)

---

If you want, I can now: create `models/Job.js` + `models/Escrow.js`, add controller stubs and routes, or add sample cURL/Postman examples for the endpoints. Which would you like me to implement next? ✅
