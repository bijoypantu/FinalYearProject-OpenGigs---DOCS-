# Milestone & Hourly Projects Documentation

## Overview

After a freelancer is hired, the project begins with milestone/hourly proposals. This document covers:

1. **Milestone-Based Projects** (Fixed Price) - Work broken into deliverables with fixed amounts
2. **Hourly-Based Projects** (Time & Materials) - Work tracked by hours at negotiated rate

> 🔑 **Core Principle**: Milestones/hours are **proposed through structured forms**, not free chat.
> - Chat = Discussion
> - Milestones/Hours = Formal, reviewable, lockable system records

---

## Part 1: Milestone-Based Projects (Fixed Price)

### 1.1 When Are Milestones Suggested?

**Correct Timing**: After hiring, before work starts

```
Proposal Accepted
    ↓
Agreement Signed & Both parties have signatures
    ↓
✅ PROJECT CREATED (Type: MILESTONE)
    ↓
Freelancer sees "Propose Milestones" button
    ↓
Freelancer suggests milestones
    ↓
Client approves/requests changes
    ↓
✅ Milestones APPROVED
    ↓
Work starts immediately (no upfront payment)
    ↓
Freelancer completes & submits Milestone 1 for approval
    ↓
Client approves completed work
    ↓
💰 Payment released for Milestone 1
```

**When NOT to suggest**:
- ❌ Before hiring/agreement signed
- ❌ During negotiation (use final offer instead)
- ❌ After project is already active

---

### 1.2 Where Are Milestones Suggested? (UI Location)

**Location**: Project Workspace → [Milestones] Tab

```
┌──────────────────────────────────┐
│ Project Workspace                │
│ ┌──────────────────────────────┐ │
│ │ [Overview] [Milestones] ...  │ │
│ ├──────────────────────────────┤ │
│ │                              │ │
│ │ Status: Awaiting Milestones  │ │
│ │                              │ │
│ │ [💡 Propose Milestones]      │ │ ← Only visible if:
│ │                              │ │   • Project type = MILESTONE
│ │                              │ │   • Status = PENDING_MILESTONE_PROPOSAL
│ │ (No milestones yet)          │ │   • User = Freelancer
│ └──────────────────────────────┘ │
└──────────────────────────────────┘
```

**Conditions for button visibility**:
```javascript
showProposeMilestonesButton = 
  project.type === 'MILESTONE' &&
  project.milestoneStatus === 'PENDING_PROPOSAL' &&
  user.id === freelancer.id &&
  project.status === 'ACTIVE'
```

---

### 1.3 Milestone Proposal Form (Structured Data)

**Cannot be done via chat** - Must use structured form.

#### Form Structure

```
┌────────────────────────────────────────┐
│ PROPOSE MILESTONES                     │
├────────────────────────────────────────┤
│                                        │
│ Project Budget: ₹20,000                │
│ Total Allocated: ₹0 (0%)               │
│                                        │
│ [+ Add Milestone]                      │
│                                        │
│ Milestone 1:                           │
│ ┌────────────────────────────────────┐ │
│ │ Title: [Homepage Design      ]     │ │
│ │ Description:                       │ │
│ │ [Design homepage mockup and    ]   │ │
│ │ [dashboard layout               ]  │ │
│ │ Amount: [5000     ] ₹              │ │
│ │ Duration: [5     ] days            │ │
│ │ [× Remove]                         │ │
│ └────────────────────────────────────┘ │
│                                        │
│ Milestone 2:                           │
│ ┌────────────────────────────────────┐ │
│ │ Title: [Backend API              ] │ │
│ │ Description:                       │ │
│ │ [Build RESTful API with auth   ]   │ │
│ │ Amount: [8000     ] ₹              │ │
│ │ Duration: [8     ] days            │ │
│ │ [× Remove]                         │ │
│ └────────────────────────────────────┘ │
│                                        │
│ Milestone 3:                           │
│ ┌────────────────────────────────────┐ │
│ │ Title: [Testing & QA             ] │ │
│ │ Description:                       │ │
│ │ [Full testing and bug fixes    ]   │ │
│ │ Amount: [7000     ] ₹              │ │
│ │ Duration: [5     ] days            │ │
│ │ [× Remove]                         │ │
│ └────────────────────────────────────┘ │
│                                        │
│ SUMMARY:                               │
│ ├─ Total Amount: ₹20,000 (100%) ✅    │
│ ├─ Total Duration: 18 days             │
│ │  Project Deadline: 20 days ✅       │
│ │                                      │
│ [Cancel] [Save as Draft] [Submit]      │
│                                        │
└────────────────────────────────────────┘
```

#### Per-Milestone Fields

| Field | Type | Required | Validation | Example |
|-------|------|----------|------------|---------|
| Title | String | ✅ | 1-100 chars | "Homepage Design" |
| Description | Text | ✅ | 10-1000 chars | "Design mockups..." |
| Amount (₹) | Number | ✅ | > 0, ≤ remaining budget | 5000 |
| Duration (days) | Number | ✅ | ≥ 1, sum ≤ project days | 5 |

#### System Validations

```javascript
// Validation rules
const validateMilestones = (milestones, project) => {
  const errors = [];
  
  // Rule 1: At least 1 milestone
  if (milestones.length === 0) {
    errors.push("Must propose at least 1 milestone");
  }
  
  // Rule 2: Maximum 10 milestones
  if (milestones.length > 10) {
    errors.push("Cannot have more than 10 milestones");
  }
  
  // Rule 3: Sum of amounts = project budget (exact match)
  const totalAmount = milestones.reduce((sum, m) => sum + m.amount, 0);
  if (totalAmount !== project.budget) {
    errors.push(
      `Total milestone amounts (₹${totalAmount}) must equal ` +
      `project budget (₹${project.budget})`
    );
  }
  
  // Rule 4: Sum of durations ≤ project deadline
  const totalDays = milestones.reduce((sum, m) => sum + m.duration, 0);
  if (totalDays > project.deliveryDays) {
    errors.push(
      `Total duration (${totalDays} days) cannot exceed ` +
      `project deadline (${project.deliveryDays} days)`
    );
  }
  
  // Rule 5: All fields required & valid
  milestones.forEach((m, idx) => {
    if (!m.title || m.title.trim().length < 1) {
      errors.push(`Milestone ${idx + 1}: Title is required`);
    }
    if (!m.description || m.description.trim().length < 10) {
      errors.push(`Milestone ${idx + 1}: Description must be at least 10 chars`);
    }
    if (m.amount <= 0 || !Number.isInteger(m.amount)) {
      errors.push(`Milestone ${idx + 1}: Amount must be a positive integer`);
    }
    if (m.duration <= 0 || !Number.isInteger(m.duration)) {
      errors.push(`Milestone ${idx + 1}: Duration must be a positive integer`);
    }
  });
  
  // Rule 6: No duplicate titles
  const titles = milestones.map(m => m.title.toLowerCase());
  const duplicates = titles.filter((title, idx) => titles.indexOf(title) !== idx);
  if (duplicates.length > 0) {
    errors.push(`Duplicate milestone title: "${duplicates[0]}"`);
  }
  
  return {
    valid: errors.length === 0,
    errors
  };
};
```

---

### 1.4 Form States

#### State A: Draft (Before Submit)

Freelancer can:
- ✅ Add/edit/remove milestones
- ✅ Save as draft
- ❌ Submit (validation errors block it)

```javascript
{
  status: 'DRAFT',
  milestones: [
    { title: 'UI Design', amount: 5000, duration: 5, description: '...' },
    { title: 'Backend', amount: 8000, duration: 8, description: '...' }
  ],
  createdAt: timestamp,
  updatedAt: timestamp,
  savedBy: freelancerId,
  submittedAt: null
}
```

#### State B: Submitted (Awaiting Client Review)

Freelancer **cannot edit** - locked for client review.

System actions:
- Create MilestoneProposal record
- Notify client
- Set project status to `PENDING_MILESTONE_APPROVAL`

```javascript
{
  status: 'SUBMITTED',
  milestones: [...],
  submittedAt: timestamp,
  submittedBy: freelancerId,
  proposalId: 'PROP-001',
  clientReviewStartedAt: null,
  clientReviewCompletedAt: null,
  clientDecision: null
}
```

---

### 1.5 Backend: MilestoneProposal Model

```javascript
const milestoneProposalSchema = new Schema({
  _id: ObjectId,
  
  projectId: {
    type: Schema.Types.ObjectId,
    ref: 'Project',
    required: true,
    index: true
  },
  
  freelancerId: {
    type: Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  
  clientId: {
    type: Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  
  // Milestone list
  milestones: [{
    _id: ObjectId,
    title: String,
    description: String,
    amount: Number,
    duration: Number,
    order: Number // 1, 2, 3...
  }],
  
  // Totals (for quick access)
  totalAmount: Number,
  totalDuration: Number,
  
  // Status
  status: {
    type: String,
    enum: ['DRAFT', 'SUBMITTED', 'APPROVED', 'CHANGES_REQUESTED', 'REJECTED'],
    default: 'DRAFT'
  },
  
  // Client decision
  clientDecision: {
    decision: String, // 'APPROVED', 'CHANGES_REQUESTED', 'REJECTED'
    feedback: String,
    decidedAt: Date,
    decidedBy: Schema.Types.ObjectId
  },
  
  // Resubmission tracking
  revisionNumber: {
    type: Number,
    default: 1
  },
  
  previousProposalId: Schema.Types.ObjectId, // For tracking changes
  
  // Audit
  createdAt: {
    type: Date,
    default: Date.now,
    index: true
  },
  
  submittedAt: Date,
  
  approvedAt: Date,
  
  updatedAt: Date
});
```

---

### 1.6 Client Review Flow

**When client receives notification:**

Client clicks → Opens Project Workspace → [Milestones] tab

```
┌────────────────────────────────────────┐
│ MILESTONE PROPOSAL REVIEW              │
├────────────────────────────────────────┤
│                                        │
│ Status: ⏳ Pending Your Approval       │ 
│ Proposed by: John (Freelancer)         │
│ Submitted: Jan 15, 2026 10:30 AM       │
│                                        │
│ ✏️ Revision #1 (Original Proposal)    │
│                                        │
│ Milestone 1:                           │
│ ├─ Title: Homepage Design              │
│ ├─ Amount: ₹5,000                      │
│ ├─ Duration: 5 days                    │
│ └─ Description: Design mockup...       │
│                                        │
│ Milestone 2:                           │
│ ├─ Title: Backend API                  │
│ ├─ Amount: ₹8,000                      │
│ ├─ Duration: 8 days                    │
│ └─ Description: Build RESTful API...   │
│                                        │
│ Milestone 3:                           │
│ ├─ Title: Testing & QA                 │
│ ├─ Amount: ₹7,000                      │
│ ├─ Duration: 5 days                    │
│ └─ Description: Full testing...        │
│                                        │
│ SUMMARY:                               │
│ ├─ Total: ₹20,000 (Project Budget)     │
│ ├─ Total Duration: 18 days             │
│ └─ Project Deadline: 20 days           │
│                                        │
│ CLIENT OPTIONS:                        │
│ [✅ Approve All] [✏️ Request Changes] [❌ Reject] │
│                                        │
└────────────────────────────────────────┘
```

#### Client Option 1: Approve All

```javascript
POST /api/projects/:projectId/milestones/approve

{
  decision: 'APPROVED'
}
```

**System Actions**:
1. Set milestones status = `APPROVED`
2. Lock milestones (read-only)
3. Set project status = `ACTIVE` (ready for work)
4. Send notification to freelancer: "Milestones approved! You can start work now."
5. Create activity log entry

> 🔑 **Payment happens AFTER milestone completion & approval**, not before.

```json
Response:
{
  "success": true,
  "data": {
    "projectId": "507f...",
    "milestonesApproved": 3,
    "totalAmount": 20000,
    "status": "APPROVED",
    "nextStep": "Freelancer can start work on Milestone 1",
    "message": "All milestones approved! Freelancer can begin work."
  }
}
```

#### Client Option 2: Request Changes

```javascript
POST /api/projects/:projectId/milestones/request-changes

{
  decision: 'CHANGES_REQUESTED',
  feedback: "Can you split Backend API into two milestones? Also, reduce Homepage Design from ₹5,000 to ₹4,000",
  changeRequests: [
    {
      milestoneIndex: 0,
      field: "amount",
      reason: "Budget constraint"
    },
    {
      milestoneIndex: 1,
      field: "split",
      reason: "Too large"
    }
  ]
}
```

**System Actions**:
1. Set status = `CHANGES_REQUESTED`
2. **UNLOCK milestones** for freelancer editing
3. Send notification to freelancer with feedback
4. Freelancer can re-edit and resubmit

```json
Response:
{
  "success": true,
  "data": {
    "projectId": "507f...",
    "status": "CHANGES_REQUESTED",
    "feedback": "Can you split Backend API...",
    "message": "Changes requested. Freelancer notified."
  }
}
```

#### Client Option 3: Reject

```javascript
POST /api/projects/:projectId/milestones/reject

{
  decision: 'REJECTED',
  feedback: "These milestones don't align with project scope. Let's discuss again."
}
```

**System Actions**:
1. Set status = `REJECTED`
2. Pause project
3. Set project status = `PENDING_MILESTONE_APPROVAL` (allow re-proposal)
4. Send notification to freelancer
5. Freelancer can submit new proposal

```json
Response:
{
  "success": true,
  "data": {
    "status": "REJECTED",
    "message": "Proposal rejected. Freelancer can resubmit new proposal.",
    "feedback": "These milestones don't align..."
  }
}
```

---

### 1.7 Freelancer Resubmission Flow

**If client requests changes:**

Freelancer sees notification → Opens workspace → [Milestones] tab

```
┌────────────────────────────────────────┐
│ CHANGES REQUESTED                      │
│                                        │
│ Client feedback: "Can you split        │
│ Backend API into two milestones?       │
│ Also, reduce Homepage Design from      │
│ ₹5,000 to ₹4,000"                      │
│                                        │
│ [📝 View Full Feedback]               │
│                                        │
│ Milestones are now UNLOCKED for edit.  │
│                                        │
│ [✏️ Edit Milestones]                  │
│                                        │
└────────────────────────────────────────┘
```

Freelancer edits:

```
Milestone 1: Homepage Design
- Old amount: ₹5,000 → New amount: ₹4,000

Milestone 2: Backend API (SPLIT into 2):
- Part 1: Backend Auth & User Management - ₹4,000 - 5 days
- Part 2: Backend API Endpoints - ₹4,000 - 3 days

Milestone 3: Testing & QA - ₹8,000 - 7 days

New Total: ₹20,000 ✅
```

Freelancer clicks: **[Resubmit Milestones]**

```javascript
POST /api/projects/:projectId/milestones/resubmit

{
  previousProposalId: "PROP-001",
  milestones: [
    { title: 'Homepage Design', amount: 4000, duration: 5, description: '...' },
    { title: 'Backend Auth & User Mgmt', amount: 4000, duration: 5, description: '...' },
    { title: 'Backend API Endpoints', amount: 4000, duration: 3, description: '...' },
    { title: 'Testing & QA', amount: 8000, duration: 7, description: '...' }
  ]
}
```

**System Actions**:
1. Create new MilestoneProposal (Revision #2)
2. Set status = `SUBMITTED`
3. Lock for editing again
4. Notify client
5. Link to previous proposal for history

```json
Response:
{
  "success": true,
  "data": {
    "proposalId": "PROP-002",
    "revisionNumber": 2,
    "status": "SUBMITTED",
    "message": "Revised proposal submitted for client review."
  }
}
```

---

### 1.8 Final Approval & Project Start

**When client approves milestones:**

```javascript
// System automatically:

1. Lock milestones (permanent read-only)
2. Set project.status = 'ACTIVE'
3. Send notification to freelancer: "Milestones approved! You can start work now."
4. Create Activity log entry
```

> ⚡ **Freelancer can START WORK IMMEDIATELY** - No payment upfront needed.

**Project Workflow (Payment on Completion):**

```
Freelancer starts work on Milestone 1
    ↓
Completes Milestone 1
    ↓
Submits Milestone 1 for approval (in Milestones tab)
    ↓
Client reviews the work
    ├─ ✅ Satisfied? → Approves milestone
    └─ ❌ Not satisfied? → Requests changes
    ↓
Client approves & payment is RELEASED
    ↓
Freelancer receives ₹X,000 for Milestone 1
    ↓
Freelancer can start Milestone 2
    ↓
(Repeat for each milestone)
```

**Key Point**: Payment is tied to work completion and client satisfaction, NOT upfront.

---

### 1.8a Payment Flow (Detailed)

```
Milestone 1: Homepage Design (₹5,000)

Day 1-5: Freelancer works
    ↓
Day 5: Freelancer submits for review
    ↓
Client reviews submitted work
    ├─ Files uploaded
    ├─ Freelancer notes
    └─ Description of work done
    ↓
Client decision:
┌────────────────────────────────────────┐
│ ✅ APPROVE                             │
│ Milestone status: APPROVED             │
│ Payment auto-released: ₹5,000          │
│ Freelancer receives amount             │
│ Status: COMPLETED (for this milestone) │
└────────────────────────────────────────┘

OR

┌────────────────────────────────────────┐
│ ✏️ REQUEST CHANGES                     │
│ Describe what needs to be fixed        │
│ Milestone locked in CHANGES_REQUESTED  │
│ No payment released yet                │
│                                        │
│ Freelancer fixes & resubmits           │
│ Client reviews again                   │
│ (If satisfied) → Payment released      │
└────────────────────────────────────────┘

OR

┌────────────────────────────────────────┐
│ ❌ REJECT                              │
│ Milestone marked as REJECTED           │
│ No payment released                    │
│ Freelancer & client discuss issue      │
│ Can resubmit or escalate               │
└────────────────────────────────────────┘
```

---

### 1.9 Chat Role During Milestone Proposal

**What chat is used for**:
- ✅ Discussing scope of each milestone
- ✅ Clarifying deliverables
- ✅ Explaining why milestones are structured that way
- ✅ Answering questions before submission

**What chat is NOT used for**:
- ❌ Creating/changing milestones
- ❌ Replacing structured milestone form
- ❌ Informal agreements

**Example Chat**:
```
Freelancer: "I've broken down the project into 3 milestones based on scope. 
Let me know if this structure works for you."

Client: "Can you split the Backend work into 2 parts? Auth seems separate 
from API endpoints."

Freelancer: "Good point. I'll update the proposal to split Backend into 
2 milestones. Will resubmit shortly."

(Freelancer edits form and resubmits via structured interface)

Client: "Perfect! These milestones look good now. I'm approving them."
```

---

### 1.10 Validation Edge Cases (Milestone)

#### Edge Case 1: Freelancer submits twice without waiting for approval

**Prevention**:
```javascript
if (currentProposal.status === 'SUBMITTED') {
  return error("A proposal is already under client review. Cannot submit another.");
}
```

#### Edge Case 2: Freelancer removes all milestones

**Prevention**:
```javascript
if (milestones.length === 0) {
  return error("Must propose at least 1 milestone");
}
```

#### Edge Case 3: Milestone amount doesn't match total budget

**Prevention**:
```javascript
const totalAmount = milestones.reduce((sum, m) => sum + m.amount, 0);
if (totalAmount !== project.budget) {
  return error(
    `Total milestones (₹${totalAmount}) must equal project budget (₹${project.budget})`
  );
}
```

#### Edge Case 4: Milestone duration exceeds project deadline

**Prevention**:
```javascript
const totalDays = milestones.reduce((sum, m) => sum + m.duration, 0);
if (totalDays > project.deliveryDays) {
  return error(
    `Total duration (${totalDays} days) exceeds project deadline (${project.deliveryDays} days)`
  );
}
```

#### Edge Case 5: Client tries to approve with invalid budget

**Prevention**:
```javascript
// Milestones are already validated before submission
// Client cannot change them during approval
// Can only: Approve, Request Changes, or Reject
```

#### Edge Case 6: Freelancer edits after submission

**Prevention**:
```javascript
if (proposalStatus === 'SUBMITTED') {
  return error("Cannot edit submitted proposal. Awaiting client decision.");
}
```

#### Edge Case 7: What if project budget is updated after milestones proposed?

**Rule**: Budget changes NOT allowed after milestone proposal starts
```javascript
if (proposalStatus !== null) { // Proposal exists
  return error("Cannot change budget while milestone proposal is under review.");
}
```

---

## Part 2: Hourly-Based Projects

### 2.1 When Are Hours Proposed?

**Same timing as milestones**:

```
Proposal Accepted
    ↓
Agreement Signed
    ↓
✅ PROJECT CREATED (Type: HOURLY)
    ↓
Freelancer proposes estimated hours
    ↓
Client approves/requests changes
    ↓
Work starts (hours tracked real-time)
```

---

### 2.2 Where Are Hours Proposed? (UI Location)

**Location**: Project Workspace → [Milestones] Tab (same interface, different fields)

For hourly projects, instead of "Propose Milestones", button says:

> **"Propose Work Plan"**

---

### 2.3 Hourly Work Plan Form (Structured)

```
┌────────────────────────────────────────┐
│ PROPOSE WORK PLAN (Hourly Project)     │
├────────────────────────────────────────┤
│                                        │
│ Negotiated Hourly Rate: ₹1,000/hour    │
│ Project Budget: ₹20,000                │
│ Available Hours: 20 hours              │
│                                        │
│ [+ Add Work Phase]                     │
│                                        │
│ Phase 1:                               │
│ ┌────────────────────────────────────┐ │
│ │ Title: [UI Design              ]   │ │
│ │ Description: [Create mockups,  ]   │ │
│ │ [wireframes for all pages      ]   │ │
│ │ Estimated Hours: [6          ]     │ │
│ │ [× Remove]                         │ │
│ └────────────────────────────────────┘ │
│                                        │
│ Phase 2:                               │
│ ┌────────────────────────────────────┐ │
│ │ Title: [Backend Development    ]   │ │
│ │ Description: [Build API, auth, ]   │ │
│ │ [database setup               ]    │ │
│ │ Estimated Hours: [10         ]     │ │
│ │ [× Remove]                         │ │
│ └────────────────────────────────────┘ │
│                                        │
│ Phase 3:                               │
│ ┌────────────────────────────────────┐ │
│ │ Title: [Testing & Deployment   ]   │ │
│ │ Description: [QA, bug fixes,   ]   │ │
│ │ [production deployment         ]   │ │
│ │ Estimated Hours: [4           ]    │ │
│ │ [× Remove]                         │ │
│ └────────────────────────────────────┘ │
│                                        │
│ SUMMARY:                               │
│ ├─ Total Hours: 20 hours ✅            │
│ ├─ Estimated Cost: ₹20,000 (100%)    │
│ │  (20 hours × ₹1,000/hour)          │
│ │                                    │
│ │ Note: Actual hours may vary.       │
│ │ You'll be paid for actual hours    │
│ │ tracked during project.            │
│ │                                    │
│ [Cancel] [Save as Draft] [Submit]    │
│                                        │
└────────────────────────────────────────┘
```

#### Per-Phase Fields

| Field | Type | Required | Validation | Example |
|-------|------|----------|------------|---------|
| Title | String | ✅ | 1-100 chars | "UI Design" |
| Description | Text | ✅ | 10-1000 chars | "Create mockups..." |
| Estimated Hours | Number | ✅ | > 0, ≤ available budget hours | 6 |

**Key Difference from Milestones**:
- Estimated hours = planning, not exact
- Actual payment based on **real hours tracked**
- No "duration in days" (work progresses at freelancer's pace)

---

### 2.4 System Validations (Hourly)

```javascript
const validateWorkPlan = (phases, project) => {
  const errors = [];
  
  // Rule 1: At least 1 phase
  if (phases.length === 0) {
    errors.push("Must propose at least 1 work phase");
  }
  
  // Rule 2: Maximum 15 phases
  if (phases.length > 15) {
    errors.push("Cannot have more than 15 phases");
  }
  
  // Rule 3: Sum of hours ≤ budget/hourly rate
  const totalEstimatedHours = phases.reduce((sum, p) => sum + p.estimatedHours, 0);
  const maxAvailableHours = project.budget / project.hourlyRate;
  
  if (totalEstimatedHours > maxAvailableHours) {
    errors.push(
      `Total estimated hours (${totalEstimatedHours}) cannot exceed ` +
      `budget hours (${maxAvailableHours} @ ₹${project.hourlyRate}/hour)`
    );
  }
  
  // Rule 4: All fields required & valid
  phases.forEach((p, idx) => {
    if (!p.title || p.title.trim().length < 1) {
      errors.push(`Phase ${idx + 1}: Title is required`);
    }
    if (!p.description || p.description.trim().length < 10) {
      errors.push(`Phase ${idx + 1}: Description must be at least 10 chars`);
    }
    if (p.estimatedHours <= 0 || !Number.isFinite(p.estimatedHours)) {
      errors.push(`Phase ${idx + 1}: Hours must be a positive number`);
    }
    if (p.estimatedHours > 100) {
      errors.push(`Phase ${idx + 1}: Hours cannot exceed 100`);
    }
  });
  
  // Rule 5: At least 0.5 hours per phase
  phases.forEach((p, idx) => {
    if (p.estimatedHours < 0.5) {
      errors.push(`Phase ${idx + 1}: Minimum 0.5 hours per phase`);
    }
  });
  
  return {
    valid: errors.length === 0,
    errors
  };
};
```

---

### 2.5 HourlyWorkPlan Model

```javascript
const hourlyWorkPlanSchema = new Schema({
  _id: ObjectId,
  
  projectId: {
    type: Schema.Types.ObjectId,
    ref: 'Project',
    required: true,
    index: true
  },
  
  freelancerId: {
    type: Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  
  clientId: {
    type: Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  
  hourlyRate: {
    type: Number,
    required: true // From negotiation
  },
  
  // Work phases
  phases: [{
    _id: ObjectId,
    title: String,
    description: String,
    estimatedHours: Number,
    order: Number
  }],
  
  // Totals
  totalEstimatedHours: Number,
  estimatedCost: Number, // totalEstimatedHours * hourlyRate
  
  // Status
  status: {
    type: String,
    enum: ['DRAFT', 'SUBMITTED', 'APPROVED', 'CHANGES_REQUESTED', 'REJECTED'],
    default: 'DRAFT'
  },
  
  // Client decision
  clientDecision: {
    decision: String,
    feedback: String,
    decidedAt: Date,
    decidedBy: Schema.Types.ObjectId
  },
  
  // Resubmission
  revisionNumber: Number,
  previousPlanId: Schema.Types.ObjectId,
  
  // Timestamps
  createdAt: {
    type: Date,
    default: Date.now
  },
  
  submittedAt: Date,
  approvedAt: Date,
  updatedAt: Date
});
```

---

### 2.6 Client Review Flow (Hourly)

Client sees similar approval interface:

```
┌────────────────────────────────────────┐
│ WORK PLAN REVIEW (Hourly Project)      │
├────────────────────────────────────────┤
│                                        │
│ Hourly Rate (Negotiated): ₹1,000/hour  │
│ Project Budget: ₹20,000                │
│ Available Hours: 20 hours              │
│                                        │
│ Phase 1:                               │
│ ├─ Title: UI Design                   │
│ ├─ Est. Hours: 6                      │
│ ├─ Est. Cost: ₹6,000                  │
│ └─ Description: Create mockups...     │
│                                        │
│ Phase 2:                               │
│ ├─ Title: Backend Development         │
│ ├─ Est. Hours: 10                     │
│ ├─ Est. Cost: ₹10,000                 │
│ └─ Description: Build API, auth...    │
│                                        │
│ Phase 3:                               │
│ ├─ Title: Testing & Deployment        │
│ ├─ Est. Hours: 4                      │
│ ├─ Est. Cost: ₹4,000                  │
│ └─ Description: QA, bug fixes...      │
│                                        │
│ SUMMARY:                               │
│ ├─ Total Estimated Hours: 20 hours    │
│ ├─ Total Estimated Cost: ₹20,000      │
│ ├─ Budget Available: ₹20,000 ✅        │
│ │                                    │
│ │ ℹ️  Actual payment based on hours  │
│ │ tracked during project execution.  │
│ │ This is an ESTIMATE only.          │
│ │                                    │
│ CLIENT OPTIONS:                        │
│ [✅ Approve] [✏️ Request Changes] [❌ Reject] │
│                                        │
└────────────────────────────────────────┘
```

**Client Actions** (same as milestones):

```javascript
// Option 1: Approve
POST /api/projects/:projectId/work-plan/approve

// Option 2: Request Changes
POST /api/projects/:projectId/work-plan/request-changes
{
  feedback: "Can you split UI Design into 2 phases - Design and Prototyping?"
}

// Option 3: Reject
POST /api/projects/:projectId/work-plan/reject
{
  feedback: "These phases are too vague. Please be more specific."
}
```

---

### 2.7 Payment Flow for Hourly Projects

**Key Difference**: Payment is based on **actual tracked hours**, not estimates.

```
Work Plan Approved
    ↓
Freelancer starts tracking time
    ↓
Uses time tracker or manual time logging
    ↓
At end of week/defined period:
├─ Total hours worked: 18 hours
├─ Hourly rate: ₹1,000/hour
├─ Amount owed: ₹18,000
│
│ Freelancer submits timesheet/invoice
│   ↓
│ Client reviews hours worked
│   ├─ ✅ Approve all hours → Payment released
│   ├─ ✏️ Question specific hours → Discussion in chat
│   └─ ❌ Reject hours → Adjust and resubmit
│
├─ Payment released for approved hours
└─ Update project spent amount
```

---

### 2.8 Time Tracking for Hourly Projects

Once work plan is approved, freelancer can track time:

```
┌────────────────────────────────────────┐
│ TIME TRACKING                          │
├────────────────────────────────────────┤
│                                        │
│ Work on: [Phase 1: UI Design    ▼]   │
│                                        │
│ Current Session:                       │
│ ⏱️  [Start] [Pause] [Stop]              │
│ Time: 00:45:23                         │
│                                        │
│ Today's Log:                           │
│ Phase 1 - UI Design:    4.5 hours      │
│ Phase 2 - Backend:      2 hours        │
│ Total Today:            6.5 hours      │
│                                        │
│ This Week:                             │
│ Phase 1 - UI Design:    6 hours        │
│ Phase 2 - Backend:      8 hours        │
│ Phase 3 - Testing:      2 hours        │
│ Total This Week:        16 hours       │
│                                        │
│ Estimated Cost: ₹16,000                │
│ (@ ₹1,000/hour)                        │
│                                        │
│ [Submit Weekly Timesheet]              │
│                                        │
└────────────────────────────────────────┘
```

---

### 2.9 Hourly Timesheet/Invoice Submission

When freelancer submits timesheet:

```javascript
POST /api/projects/:projectId/timesheets/submit

{
  week: "2026-01-13 to 2026-01-19",
  entries: [
    {
      date: "2026-01-13",
      phase: "Phase 1: UI Design",
      hours: 6,
      notes: "Completed homepage and dashboard mockups"
    },
    {
      date: "2026-01-14",
      phase: "Phase 2: Backend",
      hours: 8,
      notes: "Set up API structure and authentication"
    },
    {
      date: "2026-01-15",
      phase: "Phase 3: Testing",
      hours: 2,
      notes: "Initial testing and bug fixes"
    }
  ],
  totalHours: 16,
  estimatedAmount: 16000
}
```

**System Response**:
```json
{
  "success": true,
  "data": {
    "timesheetId": "TS-001",
    "week": "2026-01-13 to 2026-01-19",
    "totalHours": 16,
    "amount": 16000,
    "status": "PENDING_CLIENT_APPROVAL",
    "message": "Timesheet submitted. Awaiting client approval."
  }
}
```

**Client Review**:

```
┌────────────────────────────────────────┐
│ TIMESHEET REVIEW                       │
├────────────────────────────────────────┤
│                                        │
│ Week: Jan 13 - Jan 19, 2026            │
│ Freelancer: John Doe                   │
│ Hourly Rate: ₹1,000/hour               │
│                                        │
│ Monday (Jan 13):                       │
│ Phase 1 - UI Design: 6 hours           │
│ Notes: "Completed homepage..."         │
│ [✓] [✗] [?]                            │
│                                        │
│ Tuesday (Jan 14):                      │
│ Phase 2 - Backend: 8 hours             │
│ Notes: "Set up API structure..."       │
│ [✓] [✗] [?]                            │
│                                        │
│ Wednesday (Jan 15):                    │
│ Phase 3 - Testing: 2 hours             │
│ Notes: "Initial testing..."            │
│ [✓] [✗] [?]                            │
│                                        │
│ TOTAL:                                 │
│ ├─ Hours: 16                           │
│ ├─ Amount: ₹16,000                     │
│ │                                    │
│ [✅ Approve All] [❓ Query Hours] [❌ Reject] │
│                                        │
└────────────────────────────────────────┘
```

**Client Options**:

```javascript
// Option 1: Approve all hours
POST /api/projects/:projectId/timesheets/:timesheetId/approve
{
  decision: 'APPROVED'
}
// → Payment auto-released

// Option 2: Query specific hours
POST /api/projects/:projectId/timesheets/:timesheetId/query
{
  queries: [
    {
      date: "2026-01-14",
      hours: 8,
      reason: "Backend development seems like more than 8 hours?"
    }
  ]
}
// → Chat notification, freelancer can clarify/adjust

// Option 3: Reject timesheet
POST /api/projects/:projectId/timesheets/:timesheetId/reject
{
  feedback: "Some hours don't align with project progress. Let's discuss."
}
// → Freelancer can resubmit adjusted timesheet
```

---

### 2.10 Hourly Project Payment Scenarios

#### Scenario 1: Freelancer completes early (under estimated hours)

```
Estimated: 20 hours @ ₹1,000/hour = ₹20,000 budget
Actual: 15 hours worked
Payment: ₹15,000 (15 × ₹1,000)
Remaining: ₹5,000 stays with client
```

#### Scenario 2: Freelancer needs more hours (exceeds estimate)

```
Estimated: 20 hours @ ₹1,000/hour = ₹20,000 budget
Actual: 22 hours worked
Payment owed: ₹22,000 (22 × ₹1,000)
Problem: Exceeds budget!

System Rule: Check before charging
├─ If client approved unlimited hours → Pay ₹22,000
├─ If budget capped → Prompt client
│  • Approve extra ₹2,000?
│  • Or reject extra hours?
└─ Default: Freeze at ₹20,000 budget limit
```

#### Scenario 3: Client disputes hours

```
Timesheet submitted: 10 hours on one day
Client: "That seems too long for one day"

Client clicks: [Query Hours]
Chat notification sent
Freelancer explains: "I worked 10 hours that day because..."
Client can:
├─ Accept explanation → Approve hours
├─ Request adjustment → Freelancer edits
└─ Reject → Timesheet resubmitted
```

---

### 2.11 Validation Edge Cases (Hourly)

#### Edge Case 1: Freelancer estimates too many hours (exceeds budget)

**Prevention**:
```javascript
const maxHours = project.budget / project.hourlyRate;
if (totalEstimatedHours > maxHours) {
  return error(
    `Total estimated hours (${totalEstimatedHours}) exceeds ` +
    `budget capacity (${maxHours} @ ₹${project.hourlyRate}/hour)`
  );
}
```

#### Edge Case 2: Freelancer logs 0 hours in a day

**Prevention**:
```javascript
timesheet.entries.forEach(entry => {
  if (entry.hours <= 0) {
    return error(`Date ${entry.date}: Hours must be greater than 0`);
  }
});
```

#### Edge Case 3: Freelancer submits timesheet before any work

**Prevention**:
```javascript
if (totalHours === 0) {
  return error("Timesheet must have at least 1 hour logged");
}
```

#### Edge Case 4: Client approves hours, then changes hourly rate

**Rule**: Hourly rate is locked once project starts
```javascript
if (project.status === 'ACTIVE') {
  return error("Cannot change hourly rate after project started");
}
```

#### Edge Case 5: Freelancer submits hours exceeding budget

**System handles this**:
```javascript
if (timesheet.totalAmount > project.budget) {
  const shortage = timesheet.totalAmount - project.budget;
  
  // Notify client:
  // "Timesheet exceeds budget by ₹{shortage}. Approve extra charge?"
  
  // Client can:
  // • Approve extra payment
  // • Ask freelancer to reduce hours
  // • Reject timesheet
}
```

#### Edge Case 6: Multiple timesheets overlap (duplicate hours)

**Prevention**:
```javascript
// Check if hours already logged for same dates
const existingEntries = await Timesheet.find({
  projectId,
  week: newTimesheet.week,
  status: 'APPROVED'
});

if (existingEntries.length > 0) {
  return error("Timesheet already submitted for this week");
}
```

---

## 3. Comparison: Milestone vs Hourly

| Aspect | Milestone-Based | Hourly-Based |
|--------|-----------------|--------------|
| **Payment Trigger** | Milestone approved by client | Hours logged & approved by client |
| **Estimated Cost** | Fixed (known upfront) | Estimated (varies with actual hours) |
| **Budget Control** | Fixed budget per milestone | Total budget + hourly rate |
| **Freelancer Risk** | If takes longer, no extra pay | If takes longer, paid for actual hours |
| **Client Risk** | Fixed cost (predictable) | Variable cost (can exceed estimate) |
| **Proposal Form** | Title, Description, Amount, Days | Title, Description, Estimated Hours |
| **Deliverable** | Completed work per milestone | Hours logged in time tracker |
| **Payment Release** | Approval = Payment (binary) | Timesheet approval = Payment |
| **Flexibility** | Less (locked milestones) | More (hours can adjust) |
| **Tracking** | Work submission & approval | Time tracking & timesheet |
| **When To Use** | Well-defined scope | Unclear scope or exploratory work |

---

## 4. API Endpoints Summary

### Milestone Endpoints

```
POST /api/projects/:projectId/milestones/propose
- Freelancer creates milestone proposal

GET /api/projects/:projectId/milestones/proposal
- Get current milestone proposal (any status)

PUT /api/projects/:projectId/milestones/draft
- Update draft (before submit)

POST /api/projects/:projectId/milestones/submit
- Submit for client approval

POST /api/projects/:projectId/milestones/approve
- Client approves all milestones

POST /api/projects/:projectId/milestones/request-changes
- Client requests changes

POST /api/projects/:projectId/milestones/reject
- Client rejects proposal

POST /api/projects/:projectId/milestones/resubmit
- Freelancer resubmits after changes requested

GET /api/projects/:projectId/milestones/history
- View all milestone proposals & revisions
```

### Hourly Work Plan Endpoints

```
POST /api/projects/:projectId/work-plan/propose
- Freelancer creates work plan

GET /api/projects/:projectId/work-plan
- Get current work plan

PUT /api/projects/:projectId/work-plan/draft
- Update draft

POST /api/projects/:projectId/work-plan/submit
- Submit for client approval

POST /api/projects/:projectId/work-plan/approve
- Client approves

POST /api/projects/:projectId/work-plan/request-changes
- Client requests changes

POST /api/projects/:projectId/work-plan/reject
- Client rejects

POST /api/projects/:projectId/work-plan/resubmit
- Freelancer resubmits after changes

GET /api/projects/:projectId/work-plan/history
- View all work plan revisions
```

### Timesheet Endpoints

```
POST /api/projects/:projectId/timesheets/submit
- Submit timesheet for a period

GET /api/projects/:projectId/timesheets
- Get all timesheets

POST /api/projects/:projectId/timesheets/:timesheetId/approve
- Client approves timesheet

POST /api/projects/:projectId/timesheets/:timesheetId/query
- Client questions specific hours

POST /api/projects/:projectId/timesheets/:timesheetId/reject
- Client rejects timesheet

PUT /api/projects/:projectId/timesheets/:timesheetId/update
- Freelancer updates rejected timesheet
```

---

## 5. Testing & Acceptance Criteria

### Milestone Project Testing

✅ Freelancer can create milestone proposal with 1-10 milestones
✅ Sum of milestone amounts must equal project budget
✅ Sum of milestone durations must fit within project deadline
✅ Cannot submit with validation errors
✅ Submitted proposal locks for editing
✅ Client can approve all milestones
✅ Client can request changes (unlocks for editing)
✅ Client can reject (requires new proposal)
✅ Freelancer can resubmit after changes requested
✅ Milestones history shows all revisions
✅ Freelancer can start work immediately after milestone approval (no upfront payment)
✅ Project status changes to ACTIVE after milestones approved
✅ Payment released only after milestone submission & client approval
✅ Activity log captures all milestone actions

### Hourly Project Testing

✅ Freelancer can create work plan with 1-15 phases
✅ Total estimated hours fit within budget/hourly rate
✅ Cannot submit with validation errors
✅ Client can approve work plan
✅ Client can request changes
✅ Freelancer can start time tracking after approval
✅ Timesheet submitted with total hours and daily breakdown
✅ Client can approve timesheet
✅ Client can query specific hours
✅ Payment calculation correct: hours × hourly rate
✅ Cannot exceed project budget without client approval
✅ Timesheet rejection allows resubmission
✅ Activity log captures all timesheet actions

### Chat Integration Testing

✅ Chat available during milestone proposal phase
✅ Messages about milestones don't create/modify them
✅ System messages appear when proposal submitted
✅ Notifications sent to relevant party
✅ Links from chat to workspace tabs work

---

## 6. Implementation Checklist

### Phase 1: Milestone Backend

- [ ] Create MilestoneProposal schema
- [ ] Create proposal endpoints (create, submit, approve, reject, resubmit)
- [ ] Add validation logic
- [ ] Create activity logging
- [ ] Add notifications for status changes

### Phase 2: Milestone Frontend

- [ ] Build milestone proposal form
- [ ] Add form validation & error display
- [ ] Create client approval interface
- [ ] Build milestone history view
- [ ] Add status badges & indicators

### Phase 3: Hourly Backend

- [ ] Create HourlyWorkPlan schema
- [ ] Create work plan endpoints
- [ ] Create Timesheet schema
- [ ] Create timesheet endpoints
- [ ] Add time tracking backend logic

### Phase 4: Hourly Frontend

- [ ] Build work plan proposal form
- [ ] Build time tracker UI
- [ ] Create timesheet submission form
- [ ] Build client timesheet review interface
- [ ] Add hour validation & calculations

### Phase 5: Integration

- [ ] Integrate with payment system
- [ ] Integrate with project status workflow
- [ ] Add real-time WebSocket events
- [ ] Add to activity timeline
- [ ] Add notifications

### Phase 6: Testing & QA

- [ ] Unit tests for all validation
- [ ] Integration tests for workflows
- [ ] E2E tests for proposal submission
- [ ] Permission & authorization testing
- [ ] Edge case testing

---

## 7. Key Design Principles

1. **Structured over Free-Form**: Milestones/hours must use forms, not chat
2. **Client Approval, Payment on Delivery**: Client approves milestones before work, but pays after review & approval of completed work
3. **Audit Trail**: Every change logged with history
4. **Validation First**: Prevent invalid proposals early
5. **Clear Status**: Always show what state proposal is in
6. **No Surprises**: Budget locked upfront (milestones) or transparent (hourly)
7. **Dispute Prevention**: Structured data prevents misunderstandings

---

## 8. Future Enhancements

```
Phase 2 Features:
├─ Auto-generate milestones from project description (AI)
├─ Suggested hour estimates based on project complexity
├─ Recurring time entries for similar work
├─ Time tracking integrations (Toggl, Clockify)
├─ Bulk invoice generation
└─ Overtime tracking & alerts
```

---

*Document created: Comprehensive Milestone and Hourly Projects documentation covering proposal creation, client approval, payment tracking, edge cases, and implementation guide.*
