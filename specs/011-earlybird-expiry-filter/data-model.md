# Data Model: 주문 생성 시 EARLYBIRD 할인 만료일 필터링

신규 Entity/Domain/DTO 없음. `CreateOrderService` 단일 메서드 수정.

## 변경 대상

### CreateOrderService.resolveDiscounts() (수정)

**기존 EARLYBIRD 선별 로직**:
```
lessonDiscounts.stream()
    .filter(d -> d.getType() == DiscountType.EARLYBIRD)
    .min(Comparator.comparing(LessonDiscount::getCondition))
    .map(this::toOrderDiscount)
    .ifPresent(result::add);
```

**변경 후 EARLYBIRD 선별 로직**:
```
LocalDate today = LocalDate.now();
lessonDiscounts.stream()
    .filter(d -> d.getType() == DiscountType.EARLYBIRD)
    .filter(d -> !LocalDate.parse(d.getCondition()).isBefore(today))  // 만료 제외
    .min(Comparator.comparing(LessonDiscount::getCondition))
    .map(this::toOrderDiscount)
    .ifPresent(result::add);
```

**유효성 기준**: `conditionDate >= today` (당일 포함)

## 재사용 클래스

| 클래스 | 패키지 | 역할 |
|--------|--------|------|
| `LessonDiscount` | `lesson.domain` | condition(String) 제공 |
| `DiscountType` | `lesson.domain` | EARLYBIRD 구분 |
| `LocalDate` | `java.time` | 날짜 파싱 및 비교 |

## 테스트 변경

### CreateOrderServiceTest (수정)

기존 EARLYBIRD 테스트 2건을 날짜 동적 계산 방식으로 수정하고 만료 케이스를 추가한다.

| 테스트 메서드 | 변경 유형 | 검증 내용 |
|---------------|-----------|-----------|
| `createOrder_earlybirdDiscount_single_included` | **수정** | condition=미래 날짜 → 포함 |
| `createOrder_earlybirdDiscount_multiple_earliestSelected` | **수정** | 유효한 것 중 최솟값 선택 |
| `createOrder_earlybirdDiscount_expired_excluded` | **신규** | condition=과거 날짜 → 제외 |
| `createOrder_earlybirdDiscount_today_included` | **신규** | condition=오늘 날짜 → 포함 (당일 유효) |
| `createOrder_earlybirdDiscount_mixedExpiry_validOnly` | **신규** | 만료+유효 혼재 시 유효한 것 중 최솟값만 |
