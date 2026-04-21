- This spec is derived from: pack1_final_implementation_details.md
- This spec owns: Admin operations
- This spec strictly preserves original implementation details without modification
- This spec must be used with shared contract spec: docs/pack1_focused_specs/01-platform-ux-and-shared-contracts.md
- Fallback context source: docs/pack1_final_implementation_details.md

# 6. Screen-by-Screen UX Details

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


# 7. Primary Pack 1 User Flows

## 7.4 Admin approval flow

1. Admin logs in
2. Opens pending tutors or students or enrollments
3. Reviews detail page
4. Approves or rejects with optional note / required reason on rejection
5. Action recorded in audit log


# 9. Pack 1 High-Level Design (HLD)

## 9.1 Architecture modules

### Admin Operations Module
Responsibilities:
- review queues
- approve/reject actions
- admin notes and audit logs
- unpaid approval warnings at UI/API level


# 12. Database Schema Draft (Implementation-Oriented)

### `admin_profiles`

| Column | Type | Null | Notes |
|---|---|---:|---|
| id | uuid | no | PK |
| user_id | uuid | no | FK users.id unique |
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

## 13.5 Admin approval endpoints

### `GET /api/v1/admin/tutors/pending`
### `GET /api/v1/admin/students/pending`
### `GET /api/v1/admin/enrollments/requested`

Paginated list endpoints.


# 16. Security, Audit, and Operational Notes

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

