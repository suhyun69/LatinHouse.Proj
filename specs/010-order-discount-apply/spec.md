# Feature Specification: 주문 생성 시 레슨 할인 자동 적용

**Feature Branch**: `010-order-discount-apply`

**Created**: 2026-06-03

**Status**: Draft

**Input**: User description: "api-spec.md의 POST api/order 관련하여 업데이트 된 내용이 있어. 명세를 확인하고 변경사항을 구현해"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - SEX 할인 자동 적용 (Priority: P1)

구매자가 주문을 생성할 때, 레슨에 설정된 SEX 할인이 구매자의 성별과 일치하면 자동으로 주문 할인 내역에 포함된다.

**Why this priority**: 성별 할인은 레슨에서 가장 흔히 쓰이는 할인 유형이며, 주문 생성의 핵심 정확성에 직결된다.

**Independent Test**: `POST /api/order` 호출 시 응답 orderId로 주문을 조회하면 성별이 일치하는 SEX 할인이 포함되어 있고, 불일치하는 할인은 포함되어 있지 않음을 확인할 수 있다.

**Acceptance Scenarios**:

1. **Given** 레슨에 condition="M" SEX 할인이 있고 구매자 Profile.sex=M인 상태, **When** 주문 생성 요청, **Then** 주문의 discounts에 해당 LessonDiscount가 포함된다.
2. **Given** 레슨에 condition="M" SEX 할인이 있고 구매자 Profile.sex=F인 상태, **When** 주문 생성 요청, **Then** 주문의 discounts에 해당 LessonDiscount가 포함되지 않는다.
3. **Given** 레슨에 condition="F" SEX 할인이 있고 구매자 Profile.sex=F인 상태, **When** 주문 생성 요청, **Then** 주문의 discounts에 해당 LessonDiscount가 포함된다.
4. **Given** 레슨에 SEX 할인이 없는 상태, **When** 주문 생성 요청, **Then** 주문의 discounts는 비어 있다.

---

### User Story 2 - EARLYBIRD 할인 자동 적용 (Priority: P2)

구매자가 주문을 생성할 때, 레슨에 설정된 EARLYBIRD 할인 중 condition(날짜) 기준으로 가장 이른 1건만 자동으로 주문 할인 내역에 포함된다.

**Why this priority**: EARLYBIRD는 SEX와 함께 레슨에서 제공되는 표준 할인 유형이며, 복수 존재 시 선택 규칙이 명확해야 한다.

**Independent Test**: 레슨에 EARLYBIRD 할인이 2건 이상 있을 때 `POST /api/order` 호출 후, 주문 discounts에 condition이 가장 이른 1건만 포함되어 있는지 확인할 수 있다.

**Acceptance Scenarios**:

1. **Given** 레슨에 EARLYBIRD 할인이 1건(condition="2026-05-01") 있는 상태, **When** 주문 생성 요청, **Then** 주문의 discounts에 해당 1건이 포함된다.
2. **Given** 레슨에 EARLYBIRD 할인이 2건(condition="2026-06-01", "2026-05-01") 있는 상태, **When** 주문 생성 요청, **Then** 주문의 discounts에 condition="2026-05-01" 1건만 포함된다.
3. **Given** 레슨에 EARLYBIRD 할인이 없는 상태, **When** 주문 생성 요청, **Then** 주문의 discounts에 EARLYBIRD 항목은 없다.

---

### User Story 3 - SEX + EARLYBIRD 복합 할인 적용 (Priority: P3)

레슨에 SEX 할인과 EARLYBIRD 할인이 함께 존재할 때, 각각의 규칙을 독립적으로 적용하여 해당하는 모든 할인이 주문에 포함된다.

**Why this priority**: 두 유형이 동시에 존재하는 시나리오에서 적용 규칙 간 간섭이 없어야 한다.

**Independent Test**: 레슨에 SEX 할인과 EARLYBIRD 할인이 모두 있을 때 주문 생성 후 discounts에 두 유형 모두 올바르게 포함되는지 확인할 수 있다.

**Acceptance Scenarios**:

1. **Given** 레슨에 condition="M" SEX 할인 + EARLYBIRD 할인 1건이 있고 Profile.sex=M, **When** 주문 생성 요청, **Then** 주문의 discounts에 SEX 할인 1건, EARLYBIRD 할인 1건 총 2건이 포함된다.
2. **Given** 레슨에 condition="M" SEX 할인 + EARLYBIRD 할인 1건이 있고 Profile.sex=F, **When** 주문 생성 요청, **Then** 주문의 discounts에 EARLYBIRD 할인 1건만 포함된다.

---

### Edge Cases

- 레슨에 할인이 전혀 없는 경우 → `Order.discounts`는 빈 리스트로 생성된다.
- EARLYBIRD 할인이 동일한 condition 날짜를 가진 2건이 있는 경우 → 그 중 임의의 1건만 포함한다 (condition이 동일하면 id 오름차순 기준 1건).
- Profile.sex가 null인 경우 → SEX 할인은 적용되지 않는다.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: 주문 생성 시 레슨의 할인 목록(`Lesson.discounts`)을 조회하여 조건에 맞는 항목만 선별한다.
- **FR-002**: SEX 유형 할인은 `LessonDiscount.condition`이 구매자 `Profile.sex`와 일치하는 경우에만 적용한다. 불일치 시 제외한다.
- **FR-003**: EARLYBIRD 유형 할인은 `LessonDiscount.condition`(날짜 문자열, `yyyy-MM-dd`) 기준으로 가장 이른 1건만 적용한다. 복수 존재 시 나머지는 제외한다.
- **FR-004**: 적용 대상으로 선별된 각 LessonDiscount는 `OrderDiscount`로 저장되며, `discountType = LESSON`, `discountId = LessonDiscount.id`, `amount = LessonDiscount.amount`로 설정된다.
- **FR-005**: 적용 가능한 할인이 없는 경우 `Order.discounts`는 빈 리스트로 설정된다.
- **FR-006**: 할인 선별 로직은 SEX와 EARLYBIRD를 독립적으로 처리하며, 두 유형이 함께 있을 경우 각각의 규칙을 모두 적용한다.

### Key Entities

- **LessonDiscount**: 레슨에 설정된 할인 항목. `type`(SEX/EARLYBIRD), `condition`(성별 코드 또는 날짜 문자열), `amount`(할인 금액), `id`를 가진다.
- **OrderDiscount**: 주문에 기록되는 실제 적용 할인. `discountType`(LESSON/COUPON), `discountId`(원본 할인 ID), `amount`(할인 금액)를 가진다.
- **Profile**: 구매자 정보. `sex`(M/F) 필드가 SEX 할인 조건 매칭에 사용된다.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: SEX 할인 조건 매칭 정확도 100% — 성별이 일치하는 경우만 포함되고 불일치하는 경우는 항상 제외된다.
- **SC-002**: EARLYBIRD 할인 선택 정확도 100% — 복수 존재 시 condition이 가장 이른 1건만 포함된다.
- **SC-003**: 레슨에 할인이 없는 경우 주문의 discounts는 항상 빈 리스트로 반환된다.
- **SC-004**: 기존 주문 생성 기능(lessonNo/lessonOptionNo/profileId 검증, orderId 생성)은 변경 없이 동작한다.

## Assumptions

- `LessonDiscount.condition`은 SEX 유형이면 `"M"` 또는 `"F"` 문자열, EARLYBIRD 유형이면 `"yyyy-MM-dd"` 형식 날짜 문자열이다.
- Profile.sex가 null인 경우 SEX 할인은 적용하지 않는다.
- EARLYBIRD 할인의 날짜 유효성(오늘 기준 만료 여부) 검사는 이 피처 범위에 포함되지 않는다 — 명세에 유효성 검사 조건이 명시되어 있지 않으므로 condition 기준 최솟값 1건만 선택한다.
- 기존 `POST /api/order` 엔드포인트의 요청/응답 구조(orderId 반환)는 변경되지 않는다.
- 주문 생성 자체의 검증 로직(lessonNo/lessonOptionNo/profileId 404 처리)은 이미 구현되어 있으며 이 피처에서 수정하지 않는다.
