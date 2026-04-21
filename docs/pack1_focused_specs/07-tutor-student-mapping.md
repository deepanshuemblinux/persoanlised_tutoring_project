- This spec is derived from: pack1_final_implementation_details.md
- This spec owns: Tutor-student mapping
- This spec strictly preserves original implementation details without modification
- This spec must be used with shared contract spec: docs/pack1_focused_specs/01-platform-ux-and-shared-contracts.md
- Fallback context source: docs/pack1_final_implementation_details.md

# 6. Screen-by-Screen UX Details

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


# 8. UX Rules and Behavioral Rules

## 8.4 Mapping rules

- Mapping is admin-only
- One active tutor per student per course
- Tutor must be approved and teach selected subject and class
- Mapping history must be preserved for auditability


# 9. Pack 1 High-Level Design (HLD)

## 9.1 Architecture modules

### Mapping Module
Responsibilities:
- assign tutor to approved enrollment
- replace tutor while preserving history
- query current mapping


# 10. Pack 1 Low-Level Design (LLD)

## 10.5 Tutor mapping replacement sequence

1. Admin opens approved enrollment with current tutor mapping
2. Admin selects new tutor
3. Server validates tutor eligibility
4. Server marks previous mapping inactive with `active_to`
5. Server inserts new active mapping row
6. Action is logged in admin action log


# 11. State Models

## 11.4 Mapping state machine

```text
NO_MAPPING -> ACTIVE_MAPPING
ACTIVE_MAPPING -> REPLACED_MAPPING (historical row closed)
```


# 12. Database Schema Draft (Implementation-Oriented)

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


# 13. API Draft (Implementation-Oriented)

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


# 14. Validation Rules and Business Rules

## 14.6 Tutor mapping rules

- only approved tutor can be mapped
- tutor must have capability row for subject + class
- only approved enrollment can be mapped
- exactly one active mapping per enrollment

