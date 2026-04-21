- This spec is derived from: pack1_final_implementation_details.md
- This spec owns: Quality, operations, and delivery
- This spec strictly preserves original implementation details without modification
- This spec must be used with shared contract spec: docs/pack1_focused_specs/01-platform-ux-and-shared-contracts.md
- Fallback context source: docs/pack1_final_implementation_details.md

# 16. Security, Audit, and Operational Notes

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
