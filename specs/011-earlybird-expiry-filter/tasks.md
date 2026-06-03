# Tasks: 주문 생성 시 EARLYBIRD 할인 만료일 필터링

**Input**: Design documents from `specs/011-earlybird-expiry-filter/`

## Phase 1: User Story 1 — 만료된 EARLYBIRD 할인 제외 (P1)

**Goal**: `condition < today`인 EARLYBIRD 할인을 주문 생성 시 자동으로 제외

**Independent Test**: condition이 과거 날짜인 EARLYBIRD만 있을 때 POST /api/order 결과의 discounts가 비어있음을 확인

- [X] T001 [US1] `CreateOrderService.resolveDiscounts()`의 EARLYBIRD 스트림에 만료 필터 추가 및 `import java.time.LocalDate` 추가 — `Latinhouse.Be/src/main/java/com/latinhouse/api/order/application/service/CreateOrderService.java`
- [X] T002 [US1] 기존 테스트 `createOrder_earlybirdDiscount_single_included` 수정: hardcoded 날짜 → `LocalDate.now().plusDays(30).toString()` — `Latinhouse.Be/src/test/java/com/latinhouse/api/order/application/service/CreateOrderServiceTest.java`
- [X] T003 [US1] 신규 테스트 `createOrder_earlybirdDiscount_expired_excluded` 추가: condition=과거 날짜 → discounts 비어있음 — `Latinhouse.Be/src/test/java/com/latinhouse/api/order/application/service/CreateOrderServiceTest.java`
- [X] T004 [US1] 신규 테스트 `createOrder_earlybirdDiscount_today_included` 추가: condition=오늘 날짜 → 포함 — `Latinhouse.Be/src/test/java/com/latinhouse/api/order/application/service/CreateOrderServiceTest.java`

---

## Phase 2: User Story 2 — 유효한 EARLYBIRD 중 최솟값 선택 (P2)

**Goal**: 유효한 EARLYBIRD 복수 중 condition 가장 이른 1건만 적용

**Independent Test**: 유효한 EARLYBIRD 2건 중 condition이 더 이른 것만 discounts에 포함됨을 확인

- [X] T005 [US2] 기존 테스트 `createOrder_earlybirdDiscount_multiple_earliestSelected` 수정: 두 condition 모두 미래 날짜로 교체 — `Latinhouse.Be/src/test/java/com/latinhouse/api/order/application/service/CreateOrderServiceTest.java`
- [X] T006 [US2] 신규 테스트 `createOrder_earlybirdDiscount_mixedExpiry_validOnly` 추가: 만료 1건 + 유효 1건 혼재 → 유효한 것만 선택 — `Latinhouse.Be/src/test/java/com/latinhouse/api/order/application/service/CreateOrderServiceTest.java`

---

## Dependencies & Execution Order

- T001 (서비스 수정) 먼저 완료 후 T002~T006 (테스트) 실행
- T002~T006은 동일 파일이므로 순차 실행

---

## Notes

- 모든 날짜 픽스처는 `LocalDate.now()` 기준 동적 계산 (하드코딩 금지)
- 신규 클래스/인터페이스 없음, 기존 SEX 할인 로직 변경 없음
