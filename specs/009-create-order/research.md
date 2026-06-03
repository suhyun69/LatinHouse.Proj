# Research: 주문 생성 (POST /api/order)

**Date**: 2026-06-03

## 1. URL 경로 규칙

**Decision**: 컨트롤러 매핑은 `/api/v1/order`를 사용한다.

**Rationale**:
- 기존 컨트롤러 패턴은 모두 `/api/v1/{domain}` 구조를 따른다 (`ApiV1LessonController`, `ApiV1ProfileController`).
- `api-spec.md`의 `POST /api/order` 명세는 버전 접두사 없이 표기되어 있으나, 실제 구현에서는 기존 컨벤션을 따라 `/api/v1/order`로 매핑한다.

**Alternatives Considered**:
- `/api/order` 그대로 사용: 기존 컨트롤러와 불일치, ApiSecurityConfig 패턴(`/api/*/...`) 위배.

---

## 2. 인증 요구사항

**Decision**: `POST /api/v1/order`는 인증 없이 접근 가능하도록 `ApiSecurityConfig`에 permitAll을 추가한다.

**Rationale**:
- `spec.md` Assumptions: "주문 생성은 인증 없이 접근 가능하다 (기존 API와 동일한 보안 정책 적용)."
- 기존 ApiSecurityConfig의 `.anyRequest().authenticated()` 기본 정책에 의해 JWT 없이 접근하면 403이 발생한다.
- 명시적으로 `requestMatchers(HttpMethod.POST, "/api/*/order").permitAll()` 추가가 필요하다.

---

## 3. ID 생성 전략 (orderId)

**Decision**: `UUID.randomUUID().toString()` 으로 orderId를 생성한다.

**Rationale**:
- spec에서 orderId는 UUID 형식으로 정의되어 있다.
- JPA에서 `@GeneratedValue`와 함께 `@GenericGenerator(name="uuid2")` 또는 Java `UUID.randomUUID()` 모두 사용 가능하나, 기존 프로젝트의 단순성을 유지하기 위해 서비스에서 직접 생성하고 `String` PK로 JPA Entity에 `@Id`만 선언한다.

---

## 4. 기존 패턴 재사용

**Decision**: 기존 `CreateLessonPort` / `FindLessonPort` 패턴을 그대로 따른다.

**Rationale**:
- Port 인터페이스: `SaveOrderPort`, `LoadLessonPort`, `LoadLessonOptionPort`, `LoadProfilePort`
- `LoadLessonPort`: LessonPersistenceAdapter에서 기존 Repository를 재사용하거나 신규 Port 구현 추가
- `LoadLessonOptionPort`: LessonOptionRepository 조회
- `LoadProfilePort`: ProfilePersistenceAdapter 재사용

**Alternatives Considered**:
- 직접 Repository를 Service에서 사용: Hexagonal Architecture 위배.

---

## 5. ErrorCode 추가

**Decision**: `ErrorCode` enum에 `LESSON_OPTION_NOT_FOUND`와 `ORDER_NOT_FOUND`를 추가한다.

**Rationale**:
- 기존 `LESSON_NOT_FOUND`, `PROFILE_NOT_FOUND`는 이미 정의되어 있다.
- `LESSON_OPTION_NOT_FOUND`는 이번 기능에서 신규 필요.
- `ORDER_NOT_FOUND`는 향후 주문 조회 기능 대비 미리 추가한다.

---

## 6. price 초기값

**Decision**: 주문 생성 시 `Order.price`는 `Lesson.price`로 초기화한다.

**Rationale**:
- spec: "price 초기값은 Lesson.amount로 설정한다."
- Lesson 도메인 객체의 `price` 필드를 그대로 복사한다.
