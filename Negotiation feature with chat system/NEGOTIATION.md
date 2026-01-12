# Negotiation / Counter-offer System (Production-ready Spec)

✅ **Purpose:** Define a realistic negotiation system for OpenGigs modeled on real platforms (Upwork/Fiverr patterns) that supports both chat-based discussions and structured counter-offers. This doc describes every case, the data model, chat design, API endpoints, UX, and acceptance criteria.

---

## Overview

Negotiation happens after a freelancer submits a proposal (`SUBMITTED`) and before a client hires (`ACCEPT & HIRE`). This document defines a **chat-first negotiation model** (no structured counter-offer endpoints). All negotiation and agreement happen through a proposal-scoped chat room; final hiring is performed explicitly by the client via `Accept & Hire`.

Negotiation must be safe, auditable, race-condition resistant, and well-integrated with chat for natural discussion. Structured counter-offers can be considered later, but are out of scope for this document.

---

## Actors
- Client (job owner)
- Freelancer (proposal owner)
- System (notifications, events)

---

## Cases & Exact System Behavior

### Case A — Accept & Hire (Happy path)
- Trigger: Client clicks **Accept & Hire** on a proposal.
- Server actions:
  1. Verify job is `OPEN` and escrow funded
  2. Atomically create Contract/Project record (DB transaction)
  3. Update Proposal.status -> `ACCEPTED`
  4. Update Job.status -> `IN_PROGRESS`
  5. Set all other proposals for job -> `REJECTED`
  6. Notify freelancer and client (push/email/in-app)
- Result: Work starts; chat continues inside the project workspace; escrow remains locked.

### Case B — Reject
- Who can reject: Client OR Freelancer
- Trigger: Either clicks **Reject** (visible based on role & status)
- Server actions:
  1. Set Proposal.status -> `REJECTED`
  2. Freeze proposal (read-only)
  3. Notify the counterparty
- Result: No further negotiation on that proposal; freelancer may bid on other jobs.

### Case C — Negotiate (Chat-first)
- Trigger: Client or Freelancer starts or continues discussion in the proposal-scoped chat room; proposal status becomes `NEGOTIATING` when a negotiation begins.
- How it works:
  - All negotiation happens via chat messages scoped to the proposal (one chat room per proposal).
  - Participants discuss price, delivery time, and scope naturally via messages and attachments.
  - The freelancer may indicate acceptance in chat ("I accept 4200 / 5 days"), but **only the client can finalize hiring** by clicking **Accept & Hire**.
- Server actions:
  1. Create or use existing ChatRoom linked to the proposal and persist messages
  2. Set Proposal.status -> `NEGOTIATING` when the first negotiation message is posted
  3. Notify the counterparty of new messages
- Response options in the UI: Continue chat, indicate acceptance in chat, or the client can click **Accept & Hire** to create a contract and transition to `IN_PROGRESS`.

---

## Proposal Status Machine (final)
```
SUBMITTED
   |
   |----> REJECTED  (by client or freelancer)
   |
   |----> NEGOTIATING
             |
             |----> ACCEPTED (client hires)
             |
             |----> REJECTED

Also:
SUBMITTED / NEGOTIATING -> WITHDRAWN (by freelancer)
```

---

## Buttons Visibility & Permissions (UI)

### Freelancer
- SUBMITTED: Withdraw
- NEGOTIATING: Accept Counter / Reject / Counter Offer
- ACCEPTED: Go to Project
- REJECTED: None

### Client (job owner)
- SUBMITTED: Accept & Hire / Counter Offer / Reject / Start Chat
- NEGOTIATING: Accept & Hire / Counter Offer / Reject / Start Chat
- ACCEPTED: View Project
- REJECTED: None

Notes:
- `Start Chat` opens a proposal-specific chat room (see Chat design below).
- Only job owner may perform final Hire action; freelancer can accept a structured counter that the client previously offered (depending on flow choice).

---

## Data Model

### Proposal (excerpt)
```js
Proposal {
  _id: ObjectId,
  jobId: ObjectId,
  freelancerId: ObjectId,
  bidAmount: Number,
  currency: String,
  deliveryDays: Number,
  coverLetter: String,
  attachments: [String],
  status: String, // SUBMITTED | NEGOTIATING | ACCEPTED | REJECTED | WITHDRAWN
  // optional snapshot of agreed terms recorded by client when hiring or confirming
  agreedTerms: {
    amount: Number,
    days: Number,
    acceptedBy: String, // 'CLIENT' | 'FREELANCER' | null
    acceptedAt: Date
  },
  createdAt: Date,
  updatedAt: Date
}  

// Chat models remain central to negotiations (see ChatRoom and Message models below).
```
- Unique index `{ jobId, freelancerId }` enforced
- `negotiations` is append-only (audit trail)

### Chat models
```js
ChatRoom {
  _id: ObjectId,
  proposalId: ObjectId, // optional: link negotiation to a proposal
  jobId: ObjectId,
  participants: [ObjectId], // [clientId, freelancerId]
  createdAt: Date,
  lastMessageAt: Date
}

Message {
  _id: ObjectId,
  chatRoomId: ObjectId,
  senderId: ObjectId,
  text: String,
  attachments: [String],
  createdAt: Date,
  readBy: [ObjectId]
}
```

Design choices:
- One chat room per proposal (recommended) ensures messages are scoped to the negotiation.
- Chat is persisted and searchable for audits and disputes.

---

## APIs (examples)
Authentication: Bearer token; role checks

### Chat endpoints (chat-first negotiation)
- POST /api/proposals/:proposalId/chat
  - Body: { text, attachments? }
  - Auth: participant (client or freelancer)
  - Effects: create or use ChatRoom for proposal, persist Message, set Proposal.status -> `NEGOTIATING` if first negotiation message, notify other party
  - Errors: 403 if not a participant, 409 if proposal is `REJECTED` or `WITHDRAWN`

- GET /api/chat/:chatRoomId/messages?limit=50&before=messageId
  - Paginated fetch of messages

- WebSocket/Socket.IO events for real-time messages (subscribe to chatRoomId):
  - `message:create` to send message
  - `message:new` to receive broadcast messages

There are no structured negotiation endpoints in the chat-first approach; negotiation details are stored in chat messages and optionally captured in `agreedTerms` by the client when hiring.

### Chat endpoints & websocket events
- POST /api/proposals/:proposalId/chat (start chat / create room)
  - Creates ChatRoom if not exists, returns room meta

- GET /api/chat/:chatRoomId/messages?limit=50&before=messageId
  - Paginated fetch

- WS events (Socket.IO / WebSocket):
  - connect & auth with token
  - subscribe to chatRoomId
  - emit `message:create` { chatRoomId, text, attachments }
  - broadcast `message:new` to participants

Security & scaling:
- Use presence and pub/sub (Redis) for horizontal scaling
- Persist messages to DB and store attachments in object storage

---

## Sequences & Examples

### Example 1: Chat-first negotiation
1. Client starts chat from proposal page (or responds to freelancer) and says: "Can you do 4200 in 5 days?"
2. Freelancer replies: "Yes, I can do 4200 in 5 days."
3. Freelancer may write "I accept" in chat, but **only the client can finalize hiring**.
4. Client clicks **Accept & Hire** → Hire flow executes: Contract created, Proposal.status -> `ACCEPTED`, Job -> `IN_PROGRESS`, others `REJECTED`.

### Example 2: Freelancer withdraws
- Freelancer clicks Withdraw anytime while status is `SUBMITTED` or `NEGOTIATING` and job `OPEN` -> set Proposal.status -> `WITHDRAWN` and notify client.

---

## Notifications / Events
- CHAT_MESSAGE (new message in proposal chat)
- PROPOSAL_NEGOTIATION_CONFIRMED (optional: when client records agreed terms or hires)
- PROPOSAL_REJECTED
- PROPOSAL_WITHDRAWN
- FREELANCER_HIRED

Implement notification preferences (email, push, in-app) and digest strategies to avoid spam.

Implement notification preferences (email, push, in-app) and digest strategies to avoid spam.

---

## UX Patterns & Recommendations (OpenGigs)
- Provide a prominent `Start Chat` or in-page chat widget on the proposal page for negotiation
- Prefer chat-first for natural conversations; show negotiation timeline (chat history) with timestamps and participants
- Rate-limit messages if necessary and confirm destructive actions (Reject)
- Show escrow and hiring conditions prominently to reduce disputes
- Optionally provide a small UI affordance for the freelancer to mark "Accepted" in chat to help the client identify acceptance before clicking `Accept & Hire`.

---

## Security, Integrity & Concurrency
- Use DB transactions when moving from `NEGOTIATING` to `ACCEPTED` and creating Contract to prevent double hires
- Enforce RBAC and validate participants on all negotiation/counter endpoints
- Rate-limit messages and counters; virus scan attachments
- Store negotiation history append-only for auditability

---

## Testing & Acceptance Criteria
- Chat room creation returns a room tied to proposal and persists messages
- Posting messages in proposal chat sets Proposal.status -> `NEGOTIATING` and notifies the other party
- Freelancer may indicate acceptance in chat, but only the client can trigger `Accept & Hire` to create a contract; test that hiring requires client action and is atomic
- Reject marks proposal read-only and notifies counterparty
- Withdraw works in SUBMITTED/NEGOTIATING and prevents further actions
- Concurrency tests: two users trying to hire same proposal -> only one succeeds

---

## Admin & Moderation
- Admin tools to view full negotiation threads and chat logs for disputes
- Ability to redact or mute abusive users and remove attachments

---

## Migration & Implementation Notes
- Add `negotiations` array and `ChatRoom` + `Message` collections
- Add indexes: `chatRoomId` on messages, `proposalId` on chat rooms and negotiations
- Use socket auth and Redis pub/sub for message delivery at scale

---

## Example payloads
- Create counter:
```json
POST /api/proposals/613b.../negotiations
{ "amount": 4200, "days": 5, "message": "Can you do 4200 in 5 days?" }
```

- Chat message over socket:
```json
emit('message:create', { "chatRoomId": "abc123", "text": "I can do 5 days for 4200", "attachments": [] })
```

---

If you want, I can also:
- Generate Mongoose models (`models/Negotiation.js`, `models/ChatRoom.js`, `models/Message.js`)
- Implement route handlers and socket handlers
- Create sample UI wireframes for proposal negotiation page

---

*Document created: a comprehensive negotiation & chat design for production implementation.*
