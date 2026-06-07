# Feature Specification: Coupon 도메인 생성

**Feature Branch**: `013-coupon-domain`

**Created**: 2026-06-07

**Status**: Draft

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 쿠폰 템플릿 생성 (Priority: P1)

관리자가 특정 레슨에 대해 할인 쿠폰 템플릿을 생성한다. 템플릿은 쿠폰 유형, 적용 대상 레슨, 할인 금액을 정의하며, 이후 쿠폰 발행의 기준이 된다.

**Why this priority**: 쿠폰 발행(Story 2)의 전제 조건. 템플릿 없이는 쿠폰을 만들 수 없다.

**Independent Test**: `POST /api/coupon/template` 호출 후 201 응답과 `couponTemplateId`가 반환되면 독립적으로 검증 완료.

**Acceptance Scenarios**:

1. **Given** 유효한 title, type(`LESSON`), target(lessonNo), amount 값이 주어졌을 때, **When** `POST /api/coupon/template`을 호출하면, **Then** 201 Created와 함께 `{"couponTemplateId": "<id>"}` 응답이 반환된다.
2. **Given** 필수 필드 중 하나가 누락된 요청이 주어졌을 때, **When** `POST /api/coupon/template`을 호출하면, **Then** 400 Bad Request가 반환된다.
3. **Given** amount가 null인 요청이 주어졌을 때, **When** `POST /api/coupon/template`을 호출하면, **Then** 400 Bad Request가 반환된다.

---

### User Story 2 - 쿠폰 일괄 발행 (Priority: P2)

관리자가 기존 쿠폰 템플릿을 기반으로 지정한 수량만큼 쿠폰을 일괄 발행한다. 발행된 쿠폰은 소유자 미배정(owner=null) 상태로 생성되어 이후 사용자에게 배포될 수 있다.

**Why this priority**: 실제 사용자에게 쿠폰을 제공하기 위한 핵심 기능. 템플릿 생성 후 다음 단계.

**Independent Test**: 유효한 templateId와 count로 `POST /api/coupon`을 호출했을 때 201 응답이 반환되고, 내부적으로 count 개수만큼 쿠폰이 생성되었음을 확인.

**Acceptance Scenarios**:

1. **Given** 존재하는 templateId와 count=5가 주어졌을 때, **When** `POST /api/coupon`을 호출하면, **Then** 201 Created (Body 없음)가 반환되고 5개의 쿠폰이 생성된다.
2. **Given** 존재하지 않는 templateId가 주어졌을 때, **When** `POST /api/coupon`을 호출하면, **Then** 404 Not Found와 에러 코드 `COUPON_TEMPLATE_NOT_FOUND`가 반환된다.
3. **Given** count=0인 요청이 주어졌을 때, **When** `POST /api/coupon`을 호출하면, **Then** 400 Bad Request가 반환된다.

---

### Edge Cases

- count에 음수(-1, -100)가 입력되면 400 Bad Request를 반환한다.
- 동일한 templateId로 여러 번 발행 호출 시 매 호출마다 count 개수만큼 추가 쿠폰이 생성된다.
- 발행된 쿠폰의 owner는 null, status는 AVAILABLE로 초기화된다.

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: 시스템은 title, type, target, amount를 입력받아 쿠폰 템플릿을 생성할 수 있어야 한다.
- **FR-002**: 쿠폰 템플릿 생성 시 모든 필드(title, type, target, amount)는 필수이며 null을 허용하지 않는다.
- **FR-003**: 쿠폰 템플릿의 type은 `LESSON` 값만 허용한다 (`CouponTemplateType` enum).
- **FR-004**: 쿠폰 템플릿 생성 성공 시 생성된 템플릿의 ID를 반환한다.
- **FR-005**: 시스템은 templateId와 count를 입력받아 쿠폰을 count 개수만큼 일괄 생성할 수 있어야 한다.
- **FR-006**: 쿠폰 발행 시 templateId에 해당하는 쿠폰 템플릿이 존재하지 않으면 404 에러(`COUPON_TEMPLATE_NOT_FOUND`)를 반환한다.
- **FR-007**: 쿠폰 발행 시 count는 1 이상이어야 하며, 미만이면 400 에러를 반환한다.
- **FR-008**: 발행된 쿠폰은 owner=null, status=AVAILABLE 상태로 초기화된다.
- **FR-009**: 쿠폰 발행 성공 시 응답 Body 없이 201 Created를 반환한다.

### Key Entities

- **CouponTemplate**: 쿠폰의 속성을 정의하는 템플릿. (id, title, type, target, amount)
- **Coupon**: 템플릿을 기반으로 발행된 개별 쿠폰. (id, templateId, owner, status)
- **CouponTemplateType**: 쿠폰 유형 enum. `LESSON` (특정 레슨 할인)
- **CouponStatus**: 쿠폰 상태 enum. `AVAILABLE`(사용 가능, 초기값) / `USED`(사용 완료)

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: `POST /api/coupon/template` 호출 시 모든 필수 필드가 제공되면 100% 성공률로 201 응답과 `couponTemplateId`가 반환된다.
- **SC-002**: `POST /api/coupon` 호출 시 유효한 templateId와 count가 제공되면 count 개수와 정확히 일치하는 쿠폰이 생성된다.
- **SC-003**: 존재하지 않는 templateId로 쿠폰 발행 시 100% 케이스에서 `COUPON_TEMPLATE_NOT_FOUND` 에러가 반환된다.
- **SC-004**: 필수 필드 누락 또는 유효하지 않은 count(0 이하) 입력 시 100% 케이스에서 400 Bad Request가 반환된다.

---

## Assumptions

- 쿠폰 템플릿의 target은 type=LESSON일 때 lessonNo(Long)를 의미한다.
- 쿠폰 발행 시 owner 배정 기능(특정 사용자에게 쿠폰 할당)은 이번 스코프에 포함되지 않는다. 발행 후 owner는 null.
- CouponTemplate의 id는 DB auto-increment로 자동 생성된다.
- Coupon의 id도 DB auto-increment로 자동 생성된다.
- 쿠폰 조회, 사용(status 변경), 삭제 기능은 이번 스코프 밖이다.
- 기존 헥사고날(Ports & Adapters) 아키텍처 패턴을 동일하게 따른다.
