# Implementation Plan: 주문 생성 시 레슨 할인 자동 적용

**Branch**: `010-order-discount-apply` | **Date**: 2026-06-03 | **Spec**: [spec.md](spec.md)

## Summary

`POST /api/order` 주문 생성 시 레슨의 할인 목록(`Lesson.discounts`)을 조회하여 구매자 조건에 맞는 항목만 `Order.discounts`에 자동 적용한다. SEX 할인은 Profile.sex 일치 여부로, EARLYBIRD 할인은 condition(날짜) 최솟값 1건으로 선별한다. `CreateOrderService`에 할인 선별 로직을 추가하는 것이 유일한 변경사항이다.

## Technical Context

**Language/Version**: Java 21 (Spring Boot 4.x)

**Primary Dependencies**: Spring Data JPA, Lombok, Jakarta Validation

**Storage**: H2 (test) / MySQL (prod) — 기존 order_discount 테이블 재사용

**Testing**: JUnit 5, Mockito, Spring Boot Test (WebMvcTest)

**Target Platform**: Linux server (Spring Boot 내장 Tomcat)

**Project Type**: Web service (Hexagonal Architecture)

**Performance Goals**: 기존 주문 생성 응답시간 유지 (할인 선별은 in-memory 연산)

**Constraints**: Hexagonal Architecture 준수, Domain 계층 외부 의존 금지

**Scale/Scope**: 기존 `CreateOrderService` 단일 메서드 수정

## Constitution Check

| 원칙 | 준수 여부 | 비고 |
|------|-----------|------|
| Hexagonal Architecture | ✅ | 할인 선별 로직을 Application Service에 위치 |
| Two-DTO 패턴 | ✅ | Request/Response 변경 없음 |
| Contract-First API | ✅ | api-spec.md의 할인 규칙 섹션 기반 구현 |
| Unified Error Response | ✅ | 에러 응답 변경 없음 |
| Validated Inputs | ✅ | 입력 검증 변경 없음 |
| Domain 계층 외부 의존 금지 | ✅ | 할인 선별 로직이 Domain이 아닌 Service에 위치 |

## Project Structure

### Documentation (this feature)

```text
specs/010-order-discount-apply/
├── plan.md              # 이 파일
├── spec.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── post-order.md
└── tasks.md             # /speckit-tasks 출력
```

### Source Code 변경 대상

```text
Latinhouse.Be/src/main/java/com/latinhouse/api/
└── order/
    └── application/
        └── service/
            └── CreateOrderService.java       ← 수정

Latinhouse.Be/src/test/java/com/latinhouse/api/
└── order/
    └── application/
        └── service/
            └── CreateOrderServiceTest.java   ← 수정 (테스트 추가)
```

**변경 없는 파일**: Controller, WebRequest/Response, Mapper, Entity, Repository, Port 인터페이스, PersistenceAdapter, PersistenceMapper — 모두 그대로 유지.

## 핵심 구현 로직

```java
// CreateOrderService.createOrder() 내부 — 할인 선별 로직
private List<OrderDiscount> resolveDiscounts(Lesson lesson, Profile profile) {
    List<OrderDiscount> result = new ArrayList<>();

    List<LessonDiscount> discounts = lesson.getDiscounts();
    if (discounts == null || discounts.isEmpty()) return result;

    // SEX 할인
    if (profile.getSex() != null) {
        discounts.stream()
            .filter(d -> d.getType() == DiscountType.SEX)
            .filter(d -> d.getCondition().equals(profile.getSex().name()))
            .map(d -> toOrderDiscount(d))
            .forEach(result::add);
    }

    // EARLYBIRD 할인 — condition 최솟값 1건
    discounts.stream()
        .filter(d -> d.getType() == DiscountType.EARLYBIRD)
        .min(Comparator.comparing(LessonDiscount::getCondition))
        .map(d -> toOrderDiscount(d))
        .ifPresent(result::add);

    return result;
}
```

## Complexity Tracking

없음 — 헌법 위반 없음.
