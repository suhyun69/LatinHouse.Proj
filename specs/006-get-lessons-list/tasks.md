# Tasks: 레슨 목록 조회 (GET /api/lessons)

**Input**: Design documents from `specs/006-get-lessons-list/`

**Prerequisites**: plan.md ✅ spec.md ✅ research.md ✅ data-model.md ✅ contracts/ ✅

**Organization**: User Story 단위로 구성. 각 Phase 완료 시 독립 테스트 가능.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: 병렬 실행 가능 (다른 파일, 선행 의존 없음)
- **[Story]**: 어느 User Story에 해당하는지 (US1/US2/US3)
- 각 태스크에 정확한 파일 경로 포함

---

## Phase 1: Setup (공유 인프라)

**Purpose**: 신규 포트·유스케이스 골격 생성. 기존 Entity·Domain 변경 없음.

- [x] T001 `LoadLessonsPort` 인터페이스 생성 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/application/port/out/LoadLessonsPort.java` (`loadLessons(Region, String, Genre): List<Lesson>`)
- [x] T002 `GetLessonsUseCase` 인터페이스 생성 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/application/port/in/GetLessonsUseCase.java`
- [x] T003 [P] `GetLessonsAppRequest` DTO 생성 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/application/port/in/GetLessonsAppRequest.java` (필드: `region: Region`, `instructor: String`, `genre: Genre`, 모두 nullable)
- [x] T004 [P] `GetLessonsAppResponse` DTO 생성 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/application/port/in/GetLessonsAppResponse.java` (flat item: optionId, lessonNo, instructorLo, instructorLa, title, genre, startDate, startTime, endDate, endTime, region, price, discountCondition, discountAmount, status)

---

## Phase 2: Foundational (핵심 구현 — 모든 US 선행 완료 필수)

**Purpose**: JPA 동적 필터 쿼리 + status·discount 계산 로직. US1~US3 모두 이 기반에 의존.

**⚠️ CRITICAL**: 이 Phase 완료 전까지 User Story 구현 불가

- [x] T005 `LessonJpaRepository`에 `JpaSpecificationExecutor<LessonEntity>` 추가 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/adapter/out/persistence/LessonJpaRepository.java`
- [x] T006 `LessonPersistenceAdapter`에 `LoadLessonsPort` 구현 추가 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/adapter/out/persistence/LessonPersistenceAdapter.java`. JPA Specification 3개 조건(genre, instructor OR, region JOIN) AND 조합. null 파라미터는 해당 조건 제외. `DISTINCT` 적용으로 region JOIN 시 중복 제거.
- [x] T007 `GetLessonsAppMapper` 구현 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/application/port/in/GetLessonsAppMapper.java`. `List<Lesson>` → `List<GetLessonsAppResponse>` 변환. 각 Lesson × Option을 flat하게 펼침. `calcStatus()`, `findEarlybird()` 정적 헬퍼 포함.
- [x] T008 `GetLessonsService` 구현 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/application/service/GetLessonsService.java`. `GetLessonsUseCase` 구현. `LoadLessonsPort.loadLessons()` 호출 후 `GetLessonsAppMapper.toResponse()` 적용.

**Checkpoint**: Foundational 완료 — User Story 구현 시작 가능

---

## Phase 3: User Story 1 — 레슨 목록 전체 조회 (Priority: P1) 🎯 MVP

**Goal**: 필터 없이 GET /api/lessons 호출 시 모든 레슨 옵션 목록 반환

**Independent Test**: 레슨 2개(각 옵션 1개) 생성 후 `curl http://localhost:8080/api/lessons` → 2개 항목 반환, status·price·discount 필드 포함 확인

### Implementation

- [x] T009 [US1] `GetLessonsWebResponse` DTO 생성 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/adapter/in/web/GetLessonsWebResponse.java` (`GetLessonsAppResponse`와 동일 필드)
- [x] T010 [US1] `GetLessonsWebMapper` 구현 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/adapter/in/web/GetLessonsWebMapper.java`. `AppResponse → WebResponse` 변환 메서드 + 쿼리 파라미터 String → `GetLessonsAppRequest` 변환 메서드 (null safe, enum 변환 없음 — US2에서 추가)
- [x] T011 [US1] `LessonController`에 `GET /api/lessons` 엔드포인트 추가 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/adapter/in/web/LessonController.java`. `@GetMapping("/lessons")`, `@RequestParam(required=false)` 3개, `@Operation` Swagger 문서화. 반환: `ResponseEntity<List<GetLessonsWebResponse>>`
- [x] T012 [US1] `LessonControllerTest`에 GET /api/lessons 기본 조회 테스트 추가 — `Latinhouse.Be/src/test/java/com/latinhouse/api/lesson/adapter/in/web/LessonControllerTest.java`. 필터 없이 호출 시 200 OK + 올바른 필드 반환 검증
- [x] T013 [US1] `GetLessonsServiceTest` 생성 — `Latinhouse.Be/src/test/java/com/latinhouse/api/lesson/application/service/GetLessonsServiceTest.java`. 전체 조회 + status/discount 계산 검증 (INACTIVE/PENDING/IN_PROGRESS/DONE, EarlyBird 적용 및 미적용)

**Checkpoint**: US1 완료 — 필터 없는 목록 조회가 독립적으로 동작함

---

## Phase 4: User Story 2 — 필터링 조회 (Priority: P2)

**Goal**: region / instructor / genre 필터(단독 및 AND 조합) 적용 시 조건에 맞는 옵션만 반환

**Independent Test**: `curl "http://localhost:8080/api/lessons?region=GN"` → GN 옵션만 반환 확인

### Implementation

- [x] T014 [US2] `GetLessonsWebMapper`에 필터 파라미터 enum 변환 추가 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/adapter/in/web/GetLessonsWebMapper.java`. `region` String → `Region.fromCode()`, `genre` String → `Genre.fromCode()` 호출. 잘못된 코드 시 `InvalidParamException` throw (field명 포함)
- [x] T015 [US2] `LessonControllerTest`에 필터링 케이스 추가 — `Latinhouse.Be/src/test/java/com/latinhouse/api/lesson/adapter/in/web/LessonControllerTest.java`. region/genre/instructor 단독 필터, AND 조합 필터, 잘못된 코드 → 400 검증
- [x] T016 [US2] `GetLessonsServiceTest`에 필터 조합 케이스 추가 — `Latinhouse.Be/src/test/java/com/latinhouse/api/lesson/application/service/GetLessonsServiceTest.java`. 각 필터 파라미터 전달 시 `LoadLessonsPort`에 올바른 인자 전달 검증

**Checkpoint**: US2 완료 — 필터링 조회가 독립적으로 동작함

---

## Phase 5: User Story 3 — status·discount 자동 계산 (Priority: P3)

**Goal**: 각 항목의 status, discountCondition, discountAmount가 비즈니스 규칙에 따라 정확히 계산됨

**Independent Test**: isActive=false 레슨 조회 → status="INACTIVE"; EarlyBird discount(조건 미래) 있는 레슨 → discountCondition/discountAmount 반환 확인

*(이 Phase의 핵심 로직은 T007 GetLessonsAppMapper에서 구현됨. 여기서는 경계 케이스 검증 추가)*

### Implementation

- [x] T017 [P] [US3] `GetLessonsAppMapper` 경계 케이스 보완 — EarlyBird discount 복수일 때 가장 이른 1건 선택, condition이 정확히 오늘인 경우 포함(`>=`) 확인
- [x] T018 [P] [US3] `GetLessonsServiceTest`에 discount 경계 케이스 추가 — EarlyBird 복수 후보 중 가장 이른 것 선택, 모두 과거 → null 반환, condition=오늘 → 포함 검증

**Checkpoint**: US3 완료 — status·discount 계산이 모든 케이스에서 올바름

---

## Phase 6: Polish & Cross-Cutting Concerns

- [x] T019 [P] `docs/api-spec.md` GET /api/lessons 섹션 최종 검토 및 필요 시 업데이트 — `docs/api-spec.md`
- [x] T020 `quickstart.md` 기반 로컬 실행 통합 검증 — `specs/006-get-lessons-list/quickstart.md` 시나리오 수동 실행하여 전체 플로우 확인
- [x] T021 [P] Swagger UI에서 GET /api/lessons 엔드포인트 파라미터·응답 명세 확인

---

## Dependencies & Execution Order

### Phase 의존성

- **Phase 1 (Setup)**: 즉시 시작 가능
- **Phase 2 (Foundational)**: Phase 1 완료 후 시작. **US 구현 전체 블로킹**
- **Phase 3 (US1)**: Phase 2 완료 필수
- **Phase 4 (US2)**: Phase 2 완료 필수 (US1과 병렬 가능하나 동일 파일 수정 있음)
- **Phase 5 (US3)**: Phase 2 완료 필수 (T007 로직 보완)
- **Phase 6 (Polish)**: 원하는 US 완료 후

### User Story 간 의존성

- **US1**: Phase 2 완료 후 즉시 시작 가능
- **US2**: Phase 2 완료 후 시작. T010(WebMapper) 파일을 US1과 공유하므로 순차 처리 권장
- **US3**: Phase 2 완료 후 시작. T007 보완이므로 US1 완료 후 추가 권장

### 병렬 실행 기회

- T003, T004: Phase 1 내 병렬 실행 가능
- T009, T010: Phase 3 내 병렬 실행 가능 (다른 파일)
- T012, T013: Phase 3 내 병렬 실행 가능 (다른 파일)
- T017, T018: Phase 5 내 병렬 실행 가능
- T019, T021: Phase 6 내 병렬 실행 가능

---

## Parallel Example: Phase 2

```bash
# T005, T006, T007, T008은 순차 실행 (T005 → T006, T007 → T008)
# T006과 T007은 다른 파일이므로 병렬 실행 가능
Task T006: "LessonPersistenceAdapter에 LoadLessonsPort 구현"
Task T007: "GetLessonsAppMapper 구현 (status·discount 계산)"
# T008은 T006, T007 완료 후
Task T008: "GetLessonsService 구현"
```

---

## Implementation Strategy

### MVP (US1만)

1. Phase 1: Setup 완료
2. Phase 2: Foundational 완료 (블로킹)
3. Phase 3: US1 완료
4. **검증**: 필터 없는 목록 조회 동작 확인
5. 필요 시 배포/데모

### 전체 순차 전달

1. Phase 1 + 2 → 기반 완료
2. Phase 3 (US1) → 전체 조회 동작
3. Phase 4 (US2) → 필터링 추가
4. Phase 5 (US3) → 계산 정확성 보완
5. Phase 6 → 문서·검증 마무리
