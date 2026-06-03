# Research: GET /api/lessons 레슨 목록 조회

**Feature**: 006-get-lessons-list | **Date**: 2026-06-02

---

## 1. 동적 필터 쿼리 전략

**Decision**: JPA Specification + `JpaSpecificationExecutor` 사용

**Rationale**:
- region(옵션), genre·instructor(레슨) 필터가 모두 선택적이므로 동적 WHERE 절이 필요하다.
- `@Query` JPQL 방식은 필터 조합마다 별도 메서드가 필요해 조합 수(2^3=8)만큼 중복이 생긴다.
- JPA Specification은 조건 조각을 AND로 체이닝하는 표준 패턴이며 기존 `LessonJpaRepository`에 인터페이스만 추가하면 된다.
- region 필터는 LessonOptionEntity JOIN이 필요하고, `Specification<LessonEntity>` 내에서 `Root.join("options")`으로 처리 가능하다.

**Alternatives considered**:
- QueryDSL: 기존 프로젝트에 의존성 없음, 추가 빌드 설정 비용이 크다.
- 전체 조회 후 Java Stream 필터: 데이터량이 적으면 단순하지만 DB 레벨 필터가 없어 비효율적이다.

---

## 2. 응답 단위: Lesson vs Option 단위 flat list

**Decision**: Option 단위 flat list (1 option = 1 row)

**Rationale**:
- 응답 DTO에 `optionId`, `startDate`, `startTime`, `endDate`, `endTime`, `region`이 Option 필드이고 `status`도 Option 단위로 계산된다.
- 클라이언트가 옵션별로 카드/행을 렌더링하는 UI에 적합하다.
- Lesson이 N개 옵션을 가지면 N개 행 반환.

**Alternatives considered**:
- Lesson 단위 + options 배열 중첩: 단건 조회(005)와 동일한 형태지만 요구사항에서 optionId가 최상위 필드로 명시됨.

---

## 3. status 계산 시점

**Decision**: Service 계층(또는 AppMapper)에서 `LocalDateTime.now()` 기준으로 계산

**Rationale**:
- DB에서 status를 계산하면 DB 시간대에 종속되고, 로직 변경 시 쿼리 수정이 필요하다.
- 도메인/서비스에서 계산하면 테스트가 용이하고 로직이 명확하다.
- 기존 패턴(Domain → AppMapper → AppResponse)에 자연스럽게 맞음.

---

## 4. discount 계산 로직 위치

**Decision**: `GetLessonsAppMapper`에서 계산, domain 객체(`LessonDiscount`)의 기존 필드를 재사용

**Rationale**:
- `LessonDiscount.condition`은 String으로 저장됨 (EARLYBIRD: yyyy-MM-dd, SEX: M/F).
- EARLYBIRD 필터 후 `LocalDate.parse(condition)` 비교는 매핑 로직이므로 Mapper에 위치.
- Domain 객체 변경 불필요.

---

## 5. 유효하지 않은 필터 코드 처리

**Decision**: Controller 파라미터 수신 시 `Region.fromCode()` / `Genre.fromCode()` 호출, 실패 시 400 반환

**Rationale**:
- 기존 enum(`Region`, `Genre`)에 이미 `fromCode(String)` 메서드가 존재한다.
- WebMapper에서 변환 중 `IllegalArgumentException` 발생 → `GlobalExceptionHandler`에서 400으로 처리.
- 별도 Validator 클래스 불필요.

---

## 6. 재사용 가능한 기존 구성요소

| 구성요소 | 재사용 방법 |
|----------|------------|
| `LessonEntity`, `LessonOptionEntity`, `LessonDiscountEntity` | 그대로 재사용, 변경 없음 |
| `LessonPersistenceMapper.toDomain()` | Lesson 도메인 변환에 재사용 |
| `Region.fromCode()`, `Genre.fromCode()` | 필터 파라미터 변환에 재사용 |
| `LessonController` | 신규 엔드포인트 추가 |
| `LessonPersistenceAdapter` | `LoadLessonsPort` 구현 추가 |
| `GlobalExceptionHandler` | `IllegalArgumentException` 핸들러 추가 (없으면 신규) |
