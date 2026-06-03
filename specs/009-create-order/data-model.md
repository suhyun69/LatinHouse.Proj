# Data Model: 주문 생성 (POST /api/order)

**Date**: 2026-06-03

## 신규 도메인 객체

### Domain: Order

패키지: `com.latinhouse.backend.order.domain`

| 필드 | 타입 | 설명 |
|------|------|------|
| id | String | UUID 형식 주문 ID |
| lessonNo | Long | 레슨 ID |
| lessonOptionNo | Long | 수업 옵션 ID |
| buyer | String | 구매자 Profile.id |
| price | BigDecimal | 총 주문 금액 |
| paymentId | Long | 결제 ID. 생성 시 null |
| discounts | List\<OrderDiscount\> | 할인 내역. 생성 시 빈 리스트 |
| status | OrderStatus | 주문 상태. 생성 시 PAYMENT_PENDING |

### Domain: OrderDiscount

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | |
| orderId | String | |
| discountType | OrderDiscountType | |
| discountId | Long | |
| amount | BigDecimal | |

### Enum: OrderStatus

패키지: `com.latinhouse.backend.order.domain`

| 값 | 설명 |
|----|------|
| PAYMENT_PENDING | 결제 대기 (초기 상태) |
| PAYMENT_COMPLETED | 결제 완료 |
| APPROVED | 승인 완료 |
| CANCELED | 취소됨 |

### Enum: OrderDiscountType

| 값 | 설명 |
|----|------|
| Lesson | 레슨 할인 |
| Coupon | 쿠폰 할인 |

---

## 신규 JPA Entity

### Entity: OrderJpaEntity

테이블: `order_table` (MySQL 예약어 `order` 회피)

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | String | PK | UUID |
| lessonNo | Long | NOT NULL | |
| lessonOptionNo | Long | NOT NULL | |
| buyer | String | NOT NULL | Profile.id |
| price | BigDecimal | NOT NULL | |
| paymentId | Long | | |
| status | String | NOT NULL | OrderStatus enum (STRING) |

### Entity: OrderDiscountJpaEntity

테이블: `order_discount`

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | Long | PK, auto increment | |
| orderId | String | NOT NULL, FK → order_table.id | |
| discountType | String | NOT NULL | OrderDiscountType enum |
| discountId | Long | NOT NULL | |
| amount | BigDecimal | NOT NULL | |

---

## 신규 DTO 클래스

### Web Layer (`adapter/in/web/`)

#### CreateOrderWebRequest

| 필드 | 타입 | 검증 | 설명 |
|------|------|------|------|
| lessonNo | Long | `@NotNull` | 레슨 ID |
| lessonOptionNo | Long | `@NotNull` | 수업 옵션 ID |
| profileId | String | `@NotBlank` | 구매자 프로필 ID |

#### CreateOrderWebResponse

| 필드 | 타입 | 설명 |
|------|------|------|
| orderId | String | 생성된 주문 UUID |

### Application Layer (`port/in/`)

#### CreateOrderAppRequest

| 필드 | 타입 | 설명 |
|------|------|------|
| lessonNo | Long | |
| lessonOptionNo | Long | |
| profileId | String | |

#### CreateOrderAppResponse

| 필드 | 타입 | 설명 |
|------|------|------|
| orderId | String | |

---

## 재사용 클래스

| 클래스 | 위치 | 재사용 방식 |
|--------|------|------------|
| `LessonRepository` | `lesson/adapter/out/persistence/repository/` | lessonNo 존재 확인 |
| `ProfileRepository` | `profile/adapter/out/persistence/repository/` | profileId 존재 확인 |
| `LessonOptionRepository` | `lesson/adapter/out/persistence/repository/` | lessonOptionNo 존재 확인 |
| `ErrorCode` | `global/exception/` | LESSON_NOT_FOUND, PROFILE_NOT_FOUND 재사용. LESSON_OPTION_NOT_FOUND 추가 필요 |
| `CustomException` | `global/exception/` | 404 트리거 |

---

## 처리 흐름 (CreateOrderService)

```
Controller → CreateOrderAppRequest.from(webReq)
           → CreateOrderUseCase.create(appReq)
               1. Lesson lesson = loadLessonPort.load(appReq.getLessonNo())         // LESSON_NOT_FOUND
               2. loadLessonOptionPort.load(appReq.getLessonOptionNo())              // LESSON_OPTION_NOT_FOUND
               3. loadProfilePort.load(appReq.getProfileId())                       // PROFILE_NOT_FOUND
               4. Order order = Order.builder()
                      .id(UUID.randomUUID().toString())
                      .lessonNo(appReq.getLessonNo())
                      .lessonOptionNo(appReq.getLessonOptionNo())
                      .buyer(appReq.getProfileId())
                      .price(lesson.getPrice())
                      .paymentId(null)
                      .discounts(List.of())
                      .status(OrderStatus.PAYMENT_PENDING)
                      .build()
               5. Order saved = saveOrderPort.save(order)
               6. return CreateOrderAppResponse(saved.getId())
           → new CreateOrderWebResponse(appResponse.getOrderId())
           → ResponseEntity.status(201).body(webResponse)
```
