# Implementation Plan: 주문 생성 (POST /api/order)

**Branch**: `009-create-order` | **Date**: 2026-06-03 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `specs/009-create-order/spec.md`

## Summary

`POST /api/v1/order` 엔드포인트를 구현한다. 구매자 프로필, 레슨, 수업 옵션을 받아 UUID 주문 ID를 생성하고 `PAYMENT_PENDING` 상태로 저장한다.
Hexagonal Architecture 패턴을 따르며, 기존 Lesson/Profile 도메인의 Port를 재사용한다.

## Technical Context

**Language/Version**: Java 21 (Spring Boot)

**Primary Dependencies**: Spring Web MVC, Spring Data JPA, Bean Validation (Jakarta), Lombok, SpringDoc OpenAPI

**Storage**: MySQL (JPA)

**Testing**: JUnit 5, Mockito, `@WebMvcTest`

**Target Platform**: Linux 서버 (REST API)

**Project Type**: Web Service (Hexagonal Architecture)

**Performance Goals**: 기존 엔드포인트와 동일 수준

**Constraints**: `docs/api-spec.md` 계약 준수. `ApiSecurityConfig`에 permitAll 추가 필수.

**Scale/Scope**: 단일 주문 단건 생성

## Constitution Check

| Gate | 상태 | 근거 |
|------|------|------|
| I. Hexagonal Architecture | ✅ | Web Adapter → Port → Domain 단방향 의존 |
| II. Contract-First API | ✅ | `docs/api-spec.md` POST /api/order 사전 정의됨 |
| III. Unified Error Response | ✅ | `CustomException` + `GlobalExceptionHandler` 재사용 |
| IV. Validated Inputs | ✅ | Bean Validation은 WebRequest에만 적용 |
| Two-DTO 패턴 | ✅ | WebRequest/Response ↔ AppRequest/Response 분리 |

## Project Structure

### Documentation (this feature)

```text
specs/009-create-order/
├── plan.md              ← 이 파일
├── research.md          ← Phase 0 완료
├── data-model.md        ← Phase 1 완료
├── quickstart.md        ← Phase 1 완료
├── contracts/
│   └── post-order.md   ← Phase 1 완료
├── checklists/
│   └── requirements.md
└── tasks.md             ← /speckit-tasks로 생성
```

### Source Code

```text
Latinhouse.Be/src/main/java/com/latinhouse/backend/
├── global/exception/
│   └── ErrorCode.java                              ← LESSON_OPTION_NOT_FOUND 추가
├── config/
│   └── ApiSecurityConfig.java                      ← POST /api/*/order permitAll 추가
└── order/
    ├── adapter/
    │   ├── in/web/
    │   │   ├── ApiV1OrderController.java            ← 신규
    │   │   ├── request/CreateOrderWebRequest.java   ← 신규
    │   │   └── response/CreateOrderWebResponse.java ← 신규
    │   └── out/persistence/
    │       ├── entity/OrderJpaEntity.java           ← 신규
    │       ├── entity/OrderDiscountJpaEntity.java   ← 신규
    │       ├── repository/OrderRepository.java      ← 신규
    │       ├── mapper/OrderMapper.java              ← 신규
    │       └── OrderPersistenceAdapter.java         ← 신규
    ├── application/service/
    │   └── CreateOrderService.java                  ← 신규
    ├── domain/
    │   ├── Order.java                               ← 신규
    │   ├── OrderDiscount.java                       ← 신규
    │   ├── OrderStatus.java                         ← 신규
    │   └── OrderDiscountType.java                   ← 신규
    └── port/
        ├── in/
        │   ├── CreateOrderUseCase.java              ← 신규
        │   ├── request/CreateOrderAppRequest.java   ← 신규
        │   └── response/CreateOrderAppResponse.java ← 신규
        └── out/
            ├── SaveOrderPort.java                   ← 신규
            └── LoadLessonOptionPort.java            ← 신규

Latinhouse.Be/src/test/java/com/latinhouse/backend/order/
├── adapter/in/web/
│   └── ApiV1OrderControllerTest.java               ← 신규
└── application/service/
    └── CreateOrderServiceTest.java                  ← 신규
```

## Implementation Steps

### Step 1: Domain 객체

`Order`, `OrderDiscount`, `OrderStatus`, `OrderDiscountType` 생성.

### Step 2: Port 인터페이스

`CreateOrderUseCase`, `SaveOrderPort`, `LoadLessonOptionPort` 생성.

### Step 3: DTO 클래스

`CreateOrderWebRequest` (`@NotNull lessonNo`, `@NotNull lessonOptionNo`, `@NotBlank profileId`),
`CreateOrderWebResponse`, `CreateOrderAppRequest`, `CreateOrderAppResponse` 생성.

### Step 4: JPA Entity

`OrderJpaEntity` (테이블: `order_table`), `OrderDiscountJpaEntity` (테이블: `order_discount`).
`@Id private String id;` (UUID String PK, `@GeneratedValue` 없음).

### Step 5: Persistence Adapter

`OrderRepository`, `OrderMapper`, `OrderPersistenceAdapter` (`SaveOrderPort` 구현).
`LessonOptionRepository` 조회를 위한 `LoadLessonOptionPort` 구현 추가 (LessonPersistenceAdapter 또는 별도 OrderPersistenceAdapter).

### Step 6: CreateOrderService

```java
@Service
@RequiredArgsConstructor
public class CreateOrderService implements CreateOrderUseCase {
    private final LoadLessonPort loadLessonPort;
    private final LoadLessonOptionPort loadLessonOptionPort;
    private final LoadProfilePort loadProfilePort;
    private final SaveOrderPort saveOrderPort;

    @Override
    @Transactional
    public CreateOrderAppResponse create(CreateOrderAppRequest req) {
        Lesson lesson = loadLessonPort.load(req.getLessonNo());       // LESSON_NOT_FOUND
        loadLessonOptionPort.load(req.getLessonOptionNo());            // LESSON_OPTION_NOT_FOUND
        loadProfilePort.load(req.getProfileId());                      // PROFILE_NOT_FOUND

        Order order = Order.builder()
                .id(UUID.randomUUID().toString())
                .lessonNo(req.getLessonNo())
                .lessonOptionNo(req.getLessonOptionNo())
                .buyer(req.getProfileId())
                .price(lesson.getPrice())
                .paymentId(null)
                .discounts(List.of())
                .status(OrderStatus.PAYMENT_PENDING)
                .build();

        Order saved = saveOrderPort.save(order);
        return new CreateOrderAppResponse(saved.getId());
    }
}
```

### Step 7: Controller

```java
@PostMapping
@Operation(summary = "Create Order", description = "Create Order")
public ResponseEntity<CreateOrderWebResponse> create(@Valid @RequestBody CreateOrderWebRequest webReq) {
    CreateOrderAppRequest appReq = CreateOrderAppRequest.from(webReq);
    CreateOrderAppResponse appRes = createOrderUseCase.create(appReq);
    return ResponseEntity.status(HttpStatus.CREATED).body(new CreateOrderWebResponse(appRes.getOrderId()));
}
```

### Step 8: ErrorCode & SecurityConfig 수정

- `ErrorCode.java`: `LESSON_OPTION_NOT_FOUND("레슨 옵션을 찾을 수 없습니다", HttpStatus.NOT_FOUND)` 추가
- `ApiSecurityConfig.java`: `.requestMatchers(HttpMethod.POST, "/api/*/order").permitAll()` 추가

### Step 9: 테스트 작성

**ApiV1OrderControllerTest** (`@WebMvcTest`):
- 201 Created — 정상 주문 생성
- 400 — lessonNo null
- 400 — lessonOptionNo null
- 400 — profileId 빈 문자열
- 404 — 존재하지 않는 lessonNo (Mock)
- 404 — 존재하지 않는 lessonOptionNo (Mock)
- 404 — 존재하지 않는 profileId (Mock)

**CreateOrderServiceTest** (단위):
- 정상 흐름 — loadLesson → loadOption → loadProfile → save 호출 순서
- 존재하지 않는 lessonNo → CustomException(LESSON_NOT_FOUND)
- 존재하지 않는 lessonOptionNo → CustomException(LESSON_OPTION_NOT_FOUND)
- 존재하지 않는 profileId → CustomException(PROFILE_NOT_FOUND)

## Complexity Tracking

해당 없음. 모든 Constitution Check 통과.
