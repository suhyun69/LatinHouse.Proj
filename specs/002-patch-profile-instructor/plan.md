# Implementation Plan: 강사 지정 (PATCH /api/profile/{profileId}/instructor)

**Branch**: `002-patch-profile-instructor` | **Date**: 2026-05-28 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `specs/002-patch-profile-instructor/spec.md`

## Summary

`PATCH /api/profile/{profileId}/instructor` 엔드포인트를 구현한다. 경로 파라미터로 profileId를 받아 해당 프로필의 `isInstructor`를 `true`로 변경하고, 200 OK와 프로필 ID를 반환한다. Constitution의 Hexagonal Architecture를 따라 `FindProfilePort` + `UpdateProfilePort` + `Profile.asInstructor()` 도메인 메서드 조합으로 구현하며, 존재하지 않는 profileId는 `ProfileNotFoundException`으로 404를 반환한다.

## Technical Context

**Language/Version**: Java 25

**Primary Dependencies**: Spring Boot 4.0.6, Spring Data JPA, Spring Validation, Lombok, H2, springdoc-openapi 3.0.0

**Storage**: H2 (개발/테스트용 인메모리 DB)

**Testing**: JUnit 5, `@WebMvcTest`, `@MockitoBean`, MockMvc

**Target Platform**: JVM 서버 (Spring Boot embedded Tomcat)

**Project Type**: REST API (Web Service)

**Performance Goals**: 표준 웹 API 응답 수준 (단일 DB 조회 + 수정 CRUD)

**Constraints**:
- Domain 레이어는 JPA, Spring, HTTP 등 외부 의존 금지 (Constitution I)
- 계층 간 변환은 반드시 Mapper 클래스 사용 (Constitution I — Mapper 패턴)
- 모든 엔드포인트에 `@Tag`, `@Operation` Swagger 문서화 필수 (Constitution Quality Gate)
- 에러 응답은 반드시 `{ "status": ..., "errors": [...] }` 형식 (Constitution III)

**Scale/Scope**: 단일 엔드포인트 추가, 기존 코드베이스 확장

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| 원칙 | 준수 여부 | 비고 |
|------|-----------|------|
| I. Hexagonal Architecture | ✅ | Adapter → Application → Domain 단방향 의존 유지 |
| Two-DTO 패턴 | ✅ | `SetInstructorAppRequest/Response`, `SetInstructorWebResponse` 분리. WebRequest는 바디 없으므로 미생성. |
| Mapper 패턴 | ✅ | `SetInstructorWebMapper`, `SetInstructorAppMapper` 별도 클래스 |
| II. Contract-First API | ✅ | `docs/api-spec.md` 기준 구현 |
| III. Unified Error Response | ✅ | `ProfileNotFoundException` → `GlobalExceptionHandler` → `ErrorResponse` |
| IV. Validated Inputs | ✅ | Path variable 검증 없음 (8자리 String 단순 전달). 존재 여부는 Service에서 예외 처리. |
| Quality Gate: Swagger | ✅ | springdoc-openapi 이미 포함. `@Operation` 추가 필요. |

**Gate Decision**: 모든 원칙 충족. 구현 진행.

## Project Structure

### Documentation (this feature)

```text
specs/002-patch-profile-instructor/
├── plan.md              # This file
├── spec.md              # Feature specification
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/
│   └── patch-profile-instructor.md  # Phase 1 output
├── checklists/
│   └── requirements.md  # Quality checklist
└── tasks.md             # Phase 2 output (/speckit-tasks)
```

### Source Code

**신규 파일**:

```text
Latinhouse.Be/src/main/java/com/latinhouse/api/
├── profile/
│   ├── adapter/in/web/
│   │   ├── SetInstructorWebResponse.java        # HTTP 응답 DTO
│   │   └── SetInstructorWebMapper.java          # WebMapper
│   ├── application/
│   │   ├── port/in/
│   │   │   ├── SetInstructorUseCase.java        # UseCase 인터페이스
│   │   │   ├── SetInstructorAppRequest.java     # App 입력 DTO
│   │   │   ├── SetInstructorAppResponse.java    # App 출력 DTO
│   │   │   └── SetInstructorAppMapper.java      # AppMapper
│   │   └── port/out/
│   │       ├── FindProfilePort.java             # 조회 Port
│   │       └── UpdateProfilePort.java           # 수정 Port
│   └── application/service/
│       └── SetInstructorService.java            # UseCase 구현체
└── common/exception/
    └── ProfileNotFoundException.java            # 도메인 예외
```

**수정 파일**:

```text
Latinhouse.Be/src/main/java/com/latinhouse/api/
├── profile/
│   ├── domain/
│   │   └── Profile.java                        # asInstructor() 메서드 추가
│   ├── adapter/
│   │   ├── in/web/
│   │   │   └── ProfileController.java          # PATCH 엔드포인트 추가
│   │   └── out/persistence/
│   │       └── ProfilePersistenceAdapter.java  # FindProfilePort + UpdateProfilePort 구현
└── common/exception/
    └── GlobalExceptionHandler.java             # ProfileNotFoundException 핸들러 추가

Latinhouse.Be/src/test/java/com/latinhouse/api/
├── profile/
│   ├── adapter/in/web/
│   │   └── ProfileControllerTest.java          # PATCH 테스트 추가
│   └── application/service/
│       └── SetInstructorServiceTest.java       # 신규 서비스 유닛 테스트
```

**Structure Decision**: 기존 `001-create-profile`의 패키지 구조를 그대로 확장. 신규 UseCase(`set-instructor`)는 동일 패턴으로 병렬 추가.
