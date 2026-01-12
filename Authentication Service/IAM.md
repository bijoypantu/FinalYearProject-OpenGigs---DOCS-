# Identity & Access Management (IAM) ✅

## Overview
A comprehensive Identity and Access Management (IAM) module responsible for secure user authentication, authorization, and account lifecycle flows for the platform.

**Key features**

- User registration and login (email/password)
- Session management (access + refresh tokens)
- Password reset (OTP)
- Email & Mobile OTP verification
- Role-based access control (CLIENT / FREELANCER / ADMIN)
- Google Sign-In (OAuth)
- Token refresh, rotation and revocation

---

## Data Model

User document (MongoDB)

```json
{
  "_id": "ObjectId",
  "email": "string",
  "mobile": "string",
  "passwordHash": "string",
  "firstName": "string",
  "lastName": "string",
  "role": "string",           // ["GUEST","CLIENT","FREELANCER","ADMIN"]
  "status": "string",         // ["ACTIVE","INACTIVE","SUSPENDED"]
  "isEmailVerified": true,
  "isMobileVerified": false,
  "googleId": "string",       // Google OAuth identifier
  "createdAt": "Date",
  "updatedAt": "Date"
}
```

---

## Enums

| Name | Values | Description |
|------|--------|-------------|
| AuthProvider | LOCAL, GOOGLE | Authentication provider used for the account |
| UserRole | GUEST, CLIENT, FREELANCER, ADMIN | Role-based access levels |

---

## API Endpoints 🔐

> All endpoints are prefixed with `/api/v1/auth`

### 1) Register User
- **POST** `/register`

**Request Body**

```json
{
  "email": "string",
  "password": "string",
  "firstName": "string",
  "lastName": "string",
  "mobile": "string",
  "role": "CLIENT | FREELANCER"
}
```

**Success Response (201)**

```json
{
  "success": true,
  "message": "User registered successfully. OTP sent to email and mobile."
}
```

**Notes**
- Email and mobile must be unique.
- On success, email and mobile OTPs are issued for verification.

---

### 2) Login (Email + Password)
- **POST** `/login`

**Request Body**

```json
{ "email": "string", "password": "string" }
```

**Success Response (200)**

```json
{
  "success": true,
  "data": {
    "accessToken": "string",
    "refreshToken": "string",
    "expiresIn": 3600
  }
}
```

**Notes**
- Access token is short lived; refresh token is stored and rotated securely.

---

### 3) Login with Google (OAuth)
- **POST** `/google`

**Description**
- Client sends the Google `idToken` obtained from the Google Sign-In SDK.
- Backend verifies the `idToken` with Google. If the user doesn't exist, create one (no password stored).

**Request Body**

```json
{ "idToken": "string" }
```

**Success Response (200)**

```json
{
  "success": true,
  "data": {
    "accessToken": "string",
    "refreshToken": "string",
    "expiresIn": 3600,
    "user": {
      "email": "string",
      "firstName": "string",
      "lastName": "string",
      "role": "string"
    }
  }
}
```

---

### 4) Refresh Token
- **POST** `/refresh`

**Request Body**

```json
{ "refreshToken": "string" }
```

**Success Response (200)**

```json
{
  "success": true,
  "data": {
    "accessToken": "string",
    "refreshToken": "string"
  }
}
```

**Notes**
- Implement token rotation and revoke old refresh tokens when issuing new ones.

---

### 5) Logout (Revoke Tokens)
- **POST** `/logout`

**Request Body**

```json
{ "refreshToken": "string" }
```

**Success Response (200)**

```json
{ "success": true, "message": "Logged out successfully" }
```

---

### 6) Forgot Password (Send OTP)
- **POST** `/password/forgot`

**Request Body**

```json
{ "email": "string" }
```

**Success Response (200)**

```json
{ "success": true, "message": "OTP sent to your email" }
```

---

### 7) Reset Password
- **POST** `/password/reset`

**Request Body**

```json
{ "email": "string", "otp": "string", "newPassword": "string" }
```

**Success Response (200)**

```json
{ "success": true, "message": "Password reset successfully" }
```

---

### 8) Verify Email OTP
- **POST** `/verify/email`

**Request Body**

```json
{ "email": "string", "otp": "string" }
```

**Success Response (200)**

```json
{ "success": true, "message": "Email verified successfully" }
```

---

### 9) Verify Mobile OTP
- **POST** `/verify/mobile`

**Request Body**

```json
{ "mobile": "string", "otp": "string" }
```

**Success Response (200)**

```json
{ "success": true, "message": "Mobile verified successfully" }
```

---

## Business Rules 🧠

- Email and mobile are unique per user.
- Passwords must be at least 8 characters and include at least 1 uppercase, 1 lowercase, and 1 digit.
- Role must be one of: CLIENT, FREELANCER, ADMIN.
- Refresh tokens are stored securely and revoked on logout.
- For Google-authenticated users, no password is stored; `googleId` is kept.
- OTPs expire (e.g., 5–15 minutes) and have limited retry attempts.

---

## Database Design (Users Collection) 🧱

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| _id | ObjectId | Yes | Primary key |
| email | String | Yes | Unique |
| mobile | String | No | Unique if present |
| passwordHash | String | No | Not set for Google users |
| firstName | String | Yes |  |
| lastName | String | Yes |  |
| role | String | Yes | Default CLIENT |
| status | String | Yes | ACTIVE/INACTIVE/SUSPENDED |
| isEmailVerified | Boolean | Yes |  |
| isMobileVerified | Boolean | Yes |  |
| googleId | String | No | Set for Google OAuth accounts |
| createdAt | Date | Yes |  |
| updatedAt | Date | Yes |  |

---

## Validation & Security 🔒

- Validate email format and Indian mobile number format where applicable.
- Enforce strong password policy.
- Hash passwords with bcrypt (or equivalent), using a secure cost factor.
- Verify Google `idToken` server-side with Google API.
- Implement rate limiting on auth endpoints and OTP verification attempts.
- Use HTTPS and set secure cookie attributes if cookies are used.
- Store refresh tokens hashed or in a secure store (DB with rotation or Redis).
- Consider token blacklisting for immediate revocation.

---

## Error Codes

| HTTP | Code | Message | Reason |
|------|------|---------|--------|
| 400 | Bad Request | Missing params or validation failed |
| 401 | Unauthorized | Invalid credentials or token |
| 403 | Forbidden | Token expired or insufficient privileges |
| 409 | Conflict | Email or mobile already exists |
| 500 | Server Error | Internal system error |

---

## Integration Notes (Google OAuth) 🔗

1. Frontend obtains `idToken` using Google Sign-In SDK.
2. Frontend POSTs the `idToken` to `/api/v1/auth/google`.
3. Backend verifies `idToken` with Google tokeninfo endpoint or Google SDK.
4. If token is valid:
   - If user exists → issue tokens
   - Else → create user with `googleId` and issue tokens

---

## Role-specific Onboarding Docs 📚

Detailed onboarding specifications exist per role and describe exactly what fields to collect, validation rules, and UX guidance:

- **Freelancer onboarding** — `docs/Freelancer-Onboarding.md` (includes `profilePic` guidance — this is stored on the `FreelancerProfile` model as `profilePic: string` and should be a secure URL to an uploaded image).
- **Client onboarding** — `docs/Client-Onboarding.md` (no profile picture by default; company logo can be added later as a separate flow).

---

## Examples & Helpful Tips 💡

- Use short-lived access tokens (e.g., 15m–1h) and rotate refresh tokens.
- Send OTPs over both email and SMS where possible for higher reliability.
- Log auth events (register/login/password reset) for auditing.

---

## Changelog
- v1.0 — Initial documentation and API contract.

---

If you want extra sections (sequence diagrams, example cURL requests, rate-limit rules, or test cases), tell me what to add and I'll update this file. ✅


``
{
  "_id": "ObjectId",
  "email": "string",
  "mobile": "string",
  "passwordHash": "string",
  "firstName": "string",
  "lastName": "string",
  "role": "string",           // ["GUEST","CLIENT","FREELANCER","ADMIN"]
  "status": "string",         // ["ACTIVE","INACTIVE","SUSPENDED"]
  "isEmailVerified": "boolean",
  "isMobileVerified": "boolean",
  "googleId": "string",       // Google OAuth identifier
  "createdAt": "Date",
  "updatedAt": "Date"
}
``
📌 Enums
AuthProvider
Value	Description
LOCAL	Classic email + password
GOOGLE	Google OAuth

UserRole
Value	Description
GUEST	Not logged
CLIENT	Client user
FREELANCER	Worker user
ADMIN	Platform admin

🔐 Authentication Endpoints

POST /api/v1/auth/register

📌 Request Body
{
  "email": "string",
  "password": "string",
  "firstName": "string",
  "lastName": "string",
  "mobile": "string",
  "role": "CLIENT | FREELANCER"
}

📌 Response (201)
{
  "success": true,
  "message": "User registered successfully. OTP sent to email and mobile."
}

➤ 2) Login User (Email + Password)
POST /api/v1/auth/login

📌 Request Body
{
  "email": "string",
  "password": "string"
}

📌 Response (200)
{
  "success": true,
  "data": {
    "accessToken": "string",
    "refreshToken": "string",
    "expiresIn": "number"
  }
}

➤ 3) Login with Google (OAuth)
POST /api/v1/auth/google

📌 Description

Allows users to sign in using Google OAuth2.
If the user does not exist, a new user is created.

📌 Request Body
{
  "idToken": "string"
}


idToken is obtained from Google Sign-In SDK on client-side

📌 Response (200)
{
  "success": true,
  "data": {
    "accessToken": "string",
    "refreshToken": "string",
    "expiresIn": "number",
    "user": {
      "email": "string",
      "firstName": "string",
      "lastName": "string",
      "role": "string"
    }
  }
}

➤ 4) Refresh Token
POST /api/v1/auth/refresh

📌 Request Body
{
  "refreshToken": "string"
}

📌 Response
{
  "success": true,
  "data": {
    "accessToken": "string",
    "refreshToken": "string"
  }
}

➤ 5) Logout (Revoke Tokens)
POST /api/v1/auth/logout

📌 Request
{
  "refreshToken": "string"
}

📌 Response
{
  "success": true,
  "message": "Logged out successfully"
}

➤ 6) Forgot Password (Send OTP)
POST /api/v1/auth/password/forgot

📌 Request
{
  "email": "string"
}

📌 Response
{
  "success": true,
  "message": "OTP sent to your email"
}

➤ 7) Reset Password
POST /api/v1/auth/password/reset

📌 Request
{
  "email": "string",
  "otp": "string",
  "newPassword": "string"
}

📌 Response
{
  "success": true,
  "message": "Password reset successfully"
}

➤ 8) Verify Email OTP
POST /api/v1/auth/verify/email

📌 Request
{
  "email": "string",
  "otp": "string"
}

📌 Response
{
  "success": true,
  "message": "Email verified successfully"
}

➤ 9) Verify Mobile OTP
POST /api/v1/auth/verify/mobile

📌 Request
{
  "mobile": "string",
  "otp": "string"
}

📌 Response
{
  "success": true,
  "message": "Mobile verified successfully"
}

🧠 Business Rules

✔ Email and mobile are unique per user
✔ Password must be 8+ characters
✔ Role must be either CLIENT | FREELANCER
✔ Refresh token stored securely & revoked on logout
✔ If Google login is used, no password is stored

🧱 Database Design
Collection: users
Field	Type	Required
_id	ObjectId	Yes
email	String	Yes
mobile	String	No
passwordHash	String	No (not for Google)
firstName	String	Yes
lastName	String	Yes
role	String	Yes
status	String	Yes
isEmailVerified	Boolean	Yes
isMobileVerified	Boolean	Yes
googleId	String	No
createdAt	Date	Yes
updatedAt	Date	Yes
🛠 Validation

Email format: valid email

Mobile format: valid Indian mobile

Password: must include at least 1 uppercase, 1 lower, 1 digit

Google token: must be verified with Google API

🧪 Error Codes
Code	Message	Reason
400	Bad Request	Missing params
401	Unauthorized	Invalid login
403	Forbidden	Token expired
409	Conflict	Email already exists
500	Server Error	Internal system
📌 Integration Notes
Google OAuth Flow

Client uses Google Sign-In SDK to get idToken

Frontend sends idToken to backend

Backend verifies idToken with Google API

If valid:

If user exists → generate tokens

If not → create user → generate tokens
