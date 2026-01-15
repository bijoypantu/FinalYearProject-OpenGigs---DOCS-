# Chat System Module Documentation

## Overview

The Chat System is a real-time messaging platform for communication between clients and freelancers. It has **two distinct phases**:

1. **Negotiation Chat** (Before hiring) - Linked to proposals
2. **Project Chat** (After work begins) - Linked to projects and integrated into Project Workspace

This document focuses on the **Project Chat** system (after work begins).

> 📌 **Note**: Negotiation Chat is discussed in detail in [Negotiation_Chat.md](Negotiation_Chat.md)

### Architecture Change
- **Chat Location**: Embedded as [Chat] tab in Project Workspace
- **Access**: Only available within Project Workspace (no separate dashboard inbox)
- **Notifications**: Message notifications appear in system notification center
- **Single Interface**: All project tools (overview, milestones, files, chat, activity) in one unified workspace

---

## 1. Chat Architecture

### Core Concept

**Context-based Project Chat**: Every chat belongs to a specific project and is integrated directly into the Project Workspace as a dedicated tab.

```
Project Chat Room = Specific to Project + Client + Freelancer (in workspace)
```

---

## 2. When Does Project Chat Start?

### Timeline

```
Job Posted
    ↓
Freelancer Applies
    ↓
Negotiation Chat Opens (proposal-scoped)
    ↓
Agreement Signed & Freelancer Hired
    ↓
Project Created & Project Workspace Opens
    ↓
✅ PROJECT CHAT INTEGRATED - [Chat] Tab available in workspace
    ↓
Both can access chat from:
├─ Dashboard → Open Project → Workspace → [Chat] Tab
└─ Or direct project link → [Chat] Tab
```

### Automatic Chat Room Creation

**Trigger**: When `Project.status = "ACTIVE"` (immediately after hiring)

```javascript
// System automatically creates:
1. ChatRoom linked to Project
2. Add client to participants
3. Add freelancer to participants
4. Send system message: "Project chat started. Communication begins here."
```

---

## 3. Database Models

### ChatRoom Model (Project-based)

```javascript
const chatRoomSchema = new Schema({
  _id: ObjectId,
  
  // Links to Project
  projectId: {
    type: Schema.Types.ObjectId,
    ref: 'Project',
    required: true,
    unique: true
  },
  
  jobId: {
    type: Schema.Types.ObjectId,
    ref: 'Job',
    required: true
  },
  
  // Participants
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
  
  // Chat metadata
  status: {
    type: String,
    enum: ['ACTIVE', 'ARCHIVED', 'CLOSED'],
    default: 'ACTIVE'
  },
  
  lastMessageAt: Date,
  
  lastMessageBy: {
    type: Schema.Types.ObjectId,
    ref: 'User'
  },
  
  lastMessagePreview: String, // For UI display
  
  createdAt: {
    type: Date,
    default: Date.now
  },
  
  // Unread counts
  unreadByClient: {
    type: Number,
    default: 0
  },
  
  unreadByFreelancer: {
    type: Number,
    default: 0
  },
  
  // Chat history
  messageCount: {
    type: Number,
    default: 0
  }
});
```

### Message Model (Project Chat)

```javascript
const messageSchema = new Schema({
  _id: ObjectId,
  
  chatRoomId: {
    type: Schema.Types.ObjectId,
    ref: 'ChatRoom',
    required: true,
    index: true
  },
  
  projectId: {
    type: Schema.Types.ObjectId,
    ref: 'Project',
    required: true,
    index: true
  },
  
  senderId: {
    type: Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  
  senderRole: {
    type: String,
    enum: ['CLIENT', 'FREELANCER'],
    required: true
  },
  
  messageType: {
    type: String,
    enum: ['TEXT', 'FILE', 'SYSTEM', 'MILESTONE_UPDATE'],
    default: 'TEXT'
  },
  
  // Message content
  content: {
    type: String
    // For TEXT: full message
    // For SYSTEM: formatted system message
  },
  
  // For TEXT messages - content safety
  originalContent: String, // Before keyword masking
  contentBlocked: {
    type: Boolean,
    default: false
  },
  
  // For FILE messages
  file: {
    url: String,
    fileName: String,
    fileSize: Number, // in bytes
    fileType: String, // MIME type
    uploadedAt: Date
  },
  
  // For SYSTEM messages
  systemMessageType: {
    type: String,
    enum: [
      'PROJECT_STARTED',
      'MILESTONE_SUBMITTED',
      'MILESTONE_APPROVED',
      'MILESTONE_REJECTED',
      'WORK_SUBMITTED',
      'PARTICIPANT_ADDED'
    ]
  },
  
  // Message metadata
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
  
  // Read status
  readBy: [{
    userId: Schema.Types.ObjectId,
    readAt: Date
  }],
  
  // Reactions
  reactions: [{
    userId: Schema.Types.ObjectId,
    emoji: String,
    addedAt: Date
  }],
  
  // Context (for reference)
  linkedMilestoneId: {
    type: Schema.Types.ObjectId,
    ref: 'Milestone'
  },
  
  linkedMilestoneTitle: String,
  
  createdAt: {
    type: Date,
    default: Date.now,
    index: true
  }
});
```

### Message Safety Log Model

```javascript
const messageSafetyLogSchema = new Schema({
  _id: ObjectId,
  
  messageId: {
    type: Schema.Types.ObjectId,
    ref: 'Message',
    required: true
  },
  
  chatRoomId: {
    type: Schema.Types.ObjectId,
    ref: 'ChatRoom',
    required: true
  },
  
  originalContent: String,
  maskedContent: String,
  
  blockedItems: [{
    type: String, // PHONE, EMAIL, TELEGRAM, etc
    pattern: String,
    replacement: String
  }],
  
  action: {
    type: String,
    enum: ['MASKED', 'WARNING', 'BLOCKED'],
    default: 'MASKED'
  },
  
  senderId: Schema.Types.ObjectId,
  
  createdAt: {
    type: Date,
    default: Date.now
  }
});
```

---

## 4. Chat Access Point

### Single Access Point: Project Workspace [Chat] Tab

**Location**: Project Workspace (after hiring)

**Navigation Path**:
```
Dashboard → [Open Project] → Project Workspace
    ↓
    Tabs: [Overview] [Milestones] [Files] [Chat] [Activity] [Payments]
                                           ↓
                              ┌─────────────────────────┐
                              │ PROJECT CHAT            │
                              │ [Chat] Tab Content      │
                              └─────────────────────────┘
```

**Display**:
```
Project Workspace - Chat Tab
┌─────────────────────────────────────────────────┐
│ [Overview] [Milestones] [Files] [Chat] [Activity] [Payments]
├─────────────────────────────────────────────────┤
│ Project Context (Header)                        │
│ Budget: ₹5,000 | Deadline: Feb 15 | Status: Active
│ Current Milestone: Backend API (if applicable)  │
├─────────────────────────────────────────────────┤
│ Chat Messages                                   │
│ [Full chat history loads here]                  │
│ Real-time messages from both parties            │
├─────────────────────────────────────────────────┤
│ [Attach File] Message Input [Send]              │
└─────────────────────────────────────────────────┘
```

**Functionality**:
- Full project context visible in header
- All chat features (typing, files, etc.) available
- Integrated with project workspace tabs
- Seamless switching between project tools
- Message notifications in system notification center

### Message Notifications

**When Messages Arrive** (instead of inbox):
- 🔔 Notification appears in system notification center
- Shows: "New message from [Freelancer Name] in [Project Name]"
- Click notification → Opens Project Workspace → [Chat] Tab
- Badge count on Dashboard and Project

```
System Notification
┌────────────────────────────┐
│ 🔔 New Message             │
│                            │
│ John sent a message in     │
│ "Build Website"            │
│                            │
│ Preview: "Completed UI..." │
│                            │
│ [View] [Dismiss]           │
└────────────────────────────┘
```

---

## 5. Chat Features

### Feature 1: Real-time Messaging (Socket.io)

**Socket Events**:

```javascript
// Client connects
socket.on('connect', (token) => {
  // Authenticate user
  // Join all their active project rooms
})

// Join project chat
socket.on('chat:join', { projectId }) => {
  socket.join(`project_${projectId}`)
  io.to(`project_${projectId}`).emit('user:online', {
    userId,
    userName,
    role: 'CLIENT' | 'FREELANCER'
  })
}

// Send message
socket.on('message:send', {
  projectId,
  content,
  messageType: 'TEXT' | 'FILE'
}) => {
  // Validate & sanitize
  // Check keyword blocking
  // Save to DB
  // Broadcast to room
  io.to(`project_${projectId}`).emit('message:new', messageData)
}

// Mark as read
socket.on('message:read', {
  projectId,
  messageIds: []
}) => {
  // Update read status
  // Decrease unread count
}

// Disconnect
socket.on('disconnect', () => {
  // Broadcast offline status
  // Save last activity timestamp
}
```

### Feature 2: Typing Indicator

```javascript
socket.on('typing:start', { projectId }) => {
  io.to(`project_${projectId}`).emit('user:typing', {
    userId,
    userName,
    isTyping: true
  })
}

socket.on('typing:stop', { projectId }) => {
  io.to(`project_${projectId}`).emit('user:typing', {
    userId,
    isTyping: false
  })
}
```

**UI Display**:
```
Chat Window
┌──────────────────────────────┐
│ Messages...                  │
├──────────────────────────────┤
│ Freelancer is typing...      │ ← Typing indicator
│                              │
│ [Message Input Field]        │
│ [Send] [Attach File]         │
└──────────────────────────────┘
```

### Feature 3: File Sharing

**Supported Files**:
- Images: JPG, PNG, GIF, WebP (max 5MB)
- Documents: PDF, DOC, DOCX, XLS, XLSX (max 10MB)
- Archives: ZIP, RAR, 7Z (max 20MB)
- Video: MP4, MOV (max 50MB)

**Upload Flow**:

```javascript
socket.on('file:upload', {
  projectId,
  file: FileObject, // Binary data
  fileName: String,
  fileType: String
}) => {
  1. Validate file size and type
  2. Scan for malware (virus scan)
  3. Upload to cloud storage (S3)
  4. Create message record with file metadata
  5. Broadcast file message to chat room
  6. Send download link to both parties
}
```

**UI Display**:
```
Chat Message with File
┌──────────────────────────────┐
│ Freelancer                   │
│ "Here's the design file"     │
│                              │
│ 📎 design-mockup.zip        │
│ 2.5 MB • 2 hours ago         │
│ [Download] [Preview]         │
└──────────────────────────────┘
```

### Feature 4: Job Context Display

**Chat Header** (Always visible):

```
Project Chat Header
┌──────────────────────────────────┐
│ Job: "Build Website"             │
│ Project: #PRJ-001                │
│ Budget: ₹5,000 | Status: Active  │
│ Freelancer: John (★★★★★)         │
│ Deadline: Feb 15, 2026           │
│ Progress: 60% Complete           │
├──────────────────────────────────┤
│ Current Milestone: Backend API   │
│ Due: Feb 10, 2026                │
└──────────────────────────────────┘
```

**Benefits**:
- Both parties see full project context
- Quick reference without switching windows
- Deadline visibility
- Milestone progress tracking

### Feature 5: System Messages & Notifications

**Types of System Messages**:

```
1. PROJECT STARTED
   "Project started. Chat communication begins here."

2. MILESTONE SUBMITTED
   "Freelancer submitted 'UI Design' milestone"
   [View Submission] [Approve] [Reject]

3. MILESTONE APPROVED
   "Client approved 'UI Design' milestone"
   "Payment of ₹5,000 released to freelancer"

4. WORK SUBMITTED
   "Freelancer submitted final deliverables"
   [View Files] [Approve] [Request Revision]

5. PARTICIPANT ADDED
   "John has been added to this chat"
```

**UI Display**:
```
Chat with System Message
┌──────────────────────────────┐
│ ─────────────────────────────│ ← System message
│ 🔔 Freelancer submitted work │
│ ─────────────────────────────│
│                              │
│ Freelancer:                  │
│ "Completed first draft"      │
└──────────────────────────────┘
```

### Feature 6: Keyword Blocking (Phone & Email)

**Blocked Patterns**:
- Phone numbers: `(+\d{1,3}[-.\s]?)?\d{3,4}[-.\s]?\d{3,4}[-.\s]?\d{4}`
- Email addresses: `[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}`
- WhatsApp: `whatsapp|whatsapp me|wa\.me`
- Telegram: `telegram|@[a-zA-Z0-9_]+`

**Implementation**:

```javascript
function checkContentSafety(content) {
  let modifiedContent = content
  let blockedItems = []
  let hasBlockedContent = false
  
  const patterns = {
    EMAIL: /[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}/g,
    PHONE: /(\+\d{1,3}[-.\s]?)?\d{3,4}[-.\s]?\d{3,4}[-.\s]?\d{4}/g,
    WHATSAPP: /(whatsapp|whatsapp me|wa\.me)/gi,
    TELEGRAM: /(telegram|@[a-zA-Z0-9_]+)/gi
  }
  
  // Check each pattern
  for (const [type, regex] of Object.entries(patterns)) {
    if (regex.test(content)) {
      modifiedContent = modifiedContent.replace(regex, '●●●●●●')
      blockedItems.push(type)
      hasBlockedContent = true
    }
  }
  
  return {
    isBlocked: hasBlockedContent,
    originalContent: content,
    maskedContent: modifiedContent,
    blockedItems: blockedItems,
    warning: hasBlockedContent 
      ? '⚠️ For your safety, contact details are masked. Use OpenGigs messaging only.' 
      : null
  }
}
```

**User Experience**:

```
Message with Blocked Content
┌──────────────────────────────┐
│ Freelancer:                  │
│ "Contact me at ●●●●●●●"     │
│                              │
│ ⚠️ For your safety, contact  │
│ details are masked. Use       │
│ OpenGigs messaging only.      │
└──────────────────────────────┘
```

**System Action**:
1. Block contact details
2. Show warning message
3. Save safety log for moderation
4. Send warning notification to sender
5. Log action for support review

---

## 6. Unread Message Management

### Unread Count Tracking

```javascript
// When message is sent to chatroom
Message.create(messageData)
ChatRoom.updateOne(
  { _id: chatRoomId },
  {
    $inc: {
      'unreadByClient': recipient === client ? 1 : 0,
      'unreadByFreelancer': recipient === freelancer ? 1 : 0
    },
    lastMessageAt: new Date(),
    lastMessagePreview: messageContent,
    lastMessageBy: senderId
  }
)

// When user reads messages
socket.on('message:read', { projectId, messageIds }) => {
  Message.updateMany(
    { _id: { $in: messageIds } },
    { $push: { readBy: { userId, readAt: new Date() } } }
  )
  
  ChatRoom.updateOne(
    { projectId },
    { 'unreadByCurrentUser': 0 }
  )
}

// Badge display
Dashboard Message Badge: Shows unread count
Chat Open: Auto-marks as read when viewed
```

---

## 7. API Endpoints

### Chat Room Endpoints

```
GET /api/chat-rooms
- List all active chat rooms for logged-in user
- Returns: Array of ChatRoom objects with unread counts

GET /api/chat-rooms/:projectId
- Get specific chat room for project
- Returns: ChatRoom details

POST /api/chat-rooms/:projectId/messages
- Send message
- Body: { content, messageType }
- Returns: Message object with metadata

GET /api/chat-rooms/:projectId/messages?page=1&limit=50
- Get paginated message history
- Returns: Array of messages with pagination

PUT /api/messages/:messageId
- Edit message
- Body: { content }
- Returns: Updated message

DELETE /api/messages/:messageId
- Delete message
- Returns: Success confirmation

POST /api/messages/:messageId/read
- Mark message as read
- Returns: Updated read status

GET /api/chat-rooms/:projectId/unread-count
- Get unread message count
- Returns: { unreadCount }
```

---

## 8. UI Components

### For Project Workspace Chat Tab

**Chat Tab Design**:
```
Project Workspace - Chat Tab Layout
┌─────────────────────────────────────────┐
│ Project Context Header                  │
│ Budget: ₹5,000 | Deadline: Feb 15       │
│ Status: Active | Milestone: Backend API │
├─────────────────────────────────────────┤
│                                         │
│ 📅 Today                                │
│                                         │
│ ┌─────────────────────────────────────┐│
│ │ You:                                ││
│ │ "Started working on the API"        ││
│ │ 10:30 AM                            ││
│ └─────────────────────────────────────┘│
│                                         │
│ ┌─────────────────────────────────────┐│
│ │ Freelancer:                         ││
│ │ "Working on authentication"         ││
│ │ 10:45 AM                            ││
│ └─────────────────────────────────────┘│
│                                         │
│ Freelancer is typing...                 │
│                                         │
├─────────────────────────────────────────┤
│ [Attach File] [Emoji]                  │
│ ┌────────────────────────────────────┐ │
│ │ Type your message here...          │ │
│ └────────────────────────────────────┘ │
│ [Send]                                  │
└─────────────────────────────────────────┘
```

**Features Visible**:
- Project context always in header
- Full message history
- Typing indicators
- File upload option
- User presence status
- Message timestamps

---

## 9. Real-time Features

### User Presence

```javascript
// Shows online/offline status
socket.on('user:online', {
  userId,
  userName,
  projectId,
  role
})

socket.on('user:offline', {
  userId,
  projectId
})

// UI Indicator
Freelancer Online ✓ (green dot)
Freelancer Offline - Last seen 2 hours ago
```

### Message Delivery Confirmation

```
Message States:
✓ Sent - Message sent to server
✓✓ Delivered - Message received in chat room
✓✓✓ Read - Message read by recipient

Display in Chat:
"Hello" ✓ (1 check)
"Hello" ✓✓ (2 checks - delivered)
"Hello" ✓✓✓ (3 checks - read)
```

---

## 10. Security & Privacy

### Message Encryption
- All messages in transit: TLS/SSL encryption
- Store in DB: Encrypted at rest
- Sensitive data: Not stored in logs

### Keyword Blocking
- Phone numbers: Masked with ●●●●●●
- Email addresses: Masked with ●●●●●●
- Third-party contact links: Blocked
- Warning: Shown to sender
- Logging: Safety log maintained

### Access Control
- Only project participants can access chat
- Cannot access after project archived/completed
- Admin can view for support/disputes

### Data Retention
- Messages: Kept for project duration + 90 days
- Deleted messages: Marked as deleted, not purged
- Archived chats: Retained for 1 year

---

## 11. Notifications System

### Chat Notifications (Primary Discovery Method)

Since chat is integrated into Project Workspace and there's no separate Dashboard Messages inbox, notifications are the primary way users discover new messages:

**Notification Delivery**:
- 🔔 System notification center (main method)
- In-app toast alert (secondary)
- Email notification (if enabled in settings)
- Push notification (mobile app)
- Sound alert (optional)

**Notification Flow**:
```
1. New message arrives
   ↓
2. Notification appears in system notification center
   ↓
3. Shows: "New message from [Name] in [Project]"
   ↓
4. User clicks notification
   ↓
5. Opens Project Workspace → [Chat] Tab
   ↓
6. Message marked as read
```

**Notification Content**:
```
Example 1: "John sent a message in Build Website"
Example 2: "Sarah replied: 'Design looks great'"
Example 3: "Mike submitted milestone: Backend API"
```

**Notification Badge Counters**:
- Dashboard notification bell: Total unread messages across all projects
- Project card: Unread chat count for that specific project
- [Chat] tab: Unread indicator when workspace is open

**Notification Muting**:
- User can mute notifications per project
- Can disable all chat notifications in settings
- Do Not Disturb mode silences all alerts
- Snooze notifications for 1 hour/4 hours

---

## 12. Testing & Acceptance Criteria

### Chat Access & Integration
✅ Chat room auto-created when project starts
✅ Both parties added to chat room automatically
✅ Can access chat ONLY from Project Workspace [Chat] tab
✅ Dashboard does NOT have Messages inbox section
✅ Message notifications appear in system notification center
✅ Clicking notification opens Project Workspace → [Chat] tab
✅ All project context visible in chat header

### Real-time Features
✅ Messages are real-time (Socket.io)
✅ Typing indicator shows when other person is typing
✅ User presence (online/offline) shows correctly
✅ Message delivery states show (sent/delivered/read)
✅ Multiple simultaneous messages handled correctly

### Message Features
✅ Can send text messages
✅ Can upload and download files
✅ Can edit messages
✅ Can delete messages (soft delete)
✅ Phone numbers and emails are masked
✅ System messages show for milestones and updates
✅ Message reactions (emoji) work
✅ Link previews display correctly

### Unread Management
✅ Unread count displays correctly
✅ Badge counts on Dashboard notification center
✅ Badge counts on Project card
✅ Messages auto-marked as read when viewed
✅ Chat history is paginated

### Notifications
✅ New message notification appears in system center
✅ Notification shows sender name and project
✅ Notification preview shows message snippet
✅ Can dismiss notification without opening chat
✅ Notification mute works per project
✅ Do Not Disturb mode silences all notifications

---

## 13. Implementation Checklist

### Database
- [ ] Create ChatRoom model for projects
- [ ] Create Message model for project chat
- [ ] Create MessageSafetyLog model
- [ ] Add indexes for performance (projectId, chatRoomId, createdAt)
- [ ] Migrate negotiation chats (if needed)

### Backend
- [ ] Project chat room creation on hire
- [ ] Chat room endpoints (list, get, messages)
- [ ] Message endpoints (send, edit, delete, read)
- [ ] Keyword blocking function
- [ ] Unread count management
- [ ] File upload handling (virus scan)
- [ ] Socket.io implementation
- [ ] Notification creation on new message
- [ ] Remove Dashboard Messages inbox endpoints

### Socket.io/Real-time
- [ ] Socket connection & authentication
- [ ] Join/leave project chat rooms
- [ ] Real-time message events
- [ ] Typing indicator events
- [ ] Presence/online status
- [ ] Message read receipts
- [ ] Notification triggers on message send

### Frontend - Project Workspace Chat Tab
- [ ] Chat tab UI component
- [ ] Message list (paginated with infinite scroll)
- [ ] Message input with file upload
- [ ] Typing indicator UI
- [ ] Online status indicators
- [ ] Project context header (budget, deadline, status)
- [ ] System message rendering
- [ ] Unread count badges
- [ ] File preview/download
- [ ] Message reactions (emoji)
- [ ] Link previews
- [ ] Soft delete UI for deleted messages
- [ ] Edit history for edited messages

### Frontend - Remove Dashboard Messages
- [ ] Remove Messages section from Dashboard
- [ ] Remove Messages tab from Dashboard navigation
- [ ] Remove Dashboard message inbox UI
- [ ] Update Dashboard layout

### Frontend - Notifications System
- [ ] Notification center UI component
- [ ] Notification bell on Dashboard header
- [ ] Notification badge counter
- [ ] Notification click → Project Workspace [Chat] tab navigation
- [ ] Notification mute/snooze options
- [ ] Notification preferences UI
- [ ] Toast notification component
- [ ] Do Not Disturb mode toggle

### Notifications
- [ ] Create notification on new message
- [ ] Notification delivery to system notification center
- [ ] Email notification (if enabled)
- [ ] Push notification (mobile app)
- [ ] Sound alert (optional)
- [ ] Notification preferences UI
- [ ] Notification archival/cleanup

### Security
- [ ] Keyword blocking implementation
- [ ] File virus scan
- [ ] Access control validation (only project participants)
- [ ] Message encryption (TLS/SSL in transit)
- [ ] Encrypted storage at rest
- [ ] Safety log maintenance

### Testing & QA
- [ ] Integration tests for chat room creation
- [ ] Socket.io connection/disconnect tests
- [ ] Real-time message delivery tests
- [ ] Notification creation and delivery tests
- [ ] E2E test: Send message → Notification appears → Click → Open chat
- [ ] File upload tests (size, type, virus scan)
- [ ] Keyword masking tests
- [ ] Unread count accuracy tests
- [ ] Access control tests (participant vs non-participant)
- [ ] Performance tests (high message volume, many users)

---

*Document created: comprehensive project chat system for real-time communication between clients and freelancers.*
