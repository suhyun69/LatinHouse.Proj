# Data Model: 쿠폰 소유자 배정

**Feature**: 014-coupon-assign-owner
**Date**: 2026-06-07

---

## 변경되는 Domain Objects

### Coupon (기존 — 변경 없음)

```
Coupon {
    Long id
    Long templateId
    String owner       // null → profileId 로 업데이트됨
    CouponStatus status
}
```

---

## 변경되는 JPA Entity

### CouponEntity (기존 파일에 메서드 추가)

```
파일: coupon/adapter/out/persistence/CouponEntity.java

추가 메서드:
    public void updateOwner(String owner) {
        this.owner = owner;
    }
```

---

## 신규 DTO Classes

### Web Layer (`adapter/in/web/`)

| 클래스 | 필드 | 설명 |
|--------|------|------|
| `AssignCouponWebRequest` | `@NotNull Long couponId` | 배정 요청 DTO |
| `AssignCouponWebResponse` | `Long couponId` | 배정 응답 DTO |

### Application Layer (`application/port/in/`)

| 클래스 | 필드 | 설명 |
|--------|------|------|
| `AssignCouponAppRequest` | `String profileId`, `Long couponId` | 애플리케이션 계층 요청 |
| `AssignCouponAppResponse` | `Long couponId` | 애플리케이션 계층 응답 |
| `AssignCouponUseCase` | interface | `AssignCouponAppResponse assignCoupon(AssignCouponAppRequest)` |

### Port Out (`application/port/out/`)

| 인터페이스 | 메서드 | 설명 |
|-----------|--------|------|
| `LoadCouponPort` | `Optional<Coupon> findById(Long couponId)` | 쿠폰 단건 조회 |
| `UpdateCouponPort` | `Coupon update(Coupon coupon)` | 쿠폰 owner 업데이트 저장 |

---

## 계층 간 변환 흐름

```
[HTTP] PATCH /api/coupon/{profileId}
    Body: { couponId: 1 }

    → AssignCouponWebRequest
    → CouponWebMapper.toAppRequest(profileId, webRequest)
    → AssignCouponAppRequest(profileId, couponId)
    → AssignCouponService
        1. FindProfilePort.findById(profileId) → 없으면 ProfileNotFoundException
        2. LoadCouponPort.findById(couponId)   → 없으면 CouponNotFoundException
        3. Coupon 도메인 재빌드 (owner=profileId)
        4. UpdateCouponPort.update(coupon)
    → AssignCouponAppResponse(couponId)
    → CouponWebMapper.toWebResponse(appResponse)
    → AssignCouponWebResponse [HTTP 200]
```

---

## 기존 파일 변경 목록

| 파일 | 변경 내용 |
|------|----------|
| `CouponEntity.java` | `updateOwner(String owner)` 메서드 추가 |
| `CouponPersistenceAdapter.java` | `LoadCouponPort`, `UpdateCouponPort` 구현 추가 |
| `CouponWebMapper.java` | `toAppRequest(String, AssignCouponWebRequest)`, `toWebResponse(AssignCouponAppResponse)` 추가 |
| `CouponController.java` | `PATCH /coupon/{profileId}` 엔드포인트 추가 |
| `GlobalExceptionHandler.java` | `handleCouponNotFoundException()` 핸들러 추가 |

## 신규 파일 목록

| 파일 | 분류 |
|------|------|
| `AssignCouponWebRequest.java` | Web DTO |
| `AssignCouponWebResponse.java` | Web DTO |
| `AssignCouponAppRequest.java` | App DTO |
| `AssignCouponAppResponse.java` | App DTO |
| `AssignCouponUseCase.java` | UseCase |
| `LoadCouponPort.java` | Port Out |
| `UpdateCouponPort.java` | Port Out |
| `AssignCouponService.java` | Service |
| `CouponNotFoundException.java` | Exception |
