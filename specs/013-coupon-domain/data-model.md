# Data Model: Coupon 도메인

**Feature**: 013-coupon-domain
**Date**: 2026-06-07

---

## Domain Objects

### CouponTemplate

```
package: com.latinhouse.api.coupon.domain

@Getter @Builder
CouponTemplate {
    Long id
    String title
    CouponTemplateType type
    Long target        // lessonNo when type=LESSON
    BigDecimal amount
}
```

### Coupon

```
package: com.latinhouse.api.coupon.domain

@Getter @Builder
Coupon {
    Long id
    Long templateId
    String owner       // nullable — null until assigned to a user
    CouponStatus status  // default: AVAILABLE
}
```

### Enums

```java
// CouponTemplateType
LESSON

// CouponStatus
AVAILABLE, USED
```

---

## JPA Entities

### CouponTemplateEntity

```
table: coupon_templates

id          BIGINT       PK, AUTO_INCREMENT
title       VARCHAR(255) NOT NULL
type        VARCHAR(20)  NOT NULL  (CouponTemplateType.name())
target      BIGINT       NOT NULL
amount      DECIMAL(15,2) NOT NULL
```

### CouponEntity

```
table: coupons

id           BIGINT       PK, AUTO_INCREMENT
template_id  BIGINT       NOT NULL
owner        VARCHAR(8)   NULL
status       VARCHAR(20)  NOT NULL  (CouponStatus.name(), default 'AVAILABLE')
```

---

## DTO Classes

### Web Layer (`adapter/in/web/`)

| 클래스 | 필드 | 설명 |
|--------|------|------|
| `CreateCouponTemplateWebRequest` | `String title`, `String type`, `Long target`, `BigDecimal amount` | `@NotBlank`/`@NotNull` 적용 |
| `CreateCouponTemplateWebResponse` | `String couponTemplateId` | 생성된 ID 반환 |
| `CreateCouponWebRequest` | `Long templateId`, `Integer count` | `@NotNull @Min(1)` 적용 |
| `CouponWebMapper` | static methods | WebRequest → AppRequest, AppResponse → WebResponse 변환 |

### Application Layer (`application/port/in/`)

| 클래스 | 필드 | 설명 |
|--------|------|------|
| `CreateCouponTemplateAppRequest` | `String title`, `CouponTemplateType type`, `Long target`, `BigDecimal amount` | 도메인 타입 사용 |
| `CreateCouponTemplateAppResponse` | `Long couponTemplateId` | |
| `CreateCouponAppRequest` | `Long templateId`, `Integer count` | |
| `CreateCouponTemplateUseCase` | interface | `CreateCouponTemplateAppResponse createCouponTemplate(CreateCouponTemplateAppRequest)` |
| `CreateCouponUseCase` | interface | `void createCoupon(CreateCouponAppRequest)` |

### Port Out (`application/port/out/`)

| 인터페이스 | 메서드 | 설명 |
|-----------|--------|------|
| `SaveCouponTemplatePort` | `CouponTemplate save(CouponTemplate)` | 템플릿 저장 |
| `LoadCouponTemplatePort` | `Optional<CouponTemplate> findById(Long)` | 템플릿 조회 |
| `SaveCouponPort` | `void saveAll(List<Coupon>)` | 쿠폰 일괄 저장 |

---

## 계층 간 변환 흐름

```
[HTTP] CreateCouponTemplateWebRequest
    → CouponWebMapper.toAppRequest()
    → CreateCouponTemplateAppRequest
    → CreateCouponTemplateService
    → CouponTemplate (domain)
    → CouponPersistenceMapper.toEntity()
    → CouponTemplateEntity (JPA)
    → DB save
    → CouponTemplate (domain)
    → CreateCouponTemplateAppResponse
    → CouponWebMapper.toWebResponse()
    → CreateCouponTemplateWebResponse [HTTP 201]
```

```
[HTTP] CreateCouponWebRequest
    → CouponWebMapper.toAppRequest()
    → CreateCouponAppRequest
    → CreateCouponService
        → LoadCouponTemplatePort.findById() → 없으면 CouponTemplateNotFoundException
        → count 개 Coupon 생성 (owner=null, status=AVAILABLE)
        → SaveCouponPort.saveAll()
    → [HTTP 201 No Body]
```
