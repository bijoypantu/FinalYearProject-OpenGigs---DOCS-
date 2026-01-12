# Freelancer Onboarding 📋

## Overview
This document specifies the freelancer onboarding wizard: the fields collected incrementally, validation rules, and how the service expects profile image(s) to be handled. Data is stored in the `FreelancerProfile` model.

---

## Endpoint
- **PATCH** `/api/freelancer/onboarding` (authenticated, role: `freelancer`)
- Accepts partial updates and uses incremental saves so users can resume the wizard.

## Primary fields (mapped to `FreelancerProfile`)

| Field | Type | Required at finish | Notes / Validation |
|---|---|---:|---|
| experienceLevel | string | No | enum: `entry`, `intermediate`, `expert` |
| goal | string | No | enum: `side_income`, `main_income`, `experience` |
| workPreference | string | No | enum: `opportunities`, `packages`, `both` |
| category | string | No | e.g., `Web development` |
| skills | array[string] | No | individual tags, max reasonable length per skill |
| headline | string | No | max 70 chars |
| bio | string | No | free text, suggest 200–1000 chars limit on frontend |
| workHistory | array[object] | No | objects with title, company, startDate, endDate, description |
| education | array[object] | No | objects with institution, degree, dates |
| languages | array[object] | No | { name, proficiency } proficiency enum: `basic`, `conversational`, `fluent`, `native` |
| hourlyRate | number | No | positive number, currency handled by frontend/context |
| profilePic | string (url) | No | URL to stored image. See "Profile Image" section below. |
| phoneNumber | string | No | validate international format; prefer E.164 |
| address | object | No | `{ street, city, state, zip, country }` |
| isOnboardingFinished | boolean | Yes | set to `true` when final step completed |
| currentOnboardingStep | number | No | used for wizard navigation and resume |

---

## Recommended wizard flow (example)
1. Experience & goal
2. Work preference
3. Category & skills
4. Headline & bio
5. Work history & education
6. Languages & hourly rate
7. Personal details & profile image (finish)

Note: Frontend should send `{ "currentOnboardingStep": n }` to advance without validating page fields.

---

## Profile Image (profilePic) — important details 🔍
- Stored as a **URL** in `FreelancerProfile.profilePic` (string).
- Prefer direct upload to object storage (S3/GCS) using signed upload URLs (frontend requests signed URL, uploads file, then sends resulting public/secure URL to the onboarding endpoint).
- Accepted formats: **jpg, jpeg, png, webp**. Max recommended size: **2 MB**. Recommended aspect ratio: **1:1** or allow square crop on client-side.
- Security and moderation:
  - Validate `Content-Type` on upload server-side or via the upload provider (ensure image MIME types only).
  - Optional: run an image moderation (NSFW) step for public profile images.
  - Store two sizes (thumbnail and original) to optimize loading.
- If you prefer file upload via backend: implement a `POST /api/uploads/profile-pic` endpoint that accepts multipart form-data, validates the file, stores it, and returns a URL.

---

## Sample requests

Partial update example (JSON):

```json
PATCH /api/freelancer/onboarding
{
  "skills": ["Node.js", "React"],
  "headline": "Full-stack dev building reliable APIs",
  "currentOnboardingStep": 4
}
```

Final step example with profile URL:

```json
PATCH /api/freelancer/onboarding
{
  "profilePic": "https://cdn.example.com/profilepics/uid-12345.jpg",
  "phoneNumber": "+919876543210",
  "isOnboardingFinished": true
}
```

**Success response (200)**

```json
{
  "profile": { /* FreelancerProfile document */ },
  "user": { "isOnboardingCompleted": true, "currentOnboardingStep": 7 }
}
```

---

## Validation & UX notes
- Make most fields optional in earlier steps to reduce friction.
- Validate and format `phoneNumber` to E.164 on client-side.
- Encourage a short headline (<=70 chars) and a concise bio.
- Upload and preview profile image in the wizard before finalizing.
- Consider allowing users to skip profile picture but encourage adding it for better discoverability.

---

## Security & privacy
- Only accept images that pass basic validation (size, mime type).
- Treat uploaded URLs as untrusted; if possible, proxy served images through a secure CDN and set proper caching headers.
- Allow users to replace or remove profile images from account settings later.

---

## Changelog
- v1.0 — Initial freelancer onboarding spec and image guidance.

---

If you want me to add a sample signed-upload flow or an example `POST /uploads` implementation for the backend, tell me and I’ll add it. ✅
