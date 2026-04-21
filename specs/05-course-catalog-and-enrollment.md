- This spec is derived from: pack1_final_implementation_details.md
- This spec owns: Course catalog and enrollment
- This spec strictly preserves original implementation details without modification
- This spec must be used with shared contract spec: docs/pack1_focused_specs/01-platform-ux-and-shared-contracts.md
- Fallback context source: docs/pack1_final_implementation_details.md

# 2. Frozen Scope and Decisions

### Enrollment approval states
- `REQUESTED`
- `APPROVED`
- `REJECTED`

### Enrollment fee states
- `UNPAID`
- `PAID`

## 2.4 Registration fee rules

- Registration fee is **global at institute level**
- Registration fee is decided by **Admin**
- Registration fee purpose is:
  - slot booking
  - seriousness / commitment check
- Registration fee is **not** the course fee
- Registration fee reason and instructions are also global
- On enrollment creation, the system must copy a **snapshot** of:
  - registration fee amount
  - registration fee currency
  - registration fee reason
  - registration fee instructions
  - payment link
- Enrollment snapshot values are immutable after creation
- If global registration fee settings change later, existing enrollments keep their original snapshot

## 2.5 Enrollment payment behavior rules

- Student can create enrollment even before paying registration fee
- Enrollment is created with:
  - approval status = `REQUESTED`
  - fee status = `UNPAID`
- Student must see:
  - exact registration fee amount
  - reason for fee
  - instructions for payment
  - external payment link/button
- Admin can approve enrollment even when fee status is `UNPAID`
- If admin approves while fee is unpaid, UI must show warning + confirmation modal
- If payment happens after approval:
  - enrollment remains `APPROVED`
  - fee status updates from `UNPAID` to `PAID`
  - no second admin approval is required
- Student dashboard must continue showing:
  - **Enrollment approved. Registration fee pending.**
  until fee status becomes `PAID`
- The student payment path must ultimately redirect to payment gateway hosted payment
- No manual self-declaration of payment is allowed
- No receipt upload flow is allowed
- No normal admin-side manual “mark as paid” flow is part of Pack 1
- Fee status should move to PAID only from trusted system-side payment confirmation flow in future implementation


# 6. Screen-by-Screen UX Details

## 6.10 Student Dashboard Landing

### Visual References:
- `ui_mock_screens/Student_Dashboard_Landing.png`

**Pack 1 scope only**
- profile summary
- approved enrollments list
- pending enrollment requests list
- CTA: Request Enrollment
- fee payment prompts for unpaid enrollments
- no assignment details in Pack 1

**Unpaid enrollment cards must show**
- board / class / subject
- approval status
- fee status
- registration fee amount
- CTA: Pay Registration Fee to Complete Enrollment

**Approved but unpaid card message**
- **Enrollment approved. Registration fee pending.**

## 6.11 Request Course Enrollment

### Visual References:
- `ui_mock_screens/Request_Course_Enrollment.png`

**Purpose**
Allow approved student to request enrollment into a board + class + subject course.

**Fields**
- `education_board_id`: dropdown, required
- `class_id`: dropdown, required
- `subject_id`: dropdown, required

**Read-only / computed display before submit**
- registration_fee_amount (global)
- registration_fee_currency
- registration_fee_reason
- registration_fee_instructions

**Actions**
- Submit Request
- Cancel

**Validation rules**
- board/class/subject must form an existing active course
- duplicate requested enrollment not allowed
- duplicate approved enrollment not allowed
- institute registration fee settings must exist and be active
- student must be `APPROVED`

**Submission result**
On submit:
- create enrollment with:
  - approval status = `REQUESTED`
  - fee status = `UNPAID`
- copy registration fee snapshot into enrollment
- generate internal `payment_reference`
- route to Enrollment Request Submitted
After submit, the student must be able to continue to:
- Enrollment Request Submitted
- then Pay Registration Fee to Complete Enrollment

**NOTE** The payment shown here is for gateway-hosted registration fee payment only

## 6.12 Enrollment Request Submitted

### Visual References:
- `ui_mock_screens/Enrollment_Request_Submitted.png`

**Content**
- success message
- requested course summary
- approval status badge = `REQUESTED`
- fee status badge = `UNPAID`
- registration fee amount
- registration fee reason
- primary CTA:
  - Pay Registration Fee to Complete Enrollment
- helper text:
  - “Your enrollment request has been submitted. Admin may still approve it before payment is completed.”
- link to Enrollment Status View

## 6.13 Enrollment Status View

### Visual References:
- `ui_mock_screens/Enrollment_Status_View.png`

**Displays for each course request**
- board
- class
- subject
- approval status
- fee status
- assigned tutor if approved and mapped
- rejection reason if rejected
- request date
- review date if reviewed
- registration fee amount snapshot
- registration fee reason snapshot
- registration fee instructions snapshot

**Conditional UI**
- if fee status = `UNPAID`, show:
  - CTA: Pay Registration Fee
- if approval status = `APPROVED` and fee status = `UNPAID`, show:
  - **Enrollment approved. Registration fee pending.**
  - **Pay Registration Fee**
- if fee status = `PAID`, suppress fee payment CTA

**NOTE**
- Do not show:
  - upload receipt
  - mark paid
  - notify admin of payment

## 6.14 Pay Registration Fee

### Visual References:
- `ui_mock_screens/Pay_Registration_Fee.png`
- `ui_mock_screens/Registration_Fee_Payment_Instructions.png`

**Purpose**
Provide a dedicated page that takes the student to payment gateway hosted payment for registration fee.

**Displays**
- enrollment summary
- registration fee amount snapshot
- registration fee reason snapshot
- registration fee instructions snapshot
- gateway payment CTA
- payment reference (optional to show, useful for support)
- note that payment confirmation may take time once gateway integration exists

**Actions**
- Open Payment Link
- Back to Enrollment Status

**CTA**
- visible button text: Pay Registration Fee
- helper text under button:
  - “You will be redirected to our secure payment partner page.”

**Must Not Contain**
- I have paid
- Upload receipt
- Mark as paid
- Notify admin after payment

**Behavior**
- page is view-only
- no manual “mark paid” action
- no self-attestation that payment is complete

## 6.21 Enrollment Requests List

### Visual References:
- `ui_mock_screens/Enrollment_Requests_List.png`

**Table columns**
- student_name
- board
- class
- subject
- requested_at
- approval_status
- fee_status
- registration_fee_amount
- tutor_assigned
- actions

**Actions**
- View
- Approve
- Reject
- Open Mapping

**Filters**
- approval status
- fee status
- board
- class
- subject

**Optional admin display note**
- A small indicator may be shown if useful: payment expected via gateway-hosted flow

## 6.22 Enrollment Review Detail

### Visual References:
- `ui_mock_screens/Enrollment_Review_Detail.png`

**Sections**
- student info
- course info
- prior enrollment history
- mapping status
- registration fee snapshot
- admin notes

**Displays**
- approval status
- fee status
- payment reference
- registration fee amount snapshot
- registration fee reason snapshot
- registration fee instructions snapshot
- assigned tutor if mapped

**Actions**
- Approve
- Reject with reason
- Map Tutor (only after approval)

**Notes**
- Admin approval and registration payment are independent
- Payment is expected through gateway-hosted payment flow only
- Approving enrollment does not imply fee receipt

**Special behavior**
If fee status is `UNPAID` and admin clicks Approve:
- show warning modal:
  - “Registration fee is unpaid. Do you still want to approve this enrollment?”
- allow admin to confirm and continue


# 7. Primary Pack 1 User Flows

## 7.3 Student enrollment flow

1. Approved student opens Request Enrollment
2. Client loads active boards, classes, subjects or active courses
3. Client also loads current institute registration fee settings summary
4. Student selects board, class, subject
5. Student sees registration fee amount, reason, and instructions before submit
6. Student submits request
7. Server validates:
   - student is approved
   - course exists and is active
   - no duplicate requested enrollment
   - no duplicate approved enrollment
   - active registration fee settings exist
8. Server creates enrollment with:
   - approval status = `REQUESTED`
   - fee status = `UNPAID`
   - copied fee snapshot
   - internal payment reference
9. Student is routed to Enrollment Request Submitted
10. Student sees Pay Registration Fee to Complete Enrollment CTA
11. Student opens Pay Registration Fee screen
12. Student clicks CTA and is redirected to payment gateway hosted payment
13. Admin may review and approve/reject enrollment regardless of payment completion
14. If admin approves while fee status is `UNPAID`, warning + confirmation modal is shown
15. If approved, admin can map tutor
16. If payment later succeeds through trusted gateway confirmation, fee status updates to `PAID`
17. Student sees updated fee state on dashboard and status page

## 7.5 Enrollment approval with unpaid fee flow

1. Admin opens Enrollment Review Detail
2. Sees approval status = `REQUESTED`
3. Sees fee status = `UNPAID`
4. Clicks Approve
5. UI shows modal:
   - “Registration fee is unpaid. Do you still want to approve this enrollment?”
6. Admin confirms
7. Server updates approval status to `APPROVED`
8. Enrollment remains fee status = `UNPAID`
9. Student dashboard shows:
   - **Enrollment approved. Registration fee pending.**

This approval path remains valid even when gateway payment has not yet been completed.


# 8. UX Rules and Behavioral Rules

## 8.3 Enrollment rules

- Student can request enrollment only after account approval
- Student cannot request duplicate active enrollment for same course
- Student cannot create duplicate pending request for same course
- Enrollment requires separate admin approval from account approval
- Enrollment request creation requires active institute registration settings
- Enrollment snapshot must be immutable after creation
- Registration fee payment does not automatically approve enrollment
- Approval does not automatically mark fee as paid

## 8.6 Registration fee display rules

- Registration fee amount, reason, and instructions must be visible before enrollment submission
- Student dashboard must continue showing payment CTA while fee is unpaid
- Once fee becomes paid, CTA must be removed or replaced with paid indicator
- Approved-but-unpaid enrollments must show explicit pending fee message

## 8.7 Parent rule

- Parent is not modeled as a distinct role in v1
- Any parent usage is indistinguishable from student usage because the same credentials are used

## 8.8 Registration payment acceptance rules

- Registration fee payment must be accepted only through payment gateway hosted payment
- Student-facing UI must only initiate or redirect to gateway-hosted payment
- Student UI must not provide manual payment declaration
- Student UI must not provide proof upload for registration fee in Pack 1
- Admin approval must not automatically mutate fee status
- Fee status change to `PAID` must happen only from trusted payment confirmation flow in future implementation


# 9. Pack 1 High-Level Design (HLD)

## 9.1 Architecture modules

### Course Catalog Module
Responsibilities:
- provide active board/class/subject combinations
- validate enrollment requests against active courses
- expose course pricing metadata (`per_hour_rate`, `currency`) for future packs

### Enrollment Module
Responsibilities:
- create enrollment requests
- track approval status
- track fee status
- copy fee snapshot from current institute settings
- expose enrollment history and payment info
- generates gateway-correlatable internal `payment_reference`
- exposes unpaid enrollment payment initiation metadata


# 10. Pack 1 Low-Level Design (LLD)

## 10.3 Enrollment request sequence

1. Approved student opens enrollment form
2. Client loads active boards, classes, subjects or active courses
3. Client loads active institute registration settings summary
4. Student submits selected combination
5. Server resolves combination to an active `courses` row
6. Server fetches current active `institute_enrollment_settings` row
7. Server checks:
   - student exists and is approved
   - no existing `REQUESTED` enrollment for same course
   - no active approved enrollment for same course
   - active registration fee settings exist
8. Server creates internal `payment_reference`
9. Server inserts `enrollments` row with:
   - status = `REQUESTED`
   - fee_status = `UNPAID`
   - copied registration fee snapshot fields
   - payment_reference
10. Admin queue updates
11. Student sees payment CTA
    - Future implementation will redirect student to gateway-hosted payment using enrollment-linked reference/session logic

## 10.4 Enrollment approval sequence

1. Admin opens requested enrollment
2. Server returns enrollment details including fee snapshot and fee status
3. Admin clicks Approve
4. If fee status = `UNPAID`, UI shows warning modal
5. After confirmation, client calls approve endpoint
6. Server validates:
   - enrollment exists
   - current approval status = `REQUESTED`
   - admin belongs to same institute
7. Server updates approval status to `APPROVED`
8. Server writes audit log row
9. Enrollment becomes eligible for tutor mapping


# 11. State Models

## 11.2 Enrollment approval state machine

```text
REQUESTED -> APPROVED
REQUESTED -> REJECTED
```

Invalid transitions:
- `APPROVED -> REQUESTED`
- `REJECTED -> REQUESTED`
- `REJECTED -> APPROVED` in Pack 1 without explicit reopen flow

## 11.3 Enrollment fee state machine

```text
UNPAID -> PAID
```

Invalid transitions:
- `PAID -> UNPAID` in Pack 1
- direct fee state change without payment callback/admin migration tooling

**Note:** `UNPAID -> PAID` transition is allowed only from trusted payment gateway confirmation flow


# 12. Database Schema Draft (Implementation-Oriented)

### `education_boards`

| Column | Type | Null | Notes |
|---|---|---:|---|
| id | uuid | no | PK |
| institute_id | uuid | no | FK institutes.id |
| name | varchar(100) | no | |
| is_active | boolean | no | default true |
| created_at | timestamptz | no | |

**Constraints**
- unique(institute_id, name)

### `classes`

| Column | Type | Null | Notes |
|---|---|---:|---|
| id | uuid | no | PK |
| institute_id | uuid | no | FK institutes.id |
| name | varchar(50) | no | e.g. Class 8 |
| sort_order | int | yes | |
| is_active | boolean | no | default true |
| created_at | timestamptz | no | |

**Constraints**
- unique(institute_id, name)

### `subjects`

| Column | Type | Null | Notes |
|---|---|---:|---|
| id | uuid | no | PK |
| institute_id | uuid | no | FK institutes.id |
| name | varchar(100) | no | |
| is_active | boolean | no | default true |
| created_at | timestamptz | no | |

**Constraints**
- unique(institute_id, name)

### `tutor_subject_classes`

Maps tutor teaching capability.

| Column | Type | Null | Notes |
|---|---|---:|---|
| id | uuid | no | PK |
| tutor_user_id | uuid | no | FK users.id |
| subject_id | uuid | no | FK subjects.id |
| class_id | uuid | no | FK classes.id |
| created_at | timestamptz | no | |

**Constraints**
- unique(tutor_user_id, subject_id, class_id)

**Indexes**
- index(subject_id, class_id)

### `courses`

Represents unique board + class + subject combination.

| Column | Type | Null | Notes |
|---|---|---:|---|
| id | uuid | no | PK |
| institute_id | uuid | no | FK institutes.id |
| education_board_id | uuid | no | FK education_boards.id |
| class_id | uuid | no | FK classes.id |
| subject_id | uuid | no | FK subjects.id |
| per_hour_rate | numeric(12,2) | yes | future course billing metadata |
| currency | varchar(10) | yes | e.g. INR |
| is_active | boolean | no | default true |
| created_at | timestamptz | no | |
| updated_at | timestamptz | no | |

**Constraints**
- unique(institute_id, education_board_id, class_id, subject_id)

### `enrollments`

| Column | Type | Null | Notes |
|---|---|---:|---|
| id | uuid | no | PK |
| student_user_id | uuid | no | FK users.id |
| course_id | uuid | no | FK courses.id |
| status | varchar(20) | no | REQUESTED / APPROVED / REJECTED |
| fee_status | varchar(20) | no | UNPAID / PAID |
| requested_at | timestamptz | no | |
| reviewed_by_admin_user_id | uuid | yes | FK users.id |
| reviewed_at | timestamptz | yes | |
| rejection_reason | text | yes | |
| registration_fee_amount_snapshot | numeric(12,2) | no | immutable snapshot |
| registration_fee_currency_snapshot | varchar(10) | no | immutable snapshot |
| registration_fee_reason_snapshot | text | no | immutable snapshot |
| registration_fee_instructions_snapshot | text | no | immutable snapshot |
| payment_link_snapshot | text | no | immutable snapshot |
| payment_reference | varchar(100) | no | unique internal reference; generated specifically for gateway correlation; must remain unique; safe to pass to gateway session creation flow later |
| created_at | timestamptz | no | |
| updated_at | timestamptz | no | |

**Constraints**
- check(status in ('REQUESTED','APPROVED','REJECTED'))
- check(fee_status in ('UNPAID','PAID'))
- unique(payment_reference)

**Important uniqueness rule**
Use partial unique indexes:
- one `REQUESTED` enrollment per student + course
- one `APPROVED` enrollment per student + course

**Indexes**
- index(student_user_id, status)
- index(student_user_id, fee_status)
- index(course_id, status)
- index(payment_reference)


# 13. API Draft (Implementation-Oriented)

## 13.4 Student enrollment endpoints

### `GET /api/v1/student/enrollment-form-data`

Returns active boards, classes, subjects or equivalent selection data plus current active registration fee settings summary.

**Response 200**

```json
{
  "boards": [],
  "classes": [],
  "subjects": [],
  "registration_fee": {
    "amount": 200.0,
    "currency": "INR",
    "reason": "Token registration fee to confirm seriousness and reserve the slot.",
    "instructions": "Complete payment using the external payment link after submitting the request."
  }
}
```

### `POST /api/v1/student/enrollments`

**Request**

```json
{
  "education_board_id": "uuid",
  "class_id": "uuid",
  "subject_id": "uuid"
}
```

**Behavior**
Server resolves this to `course_id` internally.

**Response 201**

```json
{
  "enrollment_id": "uuid",
  "status": "REQUESTED",
  "fee_status": "UNPAID",
  "payment_reference": "ENR-REF-001",
  "registration_fee": {
    "amount": 200.0,
    "currency": "INR",
    "reason": "Token registration fee to confirm seriousness and reserve the slot.",
    "instructions": "Complete payment using the external payment link after submitting the request.",
    "payment_link": "https://example.com/pay"
  }
}
```

### `GET /api/v1/student/enrollments`

Returns current student enrollment history and statuses.

**Response shape**
Each enrollment should include:
- board
- class
- subject
- approval status
- fee status
- requested_at
- reviewed_at
- rejection_reason
- tutor summary if mapped
- fee snapshot block

### `GET /api/v1/student/enrollments/{enrollment_id}/payment-instructions`

**Note**
- response is intended only for gateway-hosted payment initiation/redirection
- no client-side status mutation allowed from this endpoint

**Response 200**

```json
{
  "enrollment_id": "uuid",
  "payment_reference": "ENR-REF-001",
  "registration_fee": {
    "amount": 200.0,
    "currency": "INR",
    "reason": "Token registration fee to confirm seriousness and reserve the slot.",
    "instructions": "Complete payment using the external payment link after submitting the request.",
    "payment_link": "https://example.com/pay"
  }
}
```

**Future endpoint: `POST /api/v1/student/enrollments/{enrollment_id}/payment-session`**

Purpose: create or fetch a payment gateway hosted session/order for registration fee.

Pack 1 note: not implemented in Pack 1; future payment integration pack will own this endpoint.

### `GET /api/v1/admin/enrollments/{enrollment_id}`

Returns student, course, fee status, fee snapshot, mapping state, and admin notes context.

### `POST /api/v1/admin/enrollments/{enrollment_id}/approve`

**Request**

```json
{
  "approve_even_if_unpaid": true
}
```

**Behavior**
- if fee status = `UNPAID` and flag missing/false, server may return a guard error to force explicit caller intent
- if fee status = `PAID`, flag may be ignored

### `POST /api/v1/admin/enrollments/{enrollment_id}/reject`

```json
{
  "reason": "No slots available"
}
```


# 14. Validation Rules and Business Rules

## 14.4 Enrollment rules

- only approved students can create enrollment requests
- selected board/class/subject must correspond to active course
- duplicate requested enrollment blocked
- duplicate approved enrollment blocked
- enrollment creation requires active institute enrollment settings
- snapshot fields must be copied atomically with enrollment creation
- snapshot fields must not be mutated by later settings changes

## 14.5 Registration fee rules

- fee status defaults to `UNPAID` on enrollment creation
- only trusted system flow / future payment callback should move fee status to `PAID`
- students cannot manually set fee status
- payment_reference must be unique and generated server-side

