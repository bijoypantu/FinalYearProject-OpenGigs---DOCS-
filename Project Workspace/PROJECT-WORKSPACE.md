# Project Workspace Documentation

## Overview

The **Project Workspace** is the unified hub where clients and freelancers collaborate after hiring. It consolidates all project management tools into a single interface with six integrated tabs, eliminating the need to switch between multiple screens.

> 📌 **Note**: The Project Workspace is created immediately after a freelancer is hired and remains the primary interface throughout the entire project lifecycle until completion or archival.

### Key Purpose
- **Single Source of Truth**: All project information, communication, and transactions in one place
- **Seamless Collaboration**: No context switching between different tools
- **Real-time Updates**: System messages and notifications keep both parties informed
- **Integrated Workflows**: Milestones → Approvals → Payments → Chat all connected

---

## 1. Workspace Architecture

### Core Concept

```
Project Workspace = Unified Interface (6 Integrated Tabs)
                   + Shared Project Context
                   + Real-time Synchronization
                   + Role-based Access Control
```

### When is Workspace Created?

**Trigger**: When `Project.status = "ACTIVE"` (immediately after hiring)

```
Job Posted
  ↓
Freelancer Applies
  ↓
Negotiation Chat Opens (proposal-scoped)
  ↓
Agreement Signed & Freelancer Hired
  ↓
✅ PROJECT WORKSPACE CREATED
  ↓
Both parties can access at:
├─ Dashboard → [Open Project] → Workspace
└─ Or direct project link
```

### Workspace Lifecycle

```
ACTIVE (Work in Progress)
  ↓ Milestones submitted, approved, payments released
  ↓
UNDER_REVIEW (Final submission pending approval)
  ↓ Client reviews deliverables
  ↓
COMPLETED (All milestones approved & payments released)
  ↓ Project marked as complete
  ↓
ARCHIVED (Can view but not edit)
```

---

## 2. Workspace Structure

### Tab-Based Interface

```
Project Workspace - Header
┌─────────────────────────────────────────────────────────┐
│ Project: Build Website | Budget: ₹50,000 | Status: Active
│ Deadline: Mar 15, 2026 | Freelancer: John (★★★★★)      │
├─────────────────────────────────────────────────────────┤
│ [Overview] [Milestones] [Files] [Chat] [Activity] [Payments] │
├─────────────────────────────────────────────────────────┤
│ [Tab Content Area]                                      │
│                                                         │
│ (Content changes based on selected tab)                 │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Tab Order & Hierarchy

1. **[Overview]** - Primary workspace entry point
2. **[Milestones]** - Deliverables & approvals
3. **[Files]** - Project file storage
4. **[Chat]** - Real-time communication
5. **[Activity]** - Event timeline
6. **[Payments]** - Transaction & payment tracking

---

## 3. Tab-by-Tab Breakdown

### Tab 1: Overview (Default Landing)

**Purpose**: Single page summary of entire project

**Components**:

```
┌──────────────────────────────────────────┐
│ PROJECT OVERVIEW                         │
├──────────────────────────────────────────┤
│                                          │
│ 1. PROJECT SUMMARY                       │
│    Title: Build Website                  │
│    Created: Jan 5, 2026 | Started: Jan 5 │
│    Budget: ₹50,000 | Spent: ₹15,000     │
│    Deadline: Mar 15, 2026 (59 days left) │
│    Status: Active (60% complete)         │
│                                          │
│ 2. FREELANCER PROFILE                    │
│    Name: John Doe                        │
│    Rating: ★★★★★ (4.8/5)                │
│    Completed: 47 projects | Earned: ₹2.3L │
│    Response Time: < 2 hours              │
│    [View Full Profile]                   │
│                                          │
│ 3. MILESTONE PROGRESS                    │
│    Total: 4 | Completed: 2 | Pending: 2 │
│    ░░░░░░░░░░░░░░░░░░░░░░ 50% Progress  │
│                                          │
│    ✅ UI Design (₹10,000) - Approved Jan 10│
│    ✅ Database Schema (₹10,000) - Approved Jan 15│
│    ⏳ Backend API (₹15,000) - Due Jan 25 │
│    ⏳ Frontend Integration (₹15,000) - Due Feb 5│
│                                          │
│    [View All Milestones] →               │
│                                          │
│ 4. RECENT ACTIVITY                       │
│    Jan 15, 10:30 AM - John uploaded files│
│    Jan 15, 9:15 AM - Database milestone approved│
│    Jan 15, 8:00 AM - John sent message   │
│    Jan 14, 4:30 PM - Payment released    │
│                                          │
│    [View Full Timeline] →                │
│                                          │
│ 5. QUICK ACTIONS                         │
│    [View All Files] [Send Message] [Release Payment] │
│    [Approve Milestone] [End Project]     │
│                                          │
└──────────────────────────────────────────┘
```

**Data Displayed**:
- Project meta (title, budget, deadline, status)
- Freelancer credentials & availability
- Milestone completion percentage
- Last 5 activities
- Quick action buttons

**Permissions**:
- **Client**: View all, edit deadline/budget (if allowed), release payments
- **Freelancer**: View all, submit milestones, upload files

---

### Tab 2: Milestones (Deliverables & Approvals)

**Purpose**: Track, submit, and approve work deliverables

**Components**:

```
┌──────────────────────────────────────────┐
│ MILESTONES                               │
├──────────────────────────────────────────┤
│ Filter: [All] [Pending] [Submitted] [Approved] [Rejected]
│ Sort: [By Due Date] [By Amount] [By Status]│
├──────────────────────────────────────────┤
│                                          │
│ 1️⃣ UI Design                            │
│    Amount: ₹10,000 | Due: Jan 10, 2026  │
│    Status: ✅ APPROVED (Jan 10)          │
│    Description: Design all UI mockups   │
│    [View Submission] [View Details]     │
│                                          │
│ 2️⃣ Database Schema                      │
│    Amount: ₹10,000 | Due: Jan 20, 2026  │
│    Status: ✅ APPROVED (Jan 15)          │
│    Description: Create normalized DB    │
│    [View Submission] [View Details]     │
│                                          │
│ 3️⃣ Backend API                          │
│    Amount: ₹15,000 | Due: Jan 25, 2026  │
│    Status: 🔄 SUBMITTED (Jan 23)        │
│    Description: RESTful API endpoints   │
│                                          │
│    Submitted Files:                      │
│    📄 api-documentation.pdf              │
│    📁 backend-code.zip                   │
│    📄 api-test-results.json              │
│                                          │
│    Freelancer Note: "All endpoints      │
│    documented and tested"                │
│                                          │
│    CLIENT ACTIONS:                       │
│    [✓ Approve] [✗ Request Changes]      │
│                                          │
│ 4️⃣ Frontend Integration                 │
│    Amount: ₹15,000 | Due: Feb 5, 2026   │
│    Status: ⏳ NOT STARTED               │
│    Description: Integrate frontend      │
│    [Submit Milestone]                    │
│                                          │
└──────────────────────────────────────────┘
```

**Milestone States**:

```
1. NOT_STARTED (Freelancer hasn't begun)
2. IN_PROGRESS (Freelancer is working)
3. SUBMITTED (Freelancer submitted for review)
4. APPROVED (Client approved)
5. CHANGES_REQUESTED (Client asked for revisions)
6. RESUBMITTED (Freelancer resubmitted after changes)
7. REJECTED (Client rejected)
8. COMPLETED (Payment released to freelancer)
```

**Data Per Milestone**:
- Title, description, amount, due date
- Submission files & links
- Freelancer notes
- Client feedback
- Approval/rejection reason
- Timeline of state changes

**Actions**:
- **Freelancer**: Submit work, view feedback, resubmit changes
- **Client**: Approve, request changes, reject, release payment

---

### Tab 3: Files (Centralized Storage)

**Purpose**: Store and organize all project files

**Components**:

```
┌──────────────────────────────────────────┐
│ FILES                                    │
├──────────────────────────────────────────┤
│ [Upload File] [Create Folder] [Search]  │
│                                          │
│ Folder View:                             │
│ 📁 Submitted_Work                        │
│    ├─ UI_Design_Mockups (Jan 10)        │
│    │  ├─ homepage.figma                  │
│    │  ├─ dashboard.figma                 │
│    │  └─ mobile-layouts.pdf              │
│    │                                     │
│    ├─ Database_Schema (Jan 15)          │
│    │  ├─ ER_Diagram.png                  │
│    │  └─ schema.sql                      │
│    │                                     │
│    └─ Backend_API (Jan 23)              │
│       ├─ api-docs.pdf                    │
│       ├─ api-code.zip                    │
│       └─ test-results.json               │
│                                          │
│ 📁 Client_Resources                      │
│    ├─ Brand_Guidelines.pdf               │
│    ├─ Content_Brief.docx                 │
│    └─ Reference_Sites.md                 │
│                                          │
│ 📁 Shared_Assets                         │
│    ├─ Fonts/ (5 files)                   │
│    ├─ Icons/ (47 files)                  │
│    └─ Color_Palette.aco                  │
│                                          │
│ Recent Files:                            │
│ 🔄 api-code.zip (1.2 MB) - Jan 23 by John│
│ 🔄 test-results.json (45 KB) - Jan 23    │
│                                          │
└──────────────────────────────────────────┘
```

**Features**:
- Folder organization
- File upload (max 100MB per file, 1GB total)
- Version control (track file changes)
- File preview (images, PDFs, code)
- Download/sharing links
- Deletion (soft delete, recoverable)
- Upload timestamps & uploader info

**Permissions**:
- **Freelancer**: Upload work files, organize folders
- **Client**: Upload resources, download all files
- **Both**: View all files, download, create shared folders

---

### Tab 4: Chat (Real-time Communication)

**Purpose**: Instant messaging with project context

> 📌 **Note**: Detailed in [Project-Chat.md](Project-Chat.md)

**Key Integration Points**:

```
Chat Features Relevant to Workspace:
├─ Real-time messaging (Socket.io)
├─ Message notifications with workspace context
├─ File sharing directly in chat
├─ System messages for milestone updates
│  ├─ "Milestone submitted for review"
│  ├─ "Milestone approved - Payment released"
│  └─ "File uploaded to project"
├─ Typing indicators
├─ Message editing/deletion
├─ Online/offline status
└─ Read receipts

Connection to Other Tabs:
├─ Submit button in [Milestones] → opens chat
├─ File upload notification → links to [Files] tab
├─ Approval → sends system message in chat
└─ Payment release → sends notification + chat message
```

---

### Tab 5: Activity (Event Timeline)

**Purpose**: Chronological log of all project events

**Components**:

```
┌──────────────────────────────────────────┐
│ ACTIVITY                                 │
├──────────────────────────────────────────┤
│ Filter: [All] [Milestones] [Payments] [Files] [Chat]
│ Date Range: [Last 7 days] [This Month] [All Time]
├──────────────────────────────────────────┤
│                                          │
│ 📅 January 23, 2026                      │
│                                          │
│ 10:30 AM - 📤 File Uploaded              │
│ John uploaded to "Backend_API" folder    │
│ Files: api-code.zip (1.2 MB)            │
│ [View Files]                             │
│                                          │
│ 09:15 AM - 🎯 Milestone Submitted       │
│ "Backend API" submitted for review      │
│ 3 files attached, 156 lines of code     │
│ [View Submission]                        │
│                                          │
│ 08:00 AM - 💬 Message Sent               │
│ John: "Completed the API endpoints"     │
│ [View Message]                           │
│                                          │
│ ─────────────────────────────────────────│
│ 📅 January 15, 2026                      │
│                                          │
│ 2:30 PM - ✅ Milestone Approved          │
│ "Database Schema" approved by client    │
│ Amount: ₹10,000                          │
│ [View Details]                           │
│                                          │
│ 1:45 PM - 💳 Payment Released            │
│ ₹10,000 released to John                 │
│ For: "Database Schema" milestone        │
│ [View Payment]                           │
│                                          │
│ 12:00 PM - 📁 Files Organized            │
│ Client created folder "Client_Resources"│
│ [View Folder]                            │
│                                          │
│ 10:00 AM - 🎯 Milestone Approved         │
│ "UI Design" approved by client          │
│ Amount: ₹10,000                          │
│ [View Details]                           │
│                                          │
│ ─────────────────────────────────────────│
│ 📅 January 10, 2026                      │
│                                          │
│ 4:00 PM - 📤 File Uploaded               │
│ John uploaded UI mockups                │
│ Files: 3 Figma files (45 MB)            │
│ [View Files]                             │
│                                          │
│ Load More Activity →                     │
│                                          │
└──────────────────────────────────────────┘
```

**Event Types**:

```
1. MILESTONE_SUBMITTED
   - When freelancer submits milestone
   - Shows files & freelancer notes

2. MILESTONE_APPROVED
   - When client approves milestone
   - Shows feedback & approval date

3. MILESTONE_REJECTED
   - When client rejects milestone
   - Shows rejection reason & feedback

4. PAYMENT_RELEASED
   - When payment is released
   - Shows amount, milestone, & payment ID

5. FILE_UPLOADED
   - When files are uploaded
   - Shows file details & uploader

6. MESSAGE_SENT
   - Recent chat messages
   - Shows message preview

7. FOLDER_CREATED
   - New folder created
   - Shows folder name & creator

8. PROJECT_STATUS_CHANGED
   - Project status changed
   - Shows old → new status

9. DEADLINE_EXTENDED
   - Deadline changed
   - Shows old & new deadline
```

**Data Per Event**:
- Timestamp (exact time)
- Event type & icon
- Actor (who did it)
- Description
- Related entities (links to tabs)
- Metadata (amounts, files, etc.)

---

### Tab 6: Payments (Transaction Tracking)

**Purpose**: Monitor project finances and payment releases

> 📌 **Note**: Detailed in [6.PROJECT-PAYMENT.md](6.PROJECT-PAYMENT.md)

**Components**:

```
┌──────────────────────────────────────────┐
│ PAYMENTS                                 │
├──────────────────────────────────────────┤
│ 📊 SUMMARY                               │
│ Total Budget: ₹50,000                    │
│ Released: ₹20,000 (40%)                  │
│ Pending: ₹30,000 (60%)                   │
│ Progress: ░░░░░░░░░░░░░░░░░░░░░░ 40%   │
├──────────────────────────────────────────┤
│                                          │
│ Payment Method: [Full Payment] / [Milestone-based] │
│ Current: Milestone-based                 │
│                                          │
│ 💳 MILESTONE PAYMENTS                    │
│                                          │
│ ✅ Milestone 1: UI Design                │
│    Amount: ₹10,000                       │
│    Status: PAID (Jan 10, 2:30 PM)       │
│    Received by: John                     │
│    Payment ID: PAY-001                   │
│    [View Receipt]                        │
│                                          │
│ ✅ Milestone 2: Database Schema          │
│    Amount: ₹10,000                       │
│    Status: PAID (Jan 15, 1:45 PM)       │
│    Received by: John                     │
│    Payment ID: PAY-002                   │
│    [View Receipt]                        │
│                                          │
│ ⏳ Milestone 3: Backend API              │
│    Amount: ₹15,000                       │
│    Status: PENDING APPROVAL              │
│    Approved by client? No (Under review) │
│    When approved → Auto-released         │
│    [Approve & Release]                   │
│                                          │
│ ⏳ Milestone 4: Frontend Integration     │
│    Amount: ₹15,000                       │
│    Status: PENDING SUBMISSION            │
│    Expected by: Feb 5, 2026              │
│    [View Details]                        │
│                                          │
│ 💰 OPTIONAL UPFRONT PAYMENT              │
│ (Shown if budget > ₹3,000)               │
│                                          │
│ Recommended: ₹5,000 (10% of ₹50,000)    │
│ Purpose: Build trust & accelerate work  │
│ [Release Upfront Payment]                │
│                                          │
│ 📈 PAYMENT HISTORY                       │
│ Sort by: [Newest] [Oldest] [Amount]     │
│                                          │
│ Jan 15 - UI Design - ₹10,000 - PAID    │
│ Jan 15 - Database - ₹10,000 - PAID      │
│                                          │
└──────────────────────────────────────────┘
```

**Payment Workflow Integration**:

```
Milestone Submission (Milestones tab)
  ↓
Client Approves (Milestones tab)
  ↓
Approval Notification (Chat & Notifications)
  ↓
Payment Auto-Released (Payments tab)
  ↓
Freelancer Receives Funds (External wallet)
  ↓
Activity Log Updated (Activity tab)
  ↓
Chat System Message (Chat tab)
```

---

## 4. Workspace Header (Persistent)

Visible on all tabs, shows critical project info:

```
┌─────────────────────────────────────────────┐
│ Project: Build Website                      │
│ Budget: ₹50,000 | Status: Active | 60% Done │
│ Deadline: Mar 15, 2026 (59 days left)       │
│ Freelancer: John Doe (★★★★★)                │
│ [View Full Profile]                         │
└─────────────────────────────────────────────┘
```

**Dynamic Elements**:
- Project title (clickable → overview)
- Budget progress bar
- Status badge (Active, Under Review, Completed, Archived)
- Countdown timer to deadline
- Freelancer quick view

**Header Actions**:
- **Client**: [Message], [Release Payment], [End Project], [More Actions ▼]
- **Freelancer**: [Message], [Submit Work], [View Invoices], [More Actions ▼]

---

## 5. User Roles & Permissions

### Client Permissions

```
[Overview]    ✅ Full access
[Milestones]  ✅ View all, approve/reject, request changes
[Files]       ✅ Upload resources, view all, download, delete own
[Chat]        ✅ Send messages, participate
[Activity]    ✅ View all events
[Payments]    ✅ View, release, track payments
```

### Freelancer Permissions

```
[Overview]    ✅ Full access
[Milestones]  ✅ View all, submit work, resubmit after changes
[Files]       ✅ Upload work, view all, download, delete own
[Chat]        ✅ Send messages, participate
[Activity]    ✅ View all events
[Payments]    ✅ View payment status, track releases, download receipts
```

### Admin Permissions (Support)

```
[Overview]    ✅ Full access (for disputes)
[Milestones]  ✅ View all (can't approve, only view)
[Files]       ✅ View all (can't delete)
[Chat]        ✅ View all (read-only)
[Activity]    ✅ View all
[Payments]    ✅ View all (dispute handling)
```

---

## 6. Integrated Workflows

### Workflow 1: Submit & Approve Milestone

```
Freelancer                          System                      Client
    │                                 │                           │
    │─── (1) Click "Submit"           │                           │
    │         in Milestones Tab       │                           │
    │                                 │                           │
    │─── (2) Upload files ──────────→ 📤 Files stored             │
    │                                 │                           │
    │─── (3) Add notes ───────────→  📝 Notes saved               │
    │                                 │                           │
    │─── (4) Click "Submit" ────────→ 🔔 Notification sent        │
    │                                 │                           │
    │                                 │         ← 🔔 Alert: Milestone submitted
    │                                 │                           │
    │                                 │         ← (5) Opens workspace
    │                                 │              & Milestones tab
    │                                 │                           │
    │                                 │         (6) Reviews files
    │                                 │             & notes        │
    │                                 │                           │
    │                                 │         (7) Click
    │                                 │             "Approve" ──→  ✅ Marked approved
    │                                 │                           │
    │  ← 💬 System Message: "Approved by client" ← (8) Chat message
    │                                 │                           │
    │  ← 💳 Payment Released ──────── ← (9) Milestone marked complete
    │                                 │     & Payment auto-released
    │  ← 💰 Funds in wallet           │
    │                                 │                           │
    │  ← 📊 Activity log updated ────→ 📊 Activity log updated
```

### Workflow 2: File Sharing During Chat

```
Client wants to share resources
    ↓
Opens [Files] tab
    ↓
Uploads "Brand Guidelines.pdf"
    ↓
Opens [Chat] tab
    ↓
Mentions file in message: "Check Brand Guidelines in Files tab"
    ↓
Freelancer receives notification
    ↓
Clicks notification → Opens workspace [Files] tab
    ↓
Downloads the file
    ↓
System logs: "File downloaded by Freelancer"
```

### Workflow 3: Payment Decision (After Hiring)

```
Project Created (Workspace opened)
    ↓
Payment system decides: Full or Milestone?
    ↓
[FULL PAYMENT]
- Single payment option
- Upfront payment suggestion (25% if budget > ₹3,000)
- Freelancer sees: [Release Full Payment]
- Client can choose when to pay (now or later)

[MILESTONE PAYMENT]
- Split by milestones
- Auto-release on approval
- Freelancer sees: Payment status per milestone
- Client sees: Release buttons per milestone
    ↓
During project:
- Freelancer submits → Payment tracks status
- Client approves → Payment auto-releases
- Freelancer receives → Notification in Chat & Payments tab
```

---

## 7. Data Models

### Project Model (Workspace-linked)

```javascript
const projectSchema = new Schema({
  _id: ObjectId,
  
  // Project basics
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
  
  title: String,
  description: String,
  
  // Budget & Timeline
  budget: {
    type: Number,
    required: true
  },
  
  spent: {
    type: Number,
    default: 0
  },
  
  deadline: Date,
  
  // Status
  status: {
    type: String,
    enum: ['ACTIVE', 'UNDER_REVIEW', 'COMPLETED', 'ARCHIVED'],
    default: 'ACTIVE'
  },
  
  // Workspace access
  workspaceCreatedAt: Date,
  workspaceAccessibleBy: ['CLIENT', 'FREELANCER'],
  
  // Timestamps
  createdAt: {
    type: Date,
    default: Date.now
  },
  
  completedAt: Date,
  archivedAt: Date
});
```

### Workspace Activity Model

```javascript
const workspaceActivitySchema = new Schema({
  _id: ObjectId,
  
  projectId: {
    type: Schema.Types.ObjectId,
    ref: 'Project',
    required: true,
    index: true
  },
  
  activityType: {
    type: String,
    enum: [
      'MILESTONE_SUBMITTED',
      'MILESTONE_APPROVED',
      'MILESTONE_REJECTED',
      'PAYMENT_RELEASED',
      'FILE_UPLOADED',
      'MESSAGE_SENT',
      'FOLDER_CREATED',
      'PROJECT_STATUS_CHANGED',
      'DEADLINE_EXTENDED'
    ],
    required: true
  },
  
  actor: {
    userId: Schema.Types.ObjectId,
    role: String, // CLIENT or FREELANCER
    name: String
  },
  
  description: String,
  
  // Linked entities
  linkedMilestoneId: Schema.Types.ObjectId,
  linkedPaymentId: Schema.Types.ObjectId,
  linkedFileId: Schema.Types.ObjectId,
  linkedMessageId: Schema.Types.ObjectId,
  
  // Metadata
  metadata: {
    amount: Number,
    oldValue: String,
    newValue: String
  },
  
  createdAt: {
    type: Date,
    default: Date.now,
    index: true
  }
});
```

---

## 8. API Endpoints

### Workspace Endpoints

```
GET /api/projects/:projectId/workspace
- Get complete workspace data
- Returns: Project, milestones, files, chat rooms, activity
- Auth: Project participants only

GET /api/projects/:projectId/overview
- Get overview tab data
- Returns: Summary, freelancer info, milestone progress

GET /api/projects/:projectId/activity
- Get activity timeline
- Query params: type, dateRange, limit, page
- Returns: Paginated activities with filters

GET /api/projects/:projectId/workspace/status
- Check workspace status
- Returns: Current status, permissions, access level
```

### Related Endpoints (Integrated)

```
Milestones:
GET /api/projects/:projectId/milestones
POST /api/projects/:projectId/milestones/:milestoneId/submit
POST /api/projects/:projectId/milestones/:milestoneId/approve

Files:
GET /api/projects/:projectId/files
POST /api/projects/:projectId/files/upload
GET /api/projects/:projectId/files/:fileId/download

Chat:
GET /api/chat-rooms/:projectId/messages
POST /api/chat-rooms/:projectId/messages

Payments:
GET /api/projects/:projectId/payments
POST /api/projects/:projectId/payments/:milestoneId/release
GET /api/projects/:projectId/payments/history
```

---

## 9. UI Components & Mockups

### Workspace Main Layout

```
┌─────────────────────────────────────────────────────────┐
│ OpenGigs Logo      Dashboard  Notifications  Account    │ ← Header
├─────────────────────────────────────────────────────────┤
│                                                         │
│ Project: Build Website | ₹50,000 | 60% Done | Deadline │ ← Workspace Header
│ Freelancer: John Doe (★★★★★)                            │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ [Overview] [Milestones] [Files] [Chat] [Activity] [Payments] ← Tabs
│                                                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ [Tab Content Rendered Here]                            │
│                                                         │
│ (Content changes based on selected tab)                │
│                                                         │
│                                                         │
│                                                         │
│                                                         │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Responsive Design

```
Desktop (>1024px):
- 6 tabs all visible
- Full workspace content
- Sidebar for quick actions

Tablet (768-1023px):
- 3 tabs visible + [More] dropdown
- Adjusted spacing
- Simplified layouts

Mobile (<768px):
- 1 tab visible + [More] dropdown
- Vertical scrolling
- Touch-optimized buttons
```

---

## 10. Real-time Synchronization

### WebSocket Events (Workspace-related)

```javascript
// New activity logged
workspace:activity-created {
  projectId,
  activity: { type, actor, description, timestamp }
}

// Tab data updated
workspace:milestone-updated {
  projectId,
  milestoneId,
  newStatus,
  updatedBy
}

workspace:payment-released {
  projectId,
  milestoneId,
  amount,
  timestamp
}

workspace:file-uploaded {
  projectId,
  fileId,
  fileName,
  uploadedBy,
  timestamp
}

// Chat message in workspace context
workspace:message-sent {
  projectId,
  message: { senderId, content, timestamp }
}
```

### Real-time Updates

```
When user is viewing [Overview] tab:
├─ New milestone submission → Refresh progress bar
├─ Payment released → Update spent amount
├─ File uploaded → Show in recent files
└─ New message → Activity notification

When user is viewing [Milestones] tab:
├─ Milestone status changes → Update status badge
├─ Files uploaded → Show in submission
└─ New feedback added → Alert user

Cross-tab consistency:
├─ Update in [Payments] → Reflects in [Overview]
├─ Update in [Milestones] → Reflects in [Activity]
└─ Update in [Chat] → Reflects in [Activity]
```

---

## 11. Notifications Integration

### Workspace Notifications

```
In System Notification Center:

1. "Milestone Submitted"
   From: John | Project: Build Website
   "Backend API milestone submitted for review"
   Click → Opens workspace [Milestones] tab

2. "Milestone Approved"
   From: Client | Project: Build Website
   "Database Schema approved - Payment released"
   Click → Opens workspace [Payments] tab

3. "Payment Released"
   From: System | Project: Build Website
   "₹10,000 released for Database Schema"
   Click → Opens workspace [Payments] tab

4. "File Uploaded"
   From: John | Project: Build Website
   "Backend code uploaded to Files"
   Click → Opens workspace [Files] tab

5. "New Message"
   From: John | Project: Build Website
   "Completed the API endpoints"
   Click → Opens workspace [Chat] tab
```

### In-App Badges

```
Overview Tab:
├─ Pending approvals: [2]
├─ Unread messages: [3]
└─ New activities: [5]

Milestones Tab:
├─ Submitted waiting approval: [1]
├─ Changes requested: [0]

Files Tab:
├─ New uploads: [2]

Chat Tab:
├─ Unread messages: [3]

Payments Tab:
├─ Pending releases: [1]
```

---

## 12. Security & Access Control

### Workspace Access Rules

```
✅ Can Access:
- Project participants (client & freelancer)
- Project owner (client)
- OpenGigs admins (view-only for support)

❌ Cannot Access:
- Non-participants
- Archived project (participants can view, not edit)
- Deleted project

Access Expires:
- Project archived → View-only after 30 days
- Project deleted → No access
```

### Data Privacy

```
Client Cannot See:
- Freelancer's personal messages in external channels
- Freelancer's other projects

Freelancer Cannot See:
- Client's payment methods
- Client's other projects
- Other freelancers' bids

Both Cannot See:
- Admin support notes (hidden from both)
```

### Audit Trail

```
Every action logged:
- Who: User ID & name
- What: Action type
- When: Timestamp
- Where: Project & workspace
- Result: Success/failure

Example: Approval audit
{
  action: "MILESTONE_APPROVED",
  actor: "client@example.com",
  milestone: "Backend API",
  timestamp: "2026-01-15T14:30:00Z",
  ipAddress: "192.168.1.1"
}
```

---

## 13. Testing & Acceptance Criteria

### Workspace Access & Loading

✅ Workspace loads immediately after project creation
✅ All 6 tabs accessible to both parties
✅ Tab switching is fast (<500ms)
✅ Workspace header shows correct project info
✅ Archived projects show as read-only
✅ Non-participants cannot access workspace

### Data Integrity

✅ Activity log captures all events
✅ No data loss when tab switching
✅ Milestone status updates reflect in Overview
✅ Payment updates reflect in all tabs
✅ File uploads appear immediately
✅ Chat messages sync across workspace

### Real-time Synchronization

✅ New activity appears in Activity tab within 1 second
✅ Milestone status changes reflect in Overview
✅ Multiple users see same data simultaneously
✅ Notification appears when event occurs
✅ WebSocket connection re-establishes on disconnect

### Permissions & Security

✅ Client cannot approve own milestones
✅ Freelancer cannot release own payments
✅ Archived project shows view-only badges
✅ Non-participants blocked from access
✅ Audit trail logs all actions

### UI/UX

✅ Tab order is intuitive
✅ All action buttons are visible
✅ Loading states show during data fetch
✅ Empty states handled gracefully
✅ Responsive design works on all devices
✅ No console errors or warnings

### Cross-Tab Integration

✅ Milestone approval → Payment released
✅ File upload → Appears in Files tab
✅ Chat message → Shows in Activity tab
✅ Payment release → Chat system message
✅ Status change → Updates in Overview

---

## 14. Implementation Checklist

### Phase 1: Core Workspace Structure

- [ ] Create Workspace component wrapper
- [ ] Build tab navigation system
- [ ] Create workspace header with project info
- [ ] Set up tab routing
- [ ] Create responsive layout

### Phase 2: Overview Tab

- [ ] Project summary section
- [ ] Freelancer profile card
- [ ] Milestone progress bar
- [ ] Recent activity feed (last 5)
- [ ] Quick action buttons
- [ ] Status indicators

### Phase 3: Milestones Tab

- [ ] Milestone list view
- [ ] Filter & sort functionality
- [ ] Milestone details expanded view
- [ ] File preview in submission
- [ ] Approval/rejection UI
- [ ] Comments/feedback section
- [ ] Client approval buttons
- [ ] Freelancer submit flow

### Phase 4: Files Tab

- [ ] Folder structure display
- [ ] File upload functionality
- [ ] Folder creation
- [ ] File preview
- [ ] Download functionality
- [ ] Delete with confirmation
- [ ] Upload progress indicator
- [ ] Version history (optional)

### Phase 5: Chat Tab Integration

- [ ] Embed Chat component
- [ ] Ensure workspace context
- [ ] Real-time message sync
- [ ] File sharing from Files tab
- [ ] System message generation

### Phase 6: Activity Tab

- [ ] Activity list component
- [ ] Filter by event type
- [ ] Date range filtering
- [ ] Pagination
- [ ] Event detail links
- [ ] Timeline view (optional)

### Phase 7: Payments Tab Integration

- [ ] Payment summary display
- [ ] Milestone payment list
- [ ] Payment history
- [ ] Release payment buttons
- [ ] Payment status badges
- [ ] Receipt generation

### Phase 8: Real-time Features

- [ ] WebSocket connection management
- [ ] Workspace activity events
- [ ] Real-time data synchronization
- [ ] Notification generation
- [ ] Badge count updates
- [ ] Cross-tab event sync

### Phase 9: Permissions & Security

- [ ] Role-based access control
- [ ] Permission checks per action
- [ ] Audit logging
- [ ] Data privacy enforcement
- [ ] XSS & CSRF protection

### Phase 10: Testing & QA

- [ ] Unit tests for components
- [ ] Integration tests for workflows
- [ ] E2E tests for user flows
- [ ] Performance testing
- [ ] Mobile responsive testing
- [ ] Accessibility testing (WCAG 2.1)

### Phase 11: Deployment

- [ ] Code review
- [ ] Documentation
- [ ] Staff training
- [ ] Staged rollout
- [ ] Monitoring & analytics
- [ ] User feedback collection

---

## 15. Performance Considerations

### Optimization Strategies

```
1. Lazy Loading
   - Load tab content only when selected
   - Load activities as user scrolls
   - Defer non-critical renders

2. Caching
   - Cache project overview (5 min)
   - Cache file list (2 min)
   - Cache payment history (10 min)
   - Invalidate on updates

3. Pagination
   - Activities: 20 per page
   - Files: 50 per page
   - Messages: 50 per page
   - Load more on scroll

4. Image Optimization
   - Compress file thumbnails
   - WebP format support
   - Lazy load preview images

5. API Optimization
   - Single workspace endpoint (all data)
   - Selective field loading
   - Minimal payload size
```

### Expected Performance

```
Workspace Load: < 2 seconds
Tab Switch: < 500ms
Activity Load: < 1 second
File Upload: Progress feedback every 100ms
Real-time Event: < 1 second propagation
```

---

## 16. Future Enhancements

```
Phase 2 Features (Post-Launch):
├─ Project timeline/Gantt chart
├─ Collaborative whiteboard
├─ Time tracking integration
├─ Invoice generation
├─ Recurring projects
├─ Team collaboration (multiple freelancers)
└─ API for third-party integrations

Analytics & Reporting:
├─ Project completion rate
├─ On-time delivery percentage
├─ Average payment release time
├─ Client satisfaction scores
└─ Freelancer performance metrics
```

---

## References

- **Negotiations**: [2.NEGOTIATION.md](2.NEGOTIATION.md)
- **Chat**: [2.PROJECT-CHAT.md](2.PROJECT-CHAT.md)
- **Payments**: [PROJECT-PAYMENT.md](PROJECT-PAYMENT.md)
- **Milestones**: (Document pending)

---

*Document created: Comprehensive Project Workspace documentation covering unified collaboration hub with 6 integrated tabs, workflows, data models, and implementation guide.*
