# Feature Specification: 레슨 단건 조회

**Feature Branch**: `005-get-lesson`

**Created**: 2026-06-01

**Status**: Draft

**Input**: User description: "api-docs의 GET /api/lessons/{lessonNo} 명세를 확인하고 구현에 필요한 사항을 확인해"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 레슨 단건 조회 (Priority: P1)

클라이언트가 `GET /api/lessons/{lessonNo}`를 호출하여 특정 레슨의 상세 정보를 조회한다.

**Why this priority**: 레슨 상세 페이지 진입의 기반 기능이며, 수강 신청·공유·편집 등 대부분의 하위 기능이 이 API에 의존한다.

**Independent Test**: 존재하는 lessonNo로 `GET /api/lessons/{lessonNo}` 호출 시 200 OK와 레슨 전체 정보가 반환되는지 확인.

**Acceptance Scenarios**:

1. **Given** 해당 lessonNo의 레슨이 존재하는 상태에서, **When** `GET /api/lessons/{lessonNo}`를 호출하면, **Then** 200 OK와 레슨 상세 정보(id, title, genre, instructors, options, amount, discounts, account, contacts, isActive, notices)가 반환된다.
2. **Given** 레슨에 남성 강사만 등록된 경우, **When** 조회하면, **Then** instructorLa는 null로 반환된다.
3. **Given** 레슨에 discounts, account, contacts, notices가 없는 경우, **When** 조회하면, **Then** 해당 필드는 각각 빈 배열 또는 null로 반환된다.

---

### User Story 2 - 존재하지 않는 레슨 조회 (Priority: P2)

존재하지 않는 lessonNo로 조회를 시도했을 때 적절한 에러가 반환된다.

**Why this priority**: 잘못된 URL 접근에 대한 명확한 피드백이 필요하다.

**Independent Test**: 존재하지 않는 lessonNo로 호출 시 404 Not Found와 LESSON_NOT_FOUND 에러가 반환되는지 확인.

**Acceptance Scenarios**:

1. **Given** 해당 lessonNo의 레슨이 없는 상태에서, **When** `GET /api/lessons/{lessonNo}`를 호출하면, **Then** 404 Not Found와 에러 코드 `LESSON_NOT_FOUND`가 반환된다.

---

### Edge Cases

- 하위 목록(options, discounts, contacts, notices)이 비어 있으면 빈 배열 `[]` 반환.
- account가 없으면 null 반환.
- instructorLo 또는 instructorLa가 없으면 null 반환.
- lessonNo가 Long 범위를 벗어나는 문자열인 경우 400 Bad Request.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: 시스템은 `GET /api/lessons/{lessonNo}` 요청에 대해 해당 레슨의 상세 정보를 200 OK로 반환해야 한다.
- **FR-002**: 응답 본문은 id, title, genre, instructorLo, instructorLa, options, amount, discounts, account, contacts, isActive, notices 필드를 포함해야 한다.
- **FR-003**: options 목록의 각 항목은 id, startDate, startTime, endDate, endTime, region, place, placeUrl을 포함해야 한다.
- **FR-004**: discounts 목록의 각 항목은 id, type, condition, amount를 포함해야 한다.
- **FR-005**: account 객체는 id, bank, account, name을 포함하며, 레슨에 계좌가 없으면 null을 반환해야 한다.
- **FR-006**: contacts 목록의 각 항목은 id, type, account, name을 포함해야 한다.
- **FR-007**: notices 목록의 각 항목은 id, type, text를 포함해야 한다.
- **FR-008**: 하위 목록(options 제외)이 비어 있으면 빈 배열 `[]`을 반환해야 한다.
- **FR-009**: 존재하지 않는 lessonNo 조회 시 404 Not Found와 에러 코드 `LESSON_NOT_FOUND`를 반환해야 한다.

### Key Entities

- **Lesson**: id(Long), title, genre(S/B), instructorLo(Profile.id), instructorLa(Profile.id), amount, isActive
- **LessonOption**: id, lessonNo, startDateTime, endDateTime, region(GN/HD), place, placeUrl
- **LessonDiscount**: id, lessonNo, type(E/S), condition, amount
- **LessonAccount**: id, lessonNo, bank, account, name
- **LessonContact**: id, lessonNo, type(Y/K/W/I/L/M), account, name
- **LessonNotice**: id, lessonNo, type(L/T/R/N/U), text

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 존재하는 레슨 조회 시 200 OK와 전체 하위 엔티티가 올바르게 반환된다.
- **SC-002**: 존재하지 않는 레슨 조회 시 404와 LESSON_NOT_FOUND 에러가 반환된다.
- **SC-003**: 하위 목록이 없는 레슨 조회 시 빈 배열/null이 정확히 반환된다.

## Assumptions

- 인증/인가 처리는 이 기능의 범위 밖이다.
- options는 최소 1개 이상 존재함이 보장되므로 빈 배열 케이스는 없다.
- 응답의 날짜/시간 형식은 저장된 LocalDateTime을 `yyyy-MM-dd` / `HH:mm`으로 분리하여 반환한다.
- `docs/api-spec.md`에 정의된 GET /api/lessons/{lessonNo} 명세를 기준으로 구현한다.
