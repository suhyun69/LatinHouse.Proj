# Feature Specification: 랜덤 수업 생성 (POST /api/lesson/random)

**Feature Branch**: `007-random-lesson-create`

**Created**: 2026-06-03

**Status**: Draft

**Input**: User description: "POST api/lesson/random 엔드포인트를 생성할거야. POST api/lesson 엔드포인트 실행에 필요한 파라미터를 랜덤으로 생성하여 수업을 생성하는 기능이야. 강사 프로필을 조회해서 랜덤으로 할당하고, 프로필이 없으면 별도로 생성해서 진행해"

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 강사 프로필이 있을 때 랜덤 수업 생성 (Priority: P1)

운영자가 테스트 또는 시연 목적으로 수업 데이터를 빠르게 생성하고 싶다. 기존에 강사 프로필이 존재할 경우 그 중에서 랜덤으로 강사를 선택하여 수업을 생성한다.

**Why this priority**: 시스템에 강사 프로필이 이미 존재하는 것이 일반적인 상황이며, 이 흐름이 핵심 기능이다.

**Independent Test**: `POST /api/lesson/random` 호출만으로 레슨 ID가 반환되면 독립적으로 검증 가능하다.

**Acceptance Scenarios**:

1. **Given** `isInstructor=true`인 프로필이 1개 이상 존재할 때, **When** `POST /api/lesson/random`을 요청하면, **Then** 201 Created와 생성된 레슨 ID가 반환된다.
2. **Given** 남성·여성 강사 프로필이 모두 존재할 때, **When** `POST /api/lesson/random`을 요청하면, **Then** 생성된 레슨의 instructorLo 또는 instructorLa 중 최소 하나 이상이 기존 프로필 ID로 설정된다.
3. **Given** 남성 강사 프로필만 존재할 때, **When** `POST /api/lesson/random`을 요청하면, **Then** instructorLo에 기존 강사가 할당되고 instructorLa는 null 또는 새로 생성된 여성 강사가 할당된다.

---

### User Story 2 - 강사 프로필이 없을 때 자동 생성 후 수업 생성 (Priority: P2)

강사 프로필이 전혀 없는 초기 상태에서도 수업을 생성할 수 있어야 한다. 시스템이 자동으로 강사 프로필을 생성하고 강사로 지정한 뒤 수업을 생성한다.

**Why this priority**: 초기 세팅 상황에서도 기능이 동작해야 한다.

**Independent Test**: 강사 프로필이 없는 상태에서 `POST /api/lesson/random` 호출 후 새 프로필 생성 여부와 레슨 생성 여부를 각각 검증한다.

**Acceptance Scenarios**:

1. **Given** `isInstructor=true`인 프로필이 전혀 없을 때, **When** `POST /api/lesson/random`을 요청하면, **Then** 신규 강사 프로필이 최소 1개 생성되고 201 Created와 레슨 ID가 반환된다.
2. **Given** 강사 프로필이 없을 때, **When** 신규 프로필을 생성하고 강사로 지정한 뒤 수업을 생성하면, **Then** 생성된 수업의 instructorLo 또는 instructorLa에 새 프로필 ID가 설정된다.
3. **Given** 강사 프로필 생성 중 오류 발생 시, **When** `POST /api/lesson/random`을 요청하면, **Then** 500 Internal Server Error가 반환된다.

---

### User Story 3 - 랜덤 생성된 수업의 필드 유효성 확인 (Priority: P3)

생성된 수업이 `POST /api/lesson`의 비즈니스 규칙을 모두 준수해야 한다. 랜덤 값이라도 유효하지 않은 데이터로 수업이 생성되어서는 안 된다.

**Why this priority**: 데이터 정합성 보장이 필요하지만, 핵심 기능이 먼저 동작해야 의미가 있다.

**Independent Test**: 생성된 레슨 ID로 `GET /api/lessons/{lessonNo}`를 조회하여 모든 필드가 유효한 값을 가지는지 검증한다.

**Acceptance Scenarios**:

1. **Given** 수업이 랜덤 생성될 때, **When** 생성된 레슨 ID로 상세 조회하면, **Then** genre는 `S` 또는 `B`, options는 1~3개, 각 option의 endTime은 startTime보다 이후인 값이 반환된다.
2. **Given** 수업이 랜덤 생성될 때, **When** 생성된 레슨 ID로 상세 조회하면, **Then** instructorLo는 `sex=M, isInstructor=true`인 프로필 ID이거나 null, instructorLa는 `sex=F, isInstructor=true`인 프로필 ID이거나 null이다.

---

### Edge Cases

- `isInstructor=true`이지만 남성만 있을 때 여성 강사 슬롯은 어떻게 처리되는가? → 새 여성 강사 프로필을 자동 생성하거나 null로 남긴다 (할당 여부는 랜덤).
- discounts의 `type=E`인 경우 condition 날짜가 항상 options 중 가장 이른 startDate보다 7일 이전이어야 하며, 과거 날짜가 될 수 있음에 유의한다.
- 동시에 여러 요청이 들어올 경우 강사 프로필이 중복 생성될 수 있다 (허용 범위로 간주).

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: 시스템은 Request Body 없이 `POST /api/lesson/random` 요청을 수락해야 한다.
- **FR-002**: 시스템은 `GET /api/profiles?isInstructor=true` API를 호출하여 기존 강사 프로필 목록을 조회해야 한다.
- **FR-003**: 시스템은 조회된 강사 프로필을 `sex=M`(남성)과 `sex=F`(여성)으로 분리하여 각각 랜덤 선택해야 한다.
- **FR-004**: 남성 강사 후보가 없을 경우, 시스템은 `POST /api/profile`로 남성 강사 프로필을 생성한 뒤 `PATCH /api/profile/{profileId}/instructor`로 강사 지정해야 한다.
- **FR-005**: 여성 강사 후보가 없을 경우, 시스템은 `POST /api/profile`로 여성 강사 프로필을 생성한 뒤 `PATCH /api/profile/{profileId}/instructor`로 강사 지정해야 한다.
- **FR-006**: 최종적으로 instructorLo와 instructorLa 중 최소 1명 이상이 할당되어야 하며, 둘 다 할당되거나 하나만 할당되는 것도 허용된다.
- **FR-007**: 시스템은 아래 규칙에 따라 레슨 파라미터를 랜덤 생성하여 `POST /api/lesson`을 내부적으로 호출해야 한다.
  - `genre`: `S` 또는 `B` 중 랜덤
  - `title`: genre에 따라 `살사 초급반` / `살사 중급반` / `살사 상급반` / `바차타 초급반` / `바차타 중급반` / `바차타 상급반` 중 랜덤
  - `options`: 1~3개 랜덤 생성. 각 option의 startDate는 오늘로부터 7~60일 이내 랜덤, startTime은 `10:00`·`14:00`·`19:00`·`20:00` 중 랜덤, endDate는 startDate와 동일, endTime은 startTime + 2시간, region은 `GN`·`HD` 중 랜덤
  - `amount`: `30000`·`50000`·`80000`·`100000` 중 랜덤
  - `discounts`: 0~2개 랜덤 생성. type=E의 condition은 options 중 가장 이른 startDate 기준 7일 전 날짜, type=S의 condition은 `M`·`F` 중 랜덤, amount는 `5000`·`10000`·`15000` 중 랜덤
  - `isActive`: `true` 고정
  - `account`, `contacts`, `notices`: null (미포함)
- **FR-008**: 수업 생성 성공 시 201 Created와 `{ "id": <Long> }` 형식으로 응답해야 한다.
- **FR-009**: 강사 프로필 생성 또는 강사 지정 실패 시 500 Internal Server Error를 반환해야 한다.
- **FR-010**: 신규 생성 강사 프로필의 nickname은 `강사_M_<4자리 랜덤 숫자>` (남성) 또는 `강사_F_<4자리 랜덤 숫자>` (여성) 형식이어야 한다.

### Key Entities

- **Profile**: 강사 프로필. `id(String)`, `nickname`, `sex(M/F)`, `isInstructor(Boolean)` 속성 보유. 이 기능에서는 강사 할당 용도로만 사용.
- **Lesson**: 생성될 수업. title, genre, instructorLo, instructorLa, options, amount, discounts, isActive 속성으로 구성.
- **LessonOption**: 수업 일정 단위. startDate, startTime, endDate, endTime, region 포함.
- **LessonDiscount**: 수업 할인. type(E/S), condition, amount 포함.

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: `POST /api/lesson/random` 단일 호출로 수업이 1건 생성되어 유효한 레슨 ID가 반환된다.
- **SC-002**: 강사 프로필이 없는 초기 상태에서도 오류 없이 수업이 생성된다.
- **SC-003**: 생성된 수업의 모든 필드는 `POST /api/lesson` 비즈니스 규칙을 100% 준수한다 (instructorLo·La 성별/강사 조건, option 시간 순서 등).
- **SC-004**: 강사 프로필 조회·생성·강사 지정·수업 생성 전 과정이 단일 API 호출로 완결된다.
- **SC-005**: 프로필 생성 또는 강사 지정 실패 시 500 에러가 반환되고 부분 생성된 데이터가 남지 않는다 (롤백 또는 무결성 유지).

---

## Assumptions

- 이 엔드포인트는 테스트·시연 목적으로 사용되며 실제 운영 트래픽은 고려하지 않는다.
- 내부적으로 기존 `POST /api/lesson`, `POST /api/profile`, `PATCH /api/profile/{profileId}/instructor` 로직을 재사용한다.
- 동시 요청에 의한 강사 프로필 중복 생성은 허용 범위로 간주한다 (동시성 제어 불필요).
- discounts의 type=E에서 condition이 오늘 기준 과거 날짜가 될 수 있으나 유효성 검증은 수업 생성 시점의 규칙을 따른다 (생성 자체는 허용).
- `account`, `contacts`, `notices`는 랜덤 생성 대상에서 제외한다.
