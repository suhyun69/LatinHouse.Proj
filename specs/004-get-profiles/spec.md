# Feature Specification: 프로필 목록 조회

**Feature Branch**: `004-get-profiles`

**Created**: 2026-06-01

**Status**: Draft

**Input**: User description: "docs/api-spec.md의 GET /api/profiles 명세를 확인하고 구현에 필요한 요구사항을 정리해줘"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 전체 프로필 목록 조회 (Priority: P1)

클라이언트가 `GET /api/profiles`를 호출하여 등록된 모든 프로필의 목록을 조회한다.

**Why this priority**: 프로필 목록 조회는 레슨 등록 화면에서 강사를 선택하는 기반 기능이며, 다른 기능들이 의존한다.

**Independent Test**: 파라미터 없이 `GET /api/profiles` 호출 시 200 OK와 프로필 배열이 반환되는지 확인. 프로필이 없으면 빈 배열 반환.

**Acceptance Scenarios**:

1. **Given** 프로필이 1개 이상 등록된 상태에서, **When** `GET /api/profiles`를 호출하면, **Then** 200 OK와 전체 프로필 목록(id, nickname, sex, isInstructor)이 반환된다.
2. **Given** 등록된 프로필이 없는 상태에서, **When** `GET /api/profiles`를 호출하면, **Then** 200 OK와 빈 배열 `[]`이 반환된다.

---

### User Story 2 - 강사 프로필만 필터링 조회 (Priority: P2)

`isInstructor=true` 쿼리 파라미터를 통해 강사로 등록된 프로필만 조회한다. 레슨 생성 시 강사 선택 목적으로 주로 사용된다.

**Why this priority**: 레슨 등록 화면에서 강사 선택 드롭다운에 필요한 기능으로, 전체 목록에서 클라이언트가 필터링하는 것보다 서버 필터링이 효율적이다.

**Independent Test**: `GET /api/profiles?isInstructor=true` 호출 시 isInstructor=true인 프로필만 반환되는지 확인.

**Acceptance Scenarios**:

1. **Given** 강사 프로필과 일반 프로필이 혼재한 상태에서, **When** `GET /api/profiles?isInstructor=true`를 호출하면, **Then** 200 OK와 isInstructor=true인 프로필만 포함된 목록이 반환된다.
2. **Given** 강사 프로필이 없는 상태에서, **When** `GET /api/profiles?isInstructor=true`를 호출하면, **Then** 200 OK와 빈 배열이 반환된다.

---

### Edge Cases

- 프로필이 아무것도 없을 때: 빈 배열 `[]` 반환 (404 아님).
- `isInstructor` 파라미터에 올바르지 않은 값(예: `"yes"`)이 입력된 경우: 400 Bad Request와 에러 메시지 반환.
- `isInstructor` 파라미터를 명시하지 않으면 전체 목록 반환.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: 시스템은 `GET /api/profiles` 요청에 대해 200 OK와 프로필 목록을 반환해야 한다.
- **FR-002**: 각 프로필 항목은 id, nickname, sex, isInstructor 필드를 포함해야 한다.
- **FR-003**: `isInstructor` 쿼리 파라미터가 없으면 전체 프로필 목록을 반환해야 한다.
- **FR-004**: `isInstructor=true` 쿼리 파라미터가 있으면 강사 프로필만 반환해야 한다.
- **FR-005**: `isInstructor=false` 쿼리 파라미터가 있으면 일반(비강사) 프로필만 반환해야 한다.
- **FR-006**: 조회 결과가 없으면 빈 배열 `[]`을 반환해야 한다 (404 아님).
- **FR-007**: `isInstructor` 파라미터에 `true`, `false` 외의 값이 입력되면 400 Bad Request를 반환해야 한다.

### Key Entities

- **Profile**: 프로필 조회 결과 항목. id(String), nickname(String), sex(M/F), isInstructor(Boolean) 포함.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 파라미터 없이 호출 시 전체 프로필 목록이 정확히 반환된다.
- **SC-002**: `isInstructor=true` 필터 적용 시 강사 프로필만 반환되고 비강사 프로필은 포함되지 않는다.
- **SC-003**: 결과가 없는 경우 빈 배열이 반환된다 (오류 없음).
- **SC-004**: 잘못된 파라미터 입력 시 400 에러가 명확한 메시지와 함께 반환된다.

## Assumptions

- 인증/인가 처리는 이 기능의 범위 밖이다.
- 페이지네이션은 이 기능의 범위 밖이다 (전체 목록 반환).
- 정렬 기준은 등록 순서(id 기준 또는 DB 기본 순서)이며 별도 정렬 파라미터는 제공하지 않는다.
- `docs/api-spec.md`에 GET /api/profiles 명세가 아직 없으므로, 본 spec에서 명세를 정의하고 api-spec.md에도 추가한다.
