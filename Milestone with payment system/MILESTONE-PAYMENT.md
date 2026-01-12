# Milestone Payment Module Documentation

## Overview

The Milestone Payment Module provides two flexible payment options for job postings:
- **Full Payment** for small projects (≤ ₹3,000)
- **Milestone-based** for large projects (> ₹10,000)

---

## 1. Payment Options

### Option A: Full Payment Jobs (Small Projects)

**Use Case:** Quick projects like logo design, bug fixes, resume design

**Examples:**
- Logo design: ₹800
- Bug fix: ₹500
- Resume design: ₹300

**System Behavior:**
1. Client posts job and pays **25% escrow** at posting
2. Freelancer completes full work
3. Client approves the submission
4. Client pays remaining **75%**
5. Money released to freelancer

**Database Storage:**
```
paymentType: "FULL"
```

**Technical Implementation:**
- Single virtual milestone created internally
- No user-visible milestone management
- Direct payment flow

---

### Option B: Milestone-based Jobs (Large Projects)

**Use Case:** Complex projects with multiple deliverables

**Examples:**
- Website development: ₹20,000
- Mobile app: ₹50,000
- E-commerce platform: ₹100,000

**System Behavior:**
1. Client posts job without escrow
2. After hiring freelancer, client defines milestones
3. Client funds **first milestone amount** (not 25% escrow)
4. Freelancer completes milestone work
5. Client approves and milestone payment released
6. Client funds next milestone
7. Process repeats for all milestones

**Example Milestone Structure:**
| Title | Amount | Status |
|-------|--------|--------|
| UI Design | ₹5,000 | Pending |
| Backend | ₹8,000 | Pending |
| Testing | ₹7,000 | Pending |
| **Total** | **₹20,000** | |

**Database Storage:**
```
paymentType: "MILESTONE"
```

---

## 2. Smart UX Guidance

### Budget-based Recommendations

**If Job Budget ≤ ₹3,000:**
```
"Full payment is recommended for small projects. 
This is faster and simpler for both you and the freelancer."
```

**If Job Budget > ₹10,000:**
```
"Milestones are recommended for better security and control. 
Break your project into deliverables and release payment as work completes."
```

**Implementation in Frontend:**
- Show recommendation when client enters budget
- Allow override if needed
- Store selection in database

---

## 3. Database Schema Changes

### Job Model Updates

```javascript
// In Job model
const jobSchema = new Schema({
  // ... existing fields
  
  paymentType: {
    type: String,
    enum: ["FULL", "MILESTONE"],
    required: true,
    default: "FULL"
  },
  
  budget: {
    type: Number,
    required: true,
    min: 0
  },
  
  currency: {
    type: String,
    default: "INR"
  },
  
  initialEscrowAmount: {
    type: Number,
    default: 0 // 25% for FULL, 0 for MILESTONE
  },
  
  totalAmount: {
    type: Number,
    required: true
  },
  
  isEscrowPaid: {
    type: Boolean,
    default: false
  }
});
```

### Project Model Updates

```javascript
// In Project model (created after hiring)
const projectSchema = new Schema({
  jobId: {
    type: Schema.Types.ObjectId,
    ref: 'Job',
    required: true
  },
  
  clientId: {
    type: Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  
  freelancerId: {
    type: Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  
  paymentType: {
    type: String,
    enum: ["FULL", "MILESTONE"],
    required: true
  },
  
  totalAmount: {
    type: Number,
    required: true
  },
  
  status: {
    type: String,
    enum: ["ACTIVE", "COMPLETED", "CANCELLED"],
    default: "ACTIVE"
  },
  
  createdAt: {
    type: Date,
    default: Date.now
  }
});
```

### Milestone Model (NEW)

```javascript
const milestoneSchema = new Schema({
  projectId: {
    type: Schema.Types.ObjectId,
    ref: 'Project',
    required: true
  },
  
  title: {
    type: String,
    required: true
  },
  
  description: {
    type: String
  },
  
  amount: {
    type: Number,
    required: true,
    min: 0
  },
  
  dueDate: {
    type: Date
  },
  
  status: {
    type: String,
    enum: ["PENDING", "IN_PROGRESS", "SUBMITTED", "APPROVED", "RELEASED", "REJECTED"],
    default: "PENDING"
  },
  
  order: {
    type: Number,
    required: true // For sequencing
  },
  
  isFunded: {
    type: Boolean,
    default: false // Client has paid this milestone amount
  },
  
  fundedAt: {
    type: Date
  },
  
  submittedAt: {
    type: Date
  },
  
  approvedAt: {
    type: Date
  },
  
  releasedAt: {
    type: Date
  },
  
  submissionDetails: {
    description: String,
    attachments: [String], // URLs
    submittedBy: Schema.Types.ObjectId
  },
  
  rejectionReason: String,
  
  createdAt: {
    type: Date,
    default: Date.now
  }
});
```

### Payment Transaction Model (NEW)

```javascript
const paymentTransactionSchema = new Schema({
  projectId: {
    type: Schema.Types.ObjectId,
    ref: 'Project',
    required: true
  },
  
  milestoneId: {
    type: Schema.Types.ObjectId,
    ref: 'Milestone'
    // null for FULL payment projects
  },
  
  amount: {
    type: Number,
    required: true
  },
  
  transactionType: {
    type: String,
    enum: ["ESCROW_PAYMENT", "MILESTONE_PAYMENT", "REFUND"],
    required: true
  },
  
  status: {
    type: String,
    enum: ["PENDING", "SUCCESS", "FAILED"],
    default: "PENDING"
  },
  
  paymentGatewayId: String,
  
  paidBy: {
    type: Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  
  releasedTo: {
    type: Schema.Types.ObjectId,
    ref: 'User'
  },
  
  releasedAt: Date,
  
  createdAt: {
    type: Date,
    default: Date.now
  }
});
```

---

## 4. API Endpoints

### 4.1 Job Creation

#### Endpoint: `POST /api/jobs`

**Request Body:**
```json
{
  "title": "Website Development",
  "description": "Build a modern e-commerce website",
  "budget": 25000,
  "paymentType": "MILESTONE",
  "category": "Web Development",
  "skillsRequired": ["React", "Node.js", "MongoDB"],
  "deadline": "2026-03-11",
  "experienceLevel": "Intermediate"
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "jobId": "507f1f77bcf86cd799439011",
    "title": "Website Development",
    "budget": 25000,
    "paymentType": "MILESTONE",
    "status": "OPEN",
    "recommendation": "Milestones are recommended for better security and control. Break your project into deliverables and release payment as work completes.",
    "createdAt": "2026-01-11T10:30:00Z"
  }
}
```

---

### 4.2 Hire Freelancer & Create Project

#### Endpoint: `POST /api/projects/create`

**Request Body:**
```json
{
  "jobId": "507f1f77bcf86cd799439011",
  "freelancerId": "507f1f77bcf86cd799439012",
  "proposalId": "507f1f77bcf86cd799439013"
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "projectId": "507f1f77bcf86cd799439014",
    "jobId": "507f1f77bcf86cd799439011",
    "clientId": "507f1f77bcf86cd799439010",
    "freelancerId": "507f1f77bcf86cd799439012",
    "paymentType": "MILESTONE",
    "totalAmount": 25000,
    "status": "ACTIVE",
    "nextStep": "Create milestones to get started",
    "createdAt": "2026-01-11T10:35:00Z"
  }
}
```

---

### 4.3 Create Milestones (For Milestone-based Projects)

#### Endpoint: `POST /api/projects/:projectId/milestones`

**Request Body:**
```json
{
  "milestones": [
    {
      "title": "UI Design & Prototyping",
      "description": "Design mockups and interactive prototypes",
      "amount": 5000,
      "dueDate": "2026-02-11",
      "order": 1
    },
    {
      "title": "Backend Development",
      "description": "API development and database setup",
      "amount": 12000,
      "dueDate": "2026-02-28",
      "order": 2
    },
    {
      "title": "Frontend Implementation",
      "description": "Implement UI in React",
      "amount": 6000,
      "dueDate": "2026-03-10",
      "order": 3
    },
    {
      "title": "Testing & Deployment",
      "description": "QA testing and production deployment",
      "amount": 2000,
      "dueDate": "2026-03-11",
      "order": 4
    }
  ]
}
```

**Validation Rules:**
- Sum of milestone amounts must equal project total amount
- All order numbers must be unique and sequential
- Due dates should be in ascending order

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "projectId": "507f1f77bcf86cd799439014",
    "milestonesCreated": 4,
    "totalAmount": 25000,
    "milestones": [
      {
        "milestoneId": "507f1f77bcf86cd799439015",
        "title": "UI Design & Prototyping",
        "amount": 5000,
        "status": "PENDING",
        "isFunded": false,
        "order": 1
      },
      {
        "milestoneId": "507f1f77bcf86cd799439016",
        "title": "Backend Development",
        "amount": 12000,
        "status": "PENDING",
        "isFunded": false,
        "order": 2
      },
      {
        "milestoneId": "507f1f77bcf86cd799439017",
        "title": "Frontend Implementation",
        "amount": 6000,
        "status": "PENDING",
        "isFunded": false,
        "order": 3
      },
      {
        "milestoneId": "507f1f77bcf86cd799439018",
        "title": "Testing & Deployment",
        "amount": 2000,
        "status": "PENDING",
        "isFunded": false,
        "order": 4
      }
    ],
    "nextStep": "Fund the first milestone to get started"
  }
}
```

---

### 4.4 Fund Milestone (Client Payment)

#### Endpoint: `POST /api/milestones/:milestoneId/fund`

**Request Body:**
```json
{
  "paymentMethodId": "pm_507f1f77bcf86cd",
  "saveCard": true
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "transactionId": "txn_507f1f77bcf86cd799439019",
    "milestoneId": "507f1f77bcf86cd799439015",
    "projectId": "507f1f77bcf86cd799439014",
    "amount": 5000,
    "status": "SUCCESS",
    "message": "Milestone funded successfully",
    "isFunded": true,
    "fundedAt": "2026-01-11T10:40:00Z",
    "nextMilestone": {
      "milestoneId": "507f1f77bcf86cd799439016",
      "title": "Backend Development",
      "amount": 12000
    }
  }
}
```

**Error Response (400):**
```json
{
  "success": false,
  "error": "Milestone already funded",
  "code": "MILESTONE_ALREADY_FUNDED"
}
```

---

### 4.5 Submit Milestone Work

#### Endpoint: `POST /api/milestones/:milestoneId/submit`

**Request Body:**
```json
{
  "description": "Completed all mockups and interactive prototypes. Delivered in Figma and Adobe XD format.",
  "attachments": [
    "https://figma.com/file/...",
    "https://storage.example.com/milestone-1-files.zip"
  ]
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "milestoneId": "507f1f77bcf86cd799439015",
    "status": "SUBMITTED",
    "submittedAt": "2026-02-11T14:20:00Z",
    "message": "Work submitted successfully. Waiting for client approval.",
    "submissionDetails": {
      "description": "Completed all mockups and interactive prototypes. Delivered in Figma and Adobe XD format.",
      "attachments": [
        "https://figma.com/file/...",
        "https://storage.example.com/milestone-1-files.zip"
      ],
      "submittedBy": "507f1f77bcf86cd799439012"
    }
  }
}
```

---

### 4.6 Approve/Reject Milestone

#### Endpoint: `POST /api/milestones/:milestoneId/review`

**Request Body (Approve):**
```json
{
  "action": "APPROVE",
  "feedback": "Excellent work! All requirements met perfectly."
}
```

**Request Body (Reject):**
```json
{
  "action": "REJECT",
  "reason": "Some design elements need revision. Please check comments."
}
```

**Response (Approve - 200 OK):**
```json
{
  "success": true,
  "data": {
    "milestoneId": "507f1f77bcf86cd799439015",
    "status": "APPROVED",
    "approvedAt": "2026-02-12T09:15:00Z",
    "message": "Milestone approved! Payment will be released to freelancer.",
    "paymentRelease": {
      "transactionId": "txn_507f1f77bcf86cd799439020",
      "amount": 5000,
      "freelancerId": "507f1f77bcf86cd799439012",
      "releasedAt": "2026-02-12T09:15:00Z",
      "status": "SUCCESS"
    },
    "nextMilestone": {
      "milestoneId": "507f1f77bcf86cd799439016",
      "title": "Backend Development",
      "amount": 12000,
      "status": "PENDING",
      "isFunded": false
    }
  }
}
```

**Response (Reject - 200 OK):**
```json
{
  "success": true,
  "data": {
    "milestoneId": "507f1f77bcf86cd799439015",
    "status": "REJECTED",
    "rejectionReason": "Some design elements need revision. Please check comments.",
    "message": "Milestone rejected. Freelancer will be notified to revise work.",
    "resubmissionDeadline": "2026-02-19T09:15:00Z"
  }
}
```

---

### 4.7 Get Project Payment Status

#### Endpoint: `GET /api/projects/:projectId/payment-status`

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "projectId": "507f1f77bcf86cd799439014",
    "paymentType": "MILESTONE",
    "totalAmount": 25000,
    "paidAmount": 5000,
    "pendingAmount": 20000,
    "releaseAmount": 0,
    "paymentProgress": "20%",
    "milestones": [
      {
        "milestoneId": "507f1f77bcf86cd799439015",
        "title": "UI Design & Prototyping",
        "amount": 5000,
        "status": "APPROVED",
        "isFunded": true,
        "releasedAt": "2026-02-12T09:15:00Z"
      },
      {
        "milestoneId": "507f1f77bcf86cd799439016",
        "title": "Backend Development",
        "amount": 12000,
        "status": "PENDING",
        "isFunded": false,
        "fundingDeadline": "2026-02-28T23:59:59Z"
      },
      {
        "milestoneId": "507f1f77bcf86cd799439017",
        "title": "Frontend Implementation",
        "amount": 6000,
        "status": "PENDING",
        "isFunded": false
      },
      {
        "milestoneId": "507f1f77bcf86cd799439018",
        "title": "Testing & Deployment",
        "amount": 2000,
        "status": "PENDING",
        "isFunded": false
      }
    ]
  }
}
```

---

### 4.8 Complete Full Payment Project

#### Endpoint: `POST /api/projects/:projectId/submit-full-payment`

**Request Body:**
```json
{
  "description": "All work completed as per requirements",
  "attachments": [
    "https://storage.example.com/final-deliverables.zip"
  ]
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "projectId": "507f1f77bcf86cd799439014",
    "status": "SUBMITTED",
    "submittedAt": "2026-01-20T16:30:00Z",
    "message": "Work submitted! Waiting for client final approval and remaining payment.",
    "paymentDue": {
      "amount": 375, // 75% of 500
      "dueBy": "2026-01-25T23:59:59Z"
    }
  }
}
```

---

### 4.9 Approve & Pay Full Payment Project

#### Endpoint: `POST /api/projects/:projectId/approve-full-payment`

**Request Body:**
```json
{
  "action": "APPROVE",
  "feedback": "Perfect work, exactly as required!",
  "paymentMethodId": "pm_507f1f77bcf86cd"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "projectId": "507f1f77bcf86cd799439014",
    "status": "COMPLETED",
    "approvedAt": "2026-01-20T17:00:00Z",
    "payment": {
      "transactionId": "txn_507f1f77bcf86cd799439021",
      "amount": 375,
      "status": "SUCCESS",
      "releasedTo": "507f1f77bcf86cd799439012",
      "releasedAt": "2026-01-20T17:00:00Z"
    },
    "totalPaid": 500,
    "message": "Project completed! Full payment released to freelancer."
  }
}
```

---

## 5. Business Logic Flow

### Flow 1: FULL Payment Project

```
1. Client posts job with budget ≤ ₹3,000
   ↓
2. System suggests "Full payment is recommended"
   ↓
3. Client pays 25% escrow (₹125 for ₹500 job)
   ↓
4. Freelancer selected and project created
   ↓
5. Single virtual milestone created internally
   ↓
6. Freelancer completes work and submits
   ↓
7. Client reviews submission
   ↓
8. Client approves + pays remaining 75% (₹375)
   ↓
9. Full amount (₹500) released to freelancer
   ↓
10. Project marked as COMPLETED
```

### Flow 2: MILESTONE Payment Project

```
1. Client posts job with budget > ₹10,000
   ↓
2. System suggests "Milestones recommended for security"
   ↓
3. NO escrow payment at posting
   ↓
4. Freelancer selected and project created
   ↓
5. Client defines milestones with breakdown
   ↓
6. Loop for each milestone:
   a) Client funds first milestone
   b) Freelancer starts work
   c) Freelancer submits work
   d) Client reviews and approves
   e) Payment released to freelancer
   f) Move to next milestone
   ↓
7. All milestones completed
   ↓
8. Project marked as COMPLETED
```

---

## 6. Implementation Checklist

### Database Changes
- [ ] Update Job model with `paymentType` field
- [ ] Update Project model with `paymentType` field
- [ ] Create Milestone model
- [ ] Create PaymentTransaction model

### Backend Controllers
- [ ] Create `milestoneController.js`
  - [ ] Create milestone
  - [ ] Fund milestone
  - [ ] Submit milestone work
  - [ ] Approve/Reject milestone
  - [ ] Get milestone details
  - [ ] Get payment status

### API Routes
- [ ] Create `milestoneRoutes.js`
- [ ] Update `jobRoutes.js` with payment type recommendation
- [ ] Update `projectRoutes.js` for full payment flow

### Payment Integration
- [ ] Integrate with payment gateway for milestone funding
- [ ] Implement payment release on approval
- [ ] Handle payment failures and refunds

### Frontend UX
- [ ] Budget input with recommendation banner
- [ ] Milestone creation form
- [ ] Milestone status dashboard
- [ ] Payment UI for milestone funding
- [ ] Work submission form per milestone
- [ ] Approval/Rejection interface

### Validation & Security
- [ ] Validate milestone amounts sum to total
- [ ] Prevent out-of-order milestone funding
- [ ] Ensure only project owner can approve milestones
- [ ] Prevent duplicate payments
- [ ] Rate limiting on payment endpoints

### Notifications
- [ ] Notify freelancer when milestone is funded
- [ ] Notify client when work is submitted
- [ ] Notify freelancer when milestone is approved/rejected
- [ ] Notify freelancer when payment is released

---

## 7. Key Rules & Constraints

### For FULL Payment Projects:
1. ✅ Client pays 25% at posting (escrow)
2. ✅ No milestone creation visible to user
3. ✅ Single virtual milestone internally
4. ✅ Freelancer submits all work once
5. ✅ Client pays 75% after approval
6. ✅ Full amount released after final approval

### For MILESTONE Projects:
1. ✅ NO escrow at posting
2. ✅ Client defines milestones after hiring
3. ✅ Client must fund each milestone before work begins
4. ✅ Milestone amounts must sum to total project amount
5. ✅ Payment released only after client approval
6. ✅ Freelancer cannot start next milestone until current is approved
7. ✅ Rejected milestones go back to "PENDING" for resubmission

### General Rules:
1. ✅ Budget ≤ ₹3,000 → Recommend FULL payment
2. ✅ Budget > ₹10,000 → Recommend MILESTONE
3. ✅ Budget between → Allow user choice
4. ✅ Payment type is immutable after job posting
5. ✅ All timestamps in UTC
6. ✅ Amounts in INR (or configurable currency)

---

## 8. Error Handling

### Common Errors:

**Milestone Already Funded**
```json
{
  "success": false,
  "error": "Milestone already funded",
  "code": "MILESTONE_ALREADY_FUNDED",
  "statusCode": 400
}
```

**Milestone Amount Mismatch**
```json
{
  "success": false,
  "error": "Sum of milestone amounts must equal total project amount",
  "code": "MILESTONE_AMOUNT_MISMATCH",
  "statusCode": 400,
  "details": {
    "expectedSum": 25000,
    "actualSum": 24000
  }
}
```

**Payment Failed**
```json
{
  "success": false,
  "error": "Payment processing failed",
  "code": "PAYMENT_FAILED",
  "statusCode": 402,
  "details": {
    "gatewayError": "Card declined"
  }
}
```

**Cannot Fund Out of Order**
```json
{
  "success": false,
  "error": "You must fund milestones in order",
  "code": "MILESTONE_OUT_OF_ORDER",
  "statusCode": 400,
  "details": {
    "currentMilestone": "UI Design & Prototyping",
    "nextAllowedMilestone": "Backend Development"
  }
}
```

---

## 9. Summary Table

| Feature | FULL Payment | MILESTONE Payment |
|---------|--------------|------------------|
| **Recommended Budget** | ≤ ₹3,000 | > ₹10,000 |
| **Escrow at Posting** | 25% | None |
| **Milestone Definition** | Auto (1 virtual) | Client defines |
| **Payment Timing** | 25% → 75% → Release | Milestone by milestone |
| **Approval Per Milestone** | Once at end | Per milestone |
| **Best For** | Quick projects | Complex projects |
| **Client Control** | Low | High |
| **Freelancer Security** | Low | High |

---

## 10. Future Enhancements

- [ ] Milestone revision tracking
- [ ] Time-based milestone extension
- [ ] Partial milestone approval (e.g., 80% approved)
- [ ] Automated payment reminders
- [ ] Milestone deadline alerts
- [ ] Dispute resolution process
- [ ] Milestone attachment versioning
- [ ] Progress tracking (% complete) per milestone
