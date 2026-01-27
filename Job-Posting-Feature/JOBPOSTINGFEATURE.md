# Job Posting Feature — Functional Design (JOBPOSTINGFEATURE.md)

## Overview
This document describes the Client flow for posting jobs, the multi-step job form, business rules (budget minimums, expiry), recommended database models, API endpoints, attachments handling, and validations.

Purpose: enable clients to post jobs and publish jobs so freelancers can discover and apply.

---

## 1) Client flow summary

### Preconditions (Step 0) 
Before posting a job, the system MUST ensure the client:
- has **completed onboarding** (User.isOnboardingCompleted === true)
- has **verified email & mobile** (isEmailVerified && isMobileVerified)

If any precondition fails, the client is redirected to the appropriate flow: onboarding or verification.

### Step 1 — Click "Post a Job"
From dashboard, client clicks "Post a Job". System checks Profile completed and verification requirements.

### Step 2 — Multi-step Job Creation Wizard
Wizard sections (recommended):
1. Job Basics
2. Budget & Timeline
3. Attachments
4. Visibility & Preferences
5. Review & Publish

(See Section 2 for detailed fields.)

### Step 3 — Job Published
When publish action is confirmed, job status becomes `OPEN` and is visible to freelancers.

---

## 2) Job form & required data

### Section 1 — Job Basics
| Field | Required | Example |
|---|---:|---|
| title | ✅ | "Build e-commerce website" |
| category | ✅ | "Web Development" |

| description | ✅ | "Need MERN developer to build..." |
| skills | ✅ | ["React","Node","MongoDB"] |

### Section 2 — Budget & Timeline
| Field | Required | Example |
|---|---:|---|
| budgetAmount (INR) | ✅ | 5000 |
| deadline (date) | ❌ | 2026-03-01 |
| experienceLevel | ❌ | `entry` / `intermediate` / `expert` |

### Section 3 — Attachments
Optional files: PDF, images, docs, ZIP. Store as signed URLs or via `/uploads` endpoint. Save metadata: { url, filename, mimeType, size }.

### Section 4 — Visibility & Preferences
| Field | Required | Notes |
|---|---:|---|
| visibility | ✅ | `public` / `invite_only` |
| allowNegotiation | ✅ | boolean (default true) |
| requireVerifiedFreelancer | ❌ | boolean, filters applicants |

### Section 5 — Review & Publish
Client reviews all job details and clicks **Publish**. The job becomes `OPEN` and is immediately visible to freelancers. No upfront payment or escrow is required to publish.

---

## 3) Business rules & lifecycle
- Optional: `maxOpenJobs` configuration per client (enforced on create)
- Job auto-expires after **30 days** (set `expiresAt = createdAt + 30d`)
- Client can **edit** job only if `proposalsCount === 0` and job is not `OPEN`/`FILLED`/`CLOSED`
- Deleting a job sets status `DELETED` or `CANCELLED` with `deletedAt`

---

## 4) Recommended statuses
- `DRAFT` — job created but not published
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
  jobType: { type: String, enum: ['fixed_price'], required: true },
  description: { type: String, required: true },
  skills: [{ type: String }],

  budgetAmount: { type: Number, required: true },
  currency: { type: String, default: 'INR' },

  deadline: { type: Date },
  experienceLevel: { type: String, enum: ['entry','intermediate','expert'] },

  attachments: [{ url: String, filename: String, mimeType: String, size: Number }],

  visibility: { type: String, enum: ['public','invite_only'], default: 'public' },
  allowNegotiation: { type: Boolean, default: true },
  requireVerifiedFreelancer: { type: Boolean, default: false },

  status: { type: String, enum: ['DRAFT','OPEN','FILLED','CLOSED','EXPIRED','CANCELLED','DELETED'], default: 'DRAFT', index: true },

  proposalsCount: { type: Number, default: 0 },

  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now },
  expiresAt: { type: Date, index: true },
  deletedAt: { type: Date }
});

```

Notes:
- Index `client`, `status`, `expiresAt`, `category` for queries.

---

## 6) API Endpoints (examples)
All endpoints should be under `/api/v1` and require authentication. Only clients may create jobs.

### POST /api/v1/jobs
Create a job draft or request to publish with payment.

Request (create and publish job):
```json
POST /api/v1/jobs
{
  "title": "Build e-commerce website",
  "category": "Web Development",
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

---

### PATCH /api/v1/jobs/:id
Update a job draft or make minor edits. Allowed only if `proposalsCount === 0` (otherwise 403).

Request (example):
```json
PATCH /api/v1/jobs/:id
{ "title": "Updated title", "budgetAmount": 6000 }
```

---

### DELETE /api/v1/jobs/:id
Delete (cancel) a job and set job status `CANCELLED` or `DELETED`.

Response (200):
```json
{ "success": true, "message": "Job cancelled" }
```

---

## 7) Attachments & file handling
- Prefer direct uploads to storage provider (S3/GCS) using signed URLs.
- Save metadata in `job.attachments` (url, filename, mimeType, size).
- If backend receives files, validate MIME and size, scan for malware if possible.

---

## 8) Indexes, performance & queries
- Index: `client`, `status`, `expiresAt`, `category` and `skills` for discovery.
- Use text index on `title` and `description` for search.
- Paginate `/jobs` list and add filters: category, budget range, experienceLevel, visibility.

---

## 9) Notifications & webhooks
- Send notifications on: job published, job expired, proposal received.
- Provide webhook for partner integrations if required.

---

## 10) Background tasks & expiry
- Scheduler (cron or queue-based) runs daily to mark jobs with `expiresAt < now` as `EXPIRED` and trigger notifications.
- On expiry evaluate business rules for refund (the default behaviour: do not auto-refund; refund only on explicit delete/cancel to avoid gaming the system). Decide based on product policy.

---

## 11) Security & validations
- Authz: only the job `client` may edit/cancel the job.
- Sanitize all text inputs to avoid XSS.
- Verify file uploads for allowed MIME types and size limits.
- Rate limit job creation to prevent abuse.

---

## 12) Error codes & responses
- 400 Bad Request — validation errors
- 401 Unauthorized — not authenticated
- 403 Forbidden — not client or not allowed to edit
- 404 Not Found — job not found
- 409 Conflict — concurrent publish attempt
- 500 Server Error — unexpected failures

---

## 13) Next steps & implementation checklist
- [ ] Add `Job` Mongoose model (`models/Job.js`)
- [ ] Add job routes & controllers (`routes/jobRoutes.js`, `controllers/jobController.js`)
- [ ] Implement cron job to expire jobs
- [ ] Add tests (unit + integration)

---

If you want, I can now: create `models/Job.js` + `models/Escrow.js`, add controller stubs and routes, or add sample cURL/Postman examples for the endpoints. Which would you like me to implement next? ✅