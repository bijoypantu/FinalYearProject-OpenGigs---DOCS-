# Project Payment Module Documentation

## Overview

The Project Payment Module handles payment processing with flexible payment type selection. 

**Payment type decision happens after hiring**, during the negotiation and agreement phase, where **client and freelancer decide together** whether to proceed with:
- **No Milestone (Fixed Price/Hourly Rate)**: Simple payment after work completion
- **Milestone-based Payment**: Staged payments as client approves each milestone

---

## 0. Payment Type Decision Process (After Hiring)

### When Does This Happen?
- **Timing**: After freelancer proposal is accepted and agreement is signed, before work begins
- **Duration**: Client and freelancer have time to discuss and agree on payment structure

### How They Decide Together

**Step 1: Client Initiates Discussion**
- After hiring freelancer and agreement is signed
- Client opens a "Payment & Project Structure" dialog in the project/job window
- Displays both payment options with details

**Step 2: Present Both Options**

```
Option A: Full Payment (Recommended for projects ≤ ₹10,000)
├─ Simple and straightforward
├─ Freelancer completes entire work
├─ Client reviews and pays in one go
└─ Best for well-defined, smaller projects

Option B: Milestone-based (Recommended for projects > ₹10,000)
├─ Break work into multiple deliverables
├─ Payment released as each milestone is approved
├─ Better control and quality assurance
└─ Recommended for complex/high-value projects
```

**Step 3: Client & Freelancer Communicate**
- Both discuss which option suits the project best
- Can chat in the negotiation/project room to discuss:
  - Project complexity
  - Delivery timeline
  - Comfort level with payment structure
  - Any concerns or preferences

**Step 4: Client Makes Final Selection**
- Client chooses one option in the job window
- System confirms: "You've selected [Option]. Work can now begin."
- Selection is locked (cannot change mid-project)

**Step 5: Proceed with Chosen Payment Flow**

**If Option A (Full Payment):**
→ Freelancer begins work immediately
→ Upon completion: Upfront payment notification (if amount > ₹3,000)
→ Client reviews and approves
→ Payment processed

**If Option B (Milestone-based):**
→ Client and freelancer together define milestones
→ Milestones stored and locked
→ Freelancer begins work on first milestone
→ Sequential approval and payment process

---

## 1. Payment Scenarios (After Decision)

### Scenario A: No Milestone Payment (Full Payment Option)

**Use Case:** Complete projects without intermediate deliverables

#### Amount ≤ ₹3,000
- Freelancer begins work
- Client reviews completed work
- Client pays **full amount**
- Project files handed over to client
- Job marked as completed
- Freelancer receives payment

**No escrow deposit required**

#### Amount > ₹3,000
- Freelancer begins work
- Upon completion, client reviews work
- **System shows notification** (in job window): "Consider paying 25% upfront to maintain trust and flexibility between parties. You'll be refunded if work isn't done properly." (Optional - can be dismissed)
- **If client decides to pay upfront:**
  - Client pays **25% upfront** (held in escrow)
  - Freelancer completes remaining work
  - Upon completion and approval: remaining **75%** is paid
  - Project files handed to client
- **If client declines upfront payment:**
  - Freelancer completes work
  - Upon completion and approval: client pays **full amount**
  - Project files handed to client

**Database Storage:**
```
paymentType: "FULL"
jobAmount: Number
upfrontPaymentSuggested: Boolean (if > 3000)
upfrontPaymentAccepted: Boolean
escrowAmount: 0.25 * jobAmount (if accepted)
```

---

### Scenario B: Milestone-based Payment (Milestone Option)

**Use Case:** Complex projects with multiple deliverables and staged payments

**When is this chosen?**
- Client and freelancer decided together (after hiring) to use milestone-based payment
- This happens during negotiation/agreement phase before work begins

**Total Amount Check:**
- If **total project cost > ₹10,000**:
  - During Option selection: System suggests milestone-based as recommended option
  - Client can still choose Full Payment if preferred
- If **total project cost ≤ ₹10,000**:
  - Either option is available; client and freelancer decide together

**System Behavior** (After milestone option is selected by client and freelancer):

1. **Milestone Definition Phase** (Immediately after payment type is selected)
   - Client and freelancer **together define:**
     - Milestone titles and descriptions
     - Amount for each milestone
     - Expected delivery timeline for each
   - Both parties must agree on milestone structure before work begins
   - Milestones stored and locked in database

2. **Upfront Payment Suggestion** (Optional, shown as notification)
   - If **total project cost > ₹3,000**:
     - System shows notification in job window: "Consider paying 25% of total amount (₹X) upfront to maintain trust and flexibility. You'll be refunded if work isn't done properly." (Optional)
     - If client decides to pay:
       - Client pays **25% of total amount** (held in escrow)
       - Work begins
     - If client declines or dismisses:
       - Work begins without upfront payment

3. **Milestone Execution Phase**
   - Freelancer begins work on first milestone
   - Upon completion:
     - Freelancer submits work with details
     - Client reviews the milestone work
     - Client **approves or rejects** the work
   
4. **Payment Release**
   - **If approved:**
     - Client pays milestone amount
     - Payment released to freelancer
     - Milestone marked as **COMPLETED**
     - Work moves to **next milestone**
     - Process repeats for all remaining milestones
   
   - **If rejected:**
     - Milestone marked as **REJECTED**
     - Freelancer notified with rejection reason
     - Freelancer can resubmit revised work
     - Upon resubmission: client reviews again
     - Payment released only after approval

5. **Final Milestone**
   - When last milestone is approved and paid:
     - All project files handed to client
     - Project marked as **COMPLETED**
     - If upfront 25% escrow was paid: it's counted toward final balance

**Example Milestone Structure:**
| # | Title | Description | Amount | Status |
|---|-------|-------------|--------|--------|
| 1 | UI Design | Homepage & dashboard mockups | ₹5,000 | PENDING |
| 2 | Backend API | REST API implementation | ₹8,000 | PENDING |
| 3 | Frontend Dev | React component implementation | ₹5,000 | PENDING |
| 4 | Testing & QA | Full testing & bug fixes | ₹2,000 | PENDING |
| | **TOTAL** | | **₹20,000** | |

**Database Storage:**
```
paymentType: "MILESTONE"
jobAmount: Number
totalAmount: Number (sum of all milestones)
upfrontPaymentSuggested: Boolean (if totalAmount > 3000)
upfrontPaymentAccepted: Boolean
escrowAmount: 0.25 * totalAmount (if accepted)
milestones: [MilestoneSchema]
milestoneLocked: Boolean (true - cannot change after work begins)
```

---

## 2. Database Schema Changes

### Job Model Updates

```javascript
// In Job model
const jobSchema = new Schema({
  // ... existing fields
  
  paymentType: {
    type: String,
    enum: ["FIXED_PRICE", "HOURLY_RATE", "MILESTONE"],
    required: true
  },
  
  jobAmount: {
    type: Number,
    required: true,
    min: 0
  },
  
  currency: {
    type: String,
    default: "INR"
  },
  
  // For FIXED_PRICE & HOURLY_RATE with amount > 3000
  upfrontPaymentSuggested: {
    type: Boolean,
    default: false
  },
  
  upfrontPaymentAccepted: {
    type: Boolean,
    default: false
  },
  
  upfrontEscrowAmount: {
    type: Number,
    default: 0 // 25% if upfrontPaymentAccepted is true
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
    enum: ["FIXED_PRICE", "HOURLY_RATE", "MILESTONE"],
    required: true
  },
  
  totalAmount: {
    type: Number,
    required: true
  },
  
  upfrontPaymentAccepted: {
    type: Boolean,
    default: false
  },
  
  upfrontEscrowAmount: {
    type: Number,
    default: 0
  },
  
  amountPaid: {
    type: Number,
    default: 0
  },
  
  status: {
    type: String,
    enum: ["ACTIVE", "COMPLETED", "CANCELLED"],
    default: "ACTIVE"
  },
  
  createdAt: {
    type: Date,
    default: Date.now
  },
  
  completedAt: {
    type: Date
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
  
  expectedDeliveryDate: {
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
  
  submittedAt: {
    type: Date
  },
  
  approvedAt: {
    type: Date
  },
  
  paidAt: {
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
    // null for NO_MILESTONE payment or UPFRONT escrow
  },
  
  amount: {
    type: Number,
    required: true
  },
  
  transactionType: {
    type: String,
    enum: ["UPFRONT_ESCROW", "MILESTONE_PAYMENT", "FULL_PAYMENT", "REFUND"],
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
    // null if payment still in escrow
  },
  
  releasedAt: {
    type: Date
  },
  
  description: String,
  
  createdAt: {
    type: Date,
    default: Date.now
  }
});
```

---

## 3. API Endpoints

### Upfront Payment Suggestion (NO MILESTONE)

**Endpoint:** `GET /api/projects/:projectId/upfront-payment-suggestion`

**Response:**
```json
{
  "success": true,
  "data": {
    "projectId": "507f1f77bcf86cd799439070",
    "paymentType": "FIXED_PRICE",
    "totalAmount": 5000,
    "amountThreshold": 3000,
    "suggestUpfrontPayment": true,
    "upfrontAmount": 1250,
    "message": "Consider paying 25% upfront (₹1,250) to maintain trust and flexibility between parties. You'll be refunded if work isn't done properly.",
    "acceptedByClient": false
  }
}
```

### Accept/Reject Upfront Payment Suggestion

**Endpoint:** `POST /api/projects/:projectId/accept-upfront-payment`

**Request:**
```json
{
  "accept": true,
  "paymentMethodId": "pm_507f1f77bcf86cd"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "projectId": "507f1f77bcf86cd799439070",
    "upfrontPaymentAccepted": true,
    "escrowAmount": 1250,
    "transactionId": "txn_507f1f77bcf86cd",
    "status": "ESCROW_PAID",
    "message": "Upfront payment of ₹1,250 has been held in escrow. Freelancer can now begin work."
  }
}
```

### Define Milestones (MILESTONE PAYMENT)

**Endpoint:** `POST /api/projects/:projectId/milestones/define`

**Request:**
```json
{
  "milestones": [
    {
      "title": "UI Design",
      "description": "Homepage & dashboard mockups",
      "amount": 5000,
      "expectedDeliveryDate": "2026-02-15",
      "order": 1
    },
    {
      "title": "Backend API",
      "description": "REST API implementation",
      "amount": 8000,
      "expectedDeliveryDate": "2026-03-01",
      "order": 2
    },
    {
      "title": "Testing & QA",
      "description": "Full testing and bug fixes",
      "amount": 7000,
      "expectedDeliveryDate": "2026-03-15",
      "order": 3
    }
  ]
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "projectId": "507f1f77bcf86cd799439070",
    "totalAmount": 20000,
    "milestonesCreated": 3,
    "upfrontPaymentSuggested": true,
    "upfrontAmount": 5000,
    "message": "Milestones defined. Total project cost ₹20,000 exceeds ₹3,000. Consider paying ₹5,000 (25%) upfront to maintain trust and flexibility.",
    "milestones": [
      {
        "milestoneId": "507f1f77bcf86cd799439080",
        "title": "UI Design",
        "amount": 5000,
        "order": 1,
        "status": "PENDING"
      },
      // ...
    ]
  }
}
```

### Accept Upfront Payment for Milestone Project

**Endpoint:** `POST /api/projects/:projectId/accept-upfront-payment-milestone`

**Request:**
```json
{
  "accept": true,
  "paymentMethodId": "pm_507f1f77bcf86cd"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "projectId": "507f1f77bcf86cd799439070",
    "upfrontPaymentAccepted": true,
    "escrowAmount": 5000,
    "transactionId": "txn_507f1f77bcf86cd",
    "status": "ESCROW_PAID",
    "message": "Upfront payment of ₹5,000 (25%) has been held in escrow. Freelancer can now begin work."
  }
}
```

### Submit Milestone Work

**Endpoint:** `POST /api/projects/:projectId/milestones/:milestoneId/submit`

**Request:**
```json
{
  "description": "Completed UI design with all requested revisions",
  "attachments": ["https://s3.example.com/design-files.zip"]
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "milestoneId": "507f1f77bcf86cd799439080",
    "projectId": "507f1f77bcf86cd799439070",
    "status": "SUBMITTED",
    "submittedAt": "2026-02-14T10:30:00Z",
    "amount": 5000,
    "message": "Work submitted for approval. Client has 5 days to review and approve."
  }
}
```

### Approve Milestone Work

**Endpoint:** `POST /api/projects/:projectId/milestones/:milestoneId/approve`

**Request:**
```json
{
  "approved": true,
  "feedback": "Excellent work! Exactly as requested."
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "milestoneId": "507f1f77bcf86cd799439080",
    "projectId": "507f1f77bcf86cd799439070",
    "status": "APPROVED",
    "approvedAt": "2026-02-14T15:00:00Z",
    "amount": 5000,
    "message": "Milestone approved. Payment of ₹5,000 being released to freelancer."
  }
}
```

### Release Milestone Payment

**Endpoint:** `POST /api/projects/:projectId/milestones/:milestoneId/release-payment`

**Response:**
```json
{
  "success": true,
  "data": {
    "milestoneId": "507f1f77bcf86cd799439080",
    "projectId": "507f1f77bcf86cd799439070",
    "status": "RELEASED",
    "paidAt": "2026-02-14T15:05:00Z",
    "amount": 5000,
    "transactionId": "txn_507f1f77bcf86cd799439088",
    "freelancerNotified": true,
    "nextMilestoneOrder": 2,
    "message": "Payment released. Freelancer can begin next milestone."
  }
}
```

### Reject Milestone Work

**Endpoint:** `POST /api/projects/:projectId/milestones/:milestoneId/reject`

**Request:**
```json
{
  "approved": false,
  "rejectionReason": "Missing some components from the design. Please revise and resubmit."
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "milestoneId": "507f1f77bcf86cd799439080",
    "projectId": "507f1f77bcf86cd799439070",
    "status": "REJECTED",
    "rejectionReason": "Missing some components from the design. Please revise and resubmit.",
    "message": "Milestone rejected. Freelancer can revise and resubmit."
  }
}
```

### Complete Project (NO MILESTONE)

**Endpoint:** `POST /api/projects/:projectId/complete`

**Request:**
```json
{
  "clientApproval": true,
  "paymentMethodId": "pm_507f1f77bcf86cd"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "projectId": "507f1f77bcf86cd799439070",
    "status": "COMPLETED",
    "totalAmount": 3500,
    "amountPaid": 3500,
    "transactionId": "txn_507f1f77bcf86cd799439088",
    "projectFiles": "https://s3.example.com/project-delivery.zip",
    "completedAt": "2026-02-14T16:00:00Z",
    "message": "Project completed and files delivered."
  }
}
```

---

## 4. UX Guidelines

### For Full Payment Option

**When Amount ≤ ₹3,000:**
- No upfront payment notification shown
- Simple flow: Freelancer completes work → Client reviews → Client pays → Files delivered

**When Amount > ₹3,000:**
- After work begins and is complete
- Show notification card in job window:
  ```
  💡 Optional Trust & Flexibility Payment
  
  Consider paying 25% (₹1,250) upfront to maintain trust between you 
  and the freelancer. You'll be fully refunded if work isn't done properly.
  
  [Accept & Pay Now] [Pay Later] [Dismiss]
  ```
- Notification appears in job window and can be dismissed
- Client can pay anytime they want (before or after completion)
- If accepted: funds held in escrow until work approval

### For Milestone-based Payment Option

**Payment Type Selection (After Hiring):**
- Display dialog showing both options with recommendations:
  ```
  Choose Payment Structure
  
  Option A: Full Payment
  ├─ Best for: Projects ≤ ₹10,000
  ├─ Simple and straightforward
  ├─ Entire payment after completion
  └─ [Select This]
  
  Option B: Milestone-based (Recommended for > ₹10,000)
  ├─ Best for: Complex projects, high budgets
  ├─ Staged payments as work completes
  ├─ Better quality control
  └─ [Select This]
  
  Or discuss with freelancer in chat first →
  ```

**During Milestone Definition:**
- Display form for both client and freelancer to add milestones together
- Show running total and verify it matches project amount
- Display preview of milestone timeline
- Show warning if total > ₹3,000: "You may want to pay 25% upfront for trust & flexibility"

**When Amount > ₹3,000 (During or After Milestone Definition):**
- Show notification in job window:
  ```
  💡 Optional Trust & Flexibility Payment
  
  Consider paying 25% (₹5,000) upfront to maintain trust between you 
  and the freelancer. You'll be fully refunded if work isn't done properly.
  
  [Accept & Pay Now] [Pay As Milestones Progress] [Dismiss]
  ```
- Notification appears and can be dismissed
- Client can pay 25% upfront or pay milestone-by-milestone

**Phase 2: Milestone Execution**
- Show milestone progress bar with current/total milestones
- For each milestone:
  - Display title, amount, expected date
  - Show status badge (PENDING → IN_PROGRESS → SUBMITTED → APPROVED → RELEASED)
  - When submitted: show "Client reviewing..." with countdown (5 days)
  - When approved: show payment release confirmation
- After last milestone: show "Project Complete - All files delivered"

---

## 5. Business Logic & Rules

### Payment Type Selection Process

1. **After Hiring & Agreement**: Client and freelancer meet to decide payment type
   - Discuss project complexity and timeline
   - Evaluate which option suits best
   - Consider team comfort level
   - Make final decision together

2. **For Projects ≤ ₹10,000**: Either option available
   - Both Full Payment and Milestone-based are equally valid
   - Team chooses based on preference

3. **For Projects > ₹10,000**: Milestone-based is recommended
   - Full Payment still available if team prefers
   - System suggests milestone-based but doesn't force

### Upfront Payment Rules

1. **Amount ≤ ₹3,000**: No upfront payment notification shown
2. **Amount > ₹3,000**: Show optional upfront payment notification in job window (after work begins)
   - Appears as notification card (can be dismissed)
   - Client can pay anytime (immediately or later)
   - Funds held in escrow until work approval
3. **Suggestion is optional**: Client can accept, decline, or dismiss
4. **Refund guarantee**: If work isn't completed properly, full upfront amount refunded
5. **Applies to all types**: Both Full Payment and Milestone-based projects

### Milestone Payment Rules

1. **Both parties define milestones together** before work begins
2. **Total milestone amounts must equal** project total amount
3. **Milestones locked**: Cannot be changed after work begins
4. **Milestones sequential**: Each must be approved before next starts
5. **5-day review window**: Client has 5 days to approve/reject after submission
6. **Resubmission allowed**: Freelancer can revise and resubmit rejected milestones
7. **Final milestone**: When last milestone is approved, all project files delivered

### Payment Rules

1. **Escrow holds**: Upfront payments held in escrow until work completion
2. **Payment sequence**: Only released after client approval (for milestones) or completion review (no milestone)
3. **Partial refunds**: If project cancelled, refund of remaining amounts handled per cancellation policy
4. **Transaction records**: All payments logged with timestamps and transaction IDs

---

## 6. Testing & Acceptance Criteria

- ✅ Upfront payment suggestion shows only when amount > ₹3,000
- ✅ Client can accept or decline upfront payment without forcing
- ✅ Milestone structure can be defined by both parties together
- ✅ Total milestone amount matches project total
- ✅ Milestones must be approved before payment released
- ✅ Rejected milestones allow freelancer to resubmit
- ✅ Final project delivery files included after last milestone approval
- ✅ Upfront escrow refunded if work not done properly
- ✅ All transactions recorded with proper status tracking

---

## 7. Implementation Checklist

### Database
- [ ] Update Job model with payment fields
- [ ] Create Project model
- [ ] Create Milestone model
- [ ] Create PaymentTransaction model
- [ ] Add indexes for efficient queries

### Backend
- [ ] Upfront payment suggestion endpoint
- [ ] Accept/reject upfront payment endpoint
- [ ] Milestone definition endpoint
- [ ] Submit milestone endpoint
- [ ] Approve/reject milestone endpoint
- [ ] Release payment endpoint
- [ ] Complete project endpoint

### Frontend
- [ ] Upfront payment suggestion UI (job window)
- [ ] Milestone definition form
- [ ] Milestone progress display
- [ ] Milestone submission form
- [ ] Approve/reject milestone interface
- [ ] Payment status tracking

### Payment Integration
- [ ] Stripe/PayPal integration for upfront escrow
- [ ] Refund handling for upfront payments
- [ ] Payment release workflow

---

*Document created: comprehensive project payment and milestone system for production implementation.*
