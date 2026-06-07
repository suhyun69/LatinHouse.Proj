# Research: 쿠폰 소유자 배정

**Feature**: 014-coupon-assign-owner
**Date**: 2026-06-07

---

## Profile 존재 검증 전략

### Decision: 기존 `FindProfilePort` 재사용
- **Rationale**: `CreateOrderService`에서 이미 `FindProfilePort.findById()`로 Profile 존재를 검증한다. 동일한 패턴 적용.
- **File**: `com.latinhouse.api.profile.application.port.out.FindProfilePort`
- **Alternatives considered**: 새 포트 생성 — 기존 포트로 충분하므로 불필요.

```java
// 기존 사용 예 (CreateOrderService)
Profile profile = findProfilePort.findById(request.getProfileId())
        .orElseThrow(() -> new ProfileNotFoundException(request.getProfileId()));
```

---

## Coupon owner 업데이트 전략

### Decision: `CouponEntity`에 `updateOwner()` 메서드 추가 후 `CouponJpaRepository.save()` 호출
- **Rationale**: JPA dirty checking 또는 명시적 save()로 owner 필드를 업데이트한다. `@Transactional` 서비스 내에서 엔티티를 조회 후 수정하면 dirty checking으로 자동 반영되지만, 명시적 save()가 의도를 명확히 한다.
- **Alternatives considered**:
  - `@Query` JPQL update — 불필요한 복잡도
  - 도메인 객체 재빌드 후 save — owner만 바꾸는데 전체 재구성 불필요

### CouponEntity 변경
`CouponEntity`에 `updateOwner(String owner)` 메서드 추가:
```java
public void updateOwner(String owner) {
    this.owner = owner;
}
```

---

## `CouponNotFoundException` 신규 추가

### Decision: `CouponNotFoundException` 클래스 신규 생성 + `GlobalExceptionHandler` 등록
- **Rationale**: `CouponTemplateNotFoundException` 패턴과 동일. `COUPON_NOT_FOUND` 404 반환.
- **Alternatives considered**: 없음.

---

## 포트 설계

### Decision: `LoadCouponPort` + `UpdateCouponPort` 신규 추가, `CouponPersistenceAdapter`에 구현
- **Rationale**: 기존 Adapter에 인터페이스를 추가하는 방식. 코드 응집도 유지.
- `LoadCouponPort.findById(Long) → Optional<Coupon>`
- `UpdateCouponPort.update(Coupon) → Coupon`

---

## 검증 순서

### Decision: Profile 검증 → Coupon 검증 순서
- **Rationale**: spec Edge Cases에 명시된 검증 순서. profileId가 path param이므로 먼저 검증.
