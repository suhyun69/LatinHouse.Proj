# Tasks: 주문 생성 (POST /api/order)

**Input**: Design documents from `specs/009-create-order/`

**Prerequisites**: plan.md ✅ spec.md ✅ research.md ✅ data-model.md ✅ contracts/ ✅

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: 기존 프로젝트에 order 도메인 패키지 구조 초기화 및 공통 인프라 수정

- [X] T001 ErrorCode.java에 `LESSON_OPTION_NOT_FOUND("레슨 옵션을 찾을 수 없습니다", HttpStatus.NOT_FOUND)` 추가 — `global/exception/ErrorCode.java`
- [X] T002 ApiSecurityConfig.java에 `requestMatchers(HttpMethod.POST, "/api/*/order").permitAll()` 추가 — `config/ApiSecurityConfig.java`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: order 도메인의 모든 User Story가 공통으로 의존하는 도메인 객체 및 Port 인터페이스 생성

**⚠️ CRITICAL**: 이 Phase 완료 전까지 User Story 구현을 시작할 수 없음

- [X] T003 [P] Order 도메인 객체 생성 — `order/domain/Order.java` (id: String UUID, lessonNo, lessonOptionNo, buyer, price, paymentId, discounts, status 필드 포함, @Getter @Builder)
- [X] T004 [P] OrderDiscount 도메인 객체 생성 — `order/domain/OrderDiscount.java`
- [X] T005 [P] OrderStatus enum 생성 — `order/domain/OrderStatus.java` (PAYMENT_PENDING, PAYMENT_COMPLETED, APPROVED, CANCELED)
- [X] T006 [P] OrderDiscountType enum 생성 — `order/domain/OrderDiscountType.java` (Lesson, Coupon)
- [X] T007 CreateOrderUseCase 인터페이스 생성 — `order/port/in/CreateOrderUseCase.java` (`CreateOrderAppResponse create(CreateOrderAppRequest req)`)
- [X] T008 [P] SaveOrderPort 인터페이스 생성 — `order/port/out/SaveOrderPort.java` (`Order save(Order order)`)
- [X] T009 [P] LoadLessonOptionPort 인터페이스 생성 — `order/port/out/LoadLessonOptionPort.java` (`void load(Long lessonOptionNo)` — 없으면 CustomException(LESSON_OPTION_NOT_FOUND))

**Checkpoint**: 도메인 객체·Port 인터페이스 완료 — User Story 구현 시작 가능

---

## Phase 3: User Story 1 - 주문 생성 (Priority: P1) 🎯 MVP

**Goal**: `POST /api/v1/order` 요청으로 주문을 생성하고 UUID orderId를 반환한다

**Independent Test**: `POST /api/v1/order`에 유효한 lessonNo+lessonOptionNo+profileId를 전달하면 201 + UUID orderId 반환

### Implementation for User Story 1

- [X] T010 [P] [US1] CreateOrderAppRequest 생성 — `order/port/in/request/CreateOrderAppRequest.java` (lessonNo, lessonOptionNo, profileId 필드. static factory `from(webReq)` 포함)
- [X] T011 [P] [US1] CreateOrderAppResponse 생성 — `order/port/in/response/CreateOrderAppResponse.java` (orderId: String)
- [X] T012 [P] [US1] CreateOrderWebRequest 생성 — `order/adapter/in/web/request/CreateOrderWebRequest.java` (@NotNull Long lessonNo, @NotNull Long lessonOptionNo, @NotBlank String profileId, 에러 메시지 한국어)
- [X] T013 [P] [US1] CreateOrderWebResponse 생성 — `order/adapter/in/web/response/CreateOrderWebResponse.java` (orderId: String)
- [X] T014 [US1] OrderJpaEntity 생성 — `order/adapter/out/persistence/entity/OrderJpaEntity.java` (테이블명 `order_table`, @Id String id, @Enumerated(EnumType.STRING) OrderStatus status, @OneToMany OrderDiscountJpaEntity 포함, CascadeType.ALL, orphanRemoval=true)
- [X] T015 [US1] OrderDiscountJpaEntity 생성 — `order/adapter/out/persistence/entity/OrderDiscountJpaEntity.java` (테이블명 `order_discount`, @ManyToOne OrderJpaEntity)
- [X] T016 [US1] OrderRepository 생성 — `order/adapter/out/persistence/repository/OrderRepository.java` (extends JpaRepository<OrderJpaEntity, String>)
- [X] T017 [US1] OrderMapper 생성 — `order/adapter/out/persistence/mapper/OrderMapper.java` (Order ↔ OrderJpaEntity 변환 정적 메서드)
- [X] T018 [US1] LessonOptionRepository 확인 또는 생성 — lesson 도메인의 LessonOptionJpaEntity 기반 Repository가 없으면 `lesson/adapter/out/persistence/repository/LessonOptionRepository.java` 추가
- [X] T019 [US1] OrderPersistenceAdapter 생성 — `order/adapter/out/persistence/OrderPersistenceAdapter.java` (SaveOrderPort, LoadLessonOptionPort 구현. LessonOptionRepository로 lessonOptionNo 존재 확인)
- [X] T020 [US1] CreateOrderService 구현 — `order/application/service/CreateOrderService.java` (CreateOrderUseCase 구현. LoadLessonPort, LoadLessonOptionPort, LoadProfilePort, SaveOrderPort 주입. UUID 생성, PAYMENT_PENDING 초기화, lesson.getPrice()로 price 설정)
- [X] T021 [US1] ApiV1OrderController 생성 — `order/adapter/in/web/ApiV1OrderController.java` (@RequestMapping("/api/v1/order"), @PostMapping, @Valid, ResponseEntity 201)

**Checkpoint**: `POST /api/v1/order` 201 Created + UUID orderId 동작 확인

---

## Phase 4: User Story 2 - 존재하지 않는 리소스 오류 (Priority: P2)

**Goal**: 없는 lessonNo/lessonOptionNo/profileId로 요청 시 정확한 404 응답 반환

**Independent Test**: 각각 없는 ID로 요청 → 404 + 해당 에러 코드 확인

- [X] T022 [US2] LoadLessonPort가 없는 lessonNo에 대해 CustomException(LESSON_NOT_FOUND)를 발생시키는지 확인 — `lesson/adapter/out/persistence/LessonPersistenceAdapter.java` (기존 구현 검토, 없으면 수정)
- [X] T023 [US2] LoadLessonOptionPort 구현에서 없는 lessonOptionNo에 대해 CustomException(LESSON_OPTION_NOT_FOUND) 발생 확인 — `order/adapter/out/persistence/OrderPersistenceAdapter.java`
- [X] T024 [US2] LoadProfilePort가 없는 profileId에 대해 CustomException(PROFILE_NOT_FOUND) 발생 확인 — `profile/adapter/out/persistence/ProfilePersistenceAdapter.java` (기존 구현 검토)

**Checkpoint**: 없는 리소스 ID 요청 시 3가지 404 모두 정상 응답

---

## Phase 5: User Story 3 - 유효성 검사 (Priority: P3)

**Goal**: 필수 필드 누락 시 400 + 필드별 에러 메시지 반환

**Independent Test**: lessonNo=null, lessonOptionNo=null, profileId="" 각각 요청 → 400 + 한국어 메시지

- [X] T025 [US3] CreateOrderWebRequest의 @NotNull/@NotBlank 메시지 검증 — 기존 GlobalExceptionHandler가 MethodArgumentNotValidException을 처리하므로 WebRequest 어노테이션 메시지만 확인 — `order/adapter/in/web/request/CreateOrderWebRequest.java`

**Checkpoint**: 400 검증 에러 3가지 모두 정상 동작

---

## Phase 6: 테스트 작성

**Purpose**: Controller 및 Service 단위 테스트

- [X] T026 [P] ApiV1OrderControllerTest 작성 — `test/.../order/adapter/in/web/ApiV1OrderControllerTest.java` (@WebMvcTest, @MockitoBean CreateOrderUseCase: 성공 201, lessonNo null 400, lessonOptionNo null 400, profileId 빈문자열 400, LESSON_NOT_FOUND 404, LESSON_OPTION_NOT_FOUND 404, PROFILE_NOT_FOUND 404)
- [X] T027 [P] CreateOrderServiceTest 작성 — `test/.../order/application/service/CreateOrderServiceTest.java` (Mockito: 정상흐름 Port 호출 순서, LESSON_NOT_FOUND, LESSON_OPTION_NOT_FOUND, PROFILE_NOT_FOUND 예외 전파)

---

## Phase 7: Polish & Cross-Cutting Concerns

- [X] T028 전체 테스트 실행 및 통과 확인 (`./gradlew test`)
- [X] T029 quickstart.md 체크리스트 수동 검증 (Swagger 또는 curl로 7개 시나리오 확인)

---

## Dependencies & Execution Order

- **Phase 1 (Setup)**: 즉시 시작 가능
- **Phase 2 (Foundational)**: Phase 1 완료 후 시작. T003-T006은 병렬 가능
- **Phase 3 (US1)**: Phase 2 완료 후 시작. T010-T013은 병렬 가능
- **Phase 4 (US2)**: Phase 3 완료 후 시작 (CreateOrderService 구현 필요)
- **Phase 5 (US3)**: Phase 3 완료 후 시작 (WebRequest 클래스 필요)
- **Phase 6 (Tests)**: Phase 3 완료 후 병렬 가능
- **Phase 7 (Polish)**: 모든 Phase 완료 후

## Implementation Strategy

### MVP First (User Story 1 Only)
1. Phase 1 완료 (T001-T002)
2. Phase 2 완료 (T003-T009)
3. Phase 3 완료 (T010-T021)
4. **STOP and VALIDATE**: `POST /api/v1/order` 201 동작 확인

### Incremental Delivery
1. Setup + Foundational → 도메인 기반 완성
2. US1 완료 → 201 동작 (MVP)
3. US2 완료 → 404 에러 검증
4. US3 완료 → 400 에러 검증
5. 테스트 + Polish
