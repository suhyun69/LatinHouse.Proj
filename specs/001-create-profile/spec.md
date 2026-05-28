# Feature Specification: Create Profile

**Feature Branch**: `001-create-profile`

**Created**: 2026-05-28

**Status**: Draft

**Input**: User description: "docs/api-spec.md의 POST /api/profiles 명세를 확인하고 구현에 필요한 요구사항을 정리해줘"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Profile Creation (Priority: P1)

A new user provides their nickname and gender to create a personal profile in the system.

**Why this priority**: Profile creation is the foundational step that enables all personalized features. Without a profile, users cannot be identified or receive customized service.

**Independent Test**: Can be fully tested by submitting a valid nickname and sex value and confirming a unique 8-character profile ID is returned with a 201 status.

**Acceptance Scenarios**:

1. **Given** a user provides a valid nickname and sex (`M` or `F`), **When** they submit the profile creation request, **Then** the system creates the profile and returns a unique 8-character profile ID with a 201 Created status.
2. **Given** a user submits the request with no nickname, **When** validation runs, **Then** the system returns a 400 error with field `nickname` and message "닉네임을 입력해 주세요."
3. **Given** a user submits the request with no sex value, **When** validation runs, **Then** the system returns a 400 error with field `sex` and message "성별을 입력해 주세요."
4. **Given** a user provides a sex value other than `M` or `F`, **When** validation runs, **Then** the system returns a 400 error with field `sex` and message "성별은 M 또는 F만 입력 가능합니다."

---

### Edge Cases

- What happens when both `nickname` and `sex` are missing simultaneously? → Multiple validation errors are returned in the `errors` array.
- What happens when `sex` is provided in lowercase (e.g., `m` or `f`)? → System rejects as invalid since only `M` or `F` are accepted.
- What happens when `nickname` is a whitespace-only string? → Treated as empty; fails the non-empty validation.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST accept a profile creation request containing a nickname and sex field.
- **FR-002**: System MUST require `nickname` to be a non-null, non-empty string.
- **FR-003**: System MUST require `sex` to be a non-null, non-empty value of either `M` or `F`.
- **FR-004**: System MUST return a 400 Bad Request with field-level error messages when any validation rule is violated.
- **FR-005**: System MUST return all validation errors in a single response (not one at a time), using the standard error format: `{ "status": 400, "errors": [{ "field": "...", "message": "..." }] }`.
- **FR-006**: System MUST generate a unique 8-character alphanumeric profile ID upon successful creation.
- **FR-007**: System MUST return the generated profile ID with a 201 Created status upon successful creation, in the format `{ "id": "..." }`.
- **FR-008**: Error messages MUST use the exact Korean strings defined in the API spec:
  - nickname empty/null → "닉네임을 입력해 주세요."
  - sex empty/null → "성별을 입력해 주세요."
  - sex invalid value → "성별은 M 또는 F만 입력 가능합니다."

### Key Entities

- **Profile**: Represents a user's identity in the system. Key attributes: unique ID (8-character alphanumeric string), nickname (display name), sex (gender code: `M` or `F`).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Users can complete profile creation in under 5 seconds under normal network conditions.
- **SC-002**: 100% of invalid requests return field-specific error messages matching the exact defined Korean error strings.
- **SC-003**: All successfully created profiles are assigned a unique 8-character ID — no duplicate IDs exist in the system.
- **SC-004**: 100% of valid profile creation requests result in a 201 response containing the generated profile ID.
- **SC-005**: Validation correctly rejects all disallowed sex values (anything other than `M` or `F`), including lowercase variants.

## Assumptions

- Profiles can be created without prior user authentication, as the API spec does not specify an auth requirement for this endpoint.
- The 8-character profile ID uses mixed-case alphanumeric characters (e.g., `Ab2Cd3Ef`), based on the example in the spec.
- Whitespace-only nickname strings are treated as empty and fail validation.
- Lowercase sex values (`m`, `f`) are not accepted — only uppercase `M` or `F` are valid.
- The canonical API path is `POST /api/profile` (singular) as defined in the spec; the user's input referenced `/api/profiles` (plural), which is treated as a reference to the same endpoint.
- When multiple fields fail validation simultaneously, all errors are returned together in a single 400 response.
