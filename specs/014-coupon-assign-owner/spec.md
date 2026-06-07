# Feature Specification: 쿠폰 소유자 배정

**Feature Branch**: `014-coupon-assign-owner`

**Created**: 2026-06-07

**Status**: Draft

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 쿠폰 소유자 배정 (Priority: P1)

관리자가 특정 쿠폰을 지정된 프로필(사용자)에게 배정한다. 배정 후 쿠폰의 소유자(owner)가 해당 프로필 ID로 업데이트된다.

**Why this priority**: 이 기능이 이 스펙의 유일한 핵심 기능이다. 쿠폰 발행 후 사용자에게 배포하기 위한 필수 단계.

**Independent Test**: `PATCH /api/coupon/{profileId}` 호출 후 200 OK + `{"couponId": <id>}` 응답 반환, 해당 쿠폰의 owner가 profileId로 변경되었음을 확인.

**Acceptance Scenarios**:

1. **Given** 존재하는 profileId와 존재하는 couponId가 주어졌을 때, **When** `PATCH /api/coupon/{profileId}`를 호출하면, **Then** 200 OK와 `{"couponId": <id>}` 응답이 반환되고 Coupon.owner가 profileId로 업데이트된다.
2. **Given** 존재하지 않는 profileId가 주어졌을 때, **When** `PATCH /api/coupon/{profileId}`를 호출하면, **Then** 404 Not Found와 에러 코드 `PROFILE_NOT_FOUND`가 반환된다.
3. **Given** 존재하지 않는 couponId가 요청 바디에 주어졌을 때, **When** `PATCH /api/coupon/{profileId}`를 호출하면, **Then** 404 Not Found와 에러 코드 `COUPON_NOT_FOUND`가 반환된다.
4. **Given** couponId가 null인 요청이 주어졌을 때, **When** `PATCH /api/coupon/{profileId}`를 호출하면, **Then** 400 Bad Request가 반환된다.

---

### Edge Cases

- couponId가 null이거나 누락된 경우 400 Bad Request를 반환한다.
- profileId와 couponId 둘 다 존재하지 않을 경우 profileId 검증이 먼저 수행된다 (순서: Profile 조회 → Coupon 조회).
- 이미 owner가 배정된 쿠폰에 재배정하는 경우 덮어쓰기 허용한다 (별도 제약 없음).

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: 시스템은 path의 profileId로 Profile을 조회하고, 존재하지 않으면 404 에러(`PROFILE_NOT_FOUND`)를 반환해야 한다.
- **FR-002**: 시스템은 요청 바디의 couponId로 Coupon을 조회하고, 존재하지 않으면 404 에러(`COUPON_NOT_FOUND`)를 반환해야 한다.
- **FR-003**: Profile과 Coupon이 모두 존재하면 Coupon.owner를 profileId로 업데이트해야 한다.
- **FR-004**: 업데이트 성공 시 200 OK와 함께 `{"couponId": <id>}` 응답을 반환해야 한다.
- **FR-005**: couponId 필드는 필수이며 null을 허용하지 않는다. null이면 400 Bad Request를 반환한다.

### Key Entities

- **Profile**: 쿠폰을 배정받는 사용자. id(String)로 식별. 존재 여부를 사전 검증한다.
- **Coupon**: 소유자가 배정될 쿠폰. id(Long)로 식별. owner(String) 필드를 profileId로 업데이트한다.

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 유효한 profileId와 couponId 조합으로 요청 시 100% 케이스에서 200 OK와 `couponId`가 반환된다.
- **SC-002**: 존재하지 않는 profileId 요청 시 100% 케이스에서 `PROFILE_NOT_FOUND` 404 에러가 반환된다.
- **SC-003**: 존재하지 않는 couponId 요청 시 100% 케이스에서 `COUPON_NOT_FOUND` 404 에러가 반환된다.
- **SC-004**: 성공 배정 후 해당 쿠폰의 소유자가 지정된 profileId와 100% 일치한다.

---

## Assumptions

- Profile 존재 확인은 기존 `FindProfilePort`를 재사용한다 (Order 도메인에서 사용 중).
- 이미 owner가 배정된 쿠폰도 재배정을 허용한다 (중복 배정 제한 없음).
- CouponStatus 변경(AVAILABLE → 등)은 이번 스코프 밖이다. owner만 업데이트한다.
- 기존 Hexagonal Architecture 패턴을 동일하게 따른다.
