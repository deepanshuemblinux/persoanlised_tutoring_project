- This spec is derived from: pack1_final_implementation_details.md
- This spec owns: Access control and error handling
- This spec strictly preserves original implementation details without modification
- This spec must be used with shared contract spec: docs/pack1_focused_specs/01-platform-ux-and-shared-contracts.md
- Fallback context source: docs/pack1_final_implementation_details.md

# 2. Frozen Scope and Decisions

## 2.6 Access behavior rules

- Users with `PENDING` status cannot access normal dashboards
- Users with `REJECTED` status cannot access normal dashboards
- Forgot password is available from both tutor login and student login
- Parent access is the same as student access because parent uses student credentials


# 6. Screen-by-Screen UX Details

## 6.5 Tutor Dashboard Landing

### Visual References:
- `ui_mock_screens/Tutor_Dashboard_Landing.png`

**Pack 1 scope only**
- welcome header
- assigned students count
- active courses count
- empty-state cards when no assignments or mappings exist

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


# 9. Pack 1 High-Level Design (HLD)

## 9.1 Architecture modules

### Access Control Module
Responsibilities:
- role-based route protection
- state-aware route protection
- dashboard routing based on role and approval status


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

