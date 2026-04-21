- This spec is derived from: pack1_final_implementation_details.md
- This spec owns: Authentication and account recovery
- This spec strictly preserves original implementation details without modification
- This spec must be used with shared contract spec: docs/pack1_focused_specs/01-platform-ux-and-shared-contracts.md
- Fallback context source: docs/pack1_final_implementation_details.md

# 6. Screen-by-Screen UX Details

## 6.1 Tutor Sign Up

### Visual References:
- `ui_mock_screens/Tutor_Sign_Up.png`

**Purpose**  
Create a tutor signup draft, verify email, then place user into approval queue.

**Fields**
- `full_name`: text, required, max 120
- `email`: email, required, unique
- `phone`: E.164-normalized phone, required, unique
- `password`: password, required
- `confirm_password`: password, required
- `subjects`: multi-select, required, at least 1
- `classes`: multi-select, required, at least 1
- `terms_accepted`: checkbox, required

**Field-level UX notes**
- Phone input must show helper text:
  - **Preferably enter the number used for WhatsApp communication.**
- Email uniqueness errors should be shown inline
- Phone uniqueness errors should be shown inline
- Subjects and classes may use searchable multi-select if the option count is large

**Actions**
- Send Email OTP
- Continue
- Go to Login

**Validation rules**
- full_name not blank
- email format valid
- phone format valid
- password length >= 8
- password must match confirm_password
- at least one subject selected
- at least one class selected
- terms accepted
- email must be unique within institute
- phone must be unique within institute

**UX states**
- idle
- client_validation_error
- email_otp_sent
- waiting_for_email_verification
- submitting
- signup_success
- duplicate_email_error
- duplicate_phone_error
- rate_limited_error

**Routing**
- successful initiation -> Tutor Verify Email OTP
- successful email verification -> Tutor Approval Pending

## 6.3 Tutor Login

### Visual References:
- `ui_mock_screens/Tutor_Login.png`

**Fields**
- email
- password

**Actions**
- Login
- Forgot Password
- Sign Up

**Routing rules**
- `APPROVED` -> Tutor Dashboard
- `PENDING` -> Tutor Approval Pending
- `REJECTED` -> Account Rejected

**API interaction**
- submit to `/api/v1/auth/login`
- on `ACCOUNT_PENDING_APPROVAL`, route to pending page
- on `ACCOUNT_REJECTED`, route to rejected page

## 6.6 Student Sign Up

### Visual References:
- `ui_mock_screens/Student_Sign_Up.png`

**Purpose**  
Create a student signup draft, verify email, then place user into approval queue.

**Fields**
- `full_name`: text, required, max 120
- `email`: email, required, unique
- `phone`: E.164-normalized phone, required, unique
- `password`: password, required
- `confirm_password`: password, required
- `terms_accepted`: checkbox, required

**Field-level UX notes**
- Phone input must show helper text:
  - **Preferably enter the number used for WhatsApp communication.**

**Actions**
- Send Email OTP
- Continue
- Go to Login

**Validation rules**
- same core validations as tutor, except no subject/class selection

**Routing**
- successful initiation -> Student Verify Email OTP
- successful email verification -> Student Approval Pending

## 6.8 Student Login

### Visual References:
- `ui_mock_screens/Student_Login.png`

**Fields**
- email
- password

**Actions**
- Login
- Forgot Password
- Sign Up

**Routing rules**
- `APPROVED` -> Student Dashboard
- `PENDING` -> Student Approval Pending
- `REJECTED` -> Account Rejected

## 6.25 Forgot Password

### Visual References:
- `ui_mock_screens/Forgot_Password.png`

**Fields**
- email

**Actions**
- Send OTP
- Back to Login

**Behavior**
- generic success response to avoid user enumeration
- OTP is sent only if the account exists and email is verified

## 6.27 Reset Password

### Visual References:
- `ui_mock_screens/Reset_Password.png`

**Fields**
- new_password
- confirm_new_password

**Actions**
- Submit

**Validation rules**
- password policy
- confirm must match
- token/session from successful reset OTP required


# 7. Primary Pack 1 User Flows

## 7.1 Tutor onboarding flow

1. Tutor opens sign-up screen
2. Enters profile and teaching metadata
3. Receives email OTP
4. Verifies email
5. System creates tutor account with `PENDING`
6. Admin reviews tutor
7. Admin approves or rejects
8. Tutor logs in
9. Approved tutor lands on tutor dashboard

## 7.2 Student onboarding flow

1. Student opens sign-up screen
2. Enters identity and credentials
3. Receives email OTP
4. Verifies email
5. System creates student account with `PENDING`
6. Admin reviews student
7. Admin approves or rejects
8. Student logs in
9. Approved student lands on student dashboard

## 7.6 Forgot password flow

1. User opens Forgot Password from login page
2. Enters email
3. OTP sent to verified email
4. User verifies reset OTP
5. User sets new password
6. Old password becomes invalid
7. User logs in with new password


# 8. UX Rules and Behavioral Rules

## 8.5 Authentication rules

- Login uses email + password
- Password reset invalidates old password immediately
- Locked, pending, or rejected states affect login routing


# 9. Pack 1 High-Level Design (HLD)

## 9.1 Architecture modules

### Auth Module
Responsibilities:
- signup initiation
- login
- logout
- forgot password
- reset password
- token issuance


# 10. Pack 1 Low-Level Design (LLD)

## 10.1 Detailed tutor signup sequence

1. Client submits tutor signup draft
2. Server validates payload
3. Server checks email and phone uniqueness against existing users
4. Server creates signup session / draft record
5. Server generates email OTP
6. OTP is delivered through email channel
7. Client submits email OTP
8. Server verifies email OTP and marks `email_verified = true` in signup session
9. Once email is verified, server creates:
   - `users` row with role `TUTOR`, approval_status `PENDING`, email_verified `true`, phone_verified not applicable
   - `tutor_profiles` row
   - tutor subject/class capability rows
10. Server deletes or expires signup session
11. Admin sees pending tutor in review queue
12. Admin approves or rejects
13. Login flow routes based on stored approval state

## 10.2 Detailed student signup sequence

1. Client submits student signup draft
2. Server validates payload
3. Server checks email and phone uniqueness against existing users
4. Server creates signup session / draft record
5. Server generates email OTP
6. OTP is delivered through email channel
7. Client submits email OTP
8. Server verifies email OTP and marks `email_verified = true` in signup session
9. Once email is verified, server creates:
   - `users` row with role `STUDENT`, approval_status `PENDING`, email_verified `true`
   - `student_profiles` row
10. Server deletes or expires signup session
11. Admin sees pending student in review queue
12. Admin approves or rejects
13. Login flow routes based on stored approval state

## 10.6 Forgot password sequence

1. User submits email from Forgot Password screen
2. Server confirms email belongs to an existing verified account
3. Server creates reset OTP record
4. Server delivers OTP
5. User submits OTP
6. Server validates OTP and returns short-lived reset token/session
7. User submits new password
8. Server hashes password and updates user record
9. Server expires all outstanding reset tokens and invalidates old sessions if configured


# 12. Database Schema Draft (Implementation-Oriented)

### `users`

| Column | Type | Null | Notes |
|---|---|---:|---|
| id | uuid | no | PK |
| institute_id | uuid | no | FK institutes.id |
| role | varchar(20) | no | ADMIN / TUTOR / STUDENT |
| full_name | varchar(120) | no | |
| email | varchar(255) | no | normalized lowercase |
| email_verified | boolean | no | default false |
| phone | varchar(20) | no | normalized E.164 |
| password_hash | varchar(255) | no | bcrypt/argon2 hash |
| approval_status | varchar(20) | no | PENDING / APPROVED / REJECTED |
| rejection_reason | text | yes | admin-set |
| is_active | boolean | no | default true |
| last_login_at | timestamptz | yes | |
| created_at | timestamptz | no | |
| updated_at | timestamptz | no | |

**Constraints**
- unique(institute_id, email)
- unique(institute_id, phone)
- check(role in ('ADMIN','TUTOR','STUDENT'))
- check(approval_status in ('PENDING','APPROVED','REJECTED'))

**Indexes**
- index on (role, approval_status)
- index on (email)
- index on (phone)

### `password_reset_sessions`

Optional but cleaner than overloading OTP tables for reset completion.

| Column | Type | Null | Notes |
|---|---|---:|---|
| id | uuid | no | PK |
| user_id | uuid | no | FK users.id |
| reset_token_hash | varchar(255) | no | short-lived token hash |
| expires_at | timestamptz | no | |
| consumed_at | timestamptz | yes | |
| created_at | timestamptz | no | |


# 13. API Draft (Implementation-Oriented)

## 13.2 Public auth endpoints

### `POST /api/v1/auth/tutor/signup/initiate`

Creates or refreshes a tutor signup session and sends email OTP.

**Request**

```json
{
  "full_name": "Aman Sharma",
  "email": "aman@example.com",
  "phone": "+919999999999",
  "password": "StrongPass123",
  "subjects": ["physics", "math"],
  "classes": ["9", "10"],
  "terms_accepted": true
}
```

**Response 200**

```json
{
  "signup_session_id": "uuid",
  "email_otp_sent": true,
  "next_step": "VERIFY_EMAIL"
}
```

**Key failures**
- duplicate email
- duplicate phone
- invalid payload
- rate limited

### `POST /api/v1/auth/student/signup/initiate`

Same pattern as tutor signup, without subjects/classes.

### `POST /api/v1/auth/login`

**Request**

```json
{
  "email": "user@example.com",
  "password": "StrongPass123"
}
```

**Response 200**

```json
{
  "access_token": "jwt-or-session-token",
  "user": {
    "id": "uuid",
    "role": "TUTOR",
    "approval_status": "APPROVED",
    "full_name": "Aman Sharma"
  },
  "route_hint": "/tutor/dashboard"
}
```

**Special responses**
- recommended: `403 ACCOUNT_PENDING_APPROVAL`
- recommended: `403 ACCOUNT_REJECTED`

### `POST /api/v1/auth/forgot-password`

**Request**

```json
{
  "email": "user@example.com"
}
```

**Response 200**

```json
{
  "message": "If the account exists, a reset OTP has been sent."
}
```

### `POST /api/v1/auth/reset-password`

**Request**

```json
{
  "reset_token": "short-lived-token",
  "new_password": "NewStrongPass123"
}
```

**Response 200**

```json
{
  "message": "Password updated successfully"
}
```

## 13.3 Self-service endpoints

### `GET /api/v1/me`
Returns basic current user profile and approval state.

### `GET /api/v1/me/status`
Returns approval status, allowed routes, and key onboarding flags.


# 14. Validation Rules and Business Rules

## 14.1 Identity and credentials

- email normalized to lowercase before uniqueness checks
- phone normalized to E.164 before uniqueness checks
- password minimum length = 8
- recommended password policy: at least 1 uppercase, 1 lowercase, 1 digit

