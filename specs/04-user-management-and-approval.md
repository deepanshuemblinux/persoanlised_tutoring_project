- This spec is derived from: pack1_final_implementation_details.md
- This spec owns: User management and approval
- This spec strictly preserves original implementation details without modification
- This spec must be used with shared contract spec: docs/pack1_focused_specs/01-platform-ux-and-shared-contracts.md
- Fallback context source: docs/pack1_final_implementation_details.md

# 2. Frozen Scope and Decisions

## 2.3 Approval state rules

### User approval states
- `PENDING`
- `APPROVED`
- `REJECTED`


# 6. Screen-by-Screen UX Details

## 6.4 Tutor Approval Pending

### Visual References:
- `ui_mock_screens/Tutor_Approval_Pending.png`

**Content**
- pending approval message
- last updated timestamp if available
- support/help text
- logout action

**Behavior**
- page should be routable directly for pending tutors
- if approval state changes to approved, next login must route to dashboard

## 6.9 Student Approval Pending

### Visual References:
- `ui_mock_screens/Student_Approval_Pending.png`

**Content**
- pending approval message
- note that enrollment is available after approval
- logout

## 6.17 Pending Tutor Approvals

### Visual References:
- `ui_mock_screens/Pending_Tutor_Approvals.png`

**Table columns**
- name
- email
- email_verified
- phone
- subjects
- classes
- submitted_at
- status
- actions

**Actions**
- View
- Approve
- Reject

## 6.18 Tutor Review Detail

### Visual References:
- `ui_mock_screens/Tutor_Review_Detail.png`

**Sections**
- identity details
- verification status
- teaching metadata
- admin notes
- action history

**Displays**
- email verified status
- phone number
- no phone verification status

**Actions**
- Approve
- Reject with reason

## 6.19 Pending Student Approvals

### Visual References:
- `ui_mock_screens/Pending_Student_Approvals.png`

**Table columns**
- name
- email
- email_verified
- phone
- submitted_at
- status
- actions

## 6.20 Student Review Detail

### Visual References:
- `ui_mock_screens/Student_Review_Detail.png`

**Sections**
- identity details
- verification status
- admin notes
- action history

**Displays**
- email verified status
- phone number
- no phone verification status

**Actions**
- Approve
- Reject with reason

## 6.28 Account Rejected

### Visual References:
- `ui_mock_screens/Account_Rejected.png`

**Content**
- rejection message
- rejection reason if configured to show
- support/help text
- logout


# 8. UX Rules and Behavioral Rules

## 8.2 Approval rules

- Only admin can approve or reject tutor and student accounts
- Rejection reason is stored even if not displayed to end user
- Rejected users cannot access normal dashboard routes


# 9. Pack 1 High-Level Design (HLD)

## 9.1 Architecture modules

### User Management Module
Responsibilities:
- tutor profile creation
- student profile creation
- user approval status handling
- user activation state handling


# 11. State Models

## 11.1 User lifecycle state machine

```text
SIGNUP_DRAFT
  -> EMAIL_VERIFIED
  -> PENDING
  -> APPROVED
  -> REJECTED
```

Practical persistent state on `users` starts at `PENDING` after email verification completes.


# 12. Database Schema Draft (Implementation-Oriented)

### `tutor_profiles`

| Column | Type | Null | Notes |
|---|---|---:|---|
| id | uuid | no | PK |
| user_id | uuid | no | FK users.id unique |
| created_at | timestamptz | no | |
| updated_at | timestamptz | no | |

### `student_profiles`

| Column | Type | Null | Notes |
|---|---|---:|---|
| id | uuid | no | PK |
| user_id | uuid | no | FK users.id unique |
| created_at | timestamptz | no | |
| updated_at | timestamptz | no | |


# 13. API Draft (Implementation-Oriented)

### `POST /api/v1/admin/tutors/{user_id}/approve`
### `POST /api/v1/admin/tutors/{user_id}/reject`
### `POST /api/v1/admin/students/{user_id}/approve`
### `POST /api/v1/admin/students/{user_id}/reject`

Reject request body example:

```json
{
  "reason": "Incomplete details"
}
```


# 14. Validation Rules and Business Rules

## 14.3 Approval rules

- only admin users with same institute_id can approve/reject
- approving an already approved entity should be idempotent or return safe error
- rejecting an approved account should not be allowed without separate suspend/deactivate flow (out of scope for Pack 1)
