# Research: 레슨 단건 조회 (GET /api/lessons/{lessonNo})

## 1. 기존 Lesson 도메인 분석

**Decision**: 기존 `LessonJpaRepository`(JpaRepository<LessonEntity, Long>)의 `findById(Long id)`를 그대로 사용한다.

**Rationale**: `LessonEntity`는 options, discounts, account, contacts, notices를 모두 `CascadeType.ALL`로 연관 관리하고 있으며, `LessonPersistenceMapper.toDomain()`이 이미 모든 하위 엔티티를 도메인 객체로 변환한다. 추가 쿼리 최적화(Fetch Join 등)는 현재 스케일에서 불필요하다.

**Alternatives considered**:
- JPQL Fetch Join 쿼리: 현재 H2 + 단건 조회 스케일에서 오버엔지니어링
- 별도 QueryRepository: 단순 findById로 충분

---

## 2. LessonNotFoundException 설계

**Decision**: `LessonNotFoundException`을 `RuntimeException`으로 신규 정의하고, `GlobalExceptionHandler`에서 404 응답으로 처리한다.

**Rationale**: 기존 `ProfileNotFoundException`과 동일한 패턴을 따른다. `common/exception/` 패키지에 위치.

**Alternatives considered**:
- `EntityNotFoundException` 재사용: 에러 코드 구분이 어려움
- `LessonValidationException` 재사용: 의미적으로 부적합

---

## 3. 응답 DTO 구조 결정

**Decision**: `GetLessonAppResponse`와 `GetLessonWebResponse` 모두 정적 내부 클래스(Option, Discount, Account, Contact, Notice)를 포함하는 구조로 설계한다.

**Rationale**: 기존 `CreateLessonWebRequest`의 정적 내부 클래스 패턴과 일관성 유지. 응답 JSON이 중첩 객체를 포함하므로 별도 클래스로 분리하는 것이 가독성에 유리하다.

**Alternatives considered**:
- 별도 파일로 각 DTO 분리: 파일 수가 많아져 관리 복잡도 증가

---

## 4. Controller URL 구조

**Decision**: 신규 엔드포인트를 `GET /api/lessons/{lessonNo}`로 정의하고, 기존 `LessonController`의 `@RequestMapping`을 `/api/lesson`에서 엔드포인트별로 분리하거나, 클래스 레벨을 `/api`로 변경하고 메서드별로 `/lesson`, `/lessons/{lessonNo}`를 지정한다.

**Rationale**: 기존 POST는 `/api/lesson`, 신규 GET은 `/api/lessons/{lessonNo}`로 명세에 정의되어 있다. 클래스 레벨 `@RequestMapping`을 `/api`로 변경하고 각 메서드에 개별 경로를 지정하는 방식이 깔끔하다.

**Alternatives considered**:
- 별도 `GetLessonController` 신설: 단일 책임 관점에서 가능하나, 규모가 작아 현재는 기존 Controller 확장으로 충분

---

## 5. 날짜/시간 직렬화

**Decision**: `LessonOptionEntity`의 `startDateTime`/`endDateTime`(LocalDateTime)을 `startDate`/`startTime`/`endDate`/`endTime` 4개 String 필드로 분리하여 응답한다. 변환은 `GetLessonWebMapper`에서 수행한다.

**Rationale**: API 명세(api-spec.md)가 `yyyy-MM-dd` / `HH:mm` 형식으로 분리하여 반환하도록 정의하고 있다.
