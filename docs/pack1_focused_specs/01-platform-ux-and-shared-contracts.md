- This spec is derived from: pack1_final_implementation_details.md
- This spec owns: Platform UX and shared contracts
- This spec strictly preserves original implementation details without modification


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


# 9. Pack 1 High-Level Design (HLD)

## 9.1 Architecture modules

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


# 12. Database Schema Draft (Implementation-Oriented)

## 12.1 Conventions

- Primary keys: UUID
- Timestamps in UTC
- Soft delete not required in Pack 1 for core entities; use `is_active` where necessary
- All status values are enums in application layer; DB may use varchar + check or native enum
- Monetary values use `numeric(12,2)` unless otherwise noted
- Currency uses ISO-style string code, e.g. `INR`

## 12.2 Tables


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

