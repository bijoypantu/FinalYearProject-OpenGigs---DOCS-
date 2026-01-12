# Client Onboarding 📋

## Overview
This document specifies the client onboarding wizard: fields collected incrementally, validation rules, and sample payloads. Data is stored in the `ClientProfile` model.

---

## Endpoint
- **PATCH** `/api/client/onboarding` (authenticated, role: `client`)
- Supports incremental saves so clients can finish the wizard later.

## Primary fields (mapped to `ClientProfile`)

| Field | Type | Required at finish | Notes / Validation |
|---|---|---:|---|
| hiringGoal | string | No | enum: `one_time_project`, `ongoing_work`, `browsing` |
| companyName | string | No | company or personal name |
| industry | string | No | free text or standardized list |
| numberOfEmployees | string | No | enum: `1-10`, `11-50`, `50-250`, `250+` |
| website | string (url) | No | validate URL format |
| roleInCompany | string | No | e.g., `Founder`, `Hiring Manager` |
| phoneNumber | string | No | validate international format; prefer E.164 |
| linkedInProfile | string (url) | No | optional, validate URL |
| isOnboardingFinished | boolean | Yes | set to `true` when final step completed |
| currentOnboardingStep | number | No | used for wizard navigation |

---

## Sample wizard flow (example)
1. Hiring goals
2. Company info
3. Your role
4. Contact & finish

**Final payload example**

```json
PATCH /api/client/onboarding
{
  "companyName": "Acme Corp",
  "industry": "Fintech",
  "numberOfEmployees": "11-50",
  "website": "https://acme.example.com",
  "phoneNumber": "+14155550123",
  "isOnboardingFinished": true
}
```

**Success response (200)**

```json
{
  "profile": { /* ClientProfile document */ },
  "user": { "isOnboardingCompleted": true, "currentOnboardingStep": 4 }
}
```

---

## Validation & UX notes
- Keep most fields optional to reduce friction; validate required fields on final submission where necessary.
- For `website` and `linkedInProfile` validate URL schema and sanitize inputs.
- Use `numberOfEmployees` enumeration to simplify filters in search / discovery.
- Allow clients to provide company logo later (separate flow) if needed — not part of the default client profile in the current model.

---

## Security & privacy
- Sanitize all inputs stored on the profile to prevent XSS in profile pages.
- Do not expose PII like phone numbers to other users unless necessary; control visibility by privacy settings.

---

## Changelog
- v1.0 — Initial client onboarding spec.

---

Tell me if you want an additional page describing a company logo upload flow, sample verified payment / billing steps, or sample UI wireframes to include. ✅
