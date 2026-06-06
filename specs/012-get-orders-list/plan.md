# Implementation Plan: 주문 목록 조회 (GET /api/orders)

**Branch**: `012-get-orders-list` | **Date**: 2026-06-04 | **Spec**: [spec.md](spec.md)

## Summary

기존 Order 도메인(POST /api/order)에 목록 조회 기능을 추가한다. buyer/lessonNo 파라미터를 AND 조건으로 필터링하고, 조건에 맞는 주문 목록을 배열로 반환한다. JPA Specification 패턴(Lesson 목록 조회 기존 패턴)을 Order에 동일하게 적용한다.

## Technical Context

**Language/Version**: Java 21, Spring Boot 4.x

**Primary Dependencies**: Spring Data JPA (`JpaSpecificationExecutor`), Lombok

**Storage**: MySQL — 기존 `orders`, `order_discounts` 테이블 사용 (스키마 변경 없음)

**Testing**: JUnit 5, Mockito, `@WebMvcTest`

**Target Platform**: Linux server (Spring Boot 내장 Tomcat)

## Constitution Check

| Gate | Status | Notes |
|------|--------|-------|
| Hexagonal Architecture | PASS | Controller→UseCase→Service→Port→Adapter 단방향 |
| Two-DTO 패턴 | PASS | WebRequest/AppRequest, AppResponse/WebResponse 분리 |
| Mapper 패턴 | PASS | GetOrdersWebMapper(static), Persistence 재사용 |
| Contract-First | PASS | api-spec.md 명세 기반 구현 |
| 기존 기능 회귀 | PASS | POST /api/order 로직 변경 없음 |
| 최소 변경 원칙 | PASS | Domain/Entity 변경 없음, Repository에 extends 추가만 |

## File Structure

```
Latinhouse.Be/src/
├── main/java/com/latinhouse/api/
│   ├── common/config/
│   │   └── SecurityConfig.java                ← GET /api/orders permitAll 추가
│   └── order/
│       ├── adapter/in/web/
│       │   ├── OrderController.java            ← GET /api/orders 엔드포인트 추가
│       │   ├── GetOrdersWebResponse.java       ← 신규
│       │   └── GetOrdersWebMapper.java         ← 신규
│       ├── adapter/out/persistence/
│       │   ├── OrderJpaRepository.java         ← JpaSpecificationExecutor 추가
│       │   └── OrderPersistenceAdapter.java    ← LoadOrderPort 구현 추가
│       └── application/
│           ├── port/in/
│           │   ├── GetOrdersUseCase.java       ← 신규
│           │   ├── GetOrdersAppRequest.java    ← 신규
│           │   └── GetOrdersAppResponse.java   ← 신규
│           ├── port/out/
│           │   └── LoadOrderPort.java          ← 신규
│           └── service/
│               └── GetOrdersService.java       ← 신규
└── test/java/com/latinhouse/api/
    ├── order/adapter/in/web/
    │   └── OrderControllerTest.java            ← GET 테스트 추가
    └── order/application/service/
        └── GetOrdersServiceTest.java           ← 신규
```

## Implementation Steps

### Phase 1: Application Layer

1. `LoadOrderPort` — `loadOrders(String buyer, Long lessonNo): List<Order>`
2. `GetOrdersAppRequest` — `buyer(String)`, `lessonNo(Long)` (nullable)
3. `GetOrdersAppResponse` — `orderId, lessonNo, lessonOptionNo, price, status, discounts` + inner `DiscountInfo`
4. `GetOrdersUseCase` — `getOrders(GetOrdersAppRequest): List<GetOrdersAppResponse>`
5. `GetOrdersService` — UseCase 구현, LoadOrderPort 호출, Order → AppResponse 변환

### Phase 2: Persistence Layer

6. `OrderJpaRepository` — `extends JpaSpecificationExecutor<OrderEntity>` 추가
7. `OrderPersistenceAdapter` — `LoadOrderPort` 구현, `buildSpec(buyer, lessonNo)` 메서드 추가

### Phase 3: Web Adapter Layer

8. `GetOrdersWebResponse` — API 응답 DTO (inner `OrderDiscountInfo` 포함)
9. `GetOrdersWebMapper` — `toAppRequest()`, `toWebResponseList()` static 메서드
10. `OrderController` — `GET /api/orders` 엔드포인트 추가
11. `SecurityConfig` — `GET /api/orders` permitAll 추가

### Phase 4: Tests

12. `GetOrdersServiceTest` — 5개 단위 테스트 (buyer필터, lessonNo필터, AND, 빈결과, 전체)
13. `OrderControllerTest` — GET /api/orders 테스트 추가 (buyer 필터, 파라미터 없음)

## Key Patterns (기존 패턴 참조)

- **JPA Specification**: `LessonPersistenceAdapter.buildSpec()` 참조
- **목록 조회 UseCase**: `GetLessonsUseCase` / `GetLessonsService` 참조
- **Web Response 구조**: `GetLessonsWebResponse` 참조 (flat DTO 대신 discounts는 중첩 클래스)
- **Controller 구조**: `LessonController.getLessons()` 참조

## Dependencies

- 기존 `Order`, `OrderDiscount`, `OrderEntity`, `OrderDiscountEntity`, `OrderPersistenceMapper` 변경 없이 재사용
- 스키마 변경 없음
