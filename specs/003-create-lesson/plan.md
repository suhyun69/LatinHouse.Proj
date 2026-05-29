# Implementation Plan: 레슨 생성 (POST /api/lesson)

**Branch**: `003-create-lesson` | **Date**: 2026-05-29 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `specs/003-create-lesson/spec.md`

## Summary

`POST /api/lesson` 엔드포인트를 구현한다. 레슨 기본 정보(제목, 장르, 강사)와 수업 옵션, 할인, 계좌, 연락처, 공지 등을 입력받아 저장하고 201 Created + 생성된 레슨 id를 반환한다. Constitution의 Hexagonal Architecture를 따라 Two-DTO 패턴(WebRequest → AppRequest) + 검증 레이어 분리(Bean Validation / Service 비즈니스 검증)로 구현한다. 강사 조회는 `LoadInstructorPort` 신규 정의 + `InstructorPersistenceAdapter`(ProfileJpaRepository 재사용)로 처리한다.

## Technical Context

**Language/Version**: Java 25

**Primary Dependencies**: Spring Boot 4.0.6, Spring Data JPA, Spring Validation, Lombok, H2, springdoc-openapi 3.0.0

**Storage**: H2 (개발/테스트용 인메모리 DB)

**Testing**: JUnit 5, `@WebMvcTest`, `@MockitoBean`, MockMvc

**Target Platform**: JVM 서버 (Spring Boot embedded Tomcat)

**Project Type**: REST API (Web Service)

**Performance Goals**: 표준 웹 API 응답 수준 (단일 트랜잭션 — 레슨 + 서브 엔티티 일괄 저장)

**Constraints**:
- Domain 레이어는 JPA, Spring, HTTP 등 외부 의존 금지 (Constitution I)
- Lesson Application이 Profile Application을 직접 의존 금지 → LoadInstructorPort 패턴 사용
- 계층 간 변환은 반드시 Mapper 클래스 사용 (Constitution I — Mapper 패턴)
- Bean Validation(@NotBlank, @Pattern 등)은 WebRequest에만, AppRequest에는 금지 (Constitution IV)
- 모든 엔드포인트에 `@Tag`, `@Operation` Swagger 문서화 필수 (Constitution Quality Gate)
- 에러 응답은 반드시 `{ "status": ..., "errors": [...] }` 형식 (Constitution III)

**Scale/Scope**: 신규 도메인(lesson) 추가. 기존 profile 도메인 코드 재사용(ProfileJpaRepository, ProfilePersistenceMapper)

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| 원칙 | 준수 여부 | 비고 |
|------|-----------|------|
| I. Hexagonal Architecture | ✅ | Adapter → Application → Domain 단방향. Lesson App이 Profile App을 직접 참조하지 않음 |
| Two-DTO 패턴 | ✅ | `CreateLessonWebRequest` / `CreateLessonAppRequest` / `CreateLessonAppResponse` / `CreateLessonWebResponse` 분리 |
| Mapper 패턴 | ✅ | `CreateLessonWebMapper`, `CreateLessonAppMapper`, `LessonPersistenceMapper` 별도 클래스 |
| II. Contract-First API | ✅ | `docs/api-spec.md` 기준 구현 |
| III. Unified Error Response | ✅ | Bean Validation 오류 + `LessonValidationException` 모두 동일 에러 형식으로 반환 |
| IV. Validated Inputs | ✅ | Bean Validation은 WebRequest에만. 비즈니스 검증은 Service에서 `LessonValidationException` 사용 |
| Quality Gate: Swagger | ✅ | `LessonController`에 `@Tag`, `@Operation` 추가 |

**Gate Decision**: 모든 원칙 충족. 구현 진행.

## Project Structure

### Documentation (this feature)

```text
specs/003-create-lesson/
├── plan.md              # This file
├── spec.md              # Feature specification
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/
│   └── create-lesson.md # Phase 1 output
├── checklists/
│   └── requirements.md  # Quality checklist
└── tasks.md             # Phase 2 output (/speckit-tasks)
```

### Source Code

**신규 파일**:

```text
Latinhouse.Be/src/main/java/com/latinhouse/api/
├── lesson/
│   ├── domain/
│   │   ├── Lesson.java
│   │   ├── LessonOption.java
│   │   ├── LessonDiscount.java
│   │   ├── LessonAccount.java
│   │   ├── LessonContact.java
│   │   ├── LessonNotice.java
│   │   ├── Genre.java
│   │   ├── Region.java
│   │   ├── DiscountType.java
│   │   ├── ContactType.java
│   │   └── NoticeType.java
│   ├── application/
│   │   ├── port/in/
│   │   │   ├── CreateLessonUseCase.java
│   │   │   ├── CreateLessonAppRequest.java   # 정적 내부 클래스 포함
│   │   │   ├── CreateLessonAppResponse.java
│   │   │   └── CreateLessonAppMapper.java
│   │   ├── port/out/
│   │   │   ├── SaveLessonPort.java
│   │   │   └── LoadInstructorPort.java
│   │   └── service/
│   │       └── CreateLessonService.java
│   ├── adapter/
│   │   ├── in/web/
│   │   │   ├── CreateLessonWebRequest.java   # Bean Validation + 정적 내부 클래스
│   │   │   ├── CreateLessonWebResponse.java
│   │   │   ├── CreateLessonWebMapper.java
│   │   │   └── LessonController.java
│   │   └── out/persistence/
│   │       ├── LessonEntity.java
│   │       ├── LessonOptionEntity.java
│   │       ├── LessonDiscountEntity.java
│   │       ├── LessonAccountEntity.java
│   │       ├── LessonContactEntity.java
│   │       ├── LessonNoticeEntity.java
│   │       ├── LessonJpaRepository.java
│   │       ├── LessonPersistenceAdapter.java
│   │       ├── LessonPersistenceMapper.java
│   │       └── InstructorPersistenceAdapter.java  # ProfileJpaRepository 사용
└── common/
    └── exception/
        └── LessonValidationException.java

Latinhouse.Be/src/test/java/com/latinhouse/api/
└── lesson/
    ├── adapter/in/web/
    │   └── LessonControllerTest.java
    └── application/service/
        └── CreateLessonServiceTest.java
```

**수정 파일**:

```text
Latinhouse.Be/src/main/java/com/latinhouse/api/
└── common/
    └── exception/
        └── GlobalExceptionHandler.java   # LessonValidationException 핸들러 추가
```

**Structure Decision**: `lesson` 패키지를 `profile` 패키지와 동일한 구조로 신규 추가. `InstructorPersistenceAdapter`는 lesson.adapter.out.persistence에 위치하며 profile.adapter.out.persistence의 `ProfileJpaRepository`, `ProfilePersistenceMapper`를 참조한다.
