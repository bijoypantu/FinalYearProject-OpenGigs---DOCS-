# Agreement Generation, Digital Signing & Hiring Module Documentation

## Overview

This module handles automatic agreement generation, digital signature collection, and the complete hiring process. It ensures legal clarity through structured, automatically-generated contracts that both parties digitally sign before work begins.

> 📌 **Note**: This document complements [5.NEGOTIATION-CHAT.md](5.NEGOTIATION-CHAT.md). Agreement generation is triggered after freelancer accepts the final offer in negotiation chat.

---

## 1. Agreement Architecture

### Core Concept

```
Final Offer Accepted (Negotiation Chat)
           ↓
Agreement Auto-Generated (This Module)
           ↓
Both Parties Digital Sign
           ↓
Hire Confirmed → Project Created
```

### Agreement Lifecycle

```
PENDING_SIGNATURES
  ↓ Freelancer signs
FREELANCER_SIGNED
  ↓ Client signs & hires
FULLY_SIGNED
  ↓ Project begins
ARCHIVED (after project completion)
```

---

## 2. Agreement Data Model

### Agreement Schema

```javascript
const agreementSchema = new Schema({
  _id: ObjectId,
  
  // Links to related entities
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
  
  // Financial Terms
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
  
  // Hourly-rate specific
  hourlyRate: Number,
  estimatedHours: Number,
  deadline: Date,
  
  // Job Context
  jobTitle: String,
  jobDescription: String,
  jobCategory: String,
  skillsRequired: [String],
  scope: String, // Detailed scope
  
  // Standard Terms (Auto-generated from policy)
  deliveryTerms: String,
  escrowTerms: String,
  cancellationPolicy: String,
  disputeResolution: String,
  revisionsAllowed: Number,
  
  // Signature Data
  freelancerSignature: {
    signed: {
      type: Boolean,
      default: false
    },
    signedAt: Date,
    ipAddress: String,
    deviceInfo: String,
    signatureMethod: {
      type: String,
      enum: ['DIGITAL_CHECKBOX', 'E_SIGNATURE', 'OTP'],
      default: 'DIGITAL_CHECKBOX'
    },
    // For OTP method
    otpSent: Boolean,
    otpVerifiedAt: Date,
    // For E-signature method (third-party)
    eSignatureProvider: String, // e.g., 'DocuSign', 'SignNow'
    eSignatureId: String,
    eSignatureUrl: String,
    // Hash of signed document for verification
    signatureHash: String,
    // Digital proof (JWT token with signature data)
    signatureProof: String
  },
  
  clientSignature: {
    signed: {
      type: Boolean,
      default: false
    },
    signedAt: Date,
    ipAddress: String,
    deviceInfo: String,
    signatureMethod: {
      type: String,
      enum: ['DIGITAL_CHECKBOX', 'E_SIGNATURE', 'OTP'],
      default: 'DIGITAL_CHECKBOX'
    },
    otpSent: Boolean,
    otpVerifiedAt: Date,
    eSignatureProvider: String,
    eSignatureId: String,
    eSignatureUrl: String,
    signatureHash: String,
    signatureProof: String
  },
  
  // Agreement Status
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
  
  // Document Generation
  agreementPdfUrl: String,
  agreementJsonData: Object, // JSON version for backup
  
  // Hire confirmation
  hiredAt: Date,
  hiredBy: Schema.Types.ObjectId, // Client's user ID
  projectId: {
    type: Schema.Types.ObjectId,
    ref: 'Project'
  },
  
  // Audit Trail
  createdAt: {
    type: Date,
    default: Date.now,
    index: true
  },
  
  updatedAt: Date,
  
  auditLog: [{
    action: String, // 'CREATED', 'FREELANCER_SIGNED', 'CLIENT_SIGNED', 'HIRED'
    actor: Schema.Types.ObjectId,
    timestamp: Date,
    ipAddress: String,
    notes: String
  }],
  
  // Revisions
  version: {
    type: Number,
    default: 1
  },
  
  previousVersions: [{
    versionNumber: Number,
    agreementPdfUrl: String,
    createdAt: Date
  }]
});
```

---

## 3. Agreement Auto-Generation

### Trigger

**When:** Freelancer accepts final offer in negotiation chat
**Action:** System automatically generates agreement document

### Generation Flow

```javascript
// In finalOffer.accept() controller
async function acceptFinalOffer(offerId) {
  const finalOffer = await FinalOffer.findById(offerId)
  const proposal = await Proposal.findById(finalOffer.proposalId)
  const job = await Job.findById(proposal.jobId)
  
  // Update proposal with final terms
  proposal.finalAmount = finalOffer.counterAmount
  proposal.finalDays = finalOffer.counterDays
  proposal.status = "AGREED"
  await proposal.save()
  
  // Auto-generate agreement
  const agreementData = {
    jobId: proposal.jobId,
    proposalId: proposal._id,
    clientId: proposal.clientId,
    freelancerId: proposal.freelancerId,
    finalAmount: finalOffer.counterAmount,
    finalDays: finalOffer.counterDays,
    paymentType: job.paymentType,
    jobTitle: job.title,
    jobDescription: job.description,
    jobCategory: job.category,
    skillsRequired: job.skillsRequired,
    scope: job.scope
  }
  
  // Generate agreement document
  const agreement = await generateAgreement(agreementData)
  
  // Notify both parties
  await sendNotification({
    userId: proposal.clientId,
    type: 'AGREEMENT_READY',
    message: 'Agreement is ready for both parties to sign',
    actionUrl: `/agreements/${agreement._id}`
  })
  
  await sendNotification({
    userId: proposal.freelancerId,
    type: 'AGREEMENT_READY',
    message: 'Agreement is ready for you to sign',
    actionUrl: `/agreements/${agreement._id}`
  })
  
  return { success: true, agreementId: agreement._id }
}
```

---

## 4. Agreement Content Template

### HTML Template Structure

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>OpenGigs Service Agreement</title>
  <style>
    body { font-family: Arial, sans-serif; line-height: 1.6; margin: 40px; }
    h1 { text-align: center; color: #333; }
    h2 { color: #555; border-bottom: 2px solid #ddd; padding-bottom: 8px; }
    table { width: 100%; border-collapse: collapse; margin: 20px 0; }
    td, th { padding: 10px; border: 1px solid #ddd; text-align: left; }
    .signature-section { margin-top: 50px; }
    .highlight { background-color: #fffacd; }
    .footer { margin-top: 30px; font-size: 12px; color: #999; }
  </style>
</head>
<body>

<h1>OpenGigs Service Agreement</h1>

<section>
  <h2>1. Parties</h2>
  <p>This Service Agreement ("Agreement") is entered into as of {{ agreementDate }}, by and between:</p>
  <table>
    <tr>
      <td><strong>Client:</strong></td>
      <td>{{ client.fullName }}</td>
    </tr>
    <tr>
      <td><strong>Email:</strong></td>
      <td>{{ client.email }}</td>
    </tr>
    <tr>
      <td><strong>Freelancer:</strong></td>
      <td>{{ freelancer.fullName }}</td>
    </tr>
    <tr>
      <td><strong>Email:</strong></td>
      <td>{{ freelancer.email }}</td>
    </tr>
    <tr>
      <td><strong>Platform:</strong></td>
      <td>OpenGigs</td>
    </tr>
  </table>
</section>

<section>
  <h2>2. Project Details</h2>
  <table>
    <tr>
      <td><strong>Project Title:</strong></td>
      <td>{{ jobTitle }}</td>
    </tr>
    <tr>
      <td><strong>Category:</strong></td>
      <td>{{ jobCategory }}</td>
    </tr>
    <tr>
      <td><strong>Skills Required:</strong></td>
      <td>{{ skillsRequired.join(', ') }}</td>
    </tr>
    <tr>
      <td><strong>Agreement Date:</strong></td>
      <td>{{ agreementDate }}</td>
    </tr>
  </table>
</section>

<section>
  <h2>3. Scope of Work</h2>
  <p>{{ jobDescription }}</p>
  <p class="highlight"><strong>Detailed Scope:</strong> {{ scope }}</p>
</section>

<section>
  <h2>4. Project Timeline</h2>
  <table>
    <tr>
      <td><strong>Start Date:</strong></td>
      <td>Upon full agreement execution (both parties sign)</td>
    </tr>
    <tr>
      <td><strong>Delivery Timeline:</strong></td>
      <td>{{ finalDays }} calendar days from project start date</td>
    </tr>
    <tr>
      <td><strong>Delivery Deadline:</strong></td>
      <td>{{ deliveryDeadline }}</td>
    </tr>
  </table>
</section>

<section>
  <h2>5. Financial Terms</h2>
  <table>
    <tr>
      <td><strong>Project Fee (INR):</strong></td>
      <td class="highlight">₹{{ finalAmount }}</td>
    </tr>
    <tr>
      <td><strong>Currency:</strong></td>
      <td>Indian Rupees (INR)</td>
    </tr>
    <tr>
      <td><strong>Payment Type:</strong></td>
      <td>{{ paymentTypeDescription }}</td>
    </tr>
    <tr>
      <td><strong>Escrow Amount (25%):</strong></td>
      <td class="highlight">₹{{ escrowAmount }}</td>
    </tr>
    <tr>
      <td><strong>Remaining Amount:</strong></td>
      <td class="highlight">₹{{ remainingAmount }}</td>
    </tr>
  </table>
  
  <p><strong>Payment Flow:</strong></p>
  <ul>
    <li>Client transfers 25% (₹{{ escrowAmount }}) to OpenGigs escrow account</li>
    <li>Freelancer begins work</li>
    <li>Upon milestone approval/final delivery, OpenGigs releases ₹{{ remainingAmount }} to freelancer</li>
    <li>Platform fee (5-10%) deducted from freelancer's earnings</li>
  </ul>
</section>

<section>
  <h2>6. Payment & Escrow Terms</h2>
  <p>
    <strong>Escrow Protection:</strong> 25% of the project fee will be held securely by OpenGigs 
    in an escrow account. This protects both parties:
  </p>
  <ul>
    <li><strong>For Client:</strong> Payment is released only after work is completed and approved</li>
    <li><strong>For Freelancer:</strong> Payment is guaranteed once work is accepted</li>
  </ul>
  
  <p><strong>Release Conditions:</strong></p>
  <ul>
    <li>Freelancer submits deliverables within agreed timeline</li>
    <li>Client reviews and approves the work</li>
    <li>Upon approval, remaining balance (75%) is released to freelancer</li>
    <li>If disputes arise, OpenGigs mediates and makes the final decision</li>
  </ul>
</section>

<section>
  <h2>7. Deliverables & Acceptance Criteria</h2>
  <p><strong>The freelancer agrees to deliver:</strong></p>
  <p>{{ scope }}</p>
  
  <p><strong>Acceptance Timeline:</strong></p>
  <ul>
    <li>Client has 5 business days to review deliverables</li>
    <li>Client can request revisions or approve</li>
    <li>Up to {{ revisionsAllowed }} revision(s) are included</li>
    <li>Additional revisions may incur extra charges (negotiated separately)</li>
  </ul>
</section>

<section>
  <h2>8. Revisions & Amendments</h2>
  <ul>
    <li><strong>Included Revisions:</strong> {{ revisionsAllowed }} revision(s) at no additional cost</li>
    <li><strong>Major Scope Changes:</strong> Require new agreement and additional payment</li>
    <li><strong>Timeline Extension:</strong> Extends deadline, may affect payment schedule</li>
    <li><strong>Any amendments:</strong> Must be agreed upon by both parties in writing</li>
  </ul>
</section>

<section>
  <h2>9. Cancellation & Refund Policy</h2>
  <ul>
    <li><strong>Work Not Started:</strong> Full refund of escrow (100%)</li>
    <li><strong>Work In Progress (0-50%):</strong> Refund based on completed percentage</li>
    <li><strong>Work 50%+ Complete:</strong> No refund; freelancer keeps escrow; client pays remaining</li>
    <li><strong>Freelancer Cancellation:</strong> Escrow and payment forfeited</li>
    <li><strong>Milestone-based Projects:</strong> Cancellation only after current milestone</li>
  </ul>
</section>

<section>
  <h2>10. Intellectual Property Rights</h2>
  <ul>
    <li><strong>Ownership:</strong> Upon final payment, all work product ownership transfers to Client</li>
    <li><strong>Pre-existing IP:</strong> Freelancer retains rights to templates, frameworks, tools created before this project</li>
    <li><strong>Third-party Content:</strong> Freelancer guarantees all content is either original or properly licensed</li>
    <li><strong>Portfolio Use:</strong> Freelancer may use work for portfolio (unless Client specifies otherwise)</li>
  </ul>
</section>

<section>
  <h2>11. Confidentiality</h2>
  <ul>
    <li>Both parties agree to keep project details confidential</li>
    <li>Client information: Freelancer will not disclose to third parties</li>
    <li>Freelancer may not publicly discuss project without Client's permission</li>
    <li>Confidentiality survives agreement termination for {{ confidentialityMonths }} months</li>
  </ul>
</section>

<section>
  <h2>12. Communication & Support</h2>
  <ul>
    <li>Primary communication: OpenGigs platform messaging only</li>
    <li>No direct contact sharing (phone, email, social media)</li>
    <li>OpenGigs monitors all conversations for quality and safety</li>
    <li>Response time: Within 24 hours during business days</li>
    <li>Violations result in platform account suspension</li>
  </ul>
</section>

<section>
  <h2>13. Dispute Resolution</h2>
  <p><strong>Process:</strong></p>
  <ol>
    <li>Direct negotiation between parties (5 business days)</li>
    <li>OpenGigs mediation (5-10 business days)</li>
    <li>OpenGigs arbitration (final decision)</li>
  </ol>
  
  <p><strong>Important:</strong></p>
  <ul>
    <li>Both parties agree to cooperate in good faith</li>
    <li>No legal action until OpenGigs process is exhausted</li>
    <li>OpenGigs decision is binding on the platform</li>
    <li>Disputes outside OpenGigs can be taken to applicable courts</li>
  </ul>
</section>

<section>
  <h2>14. Platform Terms & Conditions</h2>
  <ul>
    <li>Both parties agree to OpenGigs Terms of Service</li>
    <li>OpenGigs retains right to monitor communications</li>
    <li>Platform may suspend accounts for violations</li>
    <li>Platform fees apply (5-10% of project value)</li>
    <li>Tax compliance: Each party is responsible for their tax obligations</li>
  </ul>
</section>

<section>
  <h2>15. Entire Agreement</h2>
  <p>
    This Agreement constitutes the entire agreement between the parties 
    regarding this project. All prior discussions, proposals, and agreements 
    are superseded. No modifications are valid unless made in writing and 
    signed by both parties.
  </p>
</section>

<section class="signature-section">
  <h2>16. Digital Signatures</h2>
  
  <p>
    By signing below digitally, both parties acknowledge they have read, 
    understood, and agree to all terms in this Agreement.
  </p>
  
  <table style="margin-top: 30px;">
    <tr style="height: 100px;">
      <td>
        <strong>FREELANCER:</strong><br><br>
        Name: {{ freelancer.fullName }}<br>
        Email: {{ freelancer.email }}<br>
        Digital Signature: ___________________<br>
        Date: ___________________<br>
        <em>IP Address: {{ freelancerIpAddress }}</em><br>
        <em>Timestamp: {{ freelancerSignedAt }}</em>
      </td>
      <td>
        <strong>CLIENT:</strong><br><br>
        Name: {{ client.fullName }}<br>
        Email: {{ client.email }}<br>
        Digital Signature: ___________________<br>
        Date: ___________________<br>
        <em>IP Address: {{ clientIpAddress }}</em><br>
        <em>Timestamp: {{ clientSignedAt }}</em>
      </td>
    </tr>
  </table>
</section>

<section>
  <h2>17. Acknowledgment & Acceptance</h2>
  <p>
    <input type="checkbox" required> 
    I have read and understood all terms and conditions in this Agreement.
  </p>
  <p>
    <input type="checkbox" required> 
    I agree to be bound by this Agreement and the OpenGigs Platform Terms.
  </p>
  <p>
    <input type="checkbox" required> 
    I confirm my identity and accept the legal consequences of signing.
  </p>
</section>

<footer class="footer">
  <p>Generated on: {{ generatedDate }}</p>
  <p>Agreement ID: {{ agreementId }}</p>
  <p>Document Version: {{ version }}</p>
  <p>This is an electronically generated document. Digital signatures are legally binding.</p>
  <p>For disputes, contact: support@opengigs.com</p>
</footer>

</body>
</html>
```

---

## 5. Digital Signature Methods

### Overview

OpenGigs supports three digital signature methods:

```
1. DIGITAL_CHECKBOX (OTP-based)
   - User checks "I agree" box
   - OTP sent to email
   - User enters OTP
   - Signature recorded with timestamp & IP

2. E_SIGNATURE (Third-party integration)
   - Integration with DocuSign/SignNow
   - User signs on external platform
   - Signature data returned to OpenGigs
   - More formal, legally strong

3. OTP (One-Time Password)
   - OTP sent to user's email
   - User enters OTP as proof
   - Signature verified against mobile device
   - Medium security level
```

### Method 1: Digital Checkbox with OTP

#### Flow

```
User opens agreement page
  ↓
Reads agreement content
  ↓
Clicks "I agree" checkbox
  ↓
Clicks "Sign Agreement" button
  ↓
OTP sent to user's email
  ↓
User enters OTP in modal
  ↓
System verifies OTP
  ↓
Signature recorded with metadata:
├─ Timestamp
├─ IP Address
├─ Device Info
├─ Browser/OS
└─ Digital Proof (JWT)
  ↓
Status updated: "FREELANCER_SIGNED" or "CLIENT_SIGNED"
```

#### API: Send OTP

**Endpoint:** `POST /api/agreements/:agreementId/send-otp`

**Request:**
```json
{
  "userType": "FREELANCER", // or "CLIENT"
  "method": "DIGITAL_CHECKBOX"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "agreementId": "507f1f77bcf86cd799439060",
    "userType": "FREELANCER",
    "otpSent": true,
    "emailMasked": "fr***@example.com",
    "expiresIn": 600, // 10 minutes
    "message": "OTP sent to your registered email",
    "nextStep": "Enter OTP to complete signature"
  }
}
```

#### API: Verify OTP & Sign

**Endpoint:** `POST /api/agreements/:agreementId/sign-with-otp`

**Request:**
```json
{
  "userType": "FREELANCER",
  "otp": "123456",
  "acceptTerms": true,
  "method": "DIGITAL_CHECKBOX"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "agreementId": "507f1f77bcf86cd799439060",
    "userType": "FREELANCER",
    "signed": true,
    "signedAt": "2026-01-10T12:30:00Z",
    "status": "FREELANCER_SIGNED",
    "signature": {
      "method": "DIGITAL_CHECKBOX",
      "ipAddress": "192.168.1.100",
      "deviceInfo": "Chrome on Windows 10",
      "hash": "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6",
      "proof": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
    },
    "message": "Agreement signed successfully!",
    "nextStep": "Waiting for client to sign"
  }
}
```

#### OTP Implementation Logic

```javascript
// Send OTP
async function sendOTP(agreementId, userType) {
  const agreement = await Agreement.findById(agreementId)
  const user = userType === 'FREELANCER' ? 
    await User.findById(agreement.freelancerId) :
    await User.findById(agreement.clientId)
  
  // Generate 6-digit OTP
  const otp = Math.floor(100000 + Math.random() * 900000).toString()
  
  // Store OTP with expiry (10 minutes)
  const otpData = {
    code: otp,
    expiresAt: Date.now() + 10 * 60 * 1000,
    attempts: 0,
    maxAttempts: 3
  }
  
  if (userType === 'FREELANCER') {
    agreement.freelancerSignature.otpSent = true
    agreement.freelancerSignature.otp = otpData
  } else {
    agreement.clientSignature.otpSent = true
    agreement.clientSignature.otp = otpData
  }
  
  await agreement.save()
  
  // Send email with OTP
  await sendEmail({
    to: user.email,
    subject: 'Your OpenGigs Agreement Signature OTP',
    template: 'agreement-otp',
    variables: {
      userName: user.firstName,
      otp: otp,
      expiresIn: '10 minutes',
      agreementId: agreementId
    }
  })
  
  return { success: true, otpSent: true }
}

// Verify OTP & Sign
async function signAgreementWithOTP(agreementId, userType, otp) {
  const agreement = await Agreement.findById(agreementId)
  const signatureData = userType === 'FREELANCER' ? 
    agreement.freelancerSignature :
    agreement.clientSignature
  
  // Validate OTP
  if (!signatureData.otp) {
    throw new Error('OTP not sent or expired')
  }
  
  if (signatureData.otp.expiresAt < Date.now()) {
    throw new Error('OTP has expired')
  }
  
  if (signatureData.otp.attempts >= 3) {
    throw new Error('Maximum OTP attempts exceeded')
  }
  
  if (signatureData.otp.code !== otp) {
    signatureData.otp.attempts++
    await agreement.save()
    throw new Error('Invalid OTP')
  }
  
  // Generate signature proof (JWT)
  const signatureProof = jwt.sign({
    agreementId: agreement._id,
    userType: userType,
    userId: userType === 'FREELANCER' ? agreement.freelancerId : agreement.clientId,
    timestamp: Date.now(),
    method: 'DIGITAL_CHECKBOX_OTP'
  }, process.env.SIGNATURE_SECRET, { expiresIn: '1000y' })
  
  // Record signature
  const signatureData = {
    signed: true,
    signedAt: new Date(),
    ipAddress: req.ip,
    deviceInfo: `${req.device.type} - ${req.device.os.name}`,
    signatureMethod: 'DIGITAL_CHECKBOX',
    otpVerifiedAt: new Date(),
    signatureHash: crypto.createHash('sha256').update(signatureProof).digest('hex'),
    signatureProof: signatureProof
  }
  
  if (userType === 'FREELANCER') {
    agreement.freelancerSignature = { ...agreement.freelancerSignature, ...signatureData }
  } else {
    agreement.clientSignature = { ...agreement.clientSignature, ...signatureData }
  }
  
  // Update agreement status
  if (agreement.freelancerSignature.signed && agreement.clientSignature.signed) {
    agreement.status = 'FULLY_SIGNED'
  } else if (agreement.freelancerSignature.signed) {
    agreement.status = 'FREELANCER_SIGNED'
  }
  
  // Log action
  agreement.auditLog.push({
    action: userType === 'FREELANCER' ? 'FREELANCER_SIGNED' : 'CLIENT_SIGNED',
    actor: userType === 'FREELANCER' ? agreement.freelancerId : agreement.clientId,
    timestamp: new Date(),
    ipAddress: req.ip,
    notes: `Agreement signed via OTP verification`
  })
  
  await agreement.save()
  
  // Send notifications
  const otherUserType = userType === 'FREELANCER' ? 'CLIENT' : 'FREELANCER'
  const otherUserId = userType === 'FREELANCER' ? agreement.clientId : agreement.freelancerId
  
  await sendNotification({
    userId: otherUserId,
    type: 'AGREEMENT_SIGNED',
    message: `${userType === 'FREELANCER' ? 'Freelancer' : 'Client'} has signed the agreement`,
    actionUrl: `/agreements/${agreement._id}`
  })
  
  return {
    success: true,
    status: agreement.status,
    nextStep: agreement.status === 'FULLY_SIGNED' ? 'HIRE' : 'WAITING_FOR_OTHER_PARTY'
  }
}
```

---

### Method 2: E-Signature Integration (DocuSign)

#### Flow

```
User opens agreement
  ↓
System generates DocuSign envelope
  ↓
User clicks "Sign with DocuSign"
  ↓
Redirected to DocuSign signing page
  ↓
User signs on DocuSign platform
  ↓
DocuSign returns signed document
  ↓
System stores signature & document
  ↓
Agreement status updated
```

#### API: Create E-Signature Envelope

```javascript
async function createESignatureEnvelope(agreementId, userType) {
  const agreement = await Agreement.findById(agreementId)
  
  // Generate DocuSign envelope
  const envelopeDefinition = {
    emailSubject: 'Please sign the OpenGigs Service Agreement',
    documents: [{
      documentBase64: Buffer.from(agreementHtml).toString('base64'),
      name: `Agreement-${agreement._id}.html`,
      fileExtension: 'html',
      documentId: '1'
    }],
    recipients: {
      signers: [{
        email: userType === 'FREELANCER' ? 
          agreement.freelancer.email : agreement.client.email,
        name: userType === 'FREELANCER' ? 
          agreement.freelancer.fullName : agreement.client.fullName,
        recipientId: '1',
        routingOrder: '1',
        tabs: {
          signHereTabs: [{
            documentId: '1',
            pageNumber: String(lastPage),
            xPosition: '100',
            yPosition: '700'
          }]
        }
      }]
    },
    status: 'sent'
  }
  
  const dsApiClient = new DocuSign.ApiClient()
  dsApiClient.setBasePath(docusignApiUrl)
  dsApiClient.addDefaultHeader('Authorization', `Bearer ${accessToken}`)
  
  const envelopesApi = new DocuSign.EnvelopesApi(dsApiClient)
  const envelopeId = await envelopesApi.createEnvelope(accountId, envelopeDefinition)
  
  // Save envelope ID
  if (userType === 'FREELANCER') {
    agreement.freelancerSignature.eSignatureProvider = 'DocuSign'
    agreement.freelancerSignature.eSignatureId = envelopeId
    agreement.freelancerSignature.eSignatureUrl = `${docusignBaseUrl}/envelope/${envelopeId}`
  } else {
    agreement.clientSignature.eSignatureProvider = 'DocuSign'
    agreement.clientSignature.eSignatureId = envelopeId
    agreement.clientSignature.eSignatureUrl = `${docusignBaseUrl}/envelope/${envelopeId}`
  }
  
  await agreement.save()
  
  return {
    success: true,
    eSignatureUrl: agreement[userType === 'FREELANCER' ? 'freelancerSignature' : 'clientSignature'].eSignatureUrl
  }
}
```

#### Webhook: Handle E-Signature Completion

```javascript
// DocuSign sends webhook when signing complete
router.post('/webhooks/docusign/signing-complete', async (req, res) => {
  const { envelopeId, status } = req.body
  
  // Find agreement with this envelope ID
  const agreement = await Agreement.findOne({
    $or: [
      { 'freelancerSignature.eSignatureId': envelopeId },
      { 'clientSignature.eSignatureId': envelopeId }
    ]
  })
  
  if (!agreement) {
    return res.status(404).json({ error: 'Agreement not found' })
  }
  
  // Determine which party signed
  let userType = 'FREELANCER'
  if (agreement.clientSignature.eSignatureId === envelopeId) {
    userType = 'CLIENT'
  }
  
  if (status === 'completed') {
    // Retrieve signed document from DocuSign
    const signedDoc = await retrieveSignedDocument(envelopeId)
    
    // Update agreement
    const signatureData = {
      signed: true,
      signedAt: new Date(),
      signatureMethod: 'E_SIGNATURE',
      eSignatureProvider: 'DocuSign',
      eSignatureId: envelopeId,
      signatureHash: crypto.createHash('sha256').update(signedDoc).digest('hex'),
      signatureProof: jwt.sign({ envelopeId, timestamp: Date.now() }, process.env.SIGNATURE_SECRET)
    }
    
    if (userType === 'FREELANCER') {
      agreement.freelancerSignature = { ...agreement.freelancerSignature, ...signatureData }
    } else {
      agreement.clientSignature = { ...agreement.clientSignature, ...signatureData }
    }
    
    // Update status
    if (agreement.freelancerSignature.signed && agreement.clientSignature.signed) {
      agreement.status = 'FULLY_SIGNED'
    } else if (userType === 'FREELANCER' && agreement.freelancerSignature.signed) {
      agreement.status = 'FREELANCER_SIGNED'
    }
    
    await agreement.save()
    
    // Notify other party
    // ...
  }
  
  res.json({ success: true })
})
```

---

## 6. Agreement Signing Workflow

### Complete Flow Diagram

```
Agreement Auto-Generated
    ↓
📧 Notification sent to both parties
    ↓
Freelancer opens agreement
    ├─ Reviews terms
    ├─ Clicks "I Agree" checkbox
    ├─ Clicks "Sign Agreement"
    └─ Selects signature method
        ├─ DIGITAL_CHECKBOX (OTP)
        ├─ E_SIGNATURE (DocuSign)
        └─ OTP (Phone)
    ↓
OTP/E-Signature Process
    ├─ If OTP: Enters 6-digit code
    └─ If E-Sig: Signs on DocuSign platform
    ↓
✅ Freelancer Signature Recorded
    ├─ Timestamp: 2026-01-10T12:30:00Z
    ├─ IP Address: 192.168.1.100
    ├─ Device: Chrome on Windows 10
    └─ Proof Hash: a1b2c3d4e5f6...
    ↓
📧 Notification: "Freelancer signed, waiting for you"
    ↓
Client opens agreement
    ├─ Reviews terms
    ├─ Clicks "I Agree" checkbox
    ├─ Clicks "Sign & Hire"
    └─ Selects signature method
    ↓
OTP/E-Signature Process
    ↓
✅ Client Signature Recorded
    ↓
Agreement Status: FULLY_SIGNED
    ↓
🎯 HIRE PROCESS TRIGGERED (see Section 7)
```

---

## 7. Hiring Process

### When Agreement is Fully Signed

**Trigger:** Both parties have signed the agreement (`status = 'FULLY_SIGNED'`)

### Hire Endpoint

**Endpoint:** `POST /api/agreements/:agreementId/hire`

**Prerequisites:**
```
✅ freelancerSignature.signed = true
✅ clientSignature.signed = true
✅ Job.budget >= agreement.finalAmount
✅ No other freelancer hired for this job
✅ Client has valid payment method
```

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
    "hireConfirmation": {
      "status": "SUCCESS",
      "hiredAt": "2026-01-10T13:05:00Z",
      "message": "Freelancer successfully hired!"
    },
    "escrowDetails": {
      "escrowAmount": 1075, // 25% of 4300
      "currency": "INR",
      "deductedFrom": "Client's account",
      "status": "LOCKED",
      "releaseCondition": "Upon project milestone approval"
    },
    "projectDetails": {
      "projectId": "507f1f77bcf86cd799439070",
      "projectStatus": "ACTIVE",
      "deliveryDeadline": "2026-01-15T23:59:59Z",
      "totalBudget": 4300,
      "milestonesCount": 1 // For milestone-based
    },
    "jobUpdated": {
      "jobStatus": "IN_PROGRESS",
      "message": "Job is now in progress"
    },
    "proposalUpdated": {
      "proposalStatus": "ACCEPTED"
    },
    "otherProposalsRejected": {
      "count": 5,
      "message": "5 other proposals automatically rejected"
    },
    "notificationsSent": {
      "freelancer": "Congratulations! You're hired! Access your Project Workspace.",
      "client": "Freelancer hired successfully. Project started."
    },
    "nextSteps": [
      "Freelancer: Access Project Workspace → [Chat] tab to connect",
      "Freelancer: Review project files in [Files] tab",
      "Freelancer: Begin work on deliverables",
      "Client: Monitor project in [Milestones] tab",
      "Client: Release payment upon approval"
    ]
  }
}
```

### Hire Implementation Logic

```javascript
async function hireFreelancer(agreementId, paymentData, clientIpAddress) {
  const agreement = await Agreement.findById(agreementId)
  
  // 1. Validate signatures
  if (!agreement.freelancerSignature.signed || !agreement.clientSignature.signed) {
    throw new Error('Both parties must sign agreement first')
  }
  
  // 2. Lock escrow payment
  const escrowAmount = agreement.finalAmount * 0.25
  
  try {
    const paymentResult = await lockEscrow({
      jobId: agreement.jobId,
      amount: escrowAmount,
      paymentMethodId: paymentData.paymentMethodId,
      currency: 'INR'
    })
    
    if (!paymentResult.success) {
      throw new Error('Escrow payment failed: ' + paymentResult.error)
    }
  } catch (err) {
    throw new Error(`Escrow Payment Failed: ${err.message}`)
  }
  
  // 3. Create Project
  const project = new Project({
    jobId: agreement.jobId,
    proposalId: agreement.proposalId,
    clientId: agreement.clientId,
    freelancerId: agreement.freelancerId,
    agreementId: agreement._id,
    paymentType: agreement.paymentType,
    totalAmount: agreement.finalAmount,
    deliveryDays: agreement.finalDays,
    paymentMethod: agreement.paymentType === 'MILESTONE' ? 'MILESTONE' : 'FULL',
    escrowAmount: escrowAmount,
    status: 'ACTIVE'
  })
  await project.save()
  
  // 4. Create Chat Room (for project communication)
  const projectChatRoom = new ChatRoom({
    projectId: project._id,
    clientId: agreement.clientId,
    freelancerId: agreement.freelancerId,
    type: 'PROJECT',
    status: 'ACTIVE'
  })
  await projectChatRoom.save()
  
  // 5. Update Job
  const job = await Job.findById(agreement.jobId)
  job.status = 'IN_PROGRESS'
  job.hiredFreelancer = agreement.freelancerId
  job.projectId = project._id
  await job.save()
  
  // 6. Update Proposal
  const proposal = await Proposal.findById(agreement.proposalId)
  proposal.status = 'ACCEPTED'
  proposal.projectId = project._id
  await proposal.save()
  
  // 7. Reject other proposals
  const otherProposals = await Proposal.find({
    jobId: agreement.jobId,
    _id: { $ne: agreement.proposalId },
    status: { $in: ['SUBMITTED', 'NEGOTIATING'] }
  })
  
  for (const otherProposal of otherProposals) {
    otherProposal.status = 'REJECTED'
    otherProposal.rejectionReason = 'FREELANCER_HIRED_FOR_JOB'
    await otherProposal.save()
    
    // Notify freelancer
    await sendNotification({
      userId: otherProposal.freelancerId,
      type: 'PROPOSAL_REJECTED',
      message: `Your proposal for "${job.title}" has been rejected as another freelancer was hired`,
      jobTitle: job.title
    })
  }
  
  // 8. Update Agreement
  agreement.status = 'FULLY_SIGNED'
  agreement.hiredAt = new Date()
  agreement.hiredBy = agreement.clientId
  agreement.projectId = project._id
  agreement.auditLog.push({
    action: 'HIRED',
    actor: agreement.clientId,
    timestamp: new Date(),
    ipAddress: clientIpAddress,
    notes: `Freelancer hired. Escrow: ₹${escrowAmount} locked. Project created.`
  })
  await agreement.save()
  
  // 9. Send notifications
  await sendNotification({
    userId: agreement.freelancerId,
    type: 'HIRED_SUCCESS',
    title: 'Congratulations! You\'re Hired!',
    message: `You have been hired for "${job.title}". Check your Project Workspace to begin work.`,
    actionUrl: `/projects/${project._id}`,
    priority: 'HIGH'
  })
  
  await sendNotification({
    userId: agreement.clientId,
    type: 'HIRE_SUCCESS',
    title: 'Freelancer Hired Successfully',
    message: `${agreement.freelancer.fullName} has been hired. Escrow ₹${escrowAmount} is locked. Access your Project Workspace.`,
    actionUrl: `/projects/${project._id}`,
    priority: 'HIGH'
  })
  
  // 10. Send confirmation emails
  await sendEmail({
    to: agreement.freelancer.email,
    subject: `Congratulations! You're Hired for ${job.title}`,
    template: 'hiring-success-freelancer',
    variables: {
      freelancerName: agreement.freelancer.firstName,
      projectTitle: job.title,
      deliveryDeadline: project.deliveryDeadline,
      projectLink: `${APP_URL}/projects/${project._id}`
    }
  })
  
  await sendEmail({
    to: agreement.client.email,
    subject: `Project Started: ${job.title}`,
    template: 'hiring-success-client',
    variables: {
      clientName: agreement.client.firstName,
      freelancerName: agreement.freelancer.fullName,
      projectTitle: job.title,
      escrowAmount: escrowAmount,
      projectLink: `${APP_URL}/projects/${project._id}`
    }
  })
  
  // 11. Log to analytics
  await Analytics.logEvent({
    event: 'HIRING_COMPLETED',
    jobId: agreement.jobId,
    projectId: project._id,
    clientId: agreement.clientId,
    freelancerId: agreement.freelancerId,
    amount: agreement.finalAmount,
    timestamp: new Date()
  })
  
  return {
    success: true,
    projectId: project._id,
    escrowLocked: escrowAmount,
    nextSteps: [
      'Freelancer begins work',
      'Chat & files accessible in Project Workspace',
      'Submit milestones for approval',
      'Release payments upon approval'
    ]
  }
}
```

---

## 8. API Endpoints Summary

### Agreement Management

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/api/agreements/:agreementId` | View agreement |
| POST | `/api/agreements/:agreementId/send-otp` | Send OTP for signing |
| POST | `/api/agreements/:agreementId/sign-with-otp` | Sign with OTP |
| POST | `/api/agreements/:agreementId/create-esignature` | Create DocuSign envelope |
| POST | `/api/agreements/:agreementId/hire` | Complete hire after signing |
| GET | `/api/agreements` | List all agreements (user's) |
| GET | `/api/agreements/:agreementId/pdf` | Download signed PDF |

### Webhooks

| Event | Endpoint | Triggered By |
|-------|----------|--------------|
| E-Signature Complete | `POST /webhooks/docusign/signing-complete` | DocuSign |
| Project Created | `POST /webhooks/project/created` | Hire endpoint |

---

## 9. Security & Compliance

### Digital Signature Validation

```javascript
// Verify signature authenticity
function verifySignature(signatureProof, agreementId, expectedUserType) {
  try {
    const decoded = jwt.verify(signatureProof, process.env.SIGNATURE_SECRET)
    
    // Validate claims
    assert(decoded.agreementId === agreementId, 'Agreement ID mismatch')
    assert(decoded.userType === expectedUserType, 'User type mismatch')
    assert(decoded.method === 'DIGITAL_CHECKBOX_OTP', 'Invalid signature method')
    
    return {
      valid: true,
      timestamp: new Date(decoded.timestamp),
      userId: decoded.userId
    }
  } catch (err) {
    return { valid: false, error: err.message }
  }
}
```

### Compliance Features

✅ **Audit Trail:** Every action logged with timestamp, IP, device
✅ **Document Hashing:** SHA-256 hash prevents tampering
✅ **JWT Tokens:** Cryptographically signed proof of signature
✅ **OTP Verification:** 3-attempt limit, 10-minute expiry
✅ **Escrow Security:** Payment locked until conditions met
✅ **E-Signature:** Integration with legally-compliant providers (DocuSign)

---

## 10. Database Models Summary

### Agreement Collection

```javascript
{
  _id: ObjectId,
  jobId: ObjectId,
  proposalId: ObjectId,
  clientId: ObjectId,
  freelancerId: ObjectId,
  finalAmount: Number,
  finalDays: Number,
  paymentType: String,
  freelancerSignature: {
    signed: Boolean,
    signedAt: Date,
    ipAddress: String,
    signatureMethod: String,
    signatureProof: String
  },
  clientSignature: { ... },
  status: String,
  agreementPdfUrl: String,
  projectId: ObjectId,
  hiredAt: Date,
  auditLog: Array,
  createdAt: Date
}
```

---

## 11. Implementation Checklist

### Backend - Agreement Generation

- [ ] Create Agreement model
- [ ] Create HTML template generator
- [ ] Generate PDF from HTML
- [ ] Store agreement documents (S3)
- [ ] Create agreement endpoints (GET, LIST)

### Backend - Digital Signatures

- [ ] Implement OTP generation & verification
- [ ] Create JWT signature proof system
- [ ] Implement signature recording
- [ ] Validate signatures (tampering check)
- [ ] Implement audit logging

### Backend - E-Signature Integration

- [ ] Integrate DocuSign API
- [ ] Create DocuSign envelope generator
- [ ] Handle DocuSign webhooks
- [ ] Retrieve signed documents

### Backend - Hiring

- [ ] Implement escrow locking
- [ ] Create Project on hire
- [ ] Update Job & Proposal status
- [ ] Reject other proposals
- [ ] Create project chat room

### Frontend - Agreement Viewing

- [ ] Agreement display component
- [ ] Scroll-to-sign requirement
- [ ] Checkbox & confirmation dialogs
- [ ] Terms acceptance tracking

### Frontend - Signature Collection

- [ ] OTP input modal
- [ ] E-Signature redirect flow
- [ ] Loading & success states
- [ ] Error handling & retries

### Frontend - Hire Confirmation

- [ ] Summary display before hire
- [ ] Hire confirmation modal
- [ ] Payment method selection
- [ ] Success page with next steps

### Testing

- [ ] Unit tests for OTP generation
- [ ] Integration tests for full signing flow
- [ ] E2E tests for hire process
- [ ] Security tests (signature tampering)
- [ ] Compliance validation

---

## 12. Error Handling

### Common Scenarios

```json
{
  "error": "Both parties must sign agreement first",
  "code": "SIGNATURES_INCOMPLETE",
  "statusCode": 400
}

{
  "error": "Escrow payment failed",
  "code": "ESCROW_PAYMENT_FAILED",
  "statusCode": 402,
  "details": "Insufficient funds in account"
}

{
  "error": "OTP has expired",
  "code": "OTP_EXPIRED",
  "statusCode": 400,
  "expiresIn": 600
}

{
  "error": "A freelancer is already hired for this job",
  "code": "FREELANCER_ALREADY_HIRED",
  "statusCode": 400
}
```

---

## References

- **Negotiation Chat**: [5.NEGOTIATION-CHAT.md](5.NEGOTIATION-CHAT.md)
- **Project Workspace**: [8.PROJECT-WORKSPACE.md](8.PROJECT-WORKSPACE.md)
- **Project Payment**: [6.PROJECT-PAYMENT.md](6.PROJECT-PAYMENT.md)

---

*Document created: Comprehensive Agreement Generation, Digital Signing, and Hiring workflow with electronic signature support, escrow management, and complete audit trail.*
