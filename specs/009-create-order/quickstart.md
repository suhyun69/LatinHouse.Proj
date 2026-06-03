# Quickstart: 주문 생성 (POST /api/order)

**Date**: 2026-06-03

## 구현 요약

`POST /api/v1/order` 엔드포인트. 구매자, 레슨, 수업 옵션을 받아 주문을 생성하고 UUID 형식의 orderId를 반환한다.

## 생성할 파일 목록

### 신규 생성

| 파일 | 위치 |
|------|------|
| `ApiV1OrderController.java` | `order/adapter/in/web/` |
| `CreateOrderWebRequest.java` | `order/adapter/in/web/request/` |
| `CreateOrderWebResponse.java` | `order/adapter/in/web/response/` |
| `CreateOrderAppRequest.java` | `order/port/in/request/` |
| `CreateOrderAppResponse.java` | `order/port/in/response/` |
| `CreateOrderUseCase.java` | `order/port/in/` |
| `SaveOrderPort.java` | `order/port/out/` |
| `LoadLessonOptionPort.java` | `order/port/out/` |
| `CreateOrderService.java` | `order/application/service/` |
| `Order.java` | `order/domain/` |
| `OrderDiscount.java` | `order/domain/` |
| `OrderStatus.java` | `order/domain/` |
| `OrderDiscountType.java` | `order/domain/` |
| `OrderJpaEntity.java` | `order/adapter/out/persistence/entity/` |
| `OrderDiscountJpaEntity.java` | `order/adapter/out/persistence/entity/` |
| `OrderRepository.java` | `order/adapter/out/persistence/repository/` |
| `OrderPersistenceAdapter.java` | `order/adapter/out/persistence/` |
| `OrderMapper.java` | `order/adapter/out/persistence/mapper/` |
| `ApiV1OrderControllerTest.java` | `test/.../order/adapter/in/web/` |
| `CreateOrderServiceTest.java` | `test/.../order/application/service/` |

### 수정할 파일

| 파일 | 변경 내용 |
|------|-----------|
| `ErrorCode.java` | `LESSON_OPTION_NOT_FOUND` 추가 |
| `ApiSecurityConfig.java` | `POST /api/*/order` permitAll 추가 |

### 재사용할 Port (신규 구현 추가)

| Port | 구현 위치 |
|------|----------|
| `LoadLessonPort` | `lesson/adapter/out/persistence/LessonPersistenceAdapter.java` — 기존 메서드 활용 |
| `LoadProfilePort` | `profile/adapter/out/persistence/ProfilePersistenceAdapter.java` — 기존 메서드 활용 |

## 핵심 패턴 참조

- **WebRequest**: `CreateLessonWebRequest` — `@NotNull`, `@NotBlank` 패턴
- **AppRequest static factory**: `CreateLessonAppRequest.from(webReq)` — 동일 패턴
- **JPA Entity PK**: `Order`는 String PK (UUID). `@Id`만 사용, `@GeneratedValue` 없음
- **테이블명**: `order_table` (MySQL 예약어 회피)
- **SecurityConfig**: `ApiSecurityConfig`에 `requestMatchers(HttpMethod.POST, "/api/*/order").permitAll()` 추가

## 검증 체크리스트

- [ ] `POST /api/v1/order` 성공 → 201 Created + `{"orderId": "uuid..."}`
- [ ] lessonNo null → 400 + `"레슨을 선택해 주세요."`
- [ ] lessonOptionNo null → 400 + `"수업 옵션을 선택해 주세요."`
- [ ] profileId 빈 문자열 → 400 + `"구매자 프로필을 입력해 주세요."`
- [ ] 존재하지 않는 lessonNo → 404 + LESSON_NOT_FOUND
- [ ] 존재하지 않는 lessonOptionNo → 404 + LESSON_OPTION_NOT_FOUND
- [ ] 존재하지 않는 profileId → 404 + PROFILE_NOT_FOUND
- [ ] 생성된 주문의 status = PAYMENT_PENDING
