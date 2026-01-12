# Chat, Negotiation & Hire Module Documentation

## Overview

This module handles real-time negotiation, structured counter-offers, agreement generation, and the hire flow. It ensures legal clarity and prevents disputes by separating human conversation (chat) from contract data (proposal & agreement).

---

## 1️⃣ Chat System Architecture (Socket.io)

### Core Concept

**Context-based chat**: Every chat belongs to a specific proposal/job, not a global messaging system.

```
Chat Room = Specific to Job + Proposal
```

---

### 1.1 Database Models

#### ChatRoom Model

```javascript
const chatRoomSchema = new Schema({
  _id: ObjectId,
  
  jobId: {
    type: Schema.Types.ObjectId,
    ref: 'Job',
    required: true
  },
  
  proposalId: {
    type: Schema.Types.ObjectId,
    ref: 'Proposal',
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
  
  status: {
    type: String,
    enum: ['ACTIVE', 'ARCHIVED', 'CLOSED'],
    default: 'ACTIVE'
  },
  
  lastMessageAt: Date,
  
  createdAt: {
    type: Date,
    default: Date.now
  }
});
```

#### Message Model

```javascript
const messageSchema = new Schema({
  _id: ObjectId,
  
  chatRoomId: {
    type: Schema.Types.ObjectId,
    ref: 'ChatRoom',
    required: true,
    index: true
  },
  
  senderId: {
    type: Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  
  type: {
    type: String,
    enum: ['TEXT', 'FILE', 'SYSTEM', 'OFFER'],
    default: 'TEXT'
  },
  
  content: {
    type: String,
    required: true
  },
  
  // For file messages
  fileUrl: String,
  fileName: String,
  fileSize: Number,
  fileType: String,
  
  // For offer messages (system tracking)
  offerData: {
    offerId: Schema.Types.ObjectId,
    type: String, // 'COUNTER_OFFER' | 'OFFER_ACCEPTED' | 'OFFER_REJECTED'
    amount: Number,
    days: Number,
    status: String
  },
  
  isEdited: {
    type: Boolean,
    default: false
  },
  
  editedAt: Date,
  
  isDeleted: {
    type: Boolean,
    default: false
  },
  
  deletedAt: Date,
  
  reactions: [{
    userId: Schema.Types.ObjectId,
    emoji: String
  }],
  
  createdAt: {
    type: Date,
    default: Date.now,
    index: true
  }
});
```

---

### 1.2 Chat Room Creation Triggers

Chat room is created automatically when:

**Trigger 1: Proposal Submission**
```
POST /api/proposals (freelancer submits proposal)
  → ChatRoom created with:
    - jobId
    - proposalId
    - clientId
    - freelancerId
```

**Trigger 2: Manual Chat Request**
```
POST /api/chat/initiate (client clicks "Chat with freelancer")
  → Check if ChatRoom exists
  → If not, create new ChatRoom
  → If yes, reopen existing
```

---

### 1.3 Real-Time Features with Socket.io

#### Socket.io Room Structure

```javascript
// User joins chat room
socket.join(`chatRoom_${chatRoomId}`)
```

#### Socket Events

**User connects to chat:**
```javascript
socket.on('chat:open', (chatRoomId) => {
  socket.join(`chatRoom_${chatRoomId}`)
  io.to(`chatRoom_${chatRoomId}`).emit('user:online', {
    userId: socket.userId,
    timestamp: new Date()
  })
})
```

**Send message:**
```javascript
socket.on('message:send', (data) => {
  // Validate & sanitize
  // Check keyword blocking
  // Save to DB
  // Broadcast to room
  
  io.to(`chatRoom_${data.chatRoomId}`).emit('message:new', {
    messageId,
    senderId,
    content,
    timestamp
  })
})
```

**Typing indicator:**
```javascript
socket.on('typing:start', (chatRoomId) => {
  io.to(`chatRoom_${chatRoomId}`).emit('user:typing', {
    userId: socket.userId,
    typing: true
  })
})

socket.on('typing:stop', (chatRoomId) => {
  io.to(`chatRoom_${chatRoomId}`).emit('user:typing', {
    userId: socket.userId,
    typing: false
  })
})
```

**User disconnect:**
```javascript
socket.on('disconnect', () => {
  io.to(`chatRoom_${chatRoomId}`).emit('user:offline', {
    userId: socket.userId,
    timestamp: new Date()
  })
})
```

---

### 1.4 Keyword Blocking (Phone & Email)

**Implementation:**

```javascript
// In message:send handler
const MESSAGE_REGEX = {
  PHONE: /(\+\d{1,3}[-.\s]?)?\d{3,4}[-.\s]?\d{3,4}[-.\s]?\d{4}/,
  EMAIL: /[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}/,
  WHATSAPP: /(whatsapp|whatsapp me|wa\.me)/i,
  TELEGRAM: /(telegram|@[a-zA-Z0-9_]+)/i
}

function checkContentSafety(content) {
  let modifiedContent = content
  let hasBlockedContent = false
  
  // Check for email
  if (MESSAGE_REGEX.EMAIL.test(content)) {
    modifiedContent = content.replace(MESSAGE_REGEX.EMAIL, '***')
    hasBlockedContent = true
  }
  
  // Check for phone
  if (MESSAGE_REGEX.PHONE.test(content)) {
    modifiedContent = content.replace(MESSAGE_REGEX.PHONE, '***')
    hasBlockedContent = true
  }
  
  return {
    isBlocked: hasBlockedContent,
    originalContent: content,
    modifiedContent: modifiedContent,
    warning: hasBlockedContent ? 
      'For your safety, do not share contact details outside OpenGigs.' : null
  }
}

// Usage
const safety = checkContentSafety(messageContent)
if (safety.isBlocked) {
  // Send system message warning
  // Save message with masked content
  // Notify sender of blocked content
}
```

---

### 1.5 Chat UI Context Display

At the top of every chat window, display job context:

```javascript
{
  jobTitle: "Build ecommerce website",
  jobBudget: 25000,
  proposalAmount: 24000,
  proposalDelivery: 7,
  jobCategory: "Web Development",
  jobPostedAt: "2026-01-10T10:30:00Z"
}
```

---

## 2️⃣ Negotiation System (Counter-Offers)

### Core Principle

**Chat = Human Discussion**
**Proposal Record = Legal Source of Truth**

> ⚠️ The system NEVER reads final amount/days from chat messages.
> 
> Instead, use structured **Counter-Offer** actions.

---

### 2.1 Counter-Offer Model

```javascript
const counterOfferSchema = new Schema({
  _id: ObjectId,
  
  proposalId: {
    type: Schema.Types.ObjectId,
    ref: 'Proposal',
    required: true
  },
  
  jobId: {
    type: Schema.Types.ObjectId,
    ref: 'Job',
    required: true
  },
  
  senderId: {
    type: Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  
  receiverId: {
    type: Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  
  originalAmount: Number,
  originalDays: Number,
  
  counterAmount: {
    type: Number,
    required: true,
    min: 0
  },
  
  counterDays: {
    type: Number,
    required: true,
    min: 1
  },
  
  message: String,
  
  status: {
    type: String,
    enum: ['PENDING', 'ACCEPTED', 'REJECTED', 'COUNTERED'],
    default: 'PENDING'
  },
  
  respondedAt: Date,
  
  expiresAt: {
    type: Date,
    default: () => new Date(Date.now() + 7 * 24 * 60 * 60 * 1000) // 7 days
  },
  
  createdAt: {
    type: Date,
    default: Date.now
  }
});
```

---

### 2.2 Negotiation Flow

#### Step 1: Initial Proposal

```
Freelancer submits proposal:
{
  proposalAmount: 4500
  proposalDays: 5
  status: "PENDING"
}
```

#### Step 2: Client Negotiates

**Chat:**
```
Client: "Can you do ₹4200 in 6 days?"
Freelancer: "Yes, that works for me."
```

**Action:** Client clicks "Send Counter Offer"

```javascript
POST /api/proposals/:proposalId/counter-offer
{
  counterAmount: 4200,
  counterDays: 6,
  message: "Can you do this amount in 6 days?" // Optional
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "counterOfferId": "507f1f77bcf86cd799439050",
    "proposalId": "507f1f77bcf86cd799439011",
    "senderId": "507f1f77bcf86cd799439010", // Client
    "receiverId": "507f1f77bcf86cd799439012", // Freelancer
    "originalAmount": 4500,
    "originalDays": 5,
    "counterAmount": 4200,
    "counterDays": 6,
    "status": "PENDING",
    "expiresAt": "2026-01-18T10:30:00Z",
    "message": "Can you do this amount in 6 days?"
  }
}
```

**System Action:**
- Create CounterOffer record
- Broadcast to freelancer in real-time
- Send notification to freelancer

#### Step 3: Freelancer Responds

**Option A: Freelancer Accepts**

```javascript
POST /api/counter-offers/:offerId/accept
```

**Response:**
```json
{
  "success": true,
  "data": {
    "counterOfferId": "507f1f77bcf86cd799439050",
    "status": "ACCEPTED",
    "proposalUpdated": {
      "proposalId": "507f1f77bcf86cd799439011",
      "finalAmount": 4200,
      "finalDays": 6,
      "status": "AGREED"
    },
    "nextStep": "Agreement will be generated",
    "message": "Offer accepted! Agreement document is being generated."
  }
}
```

**System Actions:**
```javascript
// Update Proposal
proposal.finalAmount = 4200
proposal.finalDays = 6
proposal.status = "AGREED"

// Create Agreement (auto-generated)
// Send both parties agreement for review
```

---

**Option B: Freelancer Counter-Offers**

```javascript
POST /api/counter-offers/:offerId/counter
{
  counterAmount: 4300,
  counterDays: 5,
  message: "I can do 4300 but need 5 days"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "newCounterOfferId": "507f1f77bcf86cd799439051",
    "previousCounterId": "507f1f77bcf86cd799439050",
    "status": "COUNTERED",
    "message": "Counter-offer created. Client will review."
  }
}
```

---

**Option C: Freelancer Rejects**

```javascript
POST /api/counter-offers/:offerId/reject
{
  reason: "I cannot go below 4400"
}
```

---

### 2.3 Counter-Offer History

**Endpoint:** `GET /api/proposals/:proposalId/counter-offers`

**Response:**
```json
{
  "success": true,
  "data": {
    "proposalId": "507f1f77bcf86cd799439011",
    "negotiationHistory": [
      {
        "round": 1,
        "initiator": "CLIENT",
        "amount": 4500,
        "days": 5,
        "status": "INITIAL",
        "createdAt": "2026-01-10T10:30:00Z"
      },
      {
        "round": 2,
        "initiator": "CLIENT",
        "amount": 4200,
        "days": 6,
        "status": "PENDING",
        "createdAt": "2026-01-10T11:00:00Z"
      },
      {
        "round": 3,
        "initiator": "FREELANCER",
        "amount": 4300,
        "days": 5,
        "status": "ACCEPTED",
        "createdAt": "2026-01-10T11:30:00Z"
      }
    ],
    "finalAgreedAmount": 4300,
    "finalAgreedDays": 5,
    "proposalStatus": "AGREED"
  }
}
```

---

## 3️⃣ How System Determines Final Amount & Days

### The Correct Design

```
✅ Chat messages = Discussion only
✅ CounterOffer object = Source of truth for final terms
✅ Proposal object = Stores finalAmount & finalDays
```

### Data Flow

```
1. Initial Proposal
   ├─ proposalAmount: 4500
   └─ proposalDays: 5

2. Chat: "Can you do 4200 in 6 days?"

3. Client sends CounterOffer
   ├─ counterAmount: 4200
   └─ counterDays: 6

4. Freelancer accepts CounterOffer

5. System updates Proposal
   ├─ finalAmount: 4200
   ├─ finalDays: 6
   └─ status: "AGREED"

6. Agreement generated with final values
```

### Why This Works

✅ **Legal clarity**: Terms are structured, not inferred from chat  
✅ **Dispute prevention**: All changes tracked with timestamps  
✅ **Audit trail**: Full negotiation history preserved  
✅ **Database consistency**: Single source of truth per field  
✅ **Easy enforcement**: Can directly reference in contract  

---

## 4️⃣ Agreement Generation & Signing

### 4.1 Agreement Model

```javascript
const agreementSchema = new Schema({
  _id: ObjectId,
  
  jobId: {
    type: Schema.Types.ObjectId,
    ref: 'Job',
    required: true
  },
  
  proposalId: {
    type: Schema.Types.ObjectId,
    ref: 'Proposal',
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
  
  // Terms
  finalAmount: {
    type: Number,
    required: true
  },
  
  finalDays: {
    type: Number,
    required: true
  },
  
  paymentType: {
    type: String,
    enum: ['FULL', 'MILESTONE'],
    required: true
  },
  
  // Agreement content
  jobTitle: String,
  jobDescription: String,
  jobCategory: String,
  skillsRequired: [String],
  
  // Scope
  scope: String, // Detailed scope from job description
  
  // Additional terms
  escrowTerms: String,
  cancellationPolicy: String,
  disputeResolution: String,
  
  // Signatures
  clientSigned: {
    type: Boolean,
    default: false
  },
  
  clientSignedAt: Date,
  clientSignatureData: String, // e-signature or digital proof
  
  freelancerSigned: {
    type: Boolean,
    default: false
  },
  
  freelancerSignedAt: Date,
  freelancerSignatureData: String,
  
  status: {
    type: String,
    enum: [
      'PENDING_SIGNATURES',
      'FREELANCER_SIGNED',
      'FULLY_SIGNED',
      'ARCHIVED'
    ],
    default: 'PENDING_SIGNATURES'
  },
  
  // Documents
  agreementPdfUrl: String,
  agreementJsonUrl: String,
  
  createdAt: {
    type: Date,
    default: Date.now
  }
});
```

---

### 4.2 When Agreement is Generated

**Trigger:** When `proposal.status = "AGREED"`

```javascript
// In counterOffer.accept()
async function acceptCounterOffer(offerId) {
  // Update proposal
  proposal.finalAmount = counterOffer.counterAmount
  proposal.finalDays = counterOffer.counterDays
  proposal.status = "AGREED"
  
  // Auto-generate agreement
  const agreement = await generateAgreement({
    jobId: proposal.jobId,
    proposalId: proposal._id,
    clientId: proposal.clientId,
    freelancerId: proposal.freelancerId,
    finalAmount: proposal.finalAmount,
    finalDays: proposal.finalDays,
    paymentType: job.paymentType
  })
  
  // Notify both parties
  notifyUser(proposal.clientId, 'Agreement ready for signing')
  notifyUser(proposal.freelancerId, 'Agreement ready for signing')
}
```

---

### 4.3 Agreement Content Generation

**HTML Template:**

```html
<div class="agreement">
  <h1>Service Agreement</h1>
  
  <section>
    <h2>Parties</h2>
    <p>This agreement is between:</p>
    <ul>
      <li>Client: {{ client.name }}</li>
      <li>Freelancer: {{ freelancer.name }}</li>
    </ul>
  </section>
  
  <section>
    <h2>Project Details</h2>
    <table>
      <tr>
        <td>Job Title</td>
        <td>{{ job.title }}</td>
      </tr>
      <tr>
        <td>Category</td>
        <td>{{ job.category }}</td>
      </tr>
      <tr>
        <td>Required Skills</td>
        <td>{{ job.skillsRequired.join(', ') }}</td>
      </tr>
    </table>
  </section>
  
  <section>
    <h2>Scope of Work</h2>
    <p>{{ job.description }}</p>
  </section>
  
  <section>
    <h2>Financial Terms</h2>
    <table>
      <tr>
        <td>Project Fee</td>
        <td>₹{{ agreement.finalAmount }}</td>
      </tr>
      <tr>
        <td>Delivery Timeline</td>
        <td>{{ agreement.finalDays }} days from project start</td>
      </tr>
      <tr>
        <td>Payment Type</td>
        <td>{{ agreement.paymentType === 'FULL' ? 'Full Payment' : 'Milestone-based' }}</td>
      </tr>
      <tr>
        <td>Currency</td>
        <td>INR</td>
      </tr>
    </table>
  </section>
  
  <section>
    <h2>Payment & Escrow Terms</h2>
    <p>
      25% of the project fee will be held in escrow by OpenGigs to ensure 
      security for both parties. Upon successful completion and approval,
      the remaining balance will be released to the freelancer.
    </p>
  </section>
  
  <section>
    <h2>Deliverables & Acceptance</h2>
    <p>
      The freelancer agrees to deliver work that meets the scope outlined above.
      The client has 5 business days to review and approve the deliverables.
    </p>
  </section>
  
  <section>
    <h2>Cancellation & Refund Policy</h2>
    <p>
      If work has not been started, full refund of escrow amount.
      If work is in progress, refund will be determined based on completed work.
    </p>
  </section>
  
  <section>
    <h2>Dispute Resolution</h2>
    <p>
      Any disputes will be handled through OpenGigs' mediation process.
      Both parties agree to cooperate in good faith.
    </p>
  </section>
  
  <section>
    <h2>Signatures</h2>
    <table>
      <tr>
        <td>Freelancer Signature</td>
        <td>___________________</td>
        <td>Date: ___________</td>
      </tr>
      <tr>
        <td>Client Signature</td>
        <td>___________________</td>
        <td>Date: ___________</td>
      </tr>
    </table>
  </section>
  
  <footer>
    <p>Generated on {{ createdAt }} by OpenGigs</p>
    <p>Agreement ID: {{ agreement._id }}</p>
  </footer>
</div>
```

---

### 4.4 Signing Flow

#### Step 1: View Agreement

**Endpoint:** `GET /api/agreements/:agreementId`

**Response:**
```json
{
  "success": true,
  "data": {
    "agreementId": "507f1f77bcf86cd799439060",
    "clientName": "Raj Kumar",
    "freelancerName": "Priya Sharma",
    "jobTitle": "Website Development",
    "finalAmount": 4300,
    "finalDays": 5,
    "paymentType": "FULL",
    "status": "PENDING_SIGNATURES",
    "agreementHtml": "...",
    "clientSigned": false,
    "freelancerSigned": false,
    "createdAt": "2026-01-10T12:00:00Z"
  }
}
```

---

#### Step 2: Freelancer Signs

**Endpoint:** `POST /api/agreements/:agreementId/sign-freelancer`

**Request:**
```json
{
  "signatureMethod": "DIGITAL", // DIGITAL | E_SIGNATURE
  "agreementAcceptance": true
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "agreementId": "507f1f77bcf86cd799439060",
    "freelancerSigned": true,
    "freelancerSignedAt": "2026-01-10T12:30:00Z",
    "status": "FREELANCER_SIGNED",
    "message": "Agreement signed! Waiting for client to sign.",
    "clientSignatureLink": "Sent to client"
  }
}
```

**System Actions:**
- Update agreement: `freelancerSigned = true`
- Send notification to client: "Freelancer has signed the agreement"
- Display signature button to client

---

#### Step 3: Client Signs & Hires

**Endpoint:** `POST /api/agreements/:agreementId/sign-client`

**Request:**
```json
{
  "signatureMethod": "DIGITAL",
  "agreementAcceptance": true,
  "proceedWithHire": true // Important: Actually hire after signing
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "agreementId": "507f1f77bcf86cd799439060",
    "clientSigned": true,
    "clientSignedAt": "2026-01-10T13:00:00Z",
    "status": "FULLY_SIGNED",
    "message": "Agreement fully signed! Freelancer hired successfully.",
    "projectCreated": {
      "projectId": "507f1f77bcf86cd799439070",
      "status": "ACTIVE"
    }
  }
}
```

---

## 5️⃣ Hire Freelancer Flow (Final Step)

### 5.1 Complete Flow

```
Proposal submitted
     ↓
Chat opens & negotiation begins
     ↓
Chat: Client & Freelancer discuss terms
     ↓
Client sends Counter Offer
     ↓
Freelancer accepts/counters/rejects
     ↓
Once agreed: proposal.status = "AGREED"
     ↓
System auto-generates Agreement document
     ↓
Both parties review agreement
     ↓
Freelancer clicks "Sign Agreement"
     ↓
Client clicks "Sign & Hire Freelancer"
     ↓
System checks: freelancerSigned = true
     ↓
System creates Project
     ↓
Escrow payment locked
     ↓
Job status → "IN_PROGRESS"
     ↓
Proposal status → "ACCEPTED"
     ↓
Other proposals → "REJECTED"
     ↓
Chat moved to Project Workspace
```

---

### 5.2 Hire Endpoint

**Endpoint:** `POST /api/agreements/:agreementId/hire`

**Prerequisites:**
- `freelancerSigned = true`
- `clientSigned = true`
- Job has available budget
- Freelancer not already hired for this job

**Request:**
```json
{
  "paymentMethodId": "pm_507f1f77bcf86cd",
  "confirmEscrowDeduction": true
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "agreementId": "507f1f77bcf86cd799439060",
    "projectId": "507f1f77bcf86cd799439070",
    "jobId": "507f1f77bcf86cd799439011",
    "proposalId": "507f1f77bcf86cd799439012",
    "freelancerId": "507f1f77bcf86cd799439013",
    "hireConfirmation": {
      "status": "SUCCESS",
      "hiredAt": "2026-01-10T13:05:00Z",
      "escrowDeducted": 1075, // 25% of 4300
      "remainingBalance": 3225
    },
    "projectDetails": {
      "projectId": "507f1f77bcf86cd799439070",
      "projectStatus": "ACTIVE",
      "deliveryDeadline": "2026-01-15T23:59:59Z",
      "totalBudget": 4300
    },
    "jobUpdated": {
      "jobId": "507f1f77bcf86cd799439011",
      "jobStatus": "IN_PROGRESS",
      "message": "Job is now in progress"
    },
    "proposalUpdated": {
      "proposalId": "507f1f77bcf86cd799439012",
      "proposalStatus": "ACCEPTED",
      "message": "You're hired!"
    },
    "otherProposalsRejected": {
      "count": 5,
      "message": "5 other proposals have been rejected"
    },
    "nextSteps": [
      "Access project workspace",
      "Download project files",
      "Begin work on deliverables",
      "Submit work before deadline"
    ]
  }
}
```

---

### 5.3 Backend Logic for Hire

```javascript
async function hireFreelancer(agreementId, paymentData) {
  // 1. Validate signatures
  const agreement = await Agreement.findById(agreementId)
  if (!agreement.freelancerSigned || !agreement.clientSigned) {
    throw new Error('Both parties must sign agreement first')
  }
  
  // 2. Lock escrow
  const escrowAmount = agreement.finalAmount * 0.25
  const paymentResult = await lockEscrow({
    jobId: agreement.jobId,
    amount: escrowAmount,
    paymentMethodId: paymentData.paymentMethodId
  })
  
  if (!paymentResult.success) {
    throw new Error('Escrow payment failed')
  }
  
  // 3. Create Project
  const project = new Project({
    jobId: agreement.jobId,
    proposalId: agreement.proposalId,
    clientId: agreement.clientId,
    freelancerId: agreement.freelancerId,
    paymentType: agreement.paymentType,
    totalAmount: agreement.finalAmount,
    deliveryDays: agreement.finalDays,
    status: 'ACTIVE'
  })
  await project.save()
  
  // 4. Update Job
  const job = await Job.findByIdAndUpdate(agreement.jobId, {
    status: 'IN_PROGRESS',
    hiredFreelancerId: agreement.freelancerId,
    hiredAt: new Date()
  })
  
  // 5. Update Proposal
  const proposal = await Proposal.findByIdAndUpdate(
    agreement.proposalId,
    { status: 'ACCEPTED' }
  )
  
  // 6. Reject other proposals for this job
  await Proposal.updateMany(
    { jobId: agreement.jobId, _id: { $ne: agreement.proposalId } },
    { status: 'REJECTED' }
  )
  
  // 7. Update ChatRoom
  const chatRoom = await ChatRoom.findOneAndUpdate(
    { proposalId: agreement.proposalId },
    { projectId: project._id }
  )
  
  // 8. Create milestone (if MILESTONE payment)
  if (agreement.paymentType === 'MILESTONE') {
    // Milestones created separately by client
  } else {
    // Auto-create single virtual milestone for FULL payment
    const milestone = new Milestone({
      projectId: project._id,
      title: job.title,
      description: 'Full project delivery',
      amount: agreement.finalAmount,
      order: 1,
      isFunded: true,
      fundedAt: new Date()
    })
    await milestone.save()
  }
  
  return {
    projectId: project._id,
    escrowLocked: true,
    hireConfirmed: true
  }
}
```

---

## 6️⃣ Complete Data Flow Diagram

```
┌─────────────────────────┐
│   JOB POSTED            │
│   Budget: 4300          │
│   paymentType: FULL     │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│  PROPOSAL SUBMITTED     │
│  Amount: 4500           │
│  Days: 5                │
│  Status: PENDING        │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│  CHATROOM CREATED       │
│  jobId, proposalId      │
│  Real-time messaging    │
└────────────┬────────────┘
             │
             ▼
    ┌────────────────┐
    │ NEGOTIATION    │
    │  in Chat       │
    └────────┬───────┘
             │
             ▼
┌─────────────────────────┐
│ COUNTER OFFER SENT      │
│ Amount: 4200            │
│ Days: 6                 │
│ Status: PENDING         │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│ COUNTER OFFER ACCEPTED  │
│ Status: ACCEPTED        │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│ PROPOSAL UPDATED        │
│ finalAmount: 4200       │
│ finalDays: 6            │
│ Status: AGREED          │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│ AGREEMENT GENERATED     │
│ Status: PENDING_SIG     │
│ Terms locked in         │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│ FREELANCER SIGNS        │
│ Status: FREELANCER_SIGNED
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│ CLIENT SIGNS & HIRES    │
│ Status: FULLY_SIGNED    │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│ PROJECT CREATED         │
│ Escrow: ₹1050 locked    │
│ Status: ACTIVE          │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│ JOB IN_PROGRESS         │
│ Chat→Workspace          │
│ Freelancer starts work  │
└─────────────────────────┘
```

---

## 7️⃣ API Endpoints Summary

### Chat Endpoints

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/api/chat/initiate` | Open chat with freelancer |
| GET | `/api/chat/:chatRoomId` | Get chat messages |
| POST | `/api/chat/:chatRoomId/message` | Send message |
| POST | `/api/chat/:chatRoomId/file` | Send file |
| PUT | `/api/chat/message/:messageId` | Edit message |
| DELETE | `/api/chat/message/:messageId` | Delete message |
| GET | `/api/chat/rooms` | Get all chat rooms |

### Counter-Offer Endpoints

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/api/proposals/:proposalId/counter-offer` | Send counter offer |
| GET | `/api/proposals/:proposalId/counter-offers` | Get negotiation history |
| POST | `/api/counter-offers/:offerId/accept` | Accept offer |
| POST | `/api/counter-offers/:offerId/counter` | Send counter offer |
| POST | `/api/counter-offers/:offerId/reject` | Reject offer |

### Agreement Endpoints

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/api/agreements/:agreementId` | View agreement |
| POST | `/api/agreements/:agreementId/sign-freelancer` | Freelancer signs |
| POST | `/api/agreements/:agreementId/sign-client` | Client signs |
| POST | `/api/agreements/:agreementId/hire` | Hire freelancer |
| GET | `/api/agreements` | Get all agreements |

---

## 8️⃣ Socket.io Events Summary

### Client → Server Events

```javascript
// Chat
socket.emit('chat:open', { chatRoomId })
socket.emit('message:send', { chatRoomId, content })
socket.emit('typing:start', { chatRoomId })
socket.emit('typing:stop', { chatRoomId })

// User presence
socket.emit('user:online', { chatRoomId })
socket.emit('user:offline', { chatRoomId })
```

### Server → Client Events

```javascript
// Messages
socket.on('message:new', (data) => {})
socket.on('message:edited', (data) => {})
socket.on('message:deleted', (data) => {})

// User presence
socket.on('user:online', (data) => {})
socket.on('user:offline', (data) => {})
socket.on('user:typing', (data) => {})

// Counter offers
socket.on('offer:received', (data) => {})
socket.on('offer:accepted', (data) => {})
socket.on('offer:rejected', (data) => {})

// Agreement
socket.on('agreement:generated', (data) => {})
socket.on('agreement:signed', (data) => {})
socket.on('hire:confirmed', (data) => {})
```

---

## 9️⃣ Key Principles

### Design Principles

✅ **Separation of Concerns**
- Chat for discussion
- Proposal for data
- Agreement for legal terms
- Project for execution

✅ **Legal Clarity**
- All contract terms in Agreement
- Structured counter-offers (not chat text)
- Both parties sign digitally
- Audit trail complete

✅ **Prevents Disputes**
- No ambiguity on final terms
- Written agreement signed
- Full negotiation history
- Timestamps on everything

✅ **Platform Security**
- No contact sharing allowed
- Keyword blocking enforced
- Payment locked in escrow
- All actions logged

✅ **Real-Time Engagement**
- Live typing indicators
- Instant notifications
- Real-time presence
- Smooth UX

---

## 🔟 Error Handling

### Common Scenarios

**Both Not Signed:**
```json
{
  "success": false,
  "error": "Both parties must sign before hiring",
  "code": "SIGNATURES_INCOMPLETE",
  "statusCode": 400,
  "details": {
    "freelancerSigned": false,
    "clientSigned": false
  }
}
```

**Escrow Payment Failed:**
```json
{
  "success": false,
  "error": "Escrow payment failed",
  "code": "ESCROW_PAYMENT_FAILED",
  "statusCode": 402,
  "details": {
    "reason": "Insufficient funds in account"
  }
}
```

**Counter-Offer Expired:**
```json
{
  "success": false,
  "error": "Counter-offer has expired",
  "code": "OFFER_EXPIRED",
  "statusCode": 400,
  "expiresAt": "2026-01-18T10:30:00Z"
}
```

**Freelancer Already Hired:**
```json
{
  "success": false,
  "error": "A freelancer is already hired for this job",
  "code": "FREELANCER_ALREADY_HIRED",
  "statusCode": 400
}
```

---

## 1️⃣1️⃣ Implementation Checklist

### Database Models
- [ ] Update ChatRoom model
- [ ] Create Message model with keyword blocking
- [ ] Create CounterOffer model
- [ ] Create Agreement model
- [ ] Update Proposal model with finalAmount/finalDays

### Controllers
- [ ] Create `chatController.js`
- [ ] Create `counterOfferController.js`
- [ ] Create `agreementController.js`
- [ ] Update `proposalController.js`

### Routes
- [ ] Create `chatRoutes.js`
- [ ] Create `counterOfferRoutes.js`
- [ ] Create `agreementRoutes.js`

### Socket.io
- [ ] Set up Socket.io server
- [ ] Implement chat rooms
- [ ] Implement typing indicators
- [ ] Implement user presence
- [ ] Implement real-time offer notifications

### Business Logic
- [ ] Keyword blocking function
- [ ] Counter-offer acceptance flow
- [ ] Agreement generation engine
- [ ] Digital signature integration
- [ ] Hire confirmation process

### Frontend
- [ ] Chat UI with context
- [ ] Counter-offer UI
- [ ] Agreement review page
- [ ] Signature collection
- [ ] Real-time notifications

### Security
- [ ] Rate limiting on messages
- [ ] Prevent unauthorized chat access
- [ ] Validate signature authenticity
- [ ] Log all critical actions
- [ ] Prevent double-hiring

---

## Summary Table

| Component | Purpose | Real-time |
|-----------|---------|-----------|
| **Chat** | Human discussion | ✅ Yes |
| **CounterOffer** | Structured negotiation | ⏱️ Polling |
| **Agreement** | Legal contract | ⏱️ Polling |
| **Project** | Work execution | ✅ Yes |
| **Escrow** | Payment security | ✅ Yes |

---

## Next Steps

1. Implement database models
2. Build chat backend with Socket.io
3. Create counter-offer API
4. Generate agreement PDFs
5. Integrate digital signatures
6. Build frontend UI
7. Test complete flow end-to-end
8. Deploy and monitor
