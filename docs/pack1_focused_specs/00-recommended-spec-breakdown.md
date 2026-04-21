# Step 1: Recommended Spec Breakdown

## 1) Platform UX and Shared Contracts
- Spec Name: Platform UX and Shared Contracts
- Why it is a separate system: It defines shared purpose, actor responsibilities, UI contract rules, integration expectations, and common conventions consumed by all modules.
- What it owns: Document purpose, product context, role responsibilities, UI design scope, screen inventory, integration points, non-functional expectations, DB conventions, API conventions.

## 2) Authentication and Account Recovery
- Spec Name: Authentication and Account Recovery
- Why it is a separate system: It owns account creation, login, and password recovery lifecycles.
- What it owns: Signup/login/forgot-reset UX, onboarding and recovery flows, auth module details, identity tables and endpoints, identity validation rules.

## 3) Verification and OTP
- Spec Name: Verification and OTP
- Why it is a separate system: It owns email and reset OTP verification lifecycle, OTP issuance, retry, and validation constraints.
- What it owns: Verification rules, OTP UX, verification module details, OTP/session tables, OTP endpoints, OTP validation rules.

## 4) User Management and Approval
- Spec Name: User Management and Approval
- Why it is a separate system: It owns user approval state transitions and related tutor/student review outcomes.
- What it owns: User approval states, pending/rejected UX, tutor/student review screens, user management module details, profile tables, approval endpoints, approval rules.

## 5) Course Catalog and Enrollment
- Spec Name: Course Catalog and Enrollment
- Why it is a separate system: It owns enrollment request lifecycle, approval lifecycle, registration-fee behavior, and catalog dependencies.
- What it owns: Enrollment and fee rules, student enrollment UX, enrollment flows, catalog and enrollment modules, enrollment state models, catalog/enrollment tables, enrollment endpoints, enrollment validation rules.

## 6) Admin Operations
- Spec Name: Admin Operations
- Why it is a separate system: It owns administrator-facing operational workflows and decision surfaces.
- What it owns: Admin login/dashboard UX, admin approval flow overview, admin operations module details, admin profile/action-log tables, admin listing endpoints, audit notes.

## 7) Tutor-Student Mapping
- Spec Name: Tutor-Student Mapping
- Why it is a separate system: It owns tutor assignment and replacement lifecycle for approved enrollments.
- What it owns: Mapping UX, mapping rules, mapping module details, mapping sequence, mapping state model, mapping table, mapping endpoints, mapping validation rules.

## 8) Institute Enrollment Settings
- Spec Name: Institute Enrollment Settings
- Why it is a separate system: It owns institute-level enrollment defaults and policy controls.
- What it owns: Institute settings UX, institute enrollment settings module details, institute/settings tables, institute settings endpoints.

## 9) Access Control and Error Handling
- Spec Name: Access Control and Error Handling
- Why it is a separate system: It owns route/access behavior, unauthorized handling, and normalized error contract.
- What it owns: Access behavior rules, dashboard/access UX, unauthorized/session-expired UX, access control module details, error model, security notes.

## 10) Payment Callback Future Readiness
- Spec Name: Payment Callback Future Readiness
- Why it is a separate system: It isolates future gateway callback compatibility and payment integrity controls.
- What it owns: Gateway payment completion flow, future payment module details, callback-compatible fee update sequence, payment callback endpoint, gateway integrity rules.

## 11) Quality Operations and Delivery
- Spec Name: Quality Operations and Delivery
- Why it is a separate system: It owns observability, test execution baseline, implementation workflow, and declared out-of-scope guardrails.
- What it owns: Observability notes, functional and edge test cases, implementation order, definition of done, out-of-scope list.
