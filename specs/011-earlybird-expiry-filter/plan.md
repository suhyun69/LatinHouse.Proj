# Implementation Plan: 주문 생성 시 EARLYBIRD 할인 만료일 필터링

**Branch**: `011-earlybird-expiry-filter` | **Date**: 2026-06-04 | **Spec**: [spec.md](spec.md)

## Summary

`CreateOrderService.resolveDiscounts()`의 EARLYBIRD 선별 스트림에 만료일 필터를 추가한다. `condition < LocalDate.now()`인 항목을 제외한 뒤 남은 유효 후보 중 condition 최솟값 1건을 선택한다. 구현 변경은 단일 메서드 1줄 추가이며, 테스트 5건(수정 2 + 신규 3)이 수반된다.

## Technical Context

**Language/Version**: Java 21, Spring Boot 4.x

**Primary Dependencies**: Spring Data JPA, Lombok, Hibernate

**Storage**: MySQL (기존 구조 변경 없음)

**Testing**: JUnit 5, Mockito

**Target Platform**: Linux server (Spring Boot 내장 Tomcat)

## Constitution Check

| Gate | Status | Notes |
|------|--------|-------|
| 단일 책임 | PASS | resolveDiscounts() 내부 필터 추가만, 외부 인터페이스 변경 없음 |
| 테스트 커버리지 | PASS | 5개 테스트 케이스로 만료/유효/당일/혼재 시나리오 모두 검증 |
| 기존 기능 회귀 | PASS | SEX 할인 로직 및 기타 주문 생성 흐름 변경 없음 |
| 최소 변경 원칙 | PASS | 신규 클래스/인터페이스 없음, 1 filter 추가 |

## File Structure

```
Latinhouse.Be/src/
├── main/java/com/latinhouse/api/order/application/service/
│   └── CreateOrderService.java          ← EARLYBIRD 필터 추가 (resolveDiscounts)
└── test/java/com/latinhouse/api/order/application/service/
    └── CreateOrderServiceTest.java      ← 기존 2건 수정 + 신규 3건 추가
```

## Implementation Steps

### Phase 1: Service 수정

`CreateOrderService.resolveDiscounts()` EARLYBIRD 스트림에 만료 필터 추가:

```java
LocalDate today = LocalDate.now();
lessonDiscounts.stream()
    .filter(d -> d.getType() == DiscountType.EARLYBIRD)
    .filter(d -> !LocalDate.parse(d.getCondition()).isBefore(today))
    .min(Comparator.comparing(LessonDiscount::getCondition))
    .map(this::toOrderDiscount)
    .ifPresent(result::add);
```

`import java.time.LocalDate;` 추가.

### Phase 2: 테스트 수정

**수정 대상** (하드코딩 날짜 → 동적 계산):
- `createOrder_earlybirdDiscount_single_included`: condition = `LocalDate.now().plusDays(30).toString()`
- `createOrder_earlybirdDiscount_multiple_earliestSelected`: 두 condition 모두 미래 날짜로 교체

**신규 추가**:
- `createOrder_earlybirdDiscount_expired_excluded`: condition = 과거 날짜 → discounts 비어있음 검증
- `createOrder_earlybirdDiscount_today_included`: condition = 오늘 날짜 → 포함 검증
- `createOrder_earlybirdDiscount_mixedExpiry_validOnly`: 만료 1건 + 유효 1건 혼재 → 유효한 것만 선택

## Dependencies

- 010-order-discount-apply 구현 위에 적용 (CreateOrderService 기존 코드 존재 가정)
- 신규 외부 의존성 없음
