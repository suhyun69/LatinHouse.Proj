# Feature Specification: 주문 생성 (POST /api/order)

**Feature Branch**: `009-create-order`

**Created**: 2026-06-03

**Status**: Draft

**Input**: User description: "docs/api-spec.md의 POST /api/order 명세를 확인하고 구현에 필요한 요구사항을 정리해줘"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 주문 생성 (Priority: P1)

구매자가 원하는 레슨과 수업 옵션을 선택하고 주문을 생성한다. 주문이 생성되면 결제 대기 상태로 저장되고 고유한 주문 ID를 받는다.

**Why this priority**: 주문 생성은 결제·취소 등 이후 모든 주문 흐름의 시작점이다. 이 기능 없이는 어떤 거래도 시작할 수 없다.

**Independent Test**: `POST /api/order`로 요청 후 201 Created와 UUID 형식의 orderId를 반환받으면 성공. 이후 주문 조회 API로 PAYMENT_PENDING 상태임을 확인한다.

**Acceptance Scenarios**:

1. **Given** 존재하는 lessonNo, lessonOptionNo, profileId가 주어졌을 때, **When** `POST /api/order`를 호출하면, **Then** 201 Created와 UUID 형식의 `orderId`를 반환한다.
2. **Given** 주문이 생성된 직후, **When** 주문 상태를 확인하면, **Then** `PAYMENT_PENDING` 상태이다.
3. **Given** lessonOptionNo가 해당 lessonNo에 속한 옵션일 때, **When** 주문을 생성하면, **Then** 정상 처리된다.

---

### User Story 2 - 존재하지 않는 리소스로 주문 시도 (Priority: P2)

구매자가 존재하지 않는 레슨, 옵션, 프로필 ID로 주문을 시도할 때 명확한 오류 응답을 받는다.

**Why this priority**: 잘못된 ID로의 요청은 빈번한 오류 상황이며 명확한 피드백이 필요하다.

**Independent Test**: 존재하지 않는 각 ID로 요청하여 404 응답과 해당 에러 코드를 확인한다.

**Acceptance Scenarios**:

1. **Given** DB에 존재하지 않는 lessonNo가 주어졌을 때, **When** `POST /api/order`를 호출하면, **Then** `404 Not Found`와 `LESSON_NOT_FOUND`를 반환한다.
2. **Given** DB에 존재하지 않는 lessonOptionNo가 주어졌을 때, **When** 요청하면, **Then** `404 Not Found`와 `LESSON_OPTION_NOT_FOUND`를 반환한다.
3. **Given** DB에 존재하지 않는 profileId가 주어졌을 때, **When** 요청하면, **Then** `404 Not Found`와 `PROFILE_NOT_FOUND`를 반환한다.

---

### User Story 3 - 유효성 검사 실패 (Priority: P3)

필수 필드가 누락된 요청을 보낼 때 어떤 필드가 잘못되었는지 안내받는다.

**Why this priority**: 입력 오류 피드백은 사용성을 높이지만 핵심 기능 이후 처리한다.

**Independent Test**: 필수 필드를 null로 전달하고 400 응답과 필드별 에러 메시지를 확인한다.

**Acceptance Scenarios**:

1. **Given** `lessonNo`가 null인 요청, **When** 호출하면, **Then** `400 Bad Request`와 `"레슨을 선택해 주세요."` 메시지를 반환한다.
2. **Given** `lessonOptionNo`가 null인 요청, **When** 호출하면, **Then** `400 Bad Request`와 `"수업 옵션을 선택해 주세요."` 메시지를 반환한다.
3. **Given** `profileId`가 null 또는 빈 문자열인 요청, **When** 호출하면, **Then** `400 Bad Request`와 `"구매자 프로필을 입력해 주세요."` 메시지를 반환한다.

---

### Edge Cases

- lessonOptionNo가 lessonNo에 속하지 않는 다른 레슨의 옵션일 때 처리 방법 (현재 명세에 미정의 — `LESSON_OPTION_NOT_FOUND`로 처리 가정)
- 동일 구매자가 동일 레슨 옵션으로 중복 주문을 시도하는 경우 (현재 명세에 제한 없음 — 중복 허용 가정)
- price 초기값은 레슨의 amount로 설정하며, 할인 미적용 상태에서 생성

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: 시스템은 유효한 lessonNo, lessonOptionNo, profileId가 제공되면 주문을 생성해야 한다.
- **FR-002**: 주문 생성 시 orderId는 UUID 형식으로 자동 생성된다.
- **FR-003**: 주문 생성 직후 status는 `PAYMENT_PENDING`으로 초기화된다.
- **FR-004**: 주문 생성 성공 시 201 Created와 생성된 `orderId`를 반환해야 한다.
- **FR-005**: lessonNo가 null이면 `400 Bad Request`와 `"레슨을 선택해 주세요."` 메시지를 반환해야 한다.
- **FR-006**: lessonOptionNo가 null이면 `400 Bad Request`와 `"수업 옵션을 선택해 주세요."` 메시지를 반환해야 한다.
- **FR-007**: profileId가 null 또는 빈 문자열이면 `400 Bad Request`와 `"구매자 프로필을 입력해 주세요."` 메시지를 반환해야 한다.
- **FR-008**: 존재하지 않는 lessonNo로 요청 시 `404 Not Found`와 `LESSON_NOT_FOUND`를 반환해야 한다.
- **FR-009**: 존재하지 않는 lessonOptionNo로 요청 시 `404 Not Found`와 `LESSON_OPTION_NOT_FOUND`를 반환해야 한다.
- **FR-010**: 존재하지 않는 profileId로 요청 시 `404 Not Found`와 `PROFILE_NOT_FOUND`를 반환해야 한다.
- **FR-011**: 주문 생성 시 price는 해당 레슨의 amount로 초기 설정된다.

### Key Entities

- **Order**: 주문 루트 엔티티. id(UUID), lessonNo, lessonOptionNo, buyer(profileId), price, status, paymentId, discounts를 포함한다.
- **OrderDiscount**: 주문에 적용된 할인 내역. 생성 시점에는 비어 있으며 이후 단계에서 추가된다.
- **Lesson**: 주문 대상 레슨. 존재 여부 확인 및 price 초기값 제공에 사용된다.
- **LessonOption**: 수업 옵션. 존재 여부 확인에 사용된다.
- **Profile**: 구매자. 존재 여부 확인에 사용된다.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 유효한 요청은 100% 201 Created와 UUID 형식의 orderId를 반환한다.
- **SC-002**: 존재하지 않는 리소스 ID 요청은 100% 404 응답과 올바른 에러 코드를 반환한다.
- **SC-003**: 필수 필드 누락 요청은 100% 400 응답과 필드별 에러 메시지를 반환한다.
- **SC-004**: 생성된 주문의 초기 상태는 항상 PAYMENT_PENDING이다.

## Assumptions

- 주문 생성은 인증 없이 접근 가능하다 (기존 API와 동일한 보안 정책 적용).
- lessonOptionNo가 지정된 lessonNo에 속하지 않는 경우는 `LESSON_OPTION_NOT_FOUND`로 처리한다.
- 동일 구매자의 동일 레슨 옵션 중복 주문은 허용한다 (명세에 제한 없음).
- 주문 생성 시 할인은 적용되지 않는다. discounts는 빈 리스트로 초기화된다.
- price 초기값은 Lesson.amount로 설정한다.
- Payment 레코드는 주문 생성 시점에는 생성되지 않는다. paymentId는 결제 완료 후 설정된다.
- 기존 Hexagonal Architecture 패턴을 따른다.
