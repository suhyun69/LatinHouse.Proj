# Implementation Plan: 쿠폰 소유자 배정

**Branch**: `014-coupon-assign-owner` | **Date**: 2026-06-07 | **Spec**: [spec.md](spec.md)

## Summary

`PATCH /api/coupon/{profileId}` 엔드포인트를 구현한다. profileId의 Profile 존재를 확인하고, couponId의 Coupon 존재를 확인한 후, Coupon.owner를 profileId로 업데이트한다. 기존 coupon 도메인 파일들을 확장하는 방식으로 구현한다.

## Technical Context

**Language/Version**: Java 21, Spring Boot 4.x

**Primary Dependencies**: Spring Data JPA, Lombok, Bean Validation, SpringDoc OpenAPI

**Storage**: H2 in-memory (dev), DDL auto

**Testing**: JUnit 5, Mockito, `@WebMvcTest`

**Target Platform**: Linux server / embedded Tomcat

**Project Type**: REST web-service (Hexagonal Architecture)

**Constraints**: 계층 간 의존성 방향 준수. 기존 `FindProfilePort` 재사용.

---

## Constitution Check

| Gate | 결과 | 비고 |
|------|------|------|
| Hexagonal Architecture 준수 | ✅ PASS | Adapter → Application → Domain |
| Two-DTO 패턴 | ✅ PASS | AssignCouponWebRequest/AppRequest 분리 |
| Contract-First API | ✅ PASS | `docs/api-spec.md` 기준 구현 |
| Unified Error Response | ✅ PASS | `CouponNotFoundException` + 핸들러 추가 |
| Validated Inputs | ✅ PASS | `@NotNull couponId`, `@Valid` 적용 |
| Swagger 문서화 | ✅ PASS | `@Operation` 추가 예정 |

---

## Project Structure

### Documentation (this feature)

```text
specs/014-coupon-assign-owner/
├── plan.md
├── research.md
├── data-model.md
├── contracts/
│   └── coupon-assign-api.md
└── tasks.md          ← /speckit-tasks 생성
```

### Source Code 변경

```text
Latinhouse.Be/src/main/java/com/latinhouse/api/

# 신규 파일
common/exception/
└── CouponNotFoundException.java

coupon/application/port/in/
├── AssignCouponUseCase.java
├── AssignCouponAppRequest.java
└── AssignCouponAppResponse.java

coupon/application/port/out/
├── LoadCouponPort.java
└── UpdateCouponPort.java

coupon/application/service/
└── AssignCouponService.java

coupon/adapter/in/web/
├── AssignCouponWebRequest.java
└── AssignCouponWebResponse.java

# 기존 파일 수정
common/exception/
└── GlobalExceptionHandler.java        ← handleCouponNotFoundException() 추가

coupon/adapter/out/persistence/
├── CouponEntity.java                  ← updateOwner() 메서드 추가
└── CouponPersistenceAdapter.java      ← LoadCouponPort, UpdateCouponPort 구현 추가

coupon/adapter/in/web/
├── CouponWebMapper.java               ← AssignCoupon 변환 메서드 추가
└── CouponController.java             ← PATCH /coupon/{profileId} 엔드포인트 추가
```

---

## Implementation Details

### Exception

```java
// CouponNotFoundException
public CouponNotFoundException(Long couponId) {
    super("Coupon not found: " + couponId);
}

// GlobalExceptionHandler 추가
@ExceptionHandler(CouponNotFoundException.class)
@ResponseStatus(HttpStatus.NOT_FOUND)
→ field="couponId", message="쿠폰을 찾을 수 없습니다."
```

### Port Out

```java
LoadCouponPort:   Optional<Coupon> findById(Long couponId)
UpdateCouponPort: Coupon update(Coupon coupon)
```

### Service Logic (AssignCouponService)

```
1. findProfilePort.findById(profileId) → 없으면 ProfileNotFoundException
2. loadCouponPort.findById(couponId)   → 없으면 CouponNotFoundException
3. Coupon updated = Coupon.builder()
       .id(coupon.getId()).templateId(coupon.getTemplateId())
       .owner(profileId).status(coupon.getStatus()).build()
4. updateCouponPort.update(updated)
5. return new AssignCouponAppResponse(updated.getId())
```

### Persistence — CouponPersistenceAdapter.update()

```
CouponEntity entity = couponJpaRepository.findById(coupon.getId()).orElseThrow()
entity.updateOwner(coupon.getOwner())
return CouponPersistenceMapper.toCouponDomain(couponJpaRepository.save(entity))
```

### Controller

```java
@PatchMapping("/coupon/{profileId}")
ResponseEntity<AssignCouponWebResponse> assignCoupon(
    @PathVariable String profileId,
    @Valid @RequestBody AssignCouponWebRequest request)
→ ResponseEntity.ok(...)
```

---

## Verification

1. `./gradlew build` — 컴파일 오류 없음
2. `PATCH /api/coupon/{존재하는 profileId}` + 존재하는 couponId → 200 + `{"couponId": N}`
3. 없는 profileId → 404 + field=profileId
4. 없는 couponId → 404 + field=couponId
5. couponId null → 400 Bad Request
