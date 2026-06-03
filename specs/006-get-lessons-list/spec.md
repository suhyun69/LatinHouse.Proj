# Feature Specification: 레슨 목록 조회

**Feature Branch**: `006-get-lessons-list`

**Created**: 2026-06-02

**Status**: Draft

**Input**: User description: "GET /api/lessons 레슨 목록 조회 API 구현"

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 레슨 목록 전체 조회 (Priority: P1)

사용자가 필터 없이 GET /api/lessons를 호출하면 등록된 모든 레슨의 옵션 단위 목록을 받는다.

**Why this priority**: 핵심 목록 조회 기능으로, 필터 없는 전체 조회가 기본 사용 사례다.

**Independent Test**: 레슨 2개(각 옵션 1개)를 생성한 뒤 GET /api/lessons 호출 → 2개 항목 반환 확인.

**Acceptance Scenarios**:

1. **Given** 레슨 데이터가 존재할 때, **When** GET /api/lessons 호출, **Then** 200 OK와 함께 각 옵션 단위 flat list 반환
2. **Given** 레슨 데이터가 없을 때, **When** GET /api/lessons 호출, **Then** 200 OK와 함께 빈 배열 반환

---

### User Story 2 - 필터링 조회 (Priority: P2)

사용자가 region, instructor, genre 중 하나 이상의 필터를 전달하면 조건에 맞는 옵션 목록만 반환된다. 복수 필터는 AND 조건으로 결합된다.

**Why this priority**: 목록이 많아질 때 필터링은 필수 사용성 기능이다.

**Independent Test**: region=GN 필터 → GN 옵션만 반환 확인.

**Acceptance Scenarios**:

1. **Given** GN 옵션과 HD 옵션이 있을 때, **When** region=GN으로 호출, **Then** GN 옵션만 반환
2. **Given** 여러 장르 레슨이 있을 때, **When** genre=S로 호출, **Then** 살사 레슨 옵션만 반환
3. **Given** 강사 A가 등록된 레슨이 있을 때, **When** instructor=A로 호출, **Then** 해당 강사 레슨 옵션만 반환
4. **Given** region과 genre 필터를 동시에 전달, **When** 호출, **Then** 두 조건 모두 만족하는 옵션만 반환

---

### User Story 3 - 상태·할인 정보 자동 계산 (Priority: P3)

각 항목에는 status(수업 상태)와 discountCondition/discountAmount(얼리버드 할인)가 자동 계산되어 반환된다.

**Why this priority**: 클라이언트가 별도 계산 없이 바로 표시할 수 있는 핵심 부가 정보다.

**Independent Test**: isActive=false인 레슨 조회 → status="INACTIVE" 확인.

**Acceptance Scenarios**:

1. **Given** isActive=false인 레슨, **When** 목록 조회, **Then** status="INACTIVE"
2. **Given** isActive=true이고 startDateTime이 미래인 옵션, **When** 목록 조회, **Then** status="PENDING"
3. **Given** isActive=true이고 현재 시점이 startDateTime~endDateTime 사이인 옵션, **When** 목록 조회, **Then** status="IN_PROGRESS"
4. **Given** isActive=true이고 endDateTime이 과거인 옵션, **When** 목록 조회, **Then** status="DONE"
5. **Given** type=E이고 condition이 오늘 이후인 EarlyBird discount가 있을 때, **When** 목록 조회, **Then** 가장 이른 condition의 discountCondition/discountAmount 반환
6. **Given** EarlyBird discount가 없거나 condition이 모두 과거인 경우, **When** 목록 조회, **Then** discountCondition=null, discountAmount=null

---

### Edge Cases

- 레슨에 옵션이 1개도 없으면 해당 레슨은 목록에 나타나지 않는다.
- 잘못된 region 코드(GN, HD 외) 전달 시 400 반환.
- 잘못된 genre 코드(S, B 외) 전달 시 400 반환.
- EarlyBird discount 후보가 복수이면 condition 기준 가장 빠른 1건만 적용.

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: GET /api/lessons 엔드포인트는 모든 레슨의 옵션 단위 flat list를 반환해야 한다.
- **FR-002**: 쿼리 파라미터 region(GN/HD), instructor(Profile.id), genre(S/B)를 선택적으로 지원해야 한다.
- **FR-003**: 복수 필터는 AND 조건으로 결합해야 한다.
- **FR-004**: region 필터는 LessonOption.region 기준으로 적용한다.
- **FR-005**: instructor 필터는 Lesson.instructorLo 또는 Lesson.instructorLa 일치 여부로 적용한다.
- **FR-006**: genre 필터는 Lesson.genre 기준으로 적용한다.
- **FR-007**: 각 항목의 status는 isActive 및 option의 startDateTime/endDateTime으로 계산한다.
- **FR-008**: 각 항목의 discountCondition/discountAmount는 EarlyBird(type=E) discount 중 condition이 오늘 이후인 것 중 가장 이른 1건으로 계산한다.
- **FR-009**: 결과는 lessonNo 오름차순 → optionId 오름차순으로 정렬한다.
- **FR-010**: 잘못된 region/genre 코드 전달 시 400 Bad Request를 반환한다.

### Key Entities

- **Lesson**: id, title, genre, instructorLo, instructorLa, isActive, discounts, options
- **LessonOption**: id, startDateTime, endDateTime, region
- **LessonDiscount**: type(EARLYBIRD/SEX), condition(String), amount

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 필터 없이 호출 시 모든 옵션 항목이 반환된다.
- **SC-002**: 각 필터 파라미터를 단독 또는 조합하여 전달했을 때 올바른 결과만 반환된다.
- **SC-003**: 잘못된 필터값 전달 시 항상 400 에러가 반환된다.
- **SC-004**: status, discountCondition, discountAmount 값이 비즈니스 규칙에 따라 정확히 계산된다.

---

## Assumptions

- 기존 Lesson 도메인 객체(Lesson, LessonOption, LessonDiscount 등)와 JPA Entity는 변경하지 않는다.
- 인증/인가는 이 피처 범위 밖이다.
- 페이지네이션은 이 피처에서 지원하지 않는다.
