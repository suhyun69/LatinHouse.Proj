# Tasks: 레슨 수정 (PUT /api/lesson/{lessonNo})

**Input**: Design documents from `specs/008-update-lesson/`

**Prerequisites**: plan.md ✅ spec.md ✅ research.md ✅ data-model.md ✅ contracts/ ✅

**Organization**: User Story 단위로 그룹화. 각 Story는 독립적으로 구현·테스트 가능.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: 동시 실행 가능 (다른 파일, 의존성 없음)
- **[Story]**: 해당 태스크가 속하는 User Story

---

## Phase 1: Setup (공유 인프라)

**Purpose**: 신규 파일 골격 생성. 이 Phase는 기존 프로젝트 구조에 추가만 하므로 빠르게 완료된다.

- [X] T001 `UpdateLessonWebRequest.java` 파일 생성 (빈 클래스 골격) — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/adapter/in/web/UpdateLessonWebRequest.java`
- [X] T002 [P] `UpdateLessonWebResponse.java` 파일 생성 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/adapter/in/web/UpdateLessonWebResponse.java`
- [X] T003 [P] `UpdateLessonWebMapper.java` 파일 생성 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/adapter/in/web/UpdateLessonWebMapper.java`
- [X] T004 [P] `UpdateLessonAppRequest.java` 파일 생성 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/application/port/in/UpdateLessonAppRequest.java`
- [X] T005 [P] `UpdateLessonAppResponse.java` 파일 생성 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/application/port/in/UpdateLessonAppResponse.java`
- [X] T006 [P] `UpdateLessonAppMapper.java` 파일 생성 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/application/port/in/UpdateLessonAppMapper.java`
- [X] T007 [P] `UpdateLessonUseCase.java` 파일 생성 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/application/port/in/UpdateLessonUseCase.java`
- [X] T008 [P] `UpdateLessonService.java` 파일 생성 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/application/service/UpdateLessonService.java`

**Checkpoint**: 빈 클래스 골격 생성 완료 — 컴파일 가능 상태여야 함

---

## Phase 2: Foundational (블로킹 선행 작업)

**Purpose**: 모든 User Story 구현 전에 완료해야 하는 핵심 DTO·인터페이스

**⚠️ CRITICAL**: 이 Phase 완료 전 User Story 구현 불가

- [X] T009 `UpdateLessonWebRequest` 필드·Bean Validation 구현 — `CreateLessonWebRequest`와 동일한 필드·어노테이션 구조. Nested static class (OptionWebReq, DiscountWebReq, AccountWebReq, ContactWebReq, NoticeWebReq) 포함 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/adapter/in/web/UpdateLessonWebRequest.java`
- [X] T010 [P] `UpdateLessonWebResponse` 구현 — `@Builder`, `id: Long` 필드 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/adapter/in/web/UpdateLessonWebResponse.java`
- [X] T011 [P] `UpdateLessonAppResponse` 구현 — `@Builder`, `id: Long` 필드 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/application/port/in/UpdateLessonAppResponse.java`
- [X] T012 `UpdateLessonAppRequest` 구현 — `lessonNo: Long` 포함, Nested DTO 타입은 `CreateLessonAppRequest.OptionAppReq` 등 기존 inner class 직접 임포트·재사용 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/application/port/in/UpdateLessonAppRequest.java`
- [X] T013 `UpdateLessonUseCase` 인터페이스 구현 — `UpdateLessonAppResponse updateLesson(UpdateLessonAppRequest request)` — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/application/port/in/UpdateLessonUseCase.java`

**Checkpoint**: DTO·인터페이스 완료 — Mapper·Service 구현 가능

---

## Phase 3: User Story 1 — 레슨 전체 정보 수정 (Priority: P1) 🎯 MVP

**Goal**: 유효한 요청으로 레슨 데이터를 수정하고 200 OK를 반환한다

**Independent Test**: `PUT /api/lesson/{lessonNo}` 성공 후 `GET /api/lessons/{lessonNo}`로 변경 내용 확인

### Implementation

- [X] T014 [US1] `UpdateLessonWebMapper.toAppRequest()` 구현 — `lessonNo`, `UpdateLessonWebRequest`를 받아 날짜·시간 String → LocalDateTime 변환, enum 변환 포함. `private` 생성자로 인스턴스화 불가 처리 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/adapter/in/web/UpdateLessonWebMapper.java`
- [X] T015 [P] [US1] `UpdateLessonWebMapper.toWebResponse()` 구현 — `UpdateLessonAppResponse` → `UpdateLessonWebResponse` id 매핑 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/adapter/in/web/UpdateLessonWebMapper.java`
- [X] T016 [US1] `UpdateLessonAppMapper.toDomain()` 구현 — `id = req.getLessonNo()` 주입 필수 (JPA UPDATE 조건). options, discounts, account, contacts, notices 모두 매핑 포함. `private` 생성자 처리 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/application/port/in/UpdateLessonAppMapper.java`
- [X] T017 [P] [US1] `UpdateLessonAppMapper.toAppResponse()` 구현 — `Lesson` → `UpdateLessonAppResponse` id 매핑 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/application/port/in/UpdateLessonAppMapper.java`
- [X] T018 [US1] `UpdateLessonService` 구현 — `@Service @RequiredArgsConstructor`. `updateLesson()` 메서드: ① `loadLessonPort.loadLesson(lessonNo)` 호출 (없으면 LessonNotFoundException 자동 발생), ② 검증 통과 시 `UpdateLessonAppMapper.toDomain(request)`, ③ `saveLessonPort.save(lesson)`, ④ `UpdateLessonAppMapper.toAppResponse(saved)` 반환. `@Transactional` 적용 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/application/service/UpdateLessonService.java`
- [X] T019 [US1] `LessonController`에 PUT 엔드포인트 추가 — `@PutMapping("/lesson/{lessonNo}")`, `@PathVariable Long lessonNo`, `@RequestBody @Valid UpdateLessonWebRequest`. `ResponseEntity.ok(...)` 반환. `updateLessonUseCase` 의존성 주입 추가 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/adapter/in/web/LessonController.java`

### Test

- [X] T020 [P] [US1] `UpdateLessonControllerTest` 작성 — `@WebMvcTest(LessonController.class)`. 테스트 케이스: (1) 정상 수정 → 200 OK + `{"id": 1}`, (2) title 누락 → 400, (3) options 빈 배열 → 400 — `Latinhouse.Be/src/test/java/com/latinhouse/api/lesson/adapter/in/web/UpdateLessonControllerTest.java`
- [X] T021 [P] [US1] `UpdateLessonServiceTest` 작성 — 단위 테스트(Mockito). 테스트 케이스: (1) 정상 수정 — loadLesson → save 호출 순서 검증, (2) 존재하지 않는 lessonNo → LessonNotFoundException — `Latinhouse.Be/src/test/java/com/latinhouse/api/lesson/application/service/UpdateLessonServiceTest.java`

**Checkpoint**: MVP 완료 — PUT 성공 시나리오 동작 확인

---

## Phase 4: User Story 2 — 존재하지 않는 레슨 수정 (Priority: P2)

**Goal**: 존재하지 않는 lessonNo 요청 시 404 응답

**Independent Test**: 존재하지 않는 lessonNo로 PUT 요청 → 404 + `LESSON_NOT_FOUND` 확인

### Implementation

- [X] T022 [US2] `UpdateLessonService`에서 `loadLessonPort.loadLesson()` 호출이 T018에서 이미 구현되었는지 검증 (LessonNotFoundException이 GlobalExceptionHandler를 통해 404로 변환되는지 확인). 별도 코드 추가 없이 기존 `LessonNotFoundException` → `GlobalExceptionHandler` 흐름 재사용 — `Latinhouse.Be/src/main/java/com/latinhouse/api/common/exception/GlobalExceptionHandler.java`

### Test

- [X] T023 [US2] `UpdateLessonControllerTest`에 404 테스트 케이스 추가 — `updateLessonUseCase.updateLesson()` mock이 `LessonNotFoundException` throw → 404 응답 검증 — `Latinhouse.Be/src/test/java/com/latinhouse/api/lesson/adapter/in/web/UpdateLessonControllerTest.java`
- [X] T024 [P] [US2] `UpdateLessonServiceTest`에 lessonNo 없음 케이스 확인 (T021에 포함) — `Latinhouse.Be/src/test/java/com/latinhouse/api/lesson/application/service/UpdateLessonServiceTest.java`

**Checkpoint**: 404 시나리오 완료

---

## Phase 5: User Story 3 — 유효성 검사 실패 (Priority: P3)

**Goal**: 잘못된 데이터 요청 시 400 응답과 필드별 에러 메시지 반환

**Independent Test**: 필수 필드 누락·형식 오류 요청 → 400 + 필드별 에러 메시지 확인

### Implementation

- [X] T025 [US3] `UpdateLessonService`에 검증 메서드 구현 — `validateInstructors()`, `validateOptionDateTimes()`, `validateDiscountConditions()`. `CreateLessonService`의 동일 메서드와 동일한 구현, 파라미터 타입만 `UpdateLessonAppRequest`로 변경. T018 구현 시 포함되어 있어야 함 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/application/service/UpdateLessonService.java`

### Test

- [X] T026 [US3] `UpdateLessonServiceTest`에 검증 케이스 추가 — (1) instructorLo/La 모두 null → LessonValidationException, (2) 강사 성별 불일치 → LessonValidationException, (3) endDateTime ≤ startDateTime → LessonValidationException, (4) Earlybird 할인 조건 형식 오류 → LessonValidationException — `Latinhouse.Be/src/test/java/com/latinhouse/api/lesson/application/service/UpdateLessonServiceTest.java`

**Checkpoint**: 모든 User Story 동작 확인

---

## Phase 6: Polish & Cross-Cutting Concerns

- [X] T027 [P] `LessonController` Swagger 문서화 — `@Operation(summary = "레슨 수정")`, `@ApiResponse` 추가 (Constitution Quality Gate: 모든 엔드포인트 Swagger 필수) — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/adapter/in/web/LessonController.java`
- [X] T028 [P] quickstart.md 체크리스트 수동 검증 — `specs/008-update-lesson/quickstart.md`의 모든 항목 확인 후 체크 표시

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 (Setup)**: 즉시 시작 가능
- **Phase 2 (Foundational)**: Phase 1 완료 후 — 모든 User Story 블로킹
- **Phase 3 (US1)**: Phase 2 완료 후 — MVP 핵심
- **Phase 4 (US2)**: Phase 3 완료 후 (T018 의존) — 404 처리
- **Phase 5 (US3)**: Phase 3 완료 후 (T025 의존) — 검증 완성
- **Phase 6 (Polish)**: 원하는 Story 완료 후

### User Story Dependencies

- **US1 (P1)**: Phase 2 완료 후 독립 구현 가능
- **US2 (P2)**: US1의 T018(UpdateLessonService) 완료 후
- **US3 (P3)**: US1의 T018(UpdateLessonService) 완료 후. US2와 병렬 가능

### Parallel Opportunities

- Phase 1: T001~T008 모두 병렬 실행 가능
- Phase 2: T010, T011 병렬 가능. T012는 T009 완료 후
- Phase 3: T015, T017 병렬 (Mapper 내 다른 메서드). T020, T021 병렬 (다른 파일)
- Phase 4 & 5: US2, US3 병렬 가능 (T018 완료 후)

---

## Parallel Example: Phase 1

```
동시 실행:
- T001 UpdateLessonWebRequest.java 골격 생성
- T002 UpdateLessonWebResponse.java 골격 생성
- T003 UpdateLessonWebMapper.java 골격 생성
- T004 UpdateLessonAppRequest.java 골격 생성
- T005 UpdateLessonAppResponse.java 골격 생성
- T006 UpdateLessonAppMapper.java 골격 생성
- T007 UpdateLessonUseCase.java 골격 생성
- T008 UpdateLessonService.java 골격 생성
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Phase 1: Setup (T001~T008) — 골격 생성
2. Phase 2: Foundational (T009~T013) — DTO·인터페이스
3. Phase 3: US1 Implementation (T014~T019) — PUT 엔드포인트 동작
4. **STOP & VALIDATE**: `PUT /api/lesson/{lessonNo}` 성공 확인
5. Phase 3 Test (T020~T021) 작성

### Incremental Delivery

1. MVP (US1) → 정상 수정 동작
2. US2 추가 → 404 처리 완성
3. US3 추가 → 모든 검증 완성
4. Polish → Swagger, quickstart 검증

---

## Notes

- [P] 태스크 = 다른 파일, 의존성 없음 → 동시 실행 가능
- `UpdateLessonAppMapper.toDomain()`에서 `id = req.getLessonNo()` 주입은 JPA UPDATE의 핵심 — 누락 시 INSERT로 동작
- `LessonEntity.orphanRemoval = true`가 이미 설정되어 있으므로 별도 DELETE 쿼리 불필요
- Nested DTO 타입은 새로 선언하지 않고 `CreateLessonAppRequest.OptionAppReq` 등 기존 inner class를 import하여 재사용
- 각 Checkpoint에서 컴파일 오류 없음·테스트 통과 확인 후 다음 Phase 진행
