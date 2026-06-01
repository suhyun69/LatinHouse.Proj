# Implementation Plan: 프로필 목록 조회 (GET /api/profiles)

**Branch**: `004-get-profiles` | **Date**: 2026-06-01 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `specs/004-get-profiles/spec.md`

## Summary

`GET /api/profiles` 엔드포인트를 구현한다. 선택적 쿼리 파라미터 `isInstructor`(Boolean)로 전체/강사/비강사 프로필 목록을 반환한다. Constitution의 Hexagonal Architecture를 따라 기존 `FindProfilePort`에 `findAll(Boolean)` 메서드를 추가하고, `GetProfilesUseCase` + `GetProfilesService`로 구현한다. 현재 `docs/api-spec.md`에 해당 명세가 없으므로 이 피처에서 함께 추가한다.

## Technical Context

**Language/Version**: Java 25

**Primary Dependencies**: Spring Boot 4.0.6, Spring Data JPA, Lombok, springdoc-openapi 3.0.0

**Storage**: H2 (개발/테스트용 인메모리 DB)

**Testing**: JUnit 5, `@WebMvcTest`, `@MockitoBean`, MockMvc

**Target Platform**: JVM 서버 (Spring Boot embedded Tomcat)

**Project Type**: REST API (Web Service)

**Performance Goals**: 표준 웹 API 응답 수준 (단순 SELECT 쿼리)

**Constraints**:
- Domain 레이어는 JPA, Spring, HTTP 등 외부 의존 금지 (Constitution I)
- 계층 간 변환은 반드시 Mapper 클래스 사용 (Constitution I — Mapper 패턴)
- 모든 엔드포인트에 `@Tag`, `@Operation` Swagger 문서화 필수 (Constitution Quality Gate)
- 에러 응답은 반드시 `{ "status": ..., "errors": [...] }` 형식 (Constitution III)

**Scale/Scope**: 단일 엔드포인트 추가, 기존 Profile 도메인 확장

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| 원칙 | 준수 여부 | 비고 |
|------|-----------|------|
| I. Hexagonal Architecture | ✅ | Adapter → Application → Domain 단방향 의존 유지 |
| Two-DTO 패턴 | ✅ | `GetProfilesAppResponse`, `GetProfilesWebResponse` 분리. WebRequest/AppRequest 불필요 (파라미터 단순). |
| Mapper 패턴 | ✅ | `GetProfilesWebMapper`, `GetProfilesAppMapper` 별도 클래스 |
| II. Contract-First API | ✅ | `docs/api-spec.md`에 GET /api/profiles 명세 추가 후 구현 |
| III. Unified Error Response | ✅ | `MethodArgumentTypeMismatchException` → `GlobalExceptionHandler` → `ErrorResponse` |
| IV. Validated Inputs | ✅ | Spring이 Boolean 변환 실패 시 자동 400. `GlobalExceptionHandler`에서 처리. |
| Quality Gate: Swagger | ✅ | `@Operation` 추가 필요 |

**Gate Decision**: 모든 원칙 충족. 구현 진행.

## Project Structure

### Documentation (this feature)

```text
specs/004-get-profiles/
├── plan.md              # This file
├── spec.md              # Feature specification
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/
│   └── get-profiles.md  # Phase 1 output
└── tasks.md             # Phase 2 output (/speckit-tasks)
```

### Source Code

**신규 파일**:

```text
Latinhouse.Be/src/main/java/com/latinhouse/api/
└── profile/
    ├── adapter/in/web/
    │   ├── GetProfilesWebResponse.java        # HTTP 응답 DTO (개별 항목)
    │   └── GetProfilesWebMapper.java          # AppResponse → WebResponse 변환
    ├── application/
    │   ├── port/in/
    │   │   ├── GetProfilesUseCase.java        # UseCase 인터페이스
    │   │   ├── GetProfilesAppResponse.java    # Service 결과 DTO (개별 항목)
    │   │   └── GetProfilesAppMapper.java      # Domain → AppResponse 변환
    │   └── service/
    │       └── GetProfilesService.java        # UseCase 구현체
```

**수정 파일**:

```text
Latinhouse.Be/src/main/java/com/latinhouse/api/
└── profile/
    ├── adapter/in/web/
    │   └── ProfileController.java             # GET /api/profiles 엔드포인트 추가
    ├── adapter/out/persistence/
    │   ├── ProfileJpaRepository.java          # findAllByIsInstructor() 추가
    │   └── ProfilePersistenceAdapter.java     # FindProfilePort.findAll() 구현 추가
    └── application/port/out/
        └── FindProfilePort.java               # findAll(Boolean) 메서드 추가

Latinhouse.Be/src/main/java/com/latinhouse/api/common/
└── exception/
    └── GlobalExceptionHandler.java            # MethodArgumentTypeMismatchException 핸들러 추가

docs/
└── api-spec.md                                # GET /api/profiles 명세 추가
```

**테스트 파일**:

```text
Latinhouse.Be/src/test/java/com/latinhouse/api/
└── profile/
    ├── adapter/in/web/
    │   └── ProfileControllerTest.java         # GET /api/profiles 테스트 추가
    └── application/service/
        └── GetProfilesServiceTest.java        # 신규 서비스 단위 테스트
```

## Implementation Notes

### Boolean 타입 미스매치 에러 처리

Spring MVC에서 `?isInstructor=yes` 같은 잘못된 Boolean 값이 들어오면 `MethodArgumentTypeMismatchException`이 발생한다. `GlobalExceptionHandler`에 이 예외 핸들러를 추가하여 Constitution III 통합 에러 형식으로 반환한다.

```json
{
  "status": 400,
  "errors": [
    {
      "field": "isInstructor",
      "message": "isInstructor는 true 또는 false만 입력 가능합니다."
    }
  ]
}
```

### FindProfilePort.findAll() 분기 로직

```java
// ProfilePersistenceAdapter.java
public List<Profile> findAll(Boolean isInstructor) {
    List<ProfileEntity> entities = (isInstructor == null)
        ? profileJpaRepository.findAll()
        : profileJpaRepository.findAllByIsInstructor(isInstructor);
    return entities.stream()
        .map(profilePersistenceMapper::toDomain)
        .toList();
}
```
