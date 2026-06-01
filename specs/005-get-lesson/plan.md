# Implementation Plan: 레슨 단건 조회 (GET /api/lessons/{lessonNo})

**Branch**: `005-get-lesson` | **Date**: 2026-06-01 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `specs/005-get-lesson/spec.md`

## Summary

`GET /api/lessons/{lessonNo}` 엔드포인트를 구현한다. Path Variable로 lessonNo(Long)를 받아 레슨 상세 정보(Lesson + 모든 하위 엔티티)를 200 OK로 반환한다. 존재하지 않는 lessonNo 요청 시 404 + `LESSON_NOT_FOUND` 에러를 반환한다. Constitution의 Hexagonal Architecture를 따라 Two-DTO 패턴(AppResponse → WebResponse) + `GetLessonUseCase` / `LoadLessonPort`를 신규 정의하여 구현한다. 기존 `LessonPersistenceMapper.toDomain()`과 `LessonJpaRepository`를 재사용한다.

## Technical Context

**Language/Version**: Java 25

**Primary Dependencies**: Spring Boot 4.0.6, Spring Data JPA, Lombok, springdoc-openapi 3.0.0

**Storage**: H2 (개발/테스트용 인메모리 DB)

**Testing**: JUnit 5, `@WebMvcTest`, `@MockitoBean`, MockMvc

**Target Platform**: JVM 서버 (Spring Boot embedded Tomcat)

**Project Type**: REST API (Web Service)

**Performance Goals**: 표준 웹 API 응답 수준 (단일 조회 트랜잭션)

**Constraints**:
- Domain 레이어는 JPA, Spring, HTTP 등 외부 의존 금지 (Constitution I)
- 계층 간 변환은 반드시 Mapper 클래스 사용 (Constitution I — Mapper 패턴)
- WebRequest 없음(Path Variable만 사용). Bean Validation은 Controller 메서드 파라미터 수준에서 적용하지 않음 (lessonNo는 Long 타입으로 자동 변환 실패 시 400 처리됨)
- 모든 엔드포인트에 `@Tag`, `@Operation` Swagger 문서화 필수 (Constitution Quality Gate)
- 에러 응답은 반드시 `{ "status": ..., "errors": [...] }` 형식 (Constitution III)

**Scale/Scope**: 기존 lesson 도메인에 조회 유스케이스 추가. 신규 파일 위주이며 기존 파일 수정은 최소화.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| 원칙 | 준수 여부 | 비고 |
|------|-----------|------|
| I. Hexagonal Architecture | ✅ | Adapter → Application → Domain 단방향. `GetLessonUseCase` (port/in) + `LoadLessonPort` (port/out) 신규 정의 |
| Two-DTO 패턴 | ✅ | `GetLessonAppResponse` / `GetLessonWebResponse` 분리. WebRequest 없음 (Path Variable만 사용) |
| Mapper 패턴 | ✅ | `GetLessonWebMapper`(AppResponse→WebResponse), `GetLessonAppMapper`(Domain→AppResponse) 별도 클래스 |
| II. Contract-First API | ✅ | `docs/api-spec.md` GET /api/lessons/{lessonNo} 명세 기준 구현 |
| III. Unified Error Response | ✅ | `LessonNotFoundException` → GlobalExceptionHandler → 동일 에러 형식 반환 |
| IV. Validated Inputs | ✅ | lessonNo는 Long 타입 바인딩 실패 시 Spring이 400 자동 처리 |
| Quality Gate: Swagger | ✅ | `LessonController`에 신규 엔드포인트 `@Operation` 추가 |

**Gate Decision**: 모든 원칙 충족. 구현 진행.

## Project Structure

### Documentation (this feature)

```text
specs/005-get-lesson/
├── plan.md              # This file
├── spec.md              # Feature specification
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/
│   └── get-lesson.md   # Phase 1 output
└── tasks.md             # Phase 2 output (/speckit-tasks)
```

### Source Code

**신규 파일**:

```text
Latinhouse.Be/src/main/java/com/latinhouse/api/
└── lesson/
    ├── application/
    │   ├── port/in/
    │   │   ├── GetLessonUseCase.java
    │   │   ├── GetLessonAppResponse.java        # 정적 내부 클래스 포함 (Option, Discount, Account, Contact, Notice)
    │   │   └── GetLessonAppMapper.java           # Lesson 도메인 → GetLessonAppResponse
    │   ├── port/out/
    │   │   └── LoadLessonPort.java
    │   └── service/
    │       └── GetLessonService.java
    ├── adapter/
    │   ├── in/web/
    │   │   ├── GetLessonWebResponse.java         # 정적 내부 클래스 포함 (Option, Discount, Account, Contact, Notice)
    │   │   └── GetLessonWebMapper.java            # GetLessonAppResponse → GetLessonWebResponse
    │   └── out/persistence/
    │       └── (LessonPersistenceAdapter 수정)    # LoadLessonPort 구현 추가
└── common/
    └── exception/
        └── LessonNotFoundException.java

Latinhouse.Be/src/test/java/com/latinhouse/api/
└── lesson/
    ├── adapter/in/web/
    │   └── (LessonControllerTest.java 수정)      # GET 엔드포인트 테스트 추가
    └── application/service/
        └── GetLessonServiceTest.java
```

**수정 파일**:

```text
Latinhouse.Be/src/main/java/com/latinhouse/api/
├── lesson/
│   ├── adapter/
│   │   ├── in/web/
│   │   │   └── LessonController.java              # GET /api/lessons/{lessonNo} 엔드포인트 추가
│   │   └── out/persistence/
│   │       └── LessonPersistenceAdapter.java       # LoadLessonPort 구현 추가
└── common/
    └── exception/
        └── GlobalExceptionHandler.java             # LessonNotFoundException 핸들러 추가
```

**Structure Decision**: 기존 `lesson` 패키지 구조를 그대로 유지하며 조회 유스케이스를 추가한다. `LessonController`에 새 엔드포인트를 추가하고, `LessonPersistenceAdapter`에 `LoadLessonPort` 구현을 추가한다. `LessonPersistenceMapper.toDomain()`은 이미 모든 하위 엔티티를 변환하므로 재사용한다.
