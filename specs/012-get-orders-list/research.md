# Research: 주문 목록 조회 (GET /api/orders)

## 결정사항

### Decision 1: 필터링 구현 방식 — JPA Specification
- **Decision**: `OrderJpaRepository`에 `JpaSpecificationExecutor<OrderEntity>` 추가, `Specification<OrderEntity>`로 buyer/lessonNo 동적 필터링 구현
- **Rationale**: 기존 `LessonPersistenceAdapter`가 동일 패턴(`LessonJpaRepository extends JpaRepository + JpaSpecificationExecutor`, `buildSpec()` 메서드)을 사용하므로 일관성을 유지한다. 파라미터 null 여부로 AND 조건을 동적으로 구성할 수 있다.
- **Alternatives considered**: `@Query` JPQL 쿼리 직접 작성 → 조건 조합이 많을수록 유지보수 어려움. `QueryDSL` → 의존성 추가 비용.

### Decision 2: Port 설계 — LoadOrderPort 신규 추가
- **Decision**: `application/port/out/LoadOrderPort` 인터페이스를 신규 추가하고, `OrderPersistenceAdapter`가 구현한다.
- **Rationale**: 기존 `SaveOrderPort`, `LoadLessonOptionPort`와 동일한 분리 원칙. PersistenceAdapter가 Port를 구현하되 UseCase에는 인터페이스만 노출한다.
- **Alternatives considered**: 기존 `SaveOrderPort`에 목록 조회 메서드 추가 → 단일 책임 원칙 위반.

### Decision 3: 응답 DTO — GetOrdersWebResponse (중첩 DTO OrderDiscountInfo 포함)
- **Decision**: `GetOrdersWebResponse`에 `discounts` 필드를 `List<OrderDiscountInfo>` 중첩 static class로 구성한다.
- **Rationale**: 기존 `CreateOrderWebResponse`가 단일 필드만 반환하여 참조 불가. Lesson 목록 조회의 GetLessonsWebResponse(단일 flat 객체) 패턴을 따르되, discounts는 컬렉션이므로 중첩 클래스가 적절하다.
- **Alternatives considered**: 별도 `OrderDiscountWebResponse` 클래스 → 간단한 피처에 과도한 분리.

### Decision 4: SecurityConfig — GET /api/orders permitAll 추가
- **Decision**: `SecurityConfig`에 `HttpMethod.GET, "/api/orders"` permitAll 추가
- **Rationale**: 기존 패턴(GET /api/lessons, POST /api/order 등 동일 방식)과 일치. 인증 범위는 이 피처 범위 외로 명세에 기재됨.
- **Alternatives considered**: 인증 적용 → 이 피처 명세 범위 외.
