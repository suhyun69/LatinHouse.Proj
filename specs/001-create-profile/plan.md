# Implementation Plan: Create Profile (POST /api/profile)

**Branch**: `001-create-profile` | **Date**: 2026-05-28 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `specs/001-create-profile/spec.md`

## Summary

`POST /api/profile` 엔드포인트를 구현한다. 닉네임과 성별을 입력받아 프로필을 생성하고, 생성된 8자리 고유 ID를 201 Created 응답으로 반환한다. Constitution의 Hexagonal Architecture를 따라 Web Adapter → Application → Domain → Persistence Adapter 흐름으로 구성하며, Bean Validation으로 입력 검증, `@RestControllerAdvice`로 표준 에러 응답을 제공한다.

## Technical Context

**Language/Version**: Java 25

**Primary Dependencies**: Spring Boot 4.0.6, Spring Data JPA, Spring Security, Spring Validation, Lombok, H2, springdoc-openapi (추가 필요 — Constitution Quality Gate 요구사항)

**Storage**: H2 (개발/테스트용 인메모리 DB)

**Testing**: JUnit 5, Spring Boot Test (`@WebMvcTest`, `@DataJpaTest`, `@SpringBootTest`)

**Target Platform**: JVM 서버 (로컬 개발 환경 기준, Spring Boot embedded Tomcat)

**Project Type**: REST API (Web Service)

**Performance Goals**: 표준 웹 API 응답 수준 (단일 DB 저장 CRUD)

**Constraints**:
- Domain 레이어는 JPA, Spring, HTTP 등 외부 의존 금지 (Constitution I)
- Bean Validation 어노테이션은 `WebRequest`에만 적용 (Constitution IV)
- 모든 엔드포인트에 `@Tag`, `@Operation` Swagger 문서화 필수 (Constitution Quality Gate)

**Scale/Scope**: 단일 엔드포인트, 초기 개발 단계

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| 원칙 | 준수 여부 | 비고 |
|------|-----------|------|
| I. Hexagonal Architecture | ✅ | `profile` 패키지를 adapter/application/domain 3계층으로 구성 |
| Two-DTO 패턴 | ✅ | `WebRequest` / `AppRequest` / `AppResponse` / `WebResponse` 분리 |
| Mapper 패턴 | ✅ | `WebMapper`, `AppMapper`, `PersistenceMapper` 별도 클래스 |
| II. Contract-First API | ✅ | `docs/api-spec.md` 기준으로 구현 (requirements.md 충돌 → api-spec.md 우선) |
| III. Unified Error Response | ✅ | `@RestControllerAdvice` + `MethodArgumentNotValidException` 핸들러 |
| IV. Validated Inputs | ✅ | `@NotBlank`, `@Pattern`을 `CreateProfileWebRequest`에만 적용 |
| Quality Gate: Swagger | ⚠️ | `springdoc-openapi` 의존성 미포함 → `build.gradle` 추가 필요 |

**Gate Decision**: springdoc-openapi 의존성 추가를 구현 Task에 포함. 나머지 원칙은 Phase 1 설계로 충족.

## Project Structure

### Documentation (this feature)

```text
specs/001-create-profile/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/
│   └── post-profile.md  # Phase 1 output
└── tasks.md             # Phase 2 output (/speckit-tasks)
```

### Source Code

```text
Latinhouse.Be/src/main/java/com/latinhouse/api/
├── profile/
│   ├── adapter/
│   │   ├── in/web/
│   │   │   ├── ProfileController.java
│   │   │   ├── CreateProfileWebRequest.java
│   │   │   ├── CreateProfileWebResponse.java
│   │   │   └── CreateProfileWebMapper.java
│   │   └── out/persistence/
│   │       ├── ProfileEntity.java
│   │       ├── ProfileJpaRepository.java
│   │       ├── ProfilePersistenceAdapter.java
│   │       └── ProfilePersistenceMapper.java
│   ├── application/
│   │   ├── port/in/
│   │   │   ├── CreateProfileUseCase.java
│   │   │   ├── CreateProfileAppRequest.java
│   │   │   ├── CreateProfileAppResponse.java
│   │   │   └── CreateProfileAppMapper.java
│   │   ├── port/out/
│   │   │   └── SaveProfilePort.java
│   │   └── service/
│   │       └── CreateProfileService.java
│   └── domain/
│       ├── Profile.java
│       └── Sex.java
└── common/
    ├── exception/
    │   ├── GlobalExceptionHandler.java
    │   └── ErrorResponse.java
    ├── config/
    │   └── SecurityConfig.java
    └── util/
        └── ProfileIdGenerator.java

Latinhouse.Be/src/test/java/com/latinhouse/api/
└── profile/
    ├── adapter/in/web/
    │   └── ProfileControllerTest.java        (@WebMvcTest)
    └── application/service/
        └── CreateProfileServiceTest.java     (Unit test)
```

**Structure Decision**: Option 1 (단일 백엔드 프로젝트). Constitution의 Hexagonal Architecture에 따라 `profile` 도메인 패키지 하위에 adapter/application/domain 계층 구조를 구성한다. 에러 핸들러, 보안 설정, ID 생성 유틸리티 등 공통 관심사는 `common` 패키지로 분리한다.
