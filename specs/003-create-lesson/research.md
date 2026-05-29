# Research: POST /api/lesson 레슨 생성

## Decision 1: LoadInstructorPort — 강사 검증 Port 설계

**Decision**: `LoadInstructorPort` 인터페이스를 `lesson.application.port.out`에 신규 정의하고, `InstructorPersistenceAdapter`(lesson.adapter.out.persistence)가 구현.

**Rationale**:
- `CreateLessonService`는 instructorLo/instructorLa가 유효한 강사인지 검증하기 위해 Profile 조회가 필요.
- Lesson Application 레이어가 Profile Application 레이어를 직접 참조하면 도메인 간 Application 계층 결합이 발생 → Constitution I 위반.
- `LoadInstructorPort` 인터페이스를 Lesson 도메인 내 port.out에 두고, Adapter 레이어(`InstructorPersistenceAdapter`)가 `ProfileJpaRepository` + `ProfilePersistenceMapper`를 사용해 구현하면 계층 결합 없이 Profile DB 조회 가능.
- Adapter 레이어 간 JPA Repository 참조는 허용됨 (Adapter → Adapter는 허용, Application → Application 결합은 금지).

**Alternatives considered**:
- `FindProfilePort` 재사용 (profile.application.port.out): Lesson Application이 Profile Application 포트를 의존 → 도메인 간 Application 계층 결합. 기각.
- `LoadInstructorPort`를 lesson.application.port.out에 두고 Profile Application Service를 DI: 동일 이유로 기각.

---

## Decision 2: 비즈니스 검증 전략 — 다중 오류 수집

**Decision**: `CreateLessonService`에서 모든 비즈니스 규칙을 검증하고 오류를 `List<ErrorResponse.FieldError>`에 누적. 오류가 하나 이상이면 `LessonValidationException(errors)`를 throw.

**Rationale**:
- FR-026: "오류가 복수인 경우 errors 배열에 모든 오류를 담아 반환해야 한다."
- options 다수 + discounts 다수 → 여러 항목에서 동시에 오류 발생 가능.
- 단일 예외 throw 방식(첫 오류에서 중단)으로는 복수 오류를 한 번에 반환 불가.
- `LessonValidationException(List<FieldError>)` + `GlobalExceptionHandler` 확장으로 Bean Validation과 동일한 에러 응답 구조 유지.

**검증 순서**:
1. instructorLo/La 존재 여부 (둘 다 null이면 즉시 추가)
2. instructorLo 유효성: findById → isInstructor → sex=M
3. instructorLa 유효성: findById → isInstructor → sex=F
4. 각 option별 startDateTime < endDateTime
5. 각 discount별 condition 형식 (type별)
6. 오류 누적 후 비어있지 않으면 throw

**Alternatives considered**:
- 첫 오류에서 즉시 throw: 복수 오류 반환 불가, FR-026 위반. 기각.
- 예외 종류별 분리 (InstructorNotFoundException, InstructorNotValidException 등): 예외마다 핸들러 추가 필요, 복수 오류 응답 어려움. 기각.

---

## Decision 3: WebRequest 입력 구조 — 중첩 정적 클래스

**Decision**: `CreateLessonWebRequest`에 `LessonOptionWebRequest`, `LessonDiscountWebRequest`, `LessonAccountWebRequest`, `LessonContactWebRequest`, `LessonNoticeWebRequest`를 정적 내부 클래스로 정의.

**Rationale**:
- 기존 Profile 패턴과 달리 Lesson은 서브 엔티티가 많아 별도 파일로 분리하면 클래스 수가 급증.
- 정적 내부 클래스는 Bean Validation `@Valid` 전파 가능 (`@Valid @NotEmpty List<LessonOptionWebRequest>`).
- AppRequest도 동일 구조로 정적 내부 클래스 사용.

**Alternatives considered**:
- 별도 파일로 분리: 각 sub-request마다 파일 생성 → 15개 이상 클래스 추가. 기각.

---

## Decision 4: startDate+startTime → LocalDateTime 변환 위치

**Decision**: `CreateLessonWebMapper`에서 `startDate + "T" + startTime`을 `LocalDateTime.parse()`로 결합하여 변환.

**Rationale**:
- Constitution I Mapper 패턴: "HTTP 원시 타입 → 도메인 타입 변환은 WebMapper에서 수행".
- AppRequest는 이미 도메인 타입(`LocalDateTime`)을 사용.
- WebMapper에서 변환 실패(DateTimeParseException)는 Bean Validation 통과 후 도달하므로 발생하지 않음 (Pattern 검증으로 형식 보장).

**Alternatives considered**:
- AppMapper에서 변환: AppRequest에 String 필드 유지 필요 → Constitution 위반(AppRequest는 도메인 타입 사용). 기각.
- Service에서 변환: 비즈니스 로직과 변환 로직 혼합. 기각.

---

## Decision 5: JPA Entity 관계 설계

**Decision**: `LessonEntity` ↔ 자식 엔티티는 `@OneToMany(cascade = ALL, orphanRemoval = true)`, `LessonAccountEntity`는 `@OneToOne(cascade = ALL)`.

**Rationale**:
- 레슨과 서브 엔티티는 레슨의 수명주기를 공유 (레슨 삭제 시 함께 삭제).
- `cascade = ALL` + `orphanRemoval = true`로 부모 저장 시 자식 자동 저장.
- 단일 `lessonJpaRepository.save(lessonEntity)` 호출로 전체 그래프 저장 가능.

**JPA Entity 매핑 방향**:
- `LessonOptionEntity.lesson` → `@ManyToOne @JoinColumn(name="lesson_id")`
- `LessonEntity.options` → `@OneToMany(mappedBy="lesson", cascade=ALL, orphanRemoval=true)`
- `LessonAccountEntity.lesson` → `@OneToOne @JoinColumn(name="lesson_id")`
- `LessonEntity.account` → `@OneToOne(mappedBy="lesson", cascade=ALL)`

**Alternatives considered**:
- `@ElementCollection`: 서브 엔티티에 별도 ID가 있으므로 부적합. 기각.
- 별도 Repository로 자식 저장: 단순한 생성 케이스에서 불필요한 복잡도. 기각.

---

## Decision 6: LessonValidationException 추가

**Decision**: `common/exception/LessonValidationException.java` 신규 생성. `List<ErrorResponse.FieldError>` 보유. `GlobalExceptionHandler`에 핸들러 추가 (400 Bad Request).

**Rationale**:
- ProfileNotFoundException(404) 패턴과 동일하게 커스텀 예외로 분리.
- 비즈니스 검증 오류는 400으로 처리하되, 여러 errors를 한 번에 반환.
- 기존 `ErrorResponse.FieldError` 재사용 → 에러 응답 형식 일관성 유지.

---

## 신규 파일 목록 (Resolution)

| 분류 | 파일 | 역할 |
|------|------|------|
| 신규 | `lesson/domain/Lesson.java` | 레슨 도메인 객체 |
| 신규 | `lesson/domain/LessonOption.java` | 수업 옵션 도메인 객체 |
| 신규 | `lesson/domain/LessonDiscount.java` | 할인 도메인 객체 |
| 신규 | `lesson/domain/LessonAccount.java` | 계좌 도메인 객체 |
| 신규 | `lesson/domain/LessonContact.java` | 연락처 도메인 객체 |
| 신규 | `lesson/domain/LessonNotice.java` | 공지 도메인 객체 |
| 신규 | `lesson/domain/Genre.java` | 장르 Enum |
| 신규 | `lesson/domain/Region.java` | 지역 Enum |
| 신규 | `lesson/domain/DiscountType.java` | 할인 타입 Enum |
| 신규 | `lesson/domain/ContactType.java` | 연락처 타입 Enum |
| 신규 | `lesson/domain/NoticeType.java` | 공지 타입 Enum |
| 신규 | `lesson/application/port/in/CreateLessonUseCase.java` | UseCase 인터페이스 |
| 신규 | `lesson/application/port/in/CreateLessonAppRequest.java` | App 입력 DTO (내부 클래스 포함) |
| 신규 | `lesson/application/port/in/CreateLessonAppResponse.java` | App 결과 DTO |
| 신규 | `lesson/application/port/in/CreateLessonAppMapper.java` | AppMapper |
| 신규 | `lesson/application/port/out/SaveLessonPort.java` | 레슨 저장 Port |
| 신규 | `lesson/application/port/out/LoadInstructorPort.java` | 강사 조회 Port |
| 신규 | `lesson/application/service/CreateLessonService.java` | UseCase 구현체 |
| 신규 | `lesson/adapter/in/web/CreateLessonWebRequest.java` | HTTP 입력 DTO (내부 클래스 포함) |
| 신규 | `lesson/adapter/in/web/CreateLessonWebResponse.java` | HTTP 응답 DTO |
| 신규 | `lesson/adapter/in/web/CreateLessonWebMapper.java` | WebMapper |
| 신규 | `lesson/adapter/in/web/LessonController.java` | REST Controller |
| 신규 | `lesson/adapter/out/persistence/LessonEntity.java` | JPA Entity |
| 신규 | `lesson/adapter/out/persistence/LessonOptionEntity.java` | JPA Entity |
| 신규 | `lesson/adapter/out/persistence/LessonDiscountEntity.java` | JPA Entity |
| 신규 | `lesson/adapter/out/persistence/LessonAccountEntity.java` | JPA Entity |
| 신규 | `lesson/adapter/out/persistence/LessonContactEntity.java` | JPA Entity |
| 신규 | `lesson/adapter/out/persistence/LessonNoticeEntity.java` | JPA Entity |
| 신규 | `lesson/adapter/out/persistence/LessonJpaRepository.java` | Spring Data JPA Repository |
| 신규 | `lesson/adapter/out/persistence/LessonPersistenceAdapter.java` | SaveLessonPort 구현 |
| 신규 | `lesson/adapter/out/persistence/LessonPersistenceMapper.java` | PersistenceMapper |
| 신규 | `lesson/adapter/out/persistence/InstructorPersistenceAdapter.java` | LoadInstructorPort 구현 |
| 신규 | `common/exception/LessonValidationException.java` | 비즈니스 검증 예외 |
| 수정 | `common/exception/GlobalExceptionHandler.java` | LessonValidationException 핸들러 추가 |
| 신규 (테스트) | `lesson/adapter/in/web/LessonControllerTest.java` | Controller 슬라이스 테스트 |
| 신규 (테스트) | `lesson/application/service/CreateLessonServiceTest.java` | Service 유닛 테스트 |
