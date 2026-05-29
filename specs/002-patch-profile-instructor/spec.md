# Feature Specification: 강사 지정 (Patch Profile Instructor)

**Feature Branch**: `002-patch-profile-instructor`

**Created**: 2026-05-28

**Status**: Draft

**Input**: User description: "docs/api-spec.md의 PATCH /api/profile/{profileId}/instructor 명세를 확인하고 구현에 필요한 요구사항을 정리해줘"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 강사 지정 성공 (Priority: P1)

관리자가 특정 프로필을 강사로 지정한다. `PATCH /api/profile/{profileId}/instructor`를 호출하면 해당 프로필의 `isInstructor` 값이 `true`로 변경되고, 대상 프로필 ID가 반환된다.

**Why this priority**: 강사 지정은 이 기능의 핵심 목적이며, 이 유스케이스 없이는 기능 자체가 성립하지 않는다.

**Independent Test**: 존재하는 profileId로 PATCH 요청을 보내고, 응답 200 OK와 `{ "id": profileId }`를 확인한 뒤 해당 프로필의 `isInstructor`가 `true`임을 검증한다.

**Acceptance Scenarios**:

1. **Given** 유효한 profileId를 가진 프로필이 존재할 때, **When** `PATCH /api/profile/{profileId}/instructor`를 호출하면, **Then** 200 OK와 `{ "id": "{profileId}" }`가 반환된다.
2. **Given** 유효한 profileId를 가진 프로필이 존재할 때, **When** 강사 지정 요청이 성공하면, **Then** 해당 프로필의 `isInstructor`가 `true`로 변경된다.

---

### User Story 2 - 이미 강사인 프로필 재지정 (Priority: P2)

이미 `isInstructor`가 `true`인 프로필에 대해 동일 요청을 다시 호출해도 오류 없이 200 OK가 반환된다 (멱등 처리).

**Why this priority**: 중복 호출 시 오류가 발생하면 클라이언트 코드가 복잡해진다. 멱등성 보장은 API 안정성의 핵심이다.

**Independent Test**: `isInstructor`가 이미 `true`인 프로필에 PATCH 요청을 보내고, 200 OK와 동일 profileId가 반환되는지 확인한다.

**Acceptance Scenarios**:

1. **Given** `isInstructor`가 이미 `true`인 프로필이 존재할 때, **When** `PATCH /api/profile/{profileId}/instructor`를 호출하면, **Then** 200 OK와 `{ "id": "{profileId}" }`가 반환된다.
2. **Given** 동일 요청을 두 번 연속 호출했을 때, **When** 두 번째 요청이 처리되면, **Then** 첫 번째 요청과 동일한 응답이 반환된다.

---

### User Story 3 - 존재하지 않는 프로필 지정 시도 (Priority: P1)

존재하지 않는 profileId로 강사 지정 요청을 보내면 404 Not Found와 에러 코드 `PROFILE_NOT_FOUND`가 반환된다.

**Why this priority**: 잘못된 입력에 대한 명확한 에러 응답은 클라이언트가 문제를 진단할 수 있게 하며, API 신뢰성의 기반이다.

**Independent Test**: 존재하지 않는 profileId로 PATCH 요청을 보내고, 404 응답과 `PROFILE_NOT_FOUND` 에러 코드를 확인한다.

**Acceptance Scenarios**:

1. **Given** 존재하지 않는 profileId로 요청할 때, **When** `PATCH /api/profile/{profileId}/instructor`를 호출하면, **Then** 404 Not Found와 에러 코드 `PROFILE_NOT_FOUND`가 반환된다.

---

### Edge Cases

- 존재하지 않는 profileId 요청 시 404를 반환한다.
- 이미 `isInstructor`가 `true`인 프로필에 요청해도 200 OK를 반환한다 (멱등).
- Request Body가 포함되어 있어도 무시하고 정상 처리한다.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: 시스템은 유효한 profileId를 경로 파라미터로 받아 해당 프로필의 `isInstructor`를 `true`로 변경해야 한다.
- **FR-002**: 강사 지정 성공 시 HTTP 200 OK와 대상 프로필 ID(`{ "id": "..." }`)를 반환해야 한다.
- **FR-003**: 존재하지 않는 profileId에 대한 요청은 HTTP 404 Not Found와 에러 코드 `PROFILE_NOT_FOUND`로 응답해야 한다.
- **FR-004**: 이미 `isInstructor`가 `true`인 프로필에 대한 요청도 동일하게 200 OK로 응답해야 한다 (멱등).
- **FR-005**: 요청 본문(Request Body)은 없으며, 경로 파라미터 `profileId`만 입력으로 사용한다.

### Key Entities

- **Profile**: 사용자 프로필. `id`(8자리 고유 문자열), `nickname`, `sex`, `isInstructor`(Boolean, 기본값 `false`) 필드를 가진다.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 유효한 profileId로 강사 지정 요청 시 200 OK 응답과 프로필 ID가 반환된다.
- **SC-002**: 존재하지 않는 profileId로 요청 시 항상 404 응답과 `PROFILE_NOT_FOUND` 에러 코드가 반환된다.
- **SC-003**: 동일 profileId에 강사 지정 요청을 반복해도 결과가 일관되게 200 OK로 반환된다 (멱등).
- **SC-004**: 에러 응답은 공통 에러 형식(`{ "status": ..., "errors": [...] }`)을 따른다.

## Assumptions

- 이 API를 호출하는 주체에 대한 인증/권한 제어는 현재 범위 밖이다 (인증 미구현 상태).
- `isInstructor`는 단방향 변경만 지원한다 (한번 강사가 된 프로필을 일반 사용자로 되돌리는 기능은 이 스펙 범위 밖).
- profileId는 8자리 문자열로 기존 프로필 생성 규칙(`POST /api/profile`)에 따라 생성된 값이다.
- 공통 에러 응답 형식(`docs/api-spec.md` 정의)은 이미 구현되어 있다고 가정한다.
