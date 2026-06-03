# Implementation Plan: 랜덤 수업 생성 (POST /api/lesson/random)

**Branch**: `007-random-lesson-create` | **Date**: 2026-06-03 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `specs/007-random-lesson-create/spec.md`

## Summary

`POST /api/lesson/random` 엔드포인트를 구현한다. Request Body 없이 호출하면 서버가 강사 프로필을 조회하여 랜덤 할당(없으면 자동 생성·강사 지정)하고, 나머지 레슨 파라미터를 랜덤으로 생성하여 기존 `CreateLessonUseCase`를 재사용해 수업을 생성한다. 201 Created와 생성된 레슨 ID를 반환한다.

## Technical Context

**Language/Version**: Java 25

**Primary Dependencies**: Spring Boot 4.0.6, Spring Data JPA, Lombok, springdoc-openapi 3.0.0

**Storage**: H2 (개발/테스트용 인메모리 DB)

**Testing**: JUnit 5, `@WebMvcTest`, `@MockitoBean`, MockMvc

**Target Platform**: JVM 서버 (Spring Boot embedded Tomcat)

**Project Type**: REST API (Web Service)

**Performance Goals**: 표준 웹 API 응답 수준 (테스트·시연 목적)

**Constraints**:
- Domain 레이어는 JPA, Spring, HTTP 등 외부 의존 금지 (Constitution I)
- 계층 간 변환은 반드시 Mapper 클래스 사용 (Constitution I — Mapper 패턴)
- 랜덤 생성 로직은 Application 계층 Service에서 담당
- 기존 `CreateLessonUseCase`, `GetProfilesUseCase`, `CreateProfileUseCase`, `SetInstructorUseCase` 재사용
- 모든 엔드포인트에 `@Tag`, `@Operation` Swagger 문서화 필수 (Constitution Quality Gate)
- 에러 응답은 반드시 `{ "status": ..., "errors": [...] }` 형식 (Constitution III)

**Scale/Scope**: 기존 lesson 도메인에 랜덤 생성 유스케이스 추가. 기존 Entity·Domain 변경 없음.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| 원칙 | 준수 여부 | 비고 |
|------|-----------|------|
| I. Hexagonal Architecture | ✅ | `CreateRandomLessonUseCase` (port/in) 신규 정의. 기존 포트(out) 재사용 |
| Two-DTO 패턴 | ✅ | WebRequest 없음(Body 없는 엔드포인트). `CreateRandomLessonAppResponse` / `CreateRandomLessonWebResponse` 분리 |
| Mapper 패턴 | ✅ | `CreateRandomLessonWebMapper`(AppResponse→WebResponse), `CreateRandomLessonAppMapper`(랜덤 파라미터→AppRequest·AppResponse 변환) |
| II. Contract-First API | ✅ | `docs/api-spec.md` POST /api/lesson/random 명세 기준 구현 |
| III. Unified Error Response | ✅ | 프로필 생성/강사 지정 실패 → RuntimeException → GlobalExceptionHandler → 500 |
| IV. Validated Inputs | ✅ | Body 없음. 입력 없으므로 Bean Validation 불필요 |
| Quality Gate: Swagger | ✅ | `LessonController`에 `@Operation` 추가 |

**Gate Decision**: 모든 원칙 충족. 구현 진행.

## Project Structure

### Documentation (this feature)

```text
specs/007-random-lesson-create/
├── plan.md              # This file
├── spec.md              # Feature specification
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/
│   └── post-lesson-random.md  # Phase 1 output
└── tasks.md             # Phase 2 output (/speckit-tasks)
```

### Source Code

**신규 파일**:

```text
Latinhouse.Be/src/main/java/com/latinhouse/api/
└── lesson/
    ├── adapter/
    │   └── in/web/
    │       ├── CreateRandomLessonWebMapper.java     # AppResponse → WebResponse
    │       └── CreateRandomLessonWebResponse.java   # { "id": Long }
    ├── application/
    │   ├── port/in/
    │   │   ├── CreateRandomLessonUseCase.java       # createRandomLesson() : CreateRandomLessonAppResponse
    │   │   ├── CreateRandomLessonAppResponse.java   # { Long id }
    │   │   └── CreateRandomLessonAppMapper.java     # 랜덤 파라미터 → CreateLessonAppRequest 생성
    │   └── service/
    │       └── CreateRandomLessonService.java       # UseCase 구현체. 강사 조회/생성/할당 + 랜덤 레슨 파라미터 생성 + CreateLessonUseCase 호출
```

**수정 파일**:

```text
Latinhouse.Be/src/main/java/com/latinhouse/api/
└── lesson/
    └── adapter/
        └── in/web/
            └── LessonController.java               # POST /api/lesson/random 엔드포인트 추가
```

**테스트 파일**:

```text
Latinhouse.Be/src/test/java/com/latinhouse/api/
└── lesson/
    ├── adapter/in/web/
    │   └── CreateRandomLessonControllerTest.java    # @WebMvcTest: 201 반환, 500 에러 검증
    └── application/service/
        └── CreateRandomLessonServiceTest.java       # 강사 있음/없음 케이스, 랜덤 파라미터 유효성 검증
```

**Structure Decision**: lesson 도메인 내에 기존 Create/Get 유스케이스와 동일한 패턴으로 `CreateRandom` 유스케이스를 추가한다. 신규 persistence adapter 불필요 (저장은 기존 `SaveLessonPort` 재사용).

## Phase 0: Research

*연구 결과 요약 → [research.md](research.md) 참조*

**핵심 결정사항**:

1. **랜덤 생성 로직 위치**: Application Service (`CreateRandomLessonService`)에서 담당. Domain 오염 방지.
2. **기존 UseCase 재사용**: `CreateLessonUseCase`를 Service 내에서 직접 호출(UseCase → UseCase 의존). 새로운 Port(out) 불필요.
3. **강사 자동 생성 흐름**: `GetProfilesUseCase` → 성별 분리 → 랜덤 선택. 없으면 `CreateProfileUseCase` + `SetInstructorUseCase` 호출.
4. **오류 처리**: 강사 생성 실패는 `RuntimeException`을 던져 `GlobalExceptionHandler`가 500으로 처리.
5. **Request Body**: 없음. Controller에서 `@RequestBody` 없이 단순 POST 매핑.

## Phase 1: Design

### 클래스별 책임

#### `CreateRandomLessonUseCase`
```java
public interface CreateRandomLessonUseCase {
    CreateRandomLessonAppResponse createRandomLesson();
}
```

#### `CreateRandomLessonAppResponse`
```java
@Getter @Builder
public class CreateRandomLessonAppResponse {
    private final Long id;
}
```

#### `CreateRandomLessonAppMapper`
- `toCreateLessonAppRequest(String instructorLo, String instructorLa)` → `CreateLessonAppRequest`
  - genre, title, options(1~3개), amount, discounts(0~2개), isActive=true 랜덤 생성
  - options의 startDate: 오늘+7~60일, startTime: [10:00, 14:00, 19:00, 20:00] 중 랜덤, endTime: +2시간
  - discounts의 type=E condition: 가장 이른 startDate - 7일
- `toAppResponse(CreateLessonAppResponse)` → `CreateRandomLessonAppResponse`

#### `CreateRandomLessonWebMapper`
- `toWebResponse(CreateRandomLessonAppResponse)` → `CreateRandomLessonWebResponse`

#### `CreateRandomLessonWebResponse`
```java
@Getter @Builder
public class CreateRandomLessonWebResponse {
    private final Long id;
}
```

#### `CreateRandomLessonService`
```java
@Service
@RequiredArgsConstructor
public class CreateRandomLessonService implements CreateRandomLessonUseCase {
    private final GetProfilesUseCase getProfilesUseCase;
    private final CreateProfileUseCase createProfileUseCase;
    private final SetInstructorUseCase setInstructorUseCase;
    private final CreateLessonUseCase createLessonUseCase;

    public CreateRandomLessonAppResponse createRandomLesson() {
        // 1. 강사 목록 조회 (isInstructor=true)
        // 2. 남성/여성 강사 분리 및 랜덤 선택 (없으면 생성 + 강사 지정)
        // 3. instructorLo, instructorLa 결정 (최소 1명)
        // 4. CreateRandomLessonAppMapper로 랜덤 파라미터 생성
        // 5. createLessonUseCase.createLesson() 호출
        // 6. CreateRandomLessonAppResponse 반환
    }
}
```

#### `LessonController` 추가 메서드
```java
@PostMapping("/random")
@Operation(summary = "랜덤 수업 생성", description = "파라미터를 랜덤으로 생성하여 수업을 생성합니다.")
public ResponseEntity<CreateRandomLessonWebResponse> createRandomLesson() {
    CreateRandomLessonAppResponse appResponse = createRandomLessonUseCase.createRandomLesson();
    return ResponseEntity.status(HttpStatus.CREATED)
            .body(CreateRandomLessonWebMapper.toWebResponse(appResponse));
}
```

### 강사 할당 로직 (Service 내부)

```
instructors = getProfilesUseCase.getProfiles(true)  // isInstructor=true 전체

maleInstructors   = instructors.filter(sex == M)
femaleInstructors = instructors.filter(sex == F)

// 남성/여성 각각 후보가 있으면 랜덤 선택, 없으면 자동 생성
loId = maleInstructors.isEmpty()   ? createInstructor(M) : random(maleInstructors).id
laId = femaleInstructors.isEmpty() ? createInstructor(F) : random(femaleInstructors).id

// 둘 다 있는 경우 랜덤으로 하나만 null로 설정 가능 (0~1개 null 허용, 둘 다 null 금지)
// 단순 구현: 항상 둘 다 할당 (spec에서 "하나 이상" 조건만 만족하면 됨)
```

### 랜덤 파라미터 생성 규칙 (AppMapper)

| 필드 | 생성 규칙 |
|------|-----------|
| genre | `Random.nextBoolean()` → `S` 또는 `B` |
| title | genre + 레벨(`초급`/`중급`/`상급` random) |
| options 개수 | `ThreadLocalRandom.current().nextInt(1, 4)` (1~3) |
| option.startDate | `LocalDate.now().plusDays(7 + random(54))` |
| option.startTime | `[10, 14, 19, 20][random(4)]` |
| option.endTime | startTime + 2시간 |
| option.region | `GN` 또는 `HD` random |
| amount | `[30000, 50000, 80000, 100000][random(4)]` |
| discounts 개수 | `random(3)` (0~2) |
| discount.type | `E` 또는 `S` random |
| discount(E).condition | 가장 이른 startDate - 7일 |
| discount(S).condition | `M` 또는 `F` random |
| discount.amount | `[5000, 10000, 15000][random(3)]` |
| isActive | `true` 고정 |

## Complexity Tracking

해당 없음. Constitution 위반 없음.
