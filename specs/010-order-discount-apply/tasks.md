# Tasks: 주문 생성 시 레슨 할인 자동 적용

**Input**: Design documents from `specs/010-order-discount-apply/`

**Prerequisites**: plan.md ✅ | spec.md ✅ | research.md ✅ | data-model.md ✅ | contracts/ ✅

---

## Phase 1: Setup

**Purpose**: 변경 대상 파일 파악 및 기존 코드 컨텍스트 확인

- [X] T001 기존 `CreateOrderService.java` 및 관련 도메인 클래스(LessonDiscount, DiscountType, OrderDiscount, OrderDiscountType, Profile, Sex) 전체 읽기

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: 할인 선별 로직 구현 전 기존 테스트 통과 확인

- [X] T002 `./gradlew test --tests "com.latinhouse.api.order.*"` 실행하여 기존 테스트 전체 통과 확인

**Checkpoint**: 기존 테스트 통과 확인 후 구현 시작

---

## Phase 3: User Story 1 — SEX 할인 자동 적용 (Priority: P1) 🎯 MVP

**Goal**: Profile.sex가 LessonDiscount.condition과 일치하는 SEX 할인만 Order.discounts에 포함

**Independent Test**: SEX 할인 매칭/불일치 시나리오 테스트 통과

### Implementation for User Story 1

- [X] T003 [US1] `CreateOrderService.java`에 SEX 할인 선별 로직 추가: `findProfilePort.findById()` 반환값을 Profile로 보존하고, `lesson.getDiscounts()`에서 `DiscountType.SEX` 항목 중 `condition == profile.getSex().name()` 인 것만 OrderDiscount(discountType=LESSON, discountId=lessonDiscount.getId(), amount=lessonDiscount.getAmount())로 변환하여 Order.builder에 설정. Profile.sex null 시 SEX 할인 미적용. 파일: `Latinhouse.Be/src/main/java/com/latinhouse/api/order/application/service/CreateOrderService.java`

### Tests for User Story 1

- [X] T004 [US1] `CreateOrderServiceTest.java`에 SEX 할인 테스트 3건 추가
  - `createOrder_sexDiscount_matching_M_included`: Lesson에 SEX condition=M 할인 존재, Profile.sex=M → order.discounts에 1건 포함
  - `createOrder_sexDiscount_notMatching_excluded`: Lesson에 SEX condition=M 할인 존재, Profile.sex=F → order.discounts에 SEX 항목 없음
  - `createOrder_noDiscounts_emptyList`: Lesson.discounts 빈 리스트 → order.discounts 빈 리스트
  - 파일: `Latinhouse.Be/src/test/java/com/latinhouse/api/order/application/service/CreateOrderServiceTest.java`

**Checkpoint**: `./gradlew test --tests "*.CreateOrderServiceTest"` — SEX 관련 테스트 통과

---

## Phase 4: User Story 2 — EARLYBIRD 할인 자동 적용 (Priority: P2)

**Goal**: EARLYBIRD 할인 중 condition(날짜) 기준 가장 이른 1건만 Order.discounts에 포함

**Independent Test**: EARLYBIRD 단일/복수 시나리오 테스트 통과

### Implementation for User Story 2

- [X] T005 [US2] `CreateOrderService.java`에 EARLYBIRD 할인 선별 로직 추가: `lesson.getDiscounts()`에서 `DiscountType.EARLYBIRD` 항목을 `Comparator.comparing(LessonDiscount::getCondition)`으로 min 선택(1건)하여 OrderDiscount로 변환, discounts 리스트에 추가. 파일: `Latinhouse.Be/src/main/java/com/latinhouse/api/order/application/service/CreateOrderService.java`

### Tests for User Story 2

- [X] T006 [US2] `CreateOrderServiceTest.java`에 EARLYBIRD 할인 테스트 2건 추가
  - `createOrder_earlybirdDiscount_single_included`: Lesson에 EARLYBIRD 1건 → discounts에 1건 포함
  - `createOrder_earlybirdDiscount_multiple_earliestSelected`: Lesson에 EARLYBIRD 2건(condition="2026-06-01", "2026-05-01") → condition="2026-05-01" 1건만 포함
  - 파일: `Latinhouse.Be/src/test/java/com/latinhouse/api/order/application/service/CreateOrderServiceTest.java`

**Checkpoint**: `./gradlew test --tests "*.CreateOrderServiceTest"` — EARLYBIRD 관련 테스트 통과

---

## Phase 5: User Story 3 — SEX + EARLYBIRD 복합 할인 (Priority: P3)

**Goal**: 두 유형이 함께 존재할 때 각각 독립적으로 처리되어 모두 적용

**Independent Test**: 복합 할인 시나리오 테스트 통과

### Tests for User Story 3

- [X] T007 [US3] `CreateOrderServiceTest.java`에 복합 할인 테스트 2건 추가
  - `createOrder_mixedDiscounts_sexMatch_bothApplied`: SEX condition=M + EARLYBIRD 1건, Profile.sex=M → discounts 2건 포함
  - `createOrder_mixedDiscounts_sexNotMatch_onlyEarlybird`: SEX condition=M + EARLYBIRD 1건, Profile.sex=F → discounts 1건(EARLYBIRD)만 포함
  - 파일: `Latinhouse.Be/src/test/java/com/latinhouse/api/order/application/service/CreateOrderServiceTest.java`

**Checkpoint**: US3는 US1+US2 로직 완성 후 자동으로 통과 가능. 별도 구현 불필요.

---

## Phase 6: Polish & Cross-Cutting Concerns

- [X] T008 `./gradlew test` 전체 실행하여 기존 `OrderControllerTest` 포함 모든 테스트 통과 확인 (회귀 없음)

---

## Dependencies & Execution Order

- **T001**: 선행 조건 없음
- **T002**: T001 완료 후
- **T003**: T002 완료 후 (SEX 로직 먼저 구현)
- **T004**: T003 완료 후 (구현 후 테스트)
- **T005**: T004 통과 후 (EARLYBIRD 로직 추가)
- **T006**: T005 완료 후
- **T007**: T005+T006 완료 후 (추가 구현 불필요, 테스트만)
- **T008**: T007 완료 후 전체 검증

### Parallel Opportunities

- T003, T004는 같은 파일이므로 순차 실행
- T005, T006은 같은 파일이므로 순차 실행
- T007은 T004, T006과 같은 테스트 파일 — 순차 실행

---

## Implementation Strategy

### MVP (User Story 1 Only)

1. T001 → T002 → T003 → T004
2. **VALIDATE**: SEX 할인 적용 동작 확인
3. 필요 시 US2, US3 추가

### Full Delivery (권장)

T001 → T002 → T003 → T004 → T005 → T006 → T007 → T008

---

## Notes

- `CreateOrderService`의 `findProfilePort.findById()` 반환값을 현재 `.orElseThrow()` 직후 버리지 않고 `Profile profile` 변수로 보존해야 SEX 할인 선별에 사용 가능
- `Order.builder().discounts(resolvedDiscounts)` 로 설정 시 기존 `List.of()` 하드코딩 제거
- 테스트에서 `saveOrderPort.save(any())` stub이 반환하는 Order에도 discounts를 설정해야 `response.getOrderId()` 검증이 통과됨
