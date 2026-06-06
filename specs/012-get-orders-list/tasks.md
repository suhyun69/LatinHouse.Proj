# Tasks: 주문 목록 조회 (GET /api/orders)

**Input**: Design documents from `specs/012-get-orders-list/`

**Prerequisites**: plan.md ✅, spec.md ✅, data-model.md ✅, contracts/ ✅, research.md ✅

---

## Phase 1: Foundational — Application Layer Port/UseCase

**Purpose**: 상위 계층(Controller, Service)이 의존하는 Port·UseCase 인터페이스·DTO를 먼저 정의한다

**⚠️ CRITICAL**: 이 단계 완료 전 Phase 2+ 구현 불가

- [X] T001 [P] `LoadOrderPort` 인터페이스 생성: `List<Order> loadOrders(String buyer, Long lessonNo)` — `Latinhouse.Be/src/main/java/com/latinhouse/api/order/application/port/out/LoadOrderPort.java`
- [X] T002 [P] `GetOrdersAppRequest` 생성: `buyer(String)`, `lessonNo(Long)` (nullable, Lombok @Getter @Builder) — `Latinhouse.Be/src/main/java/com/latinhouse/api/order/application/port/in/GetOrdersAppRequest.java`
- [X] T003 [P] `GetOrdersAppResponse` 생성: `orderId, lessonNo, lessonOptionNo, price(BigDecimal), status(String), discounts(List<DiscountInfo>)` + inner static class `DiscountInfo(discountType, discountId, amount)` — `Latinhouse.Be/src/main/java/com/latinhouse/api/order/application/port/in/GetOrdersAppResponse.java`
- [X] T004 `GetOrdersUseCase` 인터페이스 생성: `List<GetOrdersAppResponse> getOrders(GetOrdersAppRequest request)` — `Latinhouse.Be/src/main/java/com/latinhouse/api/order/application/port/in/GetOrdersUseCase.java`

**Checkpoint**: Port·UseCase 인터페이스 완료 → Phase 2, Phase 3 병렬 진행 가능

---

## Phase 2: User Story 1 — buyer 기준 주문 목록 조회 (P1) 🎯 MVP

**Goal**: `GET /api/orders?buyer=Ab2Cd3Ef` 호출 시 해당 buyer의 주문 목록을 반환한다

**Independent Test**: `GET /api/orders?buyer=Ab2Cd3Ef` 호출 → 해당 buyer 주문만 반환, 다른 buyer 주문 미포함 확인

### Implementation

- [X] T005 [US1] `OrderJpaRepository`에 `JpaSpecificationExecutor<OrderEntity>` 추가 — `Latinhouse.Be/src/main/java/com/latinhouse/api/order/adapter/out/persistence/OrderJpaRepository.java`
- [X] T006 [US1] `OrderPersistenceAdapter`에 `LoadOrderPort` 구현 추가: `loadOrders()` + `buildSpec(buyer, lessonNo)` (buyer null 체크 후 spec.and 조건 추가, LessonPersistenceAdapter.buildSpec() 패턴 참조) — `Latinhouse.Be/src/main/java/com/latinhouse/api/order/adapter/out/persistence/OrderPersistenceAdapter.java`
- [X] T007 [US1] `GetOrdersService` 구현: `@Service`, `LoadOrderPort` 주입, `loadOrders()` 호출 후 `Order → GetOrdersAppResponse` 변환 (OrderDiscount → DiscountInfo 포함) — `Latinhouse.Be/src/main/java/com/latinhouse/api/order/application/service/GetOrdersService.java`
- [X] T008 [US1] `GetOrdersWebResponse` 생성: `orderId, lessonNo, lessonOptionNo, price, status, discounts(List<OrderDiscountInfo>)` + inner static class `OrderDiscountInfo(discountType, discountId, amount)` (Lombok @Getter @Builder) — `Latinhouse.Be/src/main/java/com/latinhouse/api/order/adapter/in/web/GetOrdersWebResponse.java`
- [X] T009 [US1] `GetOrdersWebMapper` 생성: `toAppRequest(String buyer, String lessonNo)` (lessonNo String→Long 변환, null 처리), `toWebResponseList(List<GetOrdersAppResponse>)` (static 메서드만, private 생성자) — `Latinhouse.Be/src/main/java/com/latinhouse/api/order/adapter/in/web/GetOrdersWebMapper.java`
- [X] T010 [US1] `OrderController`에 `GET /api/orders` 엔드포인트 추가: `@GetMapping("/orders")`, `@RequestParam(required=false)` buyer·lessonNo, `GetOrdersWebMapper` 사용, `ResponseEntity<List<GetOrdersWebResponse>>` 반환 — `Latinhouse.Be/src/main/java/com/latinhouse/api/order/adapter/in/web/OrderController.java`
- [X] T011 [US1] `SecurityConfig`에 `GET /api/orders` permitAll 추가: `.requestMatchers(HttpMethod.GET, "/api/orders").permitAll()` — `Latinhouse.Be/src/main/java/com/latinhouse/api/common/config/SecurityConfig.java`

### Tests

- [X] T012 [US1] `GetOrdersServiceTest` 신규 생성: `getOrders_byBuyer_returnsOnlyBuyerOrders`, `getOrders_noMatch_returnsEmptyList`, `getOrders_noParams_returnsAll` (3건, @ExtendWith(MockitoExtension.class), LoadOrderPort mock) — `Latinhouse.Be/src/test/java/com/latinhouse/api/order/application/service/GetOrdersServiceTest.java`
- [X] T013 [US1] `OrderControllerTest`에 GET 테스트 추가: `getOrders_withBuyer_returns200`, `getOrders_noParams_returns200` (2건, @WebMvcTest, GetOrdersUseCase mock) — `Latinhouse.Be/src/test/java/com/latinhouse/api/order/adapter/in/web/OrderControllerTest.java`

**Checkpoint**: `GET /api/orders?buyer=Ab2Cd3Ef` 독립적으로 동작 및 테스트 완료

---

## Phase 3: User Story 2 — lessonNo 기준 주문 목록 조회 (P2)

**Goal**: `GET /api/orders?lessonNo=1` 호출 시 해당 레슨의 주문 목록을 반환한다

**Independent Test**: `GET /api/orders?lessonNo=1` 호출 → 해당 lessonNo 주문만 반환 확인

### Implementation

- [X] T014 [US2] `OrderPersistenceAdapter.buildSpec()`의 lessonNo 조건이 이미 T006에서 구현됨. `GetOrdersServiceTest`에 `getOrders_byLessonNo_returnsOnlyLessonOrders` 테스트 추가 — `Latinhouse.Be/src/test/java/com/latinhouse/api/order/application/service/GetOrdersServiceTest.java`

**Checkpoint**: `GET /api/orders?lessonNo=1` 독립적으로 동작 확인

---

## Phase 4: User Story 3 — buyer + lessonNo AND 복합 조건 조회 (P3)

**Goal**: 두 파라미터 동시 지정 시 AND 조건으로 필터링된 주문만 반환한다

**Independent Test**: `GET /api/orders?buyer=Ab2Cd3Ef&lessonNo=1` → 두 조건 모두 만족하는 주문만 반환 확인

### Implementation

- [X] T015 [US3] `GetOrdersServiceTest`에 `getOrders_byBuyerAndLessonNo_andCondition` 테스트 추가 (buyer와 lessonNo 동시 지정, 두 조건 모두 만족하는 것만 반환 검증) — `Latinhouse.Be/src/test/java/com/latinhouse/api/order/application/service/GetOrdersServiceTest.java`

**Checkpoint**: 전체 3개 US 완료 → 통합 동작 검증

---

## Phase 5: Polish

- [X] T016 `Latinhouse.Be` 프로젝트 전체 테스트 실행하여 회귀 없음 확인: `./gradlew test`

---

## Dependencies & Execution Order

```
T001, T002, T003 [P] → T004 → T005 [P] T006 [P] T007 [P] T008 [P] T009 [P]
                             → T010 (T009 완료 후)
                             → T011 (독립)
                             → T012 (T007 완료 후)
                             → T013 (T010, T011 완료 후)
→ T014 (T006, T012 완료 후)
→ T015 (T012 완료 후)
→ T016 (모든 구현 완료 후)
```

### Phase Dependencies

- **Phase 1 (Foundational)**: 즉시 시작 가능
- **Phase 2 (US1)**: Phase 1 완료 후
- **Phase 3 (US2)**: T006 완료 후 (buildSpec 이미 구현)
- **Phase 4 (US3)**: T006, T012 완료 후
- **Phase 5 (Polish)**: 모든 구현 완료 후

---

## Parallel Opportunities

```bash
# Phase 1 — 동시 생성 가능
T001 LoadOrderPort
T002 GetOrdersAppRequest
T003 GetOrdersAppResponse

# Phase 2 — T004 완료 후 동시 진행 가능
T005 OrderJpaRepository 수정
T006 OrderPersistenceAdapter 수정
T007 GetOrdersService 구현
T008 GetOrdersWebResponse 생성
T009 GetOrdersWebMapper 생성
T011 SecurityConfig 수정
```

---

## Implementation Strategy

### MVP (User Story 1만)

1. Phase 1 완료 (T001~T004)
2. Phase 2 완료 (T005~T013)
3. `GET /api/orders?buyer=Ab2Cd3Ef` 동작 검증

### 완전 구현

MVP 이후 Phase 3~5 순서대로 진행 (lessonNo 필터, AND 조건은 buildSpec에서 이미 처리되므로 추가 구현 없이 테스트만 추가)

---

## Notes

- T005~T009, T011은 서로 다른 파일을 수정하므로 [P] 병렬 진행 가능
- US2·US3의 필터링 로직은 T006의 `buildSpec()`에서 이미 커버됨 → 추가 테스트만 필요
- `lessonNo` Query Parameter는 String으로 수신 후 WebMapper에서 Long 변환 (WebRequest 원시 타입 원칙)
