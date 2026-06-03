# Implementation Plan: 레슨 목록 조회 (GET /api/lessons)

**Branch**: `006-get-lessons-list` | **Date**: 2026-06-02 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `specs/006-get-lessons-list/spec.md`

## Summary

`GET /api/lessons` 엔드포인트를 구현한다. Query Parameter로 region/instructor/genre 필터(선택)를 받아 레슨 옵션 단위 flat list를 200 OK로 반환한다. 각 항목에는 옵션의 startDateTime/endDateTime + isActive로 계산된 `status`와 EarlyBird 할인 정보가 포함된다. Constitution의 Hexagonal Architecture를 따라 `GetLessonsUseCase` / `LoadLessonsPort`를 신규 정의하고, JPA Specification으로 동적 필터 쿼리를 구현한다.

## Technical Context

**Language/Version**: Java 25

**Primary Dependencies**: Spring Boot 4.0.6, Spring Data JPA (JpaSpecificationExecutor), Lombok, springdoc-openapi 3.0.0

**Storage**: H2 (개발/테스트용 인메모리 DB)

**Testing**: JUnit 5, `@WebMvcTest`, `@MockitoBean`, MockMvc

**Target Platform**: JVM 서버 (Spring Boot embedded Tomcat)

**Project Type**: REST API (Web Service)

**Performance Goals**: 표준 웹 API 응답 수준

**Constraints**:
- Domain 레이어는 JPA, Spring, HTTP 등 외부 의존 금지 (Constitution I)
- 계층 간 변환은 반드시 Mapper 클래스 사용 (Constitution I — Mapper 패턴)
- 쿼리 파라미터 수신은 Controller에서 String으로 받아 WebMapper에서 enum 변환 (`Region.fromCode`, `Genre.fromCode`)
- 모든 엔드포인트에 `@Tag`, `@Operation` Swagger 문서화 필수 (Constitution Quality Gate)
- 에러 응답은 반드시 `{ "status": ..., "errors": [...] }` 형식 (Constitution III)

**Scale/Scope**: 기존 lesson 도메인에 목록 조회 유스케이스 추가. 기존 Entity·Domain 변경 없음.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| 원칙 | 준수 여부 | 비고 |
|------|-----------|------|
| I. Hexagonal Architecture | ✅ | `GetLessonsUseCase` (port/in) + `LoadLessonsPort` (port/out) 신규 정의 |
| Two-DTO 패턴 | ✅ | `GetLessonsAppRequest`/`GetLessonsAppResponse` / `GetLessonsWebResponse` 분리. WebRequest 없음(쿼리 파라미터만) |
| Mapper 패턴 | ✅ | `GetLessonsWebMapper`(String param→AppRequest, AppResponse→WebResponse), `GetLessonsAppMapper`(Domain→AppResponse) |
| II. Contract-First API | ✅ | `docs/api-spec.md` GET /api/lessons 명세 기준 구현 |
| III. Unified Error Response | ✅ | `IllegalArgumentException`(잘못된 enum 코드) → GlobalExceptionHandler → 400 |
| IV. Validated Inputs | ✅ | region/genre 코드 변환 실패 시 즉시 400 처리 |
| Quality Gate: Swagger | ✅ | `LessonController`에 `@Operation` 추가 |

**Gate Decision**: 모든 원칙 충족. 구현 진행.

## Project Structure

### Documentation (this feature)

```text
specs/006-get-lessons-list/
├── plan.md              # This file
├── spec.md              # Feature specification
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/
│   └── get-lessons.md  # Phase 1 output
└── tasks.md             # Phase 2 output (/speckit-tasks)
```

### Source Code

**신규 파일**:

```text
Latinhouse.Be/src/main/java/com/latinhouse/api/
└── lesson/
    ├── application/
    │   ├── port/in/
    │   │   ├── GetLessonsUseCase.java
    │   │   ├── GetLessonsAppRequest.java          # region: Region, instructor: String, genre: Genre (모두 nullable)
    │   │   ├── GetLessonsAppResponse.java         # flat item (optionId, lessonNo, ... status, discountCondition, discountAmount)
    │   │   └── GetLessonsAppMapper.java           # List<Lesson> + filter → List<GetLessonsAppResponse>. status·discount 계산 포함
    │   ├── port/out/
    │   │   └── LoadLessonsPort.java               # loadLessons(region, instructor, genre): List<Lesson>
    │   └── service/
    │       └── GetLessonsService.java
    └── adapter/
        └── in/web/
            ├── GetLessonsWebResponse.java         # GetLessonsAppResponse와 동일 필드, HTTP 직렬화용
            └── GetLessonsWebMapper.java            # String param → AppRequest, AppResponse → WebResponse
```

**수정 파일**:

```text
Latinhouse.Be/src/main/java/com/latinhouse/api/
├── lesson/
│   ├── adapter/
│   │   ├── in/web/
│   │   │   └── LessonController.java              # GET /api/lessons 엔드포인트 추가
│   │   └── out/persistence/
│   │       ├── LessonJpaRepository.java           # JpaSpecificationExecutor<LessonEntity> 추가
│   │       └── LessonPersistenceAdapter.java       # LoadLessonsPort 구현 추가 (Specification 조합)
└── common/
    └── exception/
        └── GlobalExceptionHandler.java             # IllegalArgumentException 핸들러 추가 (없으면 신규)
```

**Structure Decision**: 기존 `lesson` 패키지 구조를 유지하며 목록 조회 유스케이스를 추가한다. Entity·Domain 객체는 변경하지 않는다.

## 구현 순서 (의존성 기준)

1. `LoadLessonsPort` 인터페이스 정의
2. `GetLessonsUseCase` + `GetLessonsAppRequest` + `GetLessonsAppResponse` 정의
3. `GetLessonsAppMapper` 구현 (status·discount 계산 로직 포함)
4. `GetLessonsService` 구현
5. `LessonJpaRepository`에 `JpaSpecificationExecutor` 추가
6. `LessonPersistenceAdapter`에 `LoadLessonsPort` 구현 (Specification 조합)
7. `GetLessonsWebResponse` + `GetLessonsWebMapper` 구현
8. `LessonController`에 엔드포인트 추가
9. `GlobalExceptionHandler`에 `IllegalArgumentException` 핸들러 확인·추가
10. 테스트: `GetLessonsServiceTest`, `LessonControllerTest` (GET /api/lessons 케이스 추가)
