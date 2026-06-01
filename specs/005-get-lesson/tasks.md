# Tasks: 레슨 단건 조회 (GET /api/lessons/{lessonNo})

**Input**: Design documents from `specs/005-get-lesson/`

**Prerequisites**: plan.md ✅, spec.md ✅, research.md ✅, data-model.md ✅, contracts/ ✅, quickstart.md ✅

**Organization**: 기존 lesson 도메인에 조회 유스케이스를 추가하는 작업. Setup/Foundational 없음(이미 프로젝트 구조 존재). US1(정상 조회) → US2(404 처리) 순으로 진행.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: 다른 파일, 의존성 없음 — 병렬 실행 가능
- **[Story]**: 해당 유저스토리 레이블 (US1, US2)
- 모든 경로는 프로젝트 루트 기준

---

## Phase 1: Foundational (공통 예외 + 포트 정의)

**Purpose**: US1, US2 모두 필요한 예외 클래스와 Port 인터페이스 먼저 생성

**⚠️ CRITICAL**: 이 단계 완료 후 US1/US2 병렬 진행 가능

- [X] T001 `LessonNotFoundException` 생성 — `Latinhouse.Be/src/main/java/com/latinhouse/api/common/exception/LessonNotFoundException.java`
- [X] T002 `LoadLessonPort` 인터페이스 생성 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/application/port/out/LoadLessonPort.java`

**Checkpoint**: 예외 클래스와 Port 인터페이스 준비 완료 → US1/US2 구현 시작 가능

---

## Phase 2: User Story 1 - 레슨 단건 조회 (Priority: P1) 🎯 MVP

**Goal**: 존재하는 lessonNo로 호출 시 200 OK + 레슨 전체 정보 반환

**Independent Test**: `GET /api/lessons/{id}` 호출 시 200 OK와 options/discounts/account/contacts/notices 포함 응답 반환 확인

### Implementation for User Story 1

- [X] T003 [P] [US1] `GetLessonAppResponse` (정적 내부 클래스 OptionResponse, DiscountResponse, AccountResponse, ContactResponse, NoticeResponse 포함) 생성 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/application/port/in/GetLessonAppResponse.java`
- [X] T004 [P] [US1] `GetLessonUseCase` 인터페이스 생성 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/application/port/in/GetLessonUseCase.java`
- [X] T005 [US1] `GetLessonAppMapper` 생성 (Lesson 도메인 → GetLessonAppResponse 변환) — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/application/port/in/GetLessonAppMapper.java`
- [X] T006 [US1] `GetLessonService` 구현 (T002 LoadLessonPort, T004 UseCase, T005 Mapper 의존) — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/application/service/GetLessonService.java`
- [X] T007 [P] [US1] `GetLessonWebResponse` (정적 내부 클래스 포함) 생성 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/adapter/in/web/GetLessonWebResponse.java`
- [X] T008 [US1] `GetLessonWebMapper` 생성 (GetLessonAppResponse → GetLessonWebResponse, LocalDateTime → startDate/startTime/endDate/endTime 분리 포함) — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/adapter/in/web/GetLessonWebMapper.java`
- [X] T009 [US1] `LessonPersistenceAdapter`에 `LoadLessonPort` 구현 추가 (`lessonJpaRepository.findById()` 사용, 없으면 LessonNotFoundException throw) — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/adapter/out/persistence/LessonPersistenceAdapter.java`
- [X] T010 [US1] `LessonController`에 `GET /api/lessons/{lessonNo}` 엔드포인트 추가, 클래스 `@RequestMapping`을 `/api`로 변경, 기존 POST에 `@PostMapping("/lesson")` 적용, `@Operation` Swagger 추가 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/adapter/in/web/LessonController.java`

### Tests for User Story 1

- [X] T011 [US1] `GetLessonServiceTest` 작성 (`getLesson_found_returnsAppResponse`) — `Latinhouse.Be/src/test/java/com/latinhouse/api/lesson/application/service/GetLessonServiceTest.java`
- [X] T012 [US1] `LessonControllerTest`에 `getLesson_existingId_returns200WithFullBody` 테스트 추가 — `Latinhouse.Be/src/test/java/com/latinhouse/api/lesson/adapter/in/web/LessonControllerTest.java`

**Checkpoint**: `GET /api/lessons/{id}` 호출 시 200 OK + 전체 응답 반환 확인 완료

---

## Phase 3: User Story 2 - 존재하지 않는 레슨 조회 (Priority: P2)

**Goal**: 존재하지 않는 lessonNo 조회 시 404 + LESSON_NOT_FOUND 에러 반환

**Independent Test**: `GET /api/lessons/9999` 호출 시 404 + `{ "status": 404, "errors": [{ "field": "lessonNo", ... }] }` 반환 확인

### Implementation for User Story 2

- [X] T013 [US2] `GlobalExceptionHandler`에 `LessonNotFoundException` 핸들러 추가 (T001 의존) — `Latinhouse.Be/src/main/java/com/latinhouse/api/common/exception/GlobalExceptionHandler.java`

### Tests for User Story 2

- [X] T014 [US2] `GetLessonServiceTest`에 `getLesson_notFound_throwsLessonNotFoundException` 테스트 추가 — `Latinhouse.Be/src/test/java/com/latinhouse/api/lesson/application/service/GetLessonServiceTest.java`
- [X] T015 [US2] `LessonControllerTest`에 `getLesson_notExistingId_returns404` 테스트 추가 — `Latinhouse.Be/src/test/java/com/latinhouse/api/lesson/adapter/in/web/LessonControllerTest.java`

**Checkpoint**: `GET /api/lessons/9999` 호출 시 404 + 에러 응답 확인 완료

---

## Phase 4: Polish & Cross-Cutting Concerns

- [X] T016 quickstart.md 검증 체크리스트 전 항목 수동 확인 (Swagger UI 포함) — `specs/005-get-lesson/quickstart.md`
- [X] T017 전체 테스트 실행하여 기존 테스트 회귀 없음 확인

---

## Dependencies & Execution Order

### Phase Dependencies

- **Foundational (Phase 1)**: 즉시 시작. US1/US2 모두 선행 필요
- **US1 (Phase 2)**: Phase 1 완료 후 시작. T003~T004는 병렬 가능
- **US2 (Phase 3)**: Phase 1 완료 후 시작 (US1과 병렬 가능). T013은 T001 의존
- **Polish (Phase 4)**: 모든 구현 완료 후

### Within User Story 1

```
T001, T002 완료
    → T003 [P], T004 [P], T007 [P] 동시 시작
    → T003, T004 완료 후 T005
    → T005 완료 후 T006
    → T007 완료 후 T008
    → T006, T008, T002 완료 후 T009
    → T009 완료 후 T010
    → T010 완료 후 T011, T012
```

### Parallel Opportunities

```bash
# Phase 1 — 병렬 실행:
Task T001: LessonNotFoundException 생성
Task T002: LoadLessonPort 생성

# Phase 2 — 병렬 실행 (T001, T002 완료 후):
Task T003: GetLessonAppResponse 생성
Task T004: GetLessonUseCase 생성
Task T007: GetLessonWebResponse 생성
```

---

## Implementation Strategy

### MVP (US1만)

1. Phase 1 완료 (T001, T002)
2. Phase 2 완료 (T003~T012)
3. **검증**: `GET /api/lessons/{id}` 200 OK 확인

### 전체 완료

1. MVP 완료
2. Phase 3 완료 (T013~T015)
3. Phase 4 완료 (T016~T017)

---

## Notes

- 기존 `LessonPersistenceMapper.toDomain()` 재사용 — 변경 없음
- 기존 `LessonJpaRepository.findById()` 재사용 — 변경 없음
- `LessonController` `@RequestMapping` 변경 시 기존 `POST /api/lesson` 경로 유지 필수 확인
- [P] tasks = 다른 파일, 의존성 없음 → 병렬 실행 가능
