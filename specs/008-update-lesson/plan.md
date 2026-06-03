# Implementation Plan: 레슨 수정 (PUT /api/lesson/{lessonNo})

**Branch**: `008-update-lesson` | **Date**: 2026-06-03 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `specs/008-update-lesson/spec.md`

## Summary

기존 레슨 데이터를 전체 교체(replace) 방식으로 수정하는 `PUT /api/lesson/{lessonNo}` 엔드포인트를 구현한다.
Hexagonal Architecture 패턴을 따르며, `CreateLesson` 구현체를 템플릿으로 재사용한다.
신규 Port 없이 기존 `LoadLessonPort`, `SaveLessonPort`, `LoadInstructorPort`를 활용하고,
`LessonEntity`의 `orphanRemoval = true` 설정으로 컬렉션 전체 교체를 처리한다.

## Technical Context

**Language/Version**: Java 21 (Spring Boot 4.x)

**Primary Dependencies**: Spring Web MVC, Spring Data JPA, Bean Validation (Jakarta), Lombok

**Storage**: MySQL (JPA `save()` = upsert by id)

**Testing**: JUnit 5, Mockito, `@WebMvcTest` (Spring Boot Test slice)

**Target Platform**: Linux 서버 (REST API)

**Project Type**: Web Service (Hexagonal Architecture)

**Performance Goals**: 기존 엔드포인트와 동일 수준

**Constraints**: `docs/api-spec.md` 계약 준수 필수 (Constitution Principle II)

**Scale/Scope**: 단일 레슨 단건 수정

## Constitution Check

| Gate | 상태 | 근거 |
|------|------|------|
| I. Hexagonal Architecture | ✅ | Web Adapter → Application Port → Domain 단방향 의존 유지 |
| II. Contract-First API | ✅ | `docs/api-spec.md` PUT 섹션이 사전 정의됨 |
| III. Unified Error Response | ✅ | `LessonValidationException`, `LessonNotFoundException` 기존 핸들러 재사용 |
| IV. Validated Inputs | ✅ | Bean Validation은 WebRequest에만, AppRequest는 어노테이션 없음 |
| Two-DTO 패턴 | ✅ | WebRequest/Response ↔ AppRequest/Response 분리 |
| Mapper 패턴 | ✅ | WebMapper(Adapter), AppMapper(Application) 분리, 정적 메서드만 |

## Project Structure

### Documentation (this feature)

```text
specs/008-update-lesson/
├── plan.md              ← 이 파일
├── research.md          ← Phase 0 완료
├── data-model.md        ← Phase 1 완료
├── quickstart.md        ← Phase 1 완료
├── contracts/
│   └── put-lesson.md   ← Phase 1 완료
├── checklists/
│   └── requirements.md
└── tasks.md             ← /speckit-tasks 명령으로 생성
```

### Source Code

```text
Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/
├── adapter/in/web/
│   ├── LessonController.java                   ← PUT 메서드 추가
│   ├── UpdateLessonWebRequest.java             ← 신규
│   ├── UpdateLessonWebResponse.java            ← 신규
│   └── UpdateLessonWebMapper.java              ← 신규
├── application/
│   ├── port/in/
│   │   ├── UpdateLessonUseCase.java            ← 신규
│   │   ├── UpdateLessonAppRequest.java         ← 신규
│   │   ├── UpdateLessonAppResponse.java        ← 신규
│   │   └── UpdateLessonAppMapper.java          ← 신규
│   └── service/
│       └── UpdateLessonService.java            ← 신규
└── (domain/, adapter/out/persistence/ 변경 없음)

Latinhouse.Be/src/test/java/com/latinhouse/api/lesson/
├── adapter/in/web/
│   └── UpdateLessonControllerTest.java         ← 신규
└── application/service/
    └── UpdateLessonServiceTest.java            ← 신규
```

## Implementation Steps

### Step 1: UpdateLessonWebRequest

`CreateLessonWebRequest`와 동일한 필드·어노테이션. Nested static class 동일 구조로 복사.

### Step 2: UpdateLessonWebResponse

```java
@Builder
public class UpdateLessonWebResponse {
    private final Long id;
}
```

### Step 3: UpdateLessonAppRequest

`lessonNo` 필드 추가. 나머지는 `CreateLessonAppRequest`와 동일.
Nested DTO 타입은 `CreateLessonAppRequest.OptionAppReq` 등을 직접 임포트·재사용.

### Step 4: UpdateLessonAppResponse

```java
@Builder
public class UpdateLessonAppResponse {
    private final Long id;
}
```

### Step 5: UpdateLessonWebMapper

```java
// adapter/in/web/UpdateLessonWebMapper.java
public static UpdateLessonAppRequest toAppRequest(Long lessonNo, UpdateLessonWebRequest webReq) { ... }
public static UpdateLessonWebResponse toWebResponse(UpdateLessonAppResponse appRes) { ... }
```

날짜/시간 String → `LocalDateTime` 변환 로직은 `CreateLessonWebMapper`와 동일.

### Step 6: UpdateLessonUseCase

```java
public interface UpdateLessonUseCase {
    UpdateLessonAppResponse updateLesson(UpdateLessonAppRequest request);
}
```

### Step 7: UpdateLessonAppMapper

```java
// application/port/in/UpdateLessonAppMapper.java
public static Lesson toDomain(UpdateLessonAppRequest req) {
    // id = req.getLessonNo() 주입 필수 — JPA UPDATE 조건
    return Lesson.builder().id(req.getLessonNo())...build();
}
public static UpdateLessonAppResponse toAppResponse(Lesson lesson) { ... }
```

### Step 8: UpdateLessonService

```java
@Service
@RequiredArgsConstructor
public class UpdateLessonService implements UpdateLessonUseCase {
    private final LoadLessonPort loadLessonPort;       // 존재 확인 (404 guard)
    private final SaveLessonPort saveLessonPort;       // UPDATE
    private final LoadInstructorPort loadInstructorPort;

    @Override
    @Transactional
    public UpdateLessonAppResponse updateLesson(UpdateLessonAppRequest request) {
        loadLessonPort.loadLesson(request.getLessonNo()); // 없으면 LessonNotFoundException
        List<ErrorResponse.FieldError> errors = new ArrayList<>();
        validateInstructors(request, errors);
        validateOptionDateTimes(request, errors);
        validateDiscountConditions(request, errors);
        if (!errors.isEmpty()) throw new LessonValidationException(errors);
        Lesson lesson = UpdateLessonAppMapper.toDomain(request);
        Lesson saved = saveLessonPort.save(lesson);
        return UpdateLessonAppMapper.toAppResponse(saved);
    }
    // validateInstructors, validateOptionDateTimes, validateDiscountConditions
    // → CreateLessonService와 동일한 구현 (CreateLessonAppRequest → UpdateLessonAppRequest 타입 변경)
}
```

### Step 9: LessonController 추가

```java
@PutMapping("/lesson/{lessonNo}")
public ResponseEntity<UpdateLessonWebResponse> updateLesson(
        @PathVariable Long lessonNo,
        @RequestBody @Valid UpdateLessonWebRequest webRequest) {
    UpdateLessonAppRequest appRequest = UpdateLessonWebMapper.toAppRequest(lessonNo, webRequest);
    UpdateLessonAppResponse appResponse = updateLessonUseCase.updateLesson(appRequest);
    return ResponseEntity.ok(UpdateLessonWebMapper.toWebResponse(appResponse));
}
```

### Step 10: 테스트 작성

**UpdateLessonControllerTest** (`@WebMvcTest`):
- 200 OK — 정상 수정
- 400 — title 누락
- 400 — options 빈 배열
- 404 — 존재하지 않는 lessonNo

**UpdateLessonServiceTest** (단위):
- 정상 수정 — loadLesson → save 호출 순서 검증
- instructorLo/La 모두 null → LessonValidationException
- 존재하지 않는 lessonNo → LessonNotFoundException
- 강사 성별 불일치 → LessonValidationException
- endDateTime < startDateTime → LessonValidationException

## Complexity Tracking

해당 없음. 모든 Constitution Check 통과.
