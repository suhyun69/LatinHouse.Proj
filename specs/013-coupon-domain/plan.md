# Implementation Plan: Coupon 도메인 생성

**Branch**: `013-coupon-domain` | **Date**: 2026-06-07 | **Spec**: [spec.md](spec.md)

## Summary

쿠폰 템플릿을 생성하고 템플릿 기반으로 쿠폰을 일괄 발행하는 두 개의 REST 엔드포인트(`POST /api/coupon/template`, `POST /api/coupon`)를 구현한다. 기존 Hexagonal Architecture 패턴(Order 도메인)을 그대로 따른다.

## Technical Context

**Language/Version**: Java 21, Spring Boot 4.x

**Primary Dependencies**: Spring Data JPA, Lombok, Bean Validation (`@NotNull`, `@Min`), SpringDoc OpenAPI

**Storage**: H2 in-memory (dev), DDL auto-create

**Testing**: JUnit 5, Mockito, `@WebMvcTest`

**Target Platform**: Linux server / embedded Tomcat

**Project Type**: REST web-service (Hexagonal Architecture)

**Performance Goals**: 기능 정확성 우선. 성능 목표 없음 (현재 스코프).

**Constraints**: 계층 간 의존성 방향 준수 (Adapter → Application → Domain), Domain은 외부 의존 금지.

**Scale/Scope**: 단일 백엔드 서비스, 단일 `coupon` 도메인 패키지 신규 추가.

---

## Constitution Check

| Gate | 결과 | 비고 |
|------|------|------|
| Hexagonal Architecture 준수 | ✅ PASS | Adapter → Application → Domain 단방향 의존 |
| Two-DTO 패턴 | ✅ PASS | WebRequest/AppRequest 분리, 변환은 Mapper 담당 |
| Contract-First API | ✅ PASS | `docs/api-spec.md` Coupon API 섹션 기준 구현 |
| Unified Error Response | ✅ PASS | `GlobalExceptionHandler`에 `CouponTemplateNotFoundException` 핸들러 추가 |
| Validated Inputs | ✅ PASS | `@NotNull`, `@NotBlank`, `@Min(1)` 적용, `@Valid` 사용 |
| Swagger 문서화 | ✅ PASS | `@Tag`, `@Operation` 추가 예정 |

---

## Project Structure

### Documentation (this feature)

```text
specs/013-coupon-domain/
├── plan.md
├── research.md
├── data-model.md
├── contracts/
│   └── coupon-api.md
└── tasks.md          ← /speckit-tasks 생성
```

### Source Code

```text
Latinhouse.Be/src/main/java/com/latinhouse/api/
└── coupon/
    ├── domain/
    │   ├── CouponTemplate.java
    │   ├── CouponTemplateType.java
    │   ├── Coupon.java
    │   └── CouponStatus.java
    ├── application/
    │   ├── port/
    │   │   ├── in/
    │   │   │   ├── CreateCouponTemplateUseCase.java
    │   │   │   ├── CreateCouponTemplateAppRequest.java
    │   │   │   ├── CreateCouponTemplateAppResponse.java
    │   │   │   ├── CreateCouponUseCase.java
    │   │   │   └── CreateCouponAppRequest.java
    │   │   └── out/
    │   │       ├── SaveCouponTemplatePort.java
    │   │       ├── LoadCouponTemplatePort.java
    │   │       └── SaveCouponPort.java
    │   └── service/
    │       ├── CreateCouponTemplateService.java
    │       └── CreateCouponService.java
    └── adapter/
        ├── in/web/
        │   ├── CouponController.java
        │   ├── CreateCouponTemplateWebRequest.java
        │   ├── CreateCouponTemplateWebResponse.java
        │   ├── CreateCouponWebRequest.java
        │   └── CouponWebMapper.java
        └── out/persistence/
            ├── CouponTemplateEntity.java
            ├── CouponEntity.java
            ├── CouponTemplateJpaRepository.java
            ├── CouponJpaRepository.java
            ├── CouponPersistenceAdapter.java
            └── CouponPersistenceMapper.java

Latinhouse.Be/src/main/java/com/latinhouse/api/common/exception/
└── CouponTemplateNotFoundException.java   ← 신규
    GlobalExceptionHandler.java            ← 핸들러 추가
```

---

## Implementation Details

### Domain Layer

```java
// CouponTemplate.java
@Getter @Builder
public class CouponTemplate {
    private Long id;
    private String title;
    private CouponTemplateType type;
    private Long target;
    private BigDecimal amount;
}

// Coupon.java
@Getter @Builder
public class Coupon {
    private Long id;
    private Long templateId;
    private String owner;       // nullable
    private CouponStatus status; // default AVAILABLE
}
```

### Application Port In (UseCase)

```java
// CreateCouponTemplateUseCase.java
public interface CreateCouponTemplateUseCase {
    CreateCouponTemplateAppResponse createCouponTemplate(CreateCouponTemplateAppRequest request);
}

// CreateCouponUseCase.java
public interface CreateCouponUseCase {
    void createCoupon(CreateCouponAppRequest request);
}
```

### Service Logic

**CreateCouponTemplateService**:
1. `AppRequest` → `CouponTemplate` 도메인 객체 생성 (id=null, DB 위임)
2. `SaveCouponTemplatePort.save()` → 저장된 `CouponTemplate` 반환
3. `CreateCouponTemplateAppResponse(savedTemplate.getId())` 반환

**CreateCouponService**:
1. `LoadCouponTemplatePort.findById(templateId)` → `Optional.empty()`이면 `CouponTemplateNotFoundException`
2. `count` 개수만큼 `Coupon` 빌드 (templateId=request.templateId, owner=null, status=AVAILABLE)
3. `SaveCouponPort.saveAll(coupons)` 호출
4. void 반환 → Controller에서 201 반환

### Web Adapter

```java
// CouponController.java
@Tag(name = "Coupon", description = "쿠폰 관리 API")
@RestController @RequestMapping("/api") @RequiredArgsConstructor
class CouponController {

    @Operation(summary = "쿠폰 템플릿 생성")
    @PostMapping("/coupon/template")
    ResponseEntity<CreateCouponTemplateWebResponse> createCouponTemplate(
            @Valid @RequestBody CreateCouponTemplateWebRequest request) {
        // CouponWebMapper.toAppRequest → useCase → CouponWebMapper.toWebResponse → 201
    }

    @Operation(summary = "쿠폰 일괄 발행")
    @PostMapping("/coupon")
    ResponseEntity<Void> createCoupon(
            @Valid @RequestBody CreateCouponWebRequest request) {
        // CouponWebMapper.toAppRequest → useCase → 201 No Body
    }
}
```

### Validation (WebRequest)

```java
// CreateCouponTemplateWebRequest
@NotBlank String title
@NotNull String type
@NotNull Long target
@NotNull BigDecimal amount

// CreateCouponWebRequest
@NotNull Long templateId
@NotNull @Min(1) Integer count
```

### Persistence

```java
// CouponTemplateEntity — @GeneratedValue(strategy = GenerationType.IDENTITY)
// CouponEntity — @GeneratedValue(strategy = GenerationType.IDENTITY)
// CouponPersistenceAdapter implements SaveCouponTemplatePort, LoadCouponTemplatePort, SaveCouponPort
```

### Exception

```java
// CouponTemplateNotFoundException
public class CouponTemplateNotFoundException extends RuntimeException {
    public CouponTemplateNotFoundException(Long templateId) {
        super("CouponTemplate not found: " + templateId);
    }
}
// GlobalExceptionHandler — handleCouponTemplateNotFoundException() 추가 → 404 + field=templateId
```

---

## Verification

1. `./gradlew build` — 컴파일 오류 없음
2. 앱 기동 → Swagger UI에서 `POST /api/coupon/template`, `POST /api/coupon` 확인
3. `POST /api/coupon/template` 유효 요청 → 201 + `couponTemplateId` 반환
4. `POST /api/coupon` 유효 요청 → 201 No Body
5. `POST /api/coupon` 없는 templateId → 404 + `COUPON_TEMPLATE_NOT_FOUND`
6. `POST /api/coupon` count=0 → 400 Bad Request
