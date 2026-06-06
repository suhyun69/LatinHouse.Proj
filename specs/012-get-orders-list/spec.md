# Feature Specification: 주문 목록 조회 (GET /api/orders)

**Feature Branch**: `012-get-orders-list`

**Created**: 2026-06-04

**Status**: Draft

**Input**: User description: "docs/api-spec.md의 GET api/orders 명세를 확인하고 구현에 필요한 요구사항을 정리해줘"

## User Scenarios & Testing *(mandatory)*

### User Story 1 — 내 주문 목록 조회 (Priority: P1)

구매자는 자신의 Profile ID를 기준으로 자신이 생성한 주문 목록을 조회할 수 있다. 각 주문 항목에는 주문 ID, 레슨 번호, 옵션 번호, 금액, 상태, 적용 할인 목록이 포함된다.

**Why this priority**: 구매자가 자신의 주문 내역을 확인하는 가장 기본적인 조회 기능이다. 결제 전 주문 확인, 내역 조회 등 핵심 사용 흐름이다.

**Independent Test**: `GET /api/orders?buyer=Ab2Cd3Ef` 호출 시 해당 buyer가 생성한 주문 목록이 반환되고, 다른 buyer의 주문은 포함되지 않는다.

**Acceptance Scenarios**:

1. **Given** buyer=Ab2Cd3Ef인 주문 2건이 존재, **When** `GET /api/orders?buyer=Ab2Cd3Ef`, **Then** 2건의 주문 목록 반환
2. **Given** buyer=Ab2Cd3Ef인 주문 없음, **When** `GET /api/orders?buyer=Ab2Cd3Ef`, **Then** 빈 배열 `[]` 반환
3. **Given** 여러 buyer의 주문이 존재, **When** `GET /api/orders?buyer=Ab2Cd3Ef`, **Then** 해당 buyer의 주문만 반환

---

### User Story 2 — 레슨별 주문 목록 조회 (Priority: P2)

운영자 또는 강사는 특정 레슨에 대한 주문 목록을 lessonNo 기준으로 조회할 수 있다.

**Why this priority**: 특정 레슨의 신청자 현황을 파악하는 관리 기능이다.

**Independent Test**: `GET /api/orders?lessonNo=1` 호출 시 해당 레슨의 주문 목록이 반환된다.

**Acceptance Scenarios**:

1. **Given** lessonNo=1인 주문 3건 존재, **When** `GET /api/orders?lessonNo=1`, **Then** 3건 반환
2. **Given** lessonNo=1인 주문 없음, **When** `GET /api/orders?lessonNo=1`, **Then** 빈 배열 반환

---

### User Story 3 — 복합 조건 조회 (Priority: P3)

buyer와 lessonNo를 동시에 지정하면 두 조건을 모두 만족하는 주문만 반환된다.

**Why this priority**: 특정 구매자의 특정 레슨 주문 확인 시 유용하다.

**Independent Test**: `GET /api/orders?buyer=Ab2Cd3Ef&lessonNo=1` 호출 시 두 조건을 AND로 만족하는 주문만 반환된다.

**Acceptance Scenarios**:

1. **Given** buyer=Ab2Cd3Ef & lessonNo=1인 주문 1건 존재, **When** 두 파라미터 동시 지정, **Then** 1건만 반환
2. **Given** buyer=Ab2Cd3Ef & lessonNo=99인 주문 없음, **When** 두 파라미터 동시 지정, **Then** 빈 배열 반환

---

### Edge Cases

- 두 파라미터 모두 생략 시 → 전체 주문 목록 반환 (빈 배열 가능)
- 존재하지 않는 buyer/lessonNo 지정 시 → 에러 없이 빈 배열 반환
- 할인이 없는 주문 → `discounts: []` 빈 배열로 반환
- 할인이 복수인 주문 → 모든 할인 항목 포함

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: `GET /api/orders` 요청 시 조건에 맞는 주문 목록을 배열로 반환한다.
- **FR-002**: `buyer` 파라미터가 지정된 경우 해당 Profile ID의 주문만 반환한다.
- **FR-003**: `lessonNo` 파라미터가 지정된 경우 해당 레슨 번호의 주문만 반환한다.
- **FR-004**: 두 파라미터 동시 지정 시 AND 조건으로 필터링한다.
- **FR-005**: 두 파라미터 모두 생략 시 전체 주문 목록을 반환한다.
- **FR-006**: 조건에 해당하는 주문이 없으면 빈 배열을 반환한다. 에러를 발생시키지 않는다.
- **FR-007**: 각 주문 항목은 orderId, lessonNo, lessonOptionNo, price, status, discounts를 포함한다.
- **FR-008**: discounts는 해당 주문에 적용된 모든 할인 항목(discountType, discountId, amount)을 포함한다.

### Key Entities

- **Order**: orderId(String/UUID), lessonNo(Long), lessonOptionNo(Long), price(BigDecimal), status(OrderStatus), discounts(List)
- **OrderDiscount**: discountType(String), discountId(Long), amount(BigDecimal)

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: buyer 파라미터 지정 시 해당 buyer의 주문만 100% 정확히 반환된다.
- **SC-002**: lessonNo 파라미터 지정 시 해당 레슨의 주문만 100% 정확히 반환된다.
- **SC-003**: AND 복합 조건 시 두 조건을 모두 만족하는 주문만 반환된다.
- **SC-004**: 조건에 맞는 주문 없을 때 빈 배열이 반환되고 에러가 발생하지 않는다.
- **SC-005**: 응답의 각 주문 항목에 FR-007에서 명시한 모든 필드가 포함된다.

## Assumptions

- 파라미터 유효성 검증(존재하지 않는 buyer, 음수 lessonNo 등)은 별도 에러 없이 빈 배열로 처리한다.
- 정렬 순서는 명세에 지정되어 있지 않아 별도 정렬 기준 없음(DB 기본 순서 반환)으로 가정한다.
- 페이징은 이 피처 범위 외로 가정한다.
- 인증/인가는 이 피처 범위 외로 가정한다 (기존 SecurityConfig에 허용 추가 필요).
