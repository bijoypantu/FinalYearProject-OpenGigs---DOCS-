# Proposal / Bidding Module (Production-ready Spec)

✅ **Purpose:** Define a robust, secure, and user-friendly proposal/bidding system like real-world freelancing platforms (Upwork, Fiverr) for the project.

---

## Overview

The Proposal/Bidding module enables freelancers to make offers (proposals) on client jobs, supports shortlisting, acceptance, and hiring to create a contract. It enforces business rules (profile completion, limits) and emits notifications and events for UX and automations.

> NOTE: Negotiation / counter-offers are documented separately in a dedicated document.

### Actors
- Client (job poster)
- Freelancer
- System / Admin

---

## Goals & Non-goals
- Goals:
  - Allow freelancers to submit one proposal per job
  - Provide shortlisting to help clients select candidates
  - Ensure safe hiring only after escrow checks

> NOTE: Negotiation / counter-offers are documented separately.
- Non-goals:
  - Payment processing details (handled elsewhere)

---

## User Flows (High-level)
1. Client posts job (Job status: `OPEN`)
2. Freelancers see job on Find Work and open Job Details
3. Freelancer submits Proposal (bid amount, days, cover letter, attachments)
4. Client reviews proposals, sorts/filters, and shortlists candidates
5. Client hires a candidate (creates Contract), job changes to `IN_PROGRESS`; other proposals auto-rejected
6. Freelancer can withdraw (before hiring) and proposals may be auto-rejected when job closes

> NOTE: Negotiation / counter-offers are covered in a separate document.

---

## Proposal Data Model (MongoDB / Mongoose style)

```js
Proposal {
  _id: ObjectId,
  jobId: ObjectId (ref: Job),
  freelancerId: ObjectId (ref: User),
  bidAmount: Number,          // smallest currency unit or float depending shipping
  currency: String,           // e.g. INR, default project currency
  deliveryDays: Number,
  coverLetter: String,
  attachments: [String],      // file URLs
  status: String,             // enum: SUBMITTED | SHORTLISTED | ACCEPTED | REJECTED | WITHDRAWN
  createdAt: Date,
  updatedAt: Date
}
```

Important indexes & constraints:
- Unique index: `{ jobId: 1, freelancerId: 1 }` to enforce one proposal per freelancer per job
- Index on `jobId` for fast listing

---

## Proposal Status Lifecycle

```
SUBMITTED
   |
SHORTLISTED
   |
ACCEPTED  --> CONTRACT CREATED
   |
REJECTED
   |
WITHDRAWN
```

Notes:
- `ACCEPTED` normally leads to `CONTRACT CREATED` and job `IN_PROGRESS`.
- `WITHDRAWN` is allowed only while job is `OPEN` and proposal not `ACCEPTED`.

---

## Business Rules & Validations
- Freelancer can submit only if:
  - Their profile is complete (required fields exist)
  - They have < 3 (configurable) active jobs
- One freelancer = one proposal per job
- Proposal cannot be edited after submission; they can withdraw and submit a new proposal only if allowed by policy
- Protect from race conditions when hiring: use DB transaction or atomic update that verifies job status before creating contract and changing statuses

> NOTE: Negotiation / counter-offers are documented separately.

---

## REST API Endpoints (examples)

Authentication: Bearer token; role-based checks required.

1) Submit Proposal

- POST /api/jobs/:jobId/proposals
- Auth: Freelancer
- Body:
  {
    "bidAmount": 4500,
    "currency": "INR",
    "deliveryDays": 4,
    "coverLetter": "I can finish this in 4 days.",
    "attachments": ["https://.../sample1.pdf"]
  }
- Responses:
  - 201 Created -> Proposal JSON
  - 400 Bad Request -> validation errors
  - 409 Conflict -> proposal already exists

2) List proposals for job (client view)
- GET /api/jobs/:jobId/proposals?sort=price&filter=shortlisted
- Auth: Client (job owner)
- Response: array of proposals with freelancer summary (hide sensitive fields for non-owners)

3) Get proposal
- GET /api/proposals/:proposalId
- Auth: Job owner or proposal owner

4) Withdraw proposal
- POST /api/proposals/:proposalId/withdraw
- Auth: proposal owner
- Rules: only while job is `OPEN` and status not `ACCEPTED` or `WITHDRAWN`

5) Negotiation endpoints (counter-offers) are documented separately.

6) Hire freelancer (creates contract)
- POST /api/jobs/:jobId/hire
- Auth: job owner
- Body: { "proposalId": "..." }
- Server actions:
  - Create Contract record
  - Job status -> `IN_PROGRESS`
  - Proposal status -> `ACCEPTED`
  - All other proposals -> `REJECTED`
  - Return Contract ID

Errors and status codes:
- 403 Forbidden (insufficient permissions)
- 409 Conflict (race conditions: job already closed/hired)
- 422 Unprocessable Entity (business rule violations)

---

## Notifications / Events

Event names (for push/email/webhooks):
- PROPOSAL_SUBMITTED
- PROPOSAL_SHORTLISTED
- PROPOSAL_ACCEPTED
- PROPOSAL_WITHDRAWN
- FREELANCER_HIRED

Implement notifications to:
- Client on new proposal (digest + push)
- Freelancer on acceptance and other important events
- System broadcasts via webhooks for downstream systems (analytics)

---

## UI / UX Notes

Freelancer view:
- Job card shows budget, skills, client rating
- Job details page: `Submit Proposal` button disabled if profile incomplete
- Proposal form with live validation and sample attachments

Client view:
- Proposals listing with columns: Freelancer, Bid, Days, Rating, Actions
- Sorting & filtering panel (price, days, rating, verified)
- ‘Shortlist’ toggle and hire action

Accessibility & Usability:
- Provide templates for cover letters
- Limit attachments size and show previews

---

## Security & Anti-fraud
- Rate-limit proposal submissions per freelancer
- Validate attachments for type & size; virus scanning
- Flag abnormal activity: many proposals with similar content
- RBAC: only job owners perform hire/shortlist actions
- Use transactions to prevent double-hire or double-accept

---

## Testing & QA
- Unit tests for model validations
- Integration tests for API flows: submit → accept → hire
- Concurrency tests for hiring (simulate two hires, expect one success)
- End-to-end scenarios covering notifications and contract creation

---

## Analytics & Metrics
- Bids per job
- Avg bid vs budget
- Time to first bid
- Time from accept to hire
- Conversion rate (proposals → hires)

---

## Admin / Moderation
- Ability to view and remove spam proposals
- Expose logs for disputes
- Tools to re-open jobs or revert contract creation (with audit)

---

## Example (short)
1. Client posts a job: "Build landing page – ₹5000"
2. A bids ₹4800 / 6 days; B bids ₹4500 / 5 days
3. Client shortlists B.
4. Client hires B → contract created, job `IN_PROGRESS`, others `REJECTED`

---

## Migration & Implementation Notes
- Add `proposals` collection with unique index `{ jobId, freelancerId }`
- Add Mongoose validators and pre-save hooks for checks
- Use two-phase commit or DB transactions when creating contracts and updating job+proposal states

---

## Acceptance Criteria
- Submit proposal with valid fields creates document and notifies job owner
- Hiring fails atomically if job already closed
- Withdraw prevents actions when not allowed

---

If you want, I can also:
- Generate a Mongoose model file (e.g., `models/Proposal.js`)
- Add route handlers and tests
- Add a small UI wireframe or API Postman collection

---

*Document created: a detailed, production-focused spec ready for implementation.*
