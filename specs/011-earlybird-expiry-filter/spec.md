# Feature Specification: 주문 생성 시 EARLYBIRD 할인 만료일 필터링

**Feature Branch**: `011-earlybird-expiry-filter`

**Created**: 2026-06-04

**Status**: Draft

**Input**: User description: "api-spec.md의 POST api/order 관련하여 업데이트 된 내용이 있어. 명세를 확인하고 변경사항을 구현해"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 만료된 EARLYBIRD 할인 제외 (Priority: P1)

주문 생성 시점에 이미 기한이 지난 EARLYBIRD 할인은 주문 할인 내역에 포함되지 않는다. EARLYBIRD 할인의 `condition` 날짜가 주문 생성 시점보다 이전이면 해당 항목은 제외된다.

**Why this priority**: 만료된 할인이 적용되면 비즈니스 손실이 발생한다. 가장 우선 처리해야 할 정합성 요구사항이다.

**Independent Test**: EARLYBIRD condition이 과거 날짜인 레슨으로 `POST /api/order`를 호출했을 때 `Order.discounts`에 해당 항목이 없음을 확인할 수 있다.

**Acceptance Scenarios**:

1. **Given** EARLYBIRD condition="2026-01-01"(과거)이 있는 레슨, **When** 2026-06-04에 주문 생성, **Then** Order.discounts에 해당 EARLYBIRD 항목이 포함되지 않는다.
2. **Given** EARLYBIRD condition="2026-06-04"(오늘)이 있는 레슨, **When** 2026-06-04에 주문 생성, **Then** Order.discounts에 해당 EARLYBIRD 항목이 포함된다 (당일은 유효).
3. **Given** EARLYBIRD condition이 모두 과거인 레슨, **When** 주문 생성, **Then** Order.discounts에 EARLYBIRD 항목이 없다.

---

### User Story 2 - 유효한 EARLYBIRD 중 가장 이른 1건 선택 (Priority: P2)

주문 생성 시점 기준으로 아직 유효한(condition >= now) EARLYBIRD 할인이 복수 존재할 때, 그 중 condition 날짜가 가장 이른 1건만 주문에 적용된다.

**Why this priority**: 유효성 필터링 후 선택 규칙이 이전 피처(010)의 단순 최솟값 선택에서 "유효한 것 중 최솟값"으로 변경된다.

**Independent Test**: 유효한 EARLYBIRD 2건이 있을 때 `POST /api/order` 호출 후 condition이 더 이른 1건만 포함되는지 확인할 수 있다.

**Acceptance Scenarios**:

1. **Given** EARLYBIRD condition="2026-07-01"(유효), condition="2026-08-01"(유효) 2건이 있는 레슨, **When** 주문 생성, **Then** condition="2026-07-01" 1건만 포함된다.
2. **Given** EARLYBIRD condition="2026-01-01"(만료), condition="2026-07-01"(유효) 2건이 있는 레슨, **When** 주문 생성, **Then** condition="2026-07-01" 1건만 포함된다 (만료 제외 후 유효한 것 중 최솟값).
3. **Given** 유효한 EARLYBIRD가 1건도 없는 레슨, **When** 주문 생성, **Then** Order.discounts에 EARLYBIRD 항목이 없다.

---

### Edge Cases

- EARLYBIRD condition이 정확히 오늘 날짜(`now`)와 같은 경우 → 유효(포함)로 처리한다.
- 레슨에 EARLYBIRD 할인이 아예 없는 경우 → Order.discounts에 EARLYBIRD 항목 없음 (정상).
- SEX 할인과 EARLYBIRD 할인이 함께 있는 경우 → SEX 규칙은 변경 없이 독립 처리, EARLYBIRD만 이번 변경 적용.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: EARLYBIRD 할인 후보 선정 시, `LessonDiscount.condition`(yyyy-MM-dd)을 주문 생성 시점의 날짜(`LocalDate.now()`)와 비교하여 `condition < now`인 항목은 후보에서 제외한다.
- **FR-002**: 유효한(`condition >= now`) EARLYBIRD 후보 중 `condition` 날짜가 가장 이른 1건만 `Order.discounts`에 적용한다.
- **FR-003**: 유효한 EARLYBIRD 후보가 없는 경우 EARLYBIRD 할인을 적용하지 않는다 (빈 상태 유지).
- **FR-004**: condition이 오늘 날짜와 동일한 EARLYBIRD 할인은 유효한 것으로 처리한다(`condition >= now`).
- **FR-005**: SEX 할인 적용 규칙 및 기타 주문 생성 로직은 변경하지 않는다.

### Key Entities

- **LessonDiscount**: `type`(EARLYBIRD), `condition`(yyyy-MM-dd 날짜 문자열), `amount`, `id` — condition 날짜가 주문 생성일 이상인 경우만 유효 후보.
- **Order**: `discounts` 리스트에 유효성 검증을 통과한 EARLYBIRD 할인만 포함.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: condition이 주문 생성일보다 이전인 EARLYBIRD 할인은 100% 제외된다.
- **SC-002**: condition이 주문 생성일 이상인 EARLYBIRD 할인 중 정확히 1건(가장 이른 것)만 포함된다.
- **SC-003**: 유효한 EARLYBIRD가 없으면 Order.discounts의 EARLYBIRD 항목은 항상 0건이다.
- **SC-004**: SEX 할인 및 기존 주문 생성 로직은 변경 없이 동작한다 (회귀 없음).

## Assumptions

- `condition` 날짜 비교는 시각(time)이 아닌 날짜(date) 단위로 수행한다. 당일(`condition == now`)은 유효로 처리한다.
- `LessonDiscount.condition`은 항상 유효한 `yyyy-MM-dd` 형식임이 보장된다 (형식 검증은 이 피처 범위 밖).
- 이 피처는 기존 010-order-discount-apply 구현 위에 EARLYBIRD 필터링 조건만 추가하는 변경이다.
- 주문 생성 요청/응답 구조(`orderId` 반환)는 변경되지 않는다.
