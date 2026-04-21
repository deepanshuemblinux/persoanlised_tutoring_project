- This spec is derived from: pack1_final_implementation_details.md
- This spec owns: Institute enrollment settings
- This spec strictly preserves original implementation details without modification
- This spec must be used with shared contract spec: docs/pack1_focused_specs/01-platform-ux-and-shared-contracts.md
- Fallback context source: docs/pack1_final_implementation_details.md

# 6. Screen-by-Screen UX Details

## 6.24 Institute Enrollment Settings

### Visual References:
- `ui_mock_screens/Institute_Enrollment_Settings.png`

**Purpose**
Allow admin to configure global registration fee settings.

**Fields**
- registration_fee_amount: decimal, required, >= 0
- registration_fee_currency: text/code, required
- registration_fee_reason: textarea, required
- registration_fee_instructions: rich text or textarea, required
- payment_link (hosted payment redirect target or placeholder gateway redirect URL): URL, required
- is_active: boolean, required

**Actions**
- Save
- Cancel

**Validation rules**
- amount must be non-negative
- payment_link must be valid URL
- reason required
- instructions required
- only one active settings row per institute

**Behavior**
- changes affect only future enrollments
- historical enrollments must keep snapshot values


# 9. Pack 1 High-Level Design (HLD)

## 9.1 Architecture modules

### Institute Enrollment Settings Module
Responsibilities:
- manage global registration fee settings
- return active settings for student enrollment screens
- preserve “active settings for future requests, snapshot for past requests” rule
- stores hosted payment redirect configuration for registration fee flow


# 12. Database Schema Draft (Implementation-Oriented)

### `institutes`

Used even in single-institute v1 so schema remains clean.

| Column | Type | Null | Notes |
|---|---|---:|---|
| id | uuid | no | PK |
| name | varchar(150) | no | institute name |
| code | varchar(50) | yes | optional unique short code |
| is_active | boolean | no | default true |
| created_at | timestamptz | no | |
| updated_at | timestamptz | no | |

**Indexes**
- unique(code) where not null

### `institute_enrollment_settings`

Stores the current global registration fee config for future enrollments.

| Column | Type | Null | Notes |
|---|---|---:|---|
| id | uuid | no | PK |
| institute_id | uuid | no | FK institutes.id |
| registration_fee_amount | numeric(12,2) | no | |
| registration_fee_currency | varchar(10) | no | e.g. INR |
| registration_fee_reason | text | no | |
| registration_fee_instructions | text | no | |
| payment_link | text | no | hosted payment destination / gateway redirect target placeholder |
| is_active | boolean | no | default true |
| updated_by_admin_user_id | uuid | no | FK users.id |
| created_at | timestamptz | no | |
| updated_at | timestamptz | no | |

**Constraints**
- one active settings row per institute

**Recommended DB enforcement**
- partial unique index on (institute_id) where is_active = true


# 13. API Draft (Implementation-Oriented)


### `GET /api/v1/admin/institute-enrollment-settings`

Returns current active settings for the institute.

### `POST /api/v1/admin/institute-enrollment-settings`

Creates or replaces active settings row.

**Request**

```json
{
  "registration_fee_amount": 200.0,
  "registration_fee_currency": "INR",
  "registration_fee_reason": "Token registration fee to confirm seriousness and reserve the slot.",
  "registration_fee_instructions": "Complete payment using the payment link after submitting the enrollment request.",
  "payment_link": "https://example.com/pay",
  "is_active": true
}
```

