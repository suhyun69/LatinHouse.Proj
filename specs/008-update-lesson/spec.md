# Feature Specification: 레슨 수정 (PUT /api/lesson/{lessonNo})

**Feature Branch**: `008-update-lesson`

**Created**: 2026-06-03

**Status**: Draft

**Input**: User description: "docs/api-spec.md의 PUT /api/lesson/{lessonNo} 명세를 확인하고 구현에 필요한 요구사항을 정리해줘"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 레슨 전체 정보 수정 (Priority: P1)

관리자가 기존에 등록된 레슨의 제목, 장르, 강사, 수업 옵션, 수강료, 할인 정보 등 전체 데이터를 수정한다.

**Why this priority**: 레슨 정보가 변경될 때 기존 데이터를 새 데이터로 완전히 교체하는 것이 핵심 기능이다.

**Independent Test**: `PUT /api/lesson/{lessonNo}` 요청에 수정된 레슨 데이터를 담아 전송 후, `GET /api/lessons/{lessonNo}`로 변경 내용이 반영되었는지 확인한다.

**Acceptance Scenarios**:

1. **Given** 존재하는 레슨 ID와 유효한 수정 데이터가 있을 때, **When** `PUT /api/lesson/{lessonNo}` 요청을 보내면, **Then** 200 OK와 함께 `{ "id": lessonNo }` 를 반환한다.
2. **Given** 수정 요청 바디에 새로운 options 리스트가 포함되어 있을 때, **When** 요청이 성공하면, **Then** 기존 옵션은 전부 삭제되고 새 옵션으로 교체된다.
3. **Given** 수정 요청 바디에 새로운 discounts 리스트가 포함되어 있을 때, **When** 요청이 성공하면, **Then** 기존 할인은 전부 삭제되고 새 할인으로 교체된다.
4. **Given** 수정 요청 바디에 `account: null`이 포함되어 있을 때, **When** 요청이 성공하면, **Then** 기존 계좌 정보가 삭제된다.

---

### User Story 2 - 존재하지 않는 레슨 수정 시도 (Priority: P2)

관리자가 존재하지 않는 lessonNo로 수정 요청을 보낼 때 명확한 오류 응답을 받는다.

**Why this priority**: 잘못된 ID로의 수정 요청은 흔한 오류 상황으로 명확한 피드백이 필요하다.

**Independent Test**: 존재하지 않는 lessonNo로 PUT 요청을 보내 404 응답을 확인한다.

**Acceptance Scenarios**:

1. **Given** DB에 존재하지 않는 lessonNo가 주어졌을 때, **When** `PUT /api/lesson/{lessonNo}` 요청을 보내면, **Then** `404 Not Found`와 에러 코드 `LESSON_NOT_FOUND`를 반환한다.

---

### User Story 3 - 유효성 검사 실패 (Priority: P3)

관리자가 잘못된 형식의 데이터로 수정 요청을 보낼 때 어떤 필드가 잘못되었는지 안내를 받는다.

**Why this priority**: 입력 오류에 대한 명확한 피드백은 사용성을 높이지만, 핵심 기능 이후 처리한다.

**Independent Test**: 필수 필드 누락 또는 형식 오류가 포함된 바디로 요청하여 400 응답과 필드별 에러 메시지를 확인한다.

**Acceptance Scenarios**:

1. **Given** `title`이 빈 문자열인 요청 바디가 있을 때, **When** PUT 요청을 보내면, **Then** `400 Bad Request`와 `{ "field": "title", "message": "제목을 입력해 주세요." }` 를 반환한다.
2. **Given** `options`가 빈 리스트인 요청 바디가 있을 때, **When** PUT 요청을 보내면, **Then** `400 Bad Request`와 `{ "field": "options", "message": "수업 옵션을 1개 이상 입력해 주세요." }` 를 반환한다.
3. **Given** `instructorLo`와 `instructorLa`가 모두 null인 요청 바디가 있을 때, **When** PUT 요청을 보내면, **Then** `400 Bad Request`와 강사 필수 에러를 반환한다.
4. **Given** 존재하지 않는 강사 ID가 `instructorLo`에 포함될 때, **When** PUT 요청을 보내면, **Then** `400 Bad Request`와 `INSTRUCTOR_NOT_FOUND` 에러를 반환한다.

---

### Edge Cases

- 수정 요청에서 `options`를 1개에서 여러 개로 늘리거나 줄이면 기존 옵션이 완전히 교체된다.
- `discounts`, `contacts`, `notices`가 요청 바디에 null 또는 빈 배열로 전달되면 기존 데이터를 전부 삭제한다.
- `instructorLo`에 여성(sex=F) 강사 ID가 주어지면 `INSTRUCTOR_NOT_VALID` 오류를 반환한다.
- `instructorLa`에 남성(sex=M) 강사 ID가 주어지면 `INSTRUCTOR_NOT_VALID` 오류를 반환한다.
- 옵션의 `endDateTime`이 `startDateTime`보다 이전이거나 같으면 `VALIDATION_ERROR`를 반환한다.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: 시스템은 유효한 lessonNo와 요청 바디가 제공되었을 때 레슨 데이터를 전체 교체(replace) 방식으로 수정해야 한다.
- **FR-002**: 수정 성공 시 시스템은 `200 OK`와 수정된 레슨 ID를 반환해야 한다.
- **FR-003**: 시스템은 수정 요청 시 `POST /api/lesson`과 동일한 필드 유효성 검사를 수행해야 한다.
- **FR-004**: 시스템은 `options` 필드를 전체 교체해야 한다 — 기존 옵션을 모두 삭제 후 새 옵션을 삽입한다.
- **FR-005**: 시스템은 `discounts` 필드를 전체 교체해야 한다 — 기존 할인을 모두 삭제 후 새 할인을 삽입한다.
- **FR-006**: 시스템은 `contacts` 필드를 전체 교체해야 한다 — 기존 연락처를 모두 삭제 후 새 연락처를 삽입한다.
- **FR-007**: 시스템은 `notices` 필드를 전체 교체해야 한다 — 기존 공지를 모두 삭제 후 새 공지를 삽입한다.
- **FR-008**: `account`가 null로 전달되면 기존 계좌 정보를 삭제해야 한다.
- **FR-009**: 존재하지 않는 lessonNo로 요청 시 `404 Not Found`와 `LESSON_NOT_FOUND` 에러 코드를 반환해야 한다.
- **FR-010**: `instructorLo`와 `instructorLa`가 모두 null이면 `400 Bad Request`를 반환해야 한다.
- **FR-011**: 제공된 강사 ID가 DB에 존재하지 않으면 `INSTRUCTOR_NOT_FOUND`를 반환해야 한다.
- **FR-012**: 강사의 성별이 포지션(Lo=남성, La=여성)과 불일치하면 `INSTRUCTOR_NOT_VALID`를 반환해야 한다.
- **FR-013**: 강사의 `isInstructor=false`이면 `INSTRUCTOR_NOT_VALID`를 반환해야 한다.
- **FR-014**: 옵션의 `startDateTime`이 `endDateTime`보다 이후이거나 같으면 `VALIDATION_ERROR`를 반환해야 한다.
- **FR-015**: Earlybird 할인 조건(`type=E`)은 `yyyy-MM-dd` 형식이어야 하며, 형식 불일치 시 `VALIDATION_ERROR`를 반환해야 한다.
- **FR-016**: Sex 할인 조건(`type=S`)은 `M` 또는 `F`이어야 하며, 그 외 값은 `VALIDATION_ERROR`를 반환해야 한다.

### Key Entities

- **Lesson**: 수정 대상 루트 엔티티. title, genre, instructorLo, instructorLa, amount, isActive를 직접 갱신한다.
- **LessonOption**: 레슨에 연결된 수업 일정. 전체 교체 방식으로 관리된다.
- **LessonDiscount**: 레슨에 연결된 할인 정보. 전체 교체 방식으로 관리된다.
- **LessonAccount**: 레슨의 입금 계좌. null 전달 시 삭제, 값 전달 시 교체된다.
- **LessonContact**: 레슨의 연락처. 전체 교체 방식으로 관리된다.
- **LessonNotice**: 레슨의 공지. 전체 교체 방식으로 관리된다.
- **Profile**: 강사 유효성 검증에 사용된다 (존재 여부, isInstructor, sex).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 유효한 수정 요청은 200 OK와 레슨 ID를 반환하며, 이후 조회 시 변경된 데이터가 반영된다.
- **SC-002**: 존재하지 않는 lessonNo 요청은 100% 404 응답을 반환한다.
- **SC-003**: 유효성 검사 실패 시 어떤 필드가 왜 잘못되었는지를 명확히 안내한다.
- **SC-004**: 수정 요청에서 컬렉션 필드(options, discounts 등)가 교체된 후, 기존 데이터가 조회되지 않는다.

## Assumptions

- 레슨 수정은 인증 없이 접근 가능하다 (기존 API와 동일한 보안 정책 적용).
- `PUT`은 전체 교체(full replace) 방식이며, 일부 필드만 변경하는 부분 수정(PATCH)은 이 기능의 범위에 포함되지 않는다.
- 강사 유효성 검사 규칙은 `POST /api/lesson`과 동일하게 재사용된다.
- 기존 `SaveLessonPort`가 저장과 수정을 모두 처리한다고 가정한다 (JPA `save()` = upsert).
- 컬렉션 필드의 orphanRemoval은 이미 JPA 엔티티에 설정되어 있다고 가정한다.
