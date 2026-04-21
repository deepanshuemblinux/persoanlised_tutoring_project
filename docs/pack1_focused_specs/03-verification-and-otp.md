- This spec is derived from: pack1_final_implementation_details.md
- This spec owns: Verification and OTP
- This spec strictly preserves original implementation details without modification
- This spec must be used with shared contract spec: docs/pack1_focused_specs/01-platform-ux-and-shared-contracts.md
- Fallback context source: docs/pack1_final_implementation_details.md

# 2. Frozen Scope and Decisions

## 2.2 Verification rules

### Tutor

Required fields:
- full name
- email
- phone
- password
- subjects taught
- classes taught

Verification required before entering approval queue:
- email OTP verification only

Phone behavior:
- phone is captured but not verified
- UI helper text must say:
  - **Preferably enter the number used for WhatsApp communication.**

### Student

Required fields:
- full name
- email
- phone
- password

Verification required before entering approval queue:
- email OTP verification only

Phone behavior:
- phone is captured but not verified
- UI helper text must say:
  - **Preferably enter the number used for WhatsApp communication.**


# 6. Screen-by-Screen UX Details

## 6.2 Tutor Verify Email OTP

### Visual References:
- `ui_mock_screens/Tutor_Verify_Email_OTP.png`

**Fields**
- masked_email
- otp_code (6 digits)

**Actions**
- Verify
- Resend OTP
- Change Email

**Rules**
- resend cooldown shown
- max active OTP attempts enforced server-side
- expired OTP message must be specific
- on changing email, prior OTP must be invalidated and signup session updated

**Completion rule**
When email is verified, account is created in `PENDING` approval state.

## 6.7 Student Verify Email OTP

### Visual References:
- `ui_mock_screens/Student_Verify_Email_OTP.png`

Same interaction model as tutor email verification.

**Completion rule**
When email is verified, account is created in `PENDING` approval state.

## 6.26 Verify Reset OTP

### Visual References:
- `ui_mock_screens/Verify_Reset_OTP.png`

**Fields**
- email
- otp_code

**Actions**
- Verify OTP
- Resend OTP


# 8. UX Rules and Behavioral Rules

## 8.1 Verification and onboarding rules

- Tutor and student require email verification only
- User record must not become active before email is verified
- Pending approval screen must clearly explain next step
- Verification resend must respect cooldown and retry limits
- Phone must be collected but is informational only in Pack 1


# 9. Pack 1 High-Level Design (HLD)

## 9.1 Architecture modules

### Verification Module
Responsibilities:
- generate OTPs
- send OTPs over email
- verify OTPs
- cooldown, expiry, retry, and lock policies


# 12. Database Schema Draft (Implementation-Oriented)

### `signup_sessions`

Stores pre-user verification progress so incomplete verifications do not create live users.

| Column | Type | Null | Notes |
|---|---|---:|---|
| id | uuid | no | PK |
| role | varchar(20) | no | TUTOR / STUDENT |
| institute_id | uuid | no | FK institutes.id |
| payload_json | jsonb | no | signup draft payload |
| email | varchar(255) | no | normalized |
| phone | varchar(20) | no | normalized |
| email_verified | boolean | no | default false |
| expires_at | timestamptz | no | session expiry |
| created_at | timestamptz | no | |
| updated_at | timestamptz | no | |

**Indexes**
- index(email)
- index(phone)
- index(expires_at)

### `otp_verifications`

| Column | Type | Null | Notes |
|---|---|---:|---|
| id | uuid | no | PK |
| signup_session_id | uuid | yes | FK signup_sessions.id |
| user_id | uuid | yes | FK users.id |
| purpose | varchar(30) | no | SIGNUP_EMAIL / RESET_PASSWORD |
| channel | varchar(20) | no | EMAIL |
| target_value | varchar(255) | no | email |
| otp_hash | varchar(255) | no | never store plain OTP |
| attempts_count | int | no | default 0 |
| resend_count | int | no | default 0 |
| expires_at | timestamptz | no | |
| verified_at | timestamptz | yes | |
| consumed_at | timestamptz | yes | |
| created_at | timestamptz | no | |

**Indexes**
- index(target_value, purpose)
- index(expires_at)


# 13. API Draft (Implementation-Oriented)

### `POST /api/v1/auth/signup/verify-email`

**Request**

```json
{
  "signup_session_id": "uuid",
  "otp": "123456"
}
```

**Response 200**

```json
{
  "email_verified": true,
  "account_created": true,
  "approval_status": "PENDING",
  "next_step": "WAIT_FOR_ADMIN_APPROVAL"
}
```

### `POST /api/v1/auth/signup/resend-otp`

**Request**

```json
{
  "signup_session_id": "uuid"
}
```

### `POST /api/v1/auth/verify-reset-otp`

**Request**

```json
{
  "email": "user@example.com",
  "otp": "123456"
}
```

**Response 200**

```json
{
  "reset_token": "short-lived-token"
}
```


# 14. Validation Rules and Business Rules

## 14.2 OTP rules

- OTP length = 6 digits
- OTP expiry = 5 minutes
- max verify attempts per OTP = 5
- resend cooldown = 30 seconds
- max resend count per session configurable, recommended 5
- expired or consumed OTP cannot be reused

