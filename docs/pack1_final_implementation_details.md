
---
title: "Pack 1 - Onboarding & Access Control"
subtitle: "V7 - Execution Blueprint"
author: "Deepanshu Sharma"
---

# 1. Document Purpose

This document freezes and specifies **Pack 1 - Onboarding & Access Control** for the tutor-assisted learning platform at **execution-blueprint depth**. It is written to be:

- **developer-ready** for backend, frontend, QA, and DevOps
- **Codex-friendly** for implementation directly from contracts
- **human-readable** for review and alignment
- **internally reconciled** with all latest decisions

This pack covers:

- tutor signup, email verification, approval, login
- student signup, email verification, approval, login
- forgot password and reset password
- course enrollment request by students
- global registration fee visibility during enrollment
- registration fee snapshotting on enrollment creation
- student payment instructions page and external payment link
- enrollment approval by admin
- tutor-student-course mapping by admin
- access control states and routing behavior
- student payment screen for gateway-hosted registration fee payment
- reserved callback-compatible payment reference model for future payment gateway integration

Registration fee collection is designed to use only payment gateway hosted payment flows in future implementation. This pack does **not** implement the actual payment gateway integration. It prepares the data model and API contracts so that a later pack can add real payment orchestration without reworking enrollment fundamentals.

# 2. Frozen Scope and Decisions

## 2.1 Product context used by this pack

- Single institute in v1
- Multiple admins supported
- Tutors and students require admin approval before normal platform use
- Parent is **not** a separate user role; parent uses student credentials
- Login method is email + password
- Course is a unique combination of **education_board + class + subject**
- One tutor per student per subject
- Student can request enrollment only after account approval
- Registration fee is a **global token enrollment fee**, not the course fee
- Course pricing metadata includes a **per-hour rate** and currency
- Payment gateway integration is not implemented in Pack 1
- Registration fee payments are intended to be accepted only through payment gateway hosted payment
- Pack 1 uses a payment screen and hosted payment redirect placeholder aligned with future gateway integration

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

## 2.3 Approval state rules

### User approval states
- `PENDING`
- `APPROVED`
- `REJECTED`

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

## 2.6 Access behavior rules

- Users with `PENDING` status cannot access normal dashboards
- Users with `REJECTED` status cannot access normal dashboards
- Forgot password is available from both tutor login and student login
- Parent access is the same as student access because parent uses student credentials

# 3. Roles and Responsibilities in This Pack

## 3.1 Tutor

Can:
- sign up
- verify email
- log in after approval
- access tutor dashboard after approval
- use forgot password flow

Cannot:
- self-approve account
- map students
- approve enrollments
- configure registration fee settings

## 3.2 Student

Can:
- sign up
- verify email
- log in after approval
- request course enrollment after approval
- view enrollment status
- view registration fee details for each enrollment
- open payment instructions page
- use external payment link
- use forgot password flow

Cannot:
- bypass admin approval
- assign tutor
- approve own enrollment
- mark own fee status as paid manually
- manually mark registration fee as paid
- submit payment proof/receipt in Pack 1

## 3.3 Admin

Can:
- log in
- review and approve/reject tutors
- review and approve/reject students
- review and approve/reject enrollment requests
- map tutor to student enrollment
- remap tutor later while preserving history
- configure institute-level registration fee settings

Cannot:
- impersonate payment success through public APIs
- bypass audit logging for approval and mapping actions
- manually collect or confirm registration fee through normal Pack 1 UI flow
- override fee status through standard admin enrollment approval flow

## 3.4 System

Responsibilities in this pack:
- manage signup sessions
- issue and verify email OTPs
- create user records after successful verification
- enforce approval-based routing
- create enrollment requests
- copy registration fee snapshot into enrollment
- expose gateway-oriented payment instructions to the student
- generate and persist internal payment reference for future gateway correlation
- support future trusted payment callback flow to update fee status
- reject non-gateway payment state mutation through student-facing flows
- support later payment callback integration through reserved fields and contracts
- log admin actions

# 4. UI/UX Design Scope

This pack includes the following UX surfaces:

- authentication screens
- email OTP verification screens
- pending and rejected status screens
- admin review and approval screens
- student enrollment request screens
- student enrollment status screens
- student Pay Registration Fee screen that redirects to payment gateway hosted payment
- tutor-student mapping screens
- forgot password and password reset screens
- unauthorized and access denied screens

The design intent is:
- low-friction onboarding
- explicit state visibility
- deterministic admin action paths
- no ambiguous routing on pending/rejected users
- clear visibility of registration fee during enrollment
- explicit separation between:
  - enrollment request state
  - registration fee payment state

Out of scope in this pack:
- payment gateway checkout UI embedded inside app
- transaction history UI
- course billing / hourly usage billing
- invoice generation
- refunds
- social login / SSO
- phone OTP verification
- manual proof-of-payment submission
- manual payment confirmation by student
- in-app payment collection UI
- admin payment collection workflow

## Visual Reference Usage Rules

For UI implementation, developer must treat the referenced image files as the primary visual source of truth for layout and presentation, while this specification remains the source of truth for behavior, data, validations, states, and API interactions.

Rules:
- For every screen that includes a visual reference filename, developer must use that image as the target UI.
- developer must match the referenced screen’s layout, spacing, hierarchy, section grouping, alignment, CTA placement, and visible component structure as closely as practical.
- developer must not redesign, simplify, or restyle the screen unless required by technical constraints or explicitly instructed.
- If the visual mockup conflicts with this spec:
  - behavior, validation, permissions, and state rules come from this spec
  - visual layout and presentation come from the referenced image
- If a screen has multiple reference images, developer must use all of them together and infer the screen states from the filenames and surrounding section details.
- If any visual detail is ambiguous, developer must choose the closest practical implementation and list the assumption in implementation notes.
- developer must reuse project components/tokens where possible while preserving the visual intent of the mockup.
- developer must compare the implemented screen against the referenced image and iterate to reduce visible mismatch.

## State-Specific Mockup Rule

If separate mockups are provided for different UI states of the same screen, developer must implement all referenced states, including but not limited to:
- default
- loading
- success
- empty
- validation error
- warning
- approved/unapproved variants
- paid/unpaid variants

If only one mockup is provided, developer must still implement required states from the behavioral rules in this spec using the same design language.

## UI Implementation Instruction

When implementing screens from this document:
- use the image filenames listed under each screen as direct visual targets
- preserve the mockup’s visible layout and structure
- implement all fields, states, and actions defined in the screen spec
- do not invent alternate layouts
- do not omit visible sections present in the referenced mockup
- after implementation, compare the rendered screen to the referenced image and adjust until the result is visually close

# 5. Actor-Wise Screen Inventory

## 5.1 Tutor screens

1. Tutor Sign Up
2. Tutor Verify Email OTP
3. Tutor Login
4. Tutor Approval Pending
5. Tutor Dashboard Landing
6. Forgot Password
7. Verify Reset OTP
8. Reset Password
9. Account Rejected

## 5.2 Student screens

1. Student Sign Up
2. Student Verify Email OTP
3. Student Login
4. Student Approval Pending
5. Student Dashboard Landing
6. Request Course Enrollment
7. Enrollment Request Submitted
8. Enrollment Status View
9. Pay Registration Fee
10. Forgot Password
11. Verify Reset OTP
12. Reset Password
13. Account Rejected

## 5.3 Admin screens

1. Admin Login
2. Admin Dashboard
3. Pending Tutor Approvals
4. Tutor Review Detail
5. Pending Student Approvals
6. Student Review Detail
7. Enrollment Requests List
8. Enrollment Review Detail
9. Tutor-Student Mapping Screen
10. Institute Enrollment Settings
11. User Action History View

## 5.4 Shared/system screens

1. Unauthorized / Access Denied
2. Session Expired
3. Generic Error State

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

## 6.5 Tutor Dashboard Landing

### Visual References:
- `ui_mock_screens/Tutor_Dashboard_Landing.png`

**Pack 1 scope only**
- welcome header
- assigned students count
- active courses count
- empty-state cards when no assignments or mappings exist

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

## 6.7 Student Verify Email OTP

### Visual References:
- `ui_mock_screens/Student_Verify_Email_OTP.png`

Same interaction model as tutor email verification.

**Completion rule**
When email is verified, account is created in `PENDING` approval state.

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

## 6.9 Student Approval Pending

### Visual References:
- `ui_mock_screens/Student_Approval_Pending.png`

**Content**
- pending approval message
- note that enrollment is available after approval
- logout

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

## 6.15 Admin Login

### Visual References:
- `ui_mock_screens/Admin_Login.png`

**Fields**
- email
- password

**Actions**
- Login
- Forgot Password (optional in Pack 1; if not implemented, hide)

## 6.16 Admin Dashboard

### Visual References:
- `ui_mock_screens/Admin_Dashboard.png`

**Widgets**
- pending tutor approvals count
- pending student approvals count
- pending enrollment requests count
- unpaid enrollment count
- approved-but-unmapped enrollments count
- recent actions feed

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

## 6.23 Tutor-Student Mapping Screen

### Visual References:
- `ui_mock_screens/Tutor-Student_Mapping_Screen.png`

**Fields**
- student (read only)
- course (read only)
- tutor dropdown filtered by approved tutors teaching that subject and class

**Actions**
- Assign Tutor
- Replace Tutor
- Save

**Rules**
- one active tutor mapping per approved enrollment
- replacement closes previous mapping by setting inactive/end timestamp
- mapping history must be retained
- mapping allowed even if fee is unpaid, provided enrollment is approved

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

## 6.26 Verify Reset OTP

### Visual References:
- `ui_mock_screens/Verify_Reset_OTP.png`

**Fields**
- email
- otp_code

**Actions**
- Verify OTP
- Resend OTP

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

## 6.28 Account Rejected

### Visual References:
- `ui_mock_screens/Account_Rejected.png`

**Content**
- rejection message
- rejection reason if configured to show
- support/help text
- logout

## 6.29 Unauthorized / Access Denied

### Visual References:
- `ui_mock_screens/Unauthorized_-_Access_Denied.png`

Shown when:
- role mismatch
- inactive session
- protected route access without permission

## 6.30 Generic Errors / Session Expired

### Visual References:
- `ui_mock_screens/Session_Expired.png`
- `ui_mock_screens/Generic_Error_State.png`

Shown when:
- session expired
- generic errors

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

## 7.4 Admin approval flow

1. Admin logs in
2. Opens pending tutors or students or enrollments
3. Reviews detail page
4. Approves or rejects with optional note / required reason on rejection
5. Action recorded in audit log

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

## 7.6 Forgot password flow

1. User opens Forgot Password from login page
2. Enters email
3. OTP sent to verified email
4. User verifies reset OTP
5. User sets new password
6. Old password becomes invalid
7. User logs in with new password

## 7.7 Gateway payment completion flow

1. Student opens Pay Registration Fee screen
2. Student clicks Proceed to Payment Gateway
3. Hosted payment page is opened
4. After successful payment in future implementation, gateway sends trusted callback/webhook
5. System validates callback
6. System resolves enrollment by `payment_reference`
7. If valid and not already processed, fee status updates to `PAID`
8. Student dashboard and enrollment status view reflect paid status

# 8. UX Rules and Behavioral Rules

## 8.1 Verification and onboarding rules

- Tutor and student require email verification only
- User record must not become active before email is verified
- Pending approval screen must clearly explain next step
- Verification resend must respect cooldown and retry limits
- Phone must be collected but is informational only in Pack 1

## 8.2 Approval rules

- Only admin can approve or reject tutor and student accounts
- Rejection reason is stored even if not displayed to end user
- Rejected users cannot access normal dashboard routes

## 8.3 Enrollment rules

- Student can request enrollment only after account approval
- Student cannot request duplicate active enrollment for same course
- Student cannot create duplicate pending request for same course
- Enrollment requires separate admin approval from account approval
- Enrollment request creation requires active institute registration settings
- Enrollment snapshot must be immutable after creation
- Registration fee payment does not automatically approve enrollment
- Approval does not automatically mark fee as paid

## 8.4 Mapping rules

- Mapping is admin-only
- One active tutor per student per course
- Tutor must be approved and teach selected subject and class
- Mapping history must be preserved for auditability

## 8.5 Authentication rules

- Login uses email + password
- Password reset invalidates old password immediately
- Locked, pending, or rejected states affect login routing

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

### Auth Module
Responsibilities:
- signup initiation
- login
- logout
- forgot password
- reset password
- token issuance

### Verification Module
Responsibilities:
- generate OTPs
- send OTPs over email
- verify OTPs
- cooldown, expiry, retry, and lock policies

### User Management Module
Responsibilities:
- tutor profile creation
- student profile creation
- user approval status handling
- user activation state handling

### Course Catalog Module
Responsibilities:
- provide active board/class/subject combinations
- validate enrollment requests against active courses
- expose course pricing metadata (`per_hour_rate`, `currency`) for future packs

### Institute Enrollment Settings Module
Responsibilities:
- manage global registration fee settings
- return active settings for student enrollment screens
- preserve “active settings for future requests, snapshot for past requests” rule
- stores hosted payment redirect configuration for registration fee flow

### Enrollment Module
Responsibilities:
- create enrollment requests
- track approval status
- track fee status
- copy fee snapshot from current institute settings
- expose enrollment history and payment info
- generates gateway-correlatable internal `payment_reference`
- exposes unpaid enrollment payment initiation metadata

### Mapping Module
Responsibilities:
- assign tutor to approved enrollment
- replace tutor while preserving history
- query current mapping

### Admin Operations Module
Responsibilities:
- review queues
- approve/reject actions
- admin notes and audit logs
- unpaid approval warnings at UI/API level

### Access Control Module
Responsibilities:
- role-based route protection
- state-aware route protection
- dashboard routing based on role and approval status

### Future Payment Integration Module (out of scope for Pack 1)

Responsibilities:
- create payment gateway session/order
- redirect to hosted payment
- receive webhook/callback
- validate callback authenticity
- update enrollment fee status
- log payment events

## 9.2 Integration points

- Email provider for OTP sending
- PostgreSQL for persistent storage
- Session or JWT provider for auth token issuance
- Audit logging to DB tables
- Future payment gateway callback integration through reserved API and payment reference fields
- future payment gateway webhook/callback integration

## 9.3 Non-functional expectations for this pack

- secure credential handling
- deterministic state transitions
- idempotent approval endpoints where possible
- consistent API error model
- auditability of admin actions
- rate limiting on OTP and login attempts
- stable fee snapshot behavior under concurrent settings changes

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

## 10.5 Tutor mapping replacement sequence

1. Admin opens approved enrollment with current tutor mapping
2. Admin selects new tutor
3. Server validates tutor eligibility
4. Server marks previous mapping inactive with `active_to`
5. Server inserts new active mapping row
6. Action is logged in admin action log

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

## 10.7 Future payment callback-compatible fee update sequence

This flow is defined now so later payment integration has a stable contract.

1. Student completes payment on hosted payment page
2. Gateway sends server-to-server callback/webhook
3. System validates callback authenticity
4. System resolves enrollment by `payment_reference`
5. System checks whether fee is already marked `PAID`
6. If not already processed, fee status updates from `UNPAID` to `PAID`
7. Callback handling must be idempotent
8. Payment event metadata should be logged in future payment pack

**Explicit prohibitions:**
- Student UI must not directly update fee status
- Admin approval flow must not directly update fee status

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

## 11.4 Mapping state machine

```text
NO_MAPPING -> ACTIVE_MAPPING
ACTIVE_MAPPING -> REPLACED_MAPPING (historical row closed)
```

# 12. Database Schema Draft (Implementation-Oriented)

## 12.1 Conventions

- Primary keys: UUID
- Timestamps in UTC
- Soft delete not required in Pack 1 for core entities; use `is_active` where necessary
- All status values are enums in application layer; DB may use varchar + check or native enum
- Monetary values use `numeric(12,2)` unless otherwise noted
- Currency uses ISO-style string code, e.g. `INR`

## 12.2 Tables

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

### `admin_profiles`

| Column | Type | Null | Notes |
|---|---|---:|---|
| id | uuid | no | PK |
| user_id | uuid | no | FK users.id unique |
| created_at | timestamptz | no | |

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

### `student_tutor_mappings`

| Column | Type | Null | Notes |
|---|---|---:|---|
| id | uuid | no | PK |
| enrollment_id | uuid | no | FK enrollments.id |
| student_user_id | uuid | no | FK users.id |
| tutor_user_id | uuid | no | FK users.id |
| course_id | uuid | no | FK courses.id |
| is_active | boolean | no | default true |
| active_from | timestamptz | no | |
| active_to | timestamptz | yes | |
| created_by_admin_user_id | uuid | no | FK users.id |
| created_at | timestamptz | no | |

**Rules**
- one active mapping per enrollment

**Indexes**
- partial unique index on (enrollment_id) where is_active = true
- index(tutor_user_id, is_active)
- index(student_user_id, is_active)

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

### `admin_action_logs`

| Column | Type | Null | Notes |
|---|---|---:|---|
| id | uuid | no | PK |
| admin_user_id | uuid | no | FK users.id |
| entity_type | varchar(50) | no | USER / ENROLLMENT / MAPPING / SETTINGS |
| entity_id | uuid | no | |
| action | varchar(50) | no | APPROVE / REJECT / MAP / REMAP / UPDATE_SETTINGS |
| notes | text | yes | |
| metadata_json | jsonb | yes | diff or context |
| created_at | timestamptz | no | |

**Future table recommendation: `enrollment_payment_events` (out of scope for Pack 1)**

Potential fields:
- id
- enrollment_id
- payment_reference
- gateway_name
- gateway_payment_id
- gateway_order_id
- event_type
- raw_payload_json
- status
- received_at
- created_at

# 13. API Draft (Implementation-Oriented)

## 13.1 API conventions

- Base path example: `/api/v1`
- JSON request and response bodies
- Errors use common shape:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Phone number already exists",
    "details": {
      "field": "phone"
    }
  }
}
```

- Authenticated endpoints require bearer token
- Timestamps are ISO 8601 UTC strings

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

## 13.5 Admin approval endpoints

### `GET /api/v1/admin/tutors/pending`
### `GET /api/v1/admin/students/pending`
### `GET /api/v1/admin/enrollments/requested`

Paginated list endpoints.

### `GET /api/v1/admin/enrollments/{enrollment_id}`

Returns student, course, fee status, fee snapshot, mapping state, and admin notes context.

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

## 13.6 Mapping endpoints

### `POST /api/v1/admin/student-tutor-mappings`

**Request**

```json
{
  "enrollment_id": "uuid",
  "tutor_user_id": "uuid"
}
```

**Response 201**

```json
{
  "mapping_id": "uuid",
  "is_active": true
}
```

### `PUT /api/v1/admin/student-tutor-mappings/{mapping_id}`

Used for remapping. Can alternatively be modeled as `/remap` command endpoint.

## 13.7 Institute enrollment settings endpoints

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

## 13.8 Future payment callback placeholder endpoint

### `POST /api/v1/payments/callback`

Reserved for future payment integration.

**Request (placeholder)**

```json
{
  "payment_reference": "ENR-REF-001",
  "status": "SUCCESS",
  "gateway_event_id": "evt_123",
  "amount": 200.0,
  "currency": "INR"
}
```

**Pack 1 note**
- endpoint may exist but stay internal/disabled until payment integration pack
- if enabled in dev/testing, it should update fee status to `PAID`

**Future implementation requirements**
- callback must be authenticated and signature-verified
- callback amount/currency should match enrollment snapshot
- callback handling must be idempotent
- duplicate gateway events must not double-process payment

# 14. Validation Rules and Business Rules

## 14.1 Identity and credentials

- email normalized to lowercase before uniqueness checks
- phone normalized to E.164 before uniqueness checks
- password minimum length = 8
- recommended password policy: at least 1 uppercase, 1 lowercase, 1 digit

## 14.2 OTP rules

- OTP length = 6 digits
- OTP expiry = 5 minutes
- max verify attempts per OTP = 5
- resend cooldown = 30 seconds
- max resend count per session configurable, recommended 5
- expired or consumed OTP cannot be reused

## 14.3 Approval rules

- only admin users with same institute_id can approve/reject
- approving an already approved entity should be idempotent or return safe error
- rejecting an approved account should not be allowed without separate suspend/deactivate flow (out of scope for Pack 1)

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

## 14.6 Tutor mapping rules

- only approved tutor can be mapped
- tutor must have capability row for subject + class
- only approved enrollment can be mapped
- exactly one active mapping per enrollment

## 14.7 Gateway payment integrity rules

- Only trusted payment callback flow can mark fee status as `PAID`
- Student-facing endpoints cannot mutate fee status
- Admin enrollment approval endpoints cannot mutate fee status
- Future callback amount must match `registration_fee_amount_snapshot`
- Future callback currency must match `registration_fee_currency_snapshot`
- Callback processing must be idempotent

# 15. Error Model

Recommended API error codes:

- `VALIDATION_ERROR`
- `DUPLICATE_EMAIL`
- `DUPLICATE_PHONE`
- `OTP_INVALID`
- `OTP_EXPIRED`
- `OTP_ATTEMPT_LIMIT_EXCEEDED`
- `OTP_RESEND_LIMIT_EXCEEDED`
- `ACCOUNT_PENDING_APPROVAL`
- `ACCOUNT_REJECTED`
- `UNAUTHORIZED`
- `FORBIDDEN`
- `COURSE_NOT_FOUND`
- `DUPLICATE_ENROLLMENT_REQUEST`
- `DUPLICATE_APPROVED_ENROLLMENT`
- `REGISTRATION_SETTINGS_NOT_CONFIGURED`
- `UNPAID_ENROLLMENT_APPROVAL_REQUIRES_CONFIRMATION`
- `TUTOR_NOT_ELIGIBLE`
- `MAPPING_ALREADY_EXISTS`
- `PAYMENT_REFERENCE_NOT_FOUND`
- `RESOURCE_NOT_FOUND`
- `PAYMENT_GATEWAY_CALLBACK_INVALID`
- `PAYMENT_AMOUNT_MISMATCH`
- `PAYMENT_CURRENCY_MISMATCH`
- `PAYMENT_ALREADY_PROCESSED`

# 16. Security, Audit, and Operational Notes

## 16.1 Security

- store password hashes using bcrypt or argon2
- never store plain OTP
- rate limit login and OTP endpoints
- return generic response for forgot password to avoid user enumeration
- secure JWT/session cookies depending on auth strategy
- enforce HTTPS in all environments except isolated local dev
- future payment gateway callbacks must be authenticated and signature verified
- payment references should not be trivially guessable if exposed externally

## 16.2 Audit

Admin actions to log:
- approve tutor
- reject tutor
- approve student
- reject student
- approve enrollment
- reject enrollment
- create mapping
- replace mapping
- update institute enrollment settings
- fee status transition to `PAID` should be logged once payment integration is enabled

## 16.3 Observability

Track metrics such as:
- signup initiation count by role
- OTP success/failure rate
- approval turnaround time
- enrollment request volume
- unpaid enrollment count
- approved-but-unpaid enrollment count
- mapping completion time
- password reset success rate
- unpaid-to-paid conversion rate
- payment CTA click-through rate
- callback success/failure rate
- duplicate callback rate

# 17. Test Cases

## 17.1 Functional test cases

1. Tutor signup initiates successfully with valid payload
2. Tutor email OTP verification succeeds with valid code
3. Tutor account is created only after email verification passes
4. Admin can approve tutor account
5. Approved tutor can log in successfully
6. Tutor login routes to dashboard
7. Student signup initiates successfully with valid payload
8. Student email OTP verification succeeds with valid code
9. Student account is created only after email verification passes
10. Admin can approve student account
11. Approved student can log in successfully
12. Approved student can load enrollment form data including current registration fee settings
13. Approved student can request valid enrollment
14. Enrollment is created with approval status `REQUESTED`
15. Enrollment is created with fee status `UNPAID`
16. Enrollment stores fee snapshot fields from current institute settings
17. Enrollment stores unique payment reference
18. Student can view payment instructions page
19. Admin can approve enrollment request with fee already `PAID`
20. Admin can approve unpaid enrollment when explicit approval flag is provided
21. Approved student dashboard shows “Enrollment approved. Registration fee pending.” when fee remains unpaid
22. Admin can assign eligible tutor to approved enrollment
23. Student enrollment status screen shows assigned tutor after mapping
24. Tutor sees linked student relationship after mapping
25. Forgot password sends reset OTP
26. Verify reset OTP returns reset token
27. Reset password updates stored password hash
28. Old password fails after reset
29. New password succeeds after reset
30. Replacing tutor mapping closes previous mapping and creates new active mapping
31. Admin action log is created for approve/reject/map/settings actions
32. Future payment callback-compatible handler updates fee status from `UNPAID` to `PAID` in test mode if endpoint is enabled
33. Student sees Pay Registration Fee to Complete Enrollment immediately after enrollment request
34. Approved-but-unpaid enrollment continues showing payment prompt
35. Fee status changes to `PAID` only after trusted callback flow in test mode

## 17.2 Negative and edge test cases

1. Duplicate tutor email is rejected during signup initiation
2. Duplicate tutor phone is rejected during signup initiation
3. Duplicate student email is rejected during signup initiation
4. Duplicate student phone is rejected during signup initiation
5. Invalid email OTP returns `OTP_INVALID`
6. Expired OTP returns `OTP_EXPIRED`
7. OTP verify attempts over limit are blocked
8. OTP resend over limit is blocked
9. Pending tutor cannot access tutor dashboard APIs
10. Pending student cannot access student dashboard APIs
11. Rejected user cannot access normal dashboard APIs
12. Login with wrong password fails
13. Student cannot request enrollment before account approval
14. Student cannot request enrollment for nonexistent course combination
15. Student cannot create duplicate `REQUESTED` enrollment for same course
16. Student cannot create duplicate approved enrollment for same course
17. Student cannot create enrollment when institute registration settings are missing or inactive
18. Admin cannot approve enrollment from another institute
19. Admin approve endpoint for unpaid enrollment without explicit confirmation flag returns guard error if that mode is enabled
20. Admin cannot map tutor to rejected enrollment
21. Admin cannot map ineligible tutor for course subject/class
22. Admin cannot create second active mapping for same enrollment
23. Reset password with invalid token fails
24. Reset password with expired token fails
25. Forgot password does not leak whether account exists
26. Disabled course cannot be requested
27. Payment reference collision is rejected
28. Snapshot fields cannot be mutated through enrollment update flow
29. Historical enrollment snapshot remains unchanged after global settings update
30. Cross-institute admin access must be forbidden, even if future v1.1 adds more institutes
31. Student cannot manually mark fee as paid
32. Admin approval does not mutate fee status
33. Invalid callback signature rejected
34. Duplicate callback does not double-process fee status update
35. Callback amount mismatch rejected
36. Callback currency mismatch rejected

# 18. Implementation Notes for Codex / Developer Workflow

## 18.1 Recommended implementation order

1. Create DB migrations for core tables
2. Implement enum/constants package for states and roles
3. Implement signup session + email OTP subsystem
4. Implement tutor/student signup flows
5. Implement login and state-aware routing behavior
6. Implement admin approval APIs
7. Implement institute enrollment settings APIs
8. Implement course catalog read APIs
9. Implement student enrollment APIs with fee snapshot logic
10. Implement student payment instructions APIs/views
11. Define future payment gateway integration seam clearly, even if not implemented yet
12. Implement tutor mapping APIs
13. Implement forgot password flow
14. Add audit logging and metrics
15. Add integration tests and E2E tests

**Note:** Avoid building any manual payment-confirmation shortcut in Pack 1 because it will conflict with the gateway-only future design.

## 18.2 Definition of done for Pack 1

Pack 1 is complete when:
- tutor and student can sign up with email verification
- admin can approve/reject both roles
- approved users can log in
- approved student can request course enrollment
- enrollment stores immutable registration fee snapshot
- student can see payment instructions and external link
- admin can approve/reject enrollment
- admin can approve unpaid enrollment with warning/explicit confirmation
- admin can map eligible tutor to approved enrollment
- forgot password works end-to-end
- all core functional and negative tests pass
- audit logs exist for admin actions
- API contracts match this document

# 19. Out of Scope for Pack 1

- assignment creation
- rubric generation
- knowledge base workflows
- student submissions
- evaluation flows
- targeted assignment suggestions
- parent-specific separate account system
- suspension/deactivation lifecycle beyond approve/reject
- advanced auth methods such as social login or SSO
- phone OTP verification
- actual payment gateway integration
- course fee billing and hourly billing collection
- refunds and payment reconciliation UI
- manual payment confirmation flow
- proof-of-payment upload flow
- admin-side registration fee collection flow
- embedded checkout flow inside app
- payment reconciliation tooling
