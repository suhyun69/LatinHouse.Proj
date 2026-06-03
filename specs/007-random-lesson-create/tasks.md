# Tasks: 랜덤 수업 생성 (POST /api/lesson/random)

**Input**: Design documents from `/specs/007-random-lesson-create/`

**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/post-lesson-random.md

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (US1, US2, US3)
- Include exact file paths in descriptions

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: 신규 유스케이스를 위한 기반 DTO 및 인터페이스 생성

- [x] T001 [P] `CreateRandomLessonAppResponse` 생성 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/application/port/in/CreateRandomLessonAppResponse.java` (`Long id` 필드)
- [x] T002 [P] `CreateRandomLessonWebResponse` 생성 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/adapter/in/web/CreateRandomLessonWebResponse.java` (`Long id` 필드)
- [x] T003 `CreateRandomLessonUseCase` 인터페이스 생성 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/application/port/in/CreateRandomLessonUseCase.java` (`CreateRandomLessonAppResponse createRandomLesson()` 메서드)

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Service 구현 전에 반드시 완료되어야 하는 Mapper 클래스 생성

**⚠️ CRITICAL**: Mapper 없이는 Service 및 Controller를 구현할 수 없음

- [x] T004 `CreateRandomLessonAppMapper` 생성 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/application/port/in/CreateRandomLessonAppMapper.java`
  - `toCreateLessonAppRequest(String instructorLo, String instructorLa)`: genre(S/B 랜덤), title(장르+레벨 조합), options(1~3개, startDate=오늘+7~60일, startTime=[10:00/14:00/19:00/20:00], endTime=startTime+2h, region=GN/HD), amount([30000/50000/80000/100000]), discounts(0~2개, type=E의 condition=가장이른startDate-7일, type=S의 condition=M/F), isActive=true 랜덤 생성. `ThreadLocalRandom.current()` 사용
  - `toAppResponse(CreateLessonAppResponse)` → `CreateRandomLessonAppResponse`
- [x] T005 `CreateRandomLessonWebMapper` 생성 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/adapter/in/web/CreateRandomLessonWebMapper.java`
  - `toWebResponse(CreateRandomLessonAppResponse)` → `CreateRandomLessonWebResponse`

**Checkpoint**: Foundation ready — Service 및 Controller 구현 시작 가능

---

## Phase 3: User Story 1 — 강사 프로필이 있을 때 랜덤 수업 생성 (Priority: P1) 🎯 MVP

**Goal**: 기존 강사 프로필을 랜덤 선택하여 수업을 생성하고 201 Created + 레슨 ID를 반환

**Independent Test**: 강사 프로필이 존재하는 상태에서 `POST /api/lesson/random` 호출 → 201 응답 및 기존 강사 ID가 instructorLo/La에 할당되었는지 확인

### Implementation for User Story 1

- [x] T006 [US1] `CreateRandomLessonService` 생성 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/application/service/CreateRandomLessonService.java`
  - `@Service`, `@RequiredArgsConstructor`
  - 생성자 주입: `GetProfilesUseCase`, `CreateProfileUseCase`, `SetInstructorUseCase`, `CreateLessonUseCase`
  - `createRandomLesson()` 구현:
    1. `getProfilesUseCase.getProfiles(true)` 호출
    2. 결과를 `sex=M`(남성)과 `sex=F`(여성)으로 분리
    3. 각각 `ThreadLocalRandom`으로 랜덤 선택
    4. `CreateRandomLessonAppMapper.toCreateLessonAppRequest(loId, laId)` 호출
    5. `createLessonUseCase.createLesson(appRequest)` 호출
    6. `CreateRandomLessonAppMapper.toAppResponse(createLessonAppResponse)` 반환
- [x] T007 [US1] `LessonController`에 `POST /random` 엔드포인트 추가 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/adapter/in/web/LessonController.java`
  - `CreateRandomLessonUseCase` 필드 추가 (생성자 주입)
  - `@PostMapping("/random")` 메서드 추가
  - `@Operation(summary = "랜덤 수업 생성", description = "파라미터를 랜덤으로 생성하여 수업을 생성합니다.")` 추가
  - `ResponseEntity.status(HttpStatus.CREATED).body(CreateRandomLessonWebMapper.toWebResponse(appResponse))` 반환
- [x] T008 [US1] `CreateRandomLessonControllerTest` 생성 — `Latinhouse.Be/src/test/java/com/latinhouse/api/lesson/adapter/in/web/CreateRandomLessonControllerTest.java`
  - `@WebMvcTest(LessonController.class)`
  - 테스트: `POST /api/lesson/random` → 201 Created + `{ "id": N }` 검증
  - `createRandomLessonUseCase` Mock 처리

**Checkpoint**: User Story 1 완료 — 강사 있는 상태에서 랜덤 수업 생성 가능

---

## Phase 4: User Story 2 — 강사 프로필이 없을 때 자동 생성 후 수업 생성 (Priority: P2)

**Goal**: 강사가 없을 때 신규 프로필을 생성하고 강사 지정 후 수업 생성

**Independent Test**: 강사 프로필 없는 초기 상태에서 `POST /api/lesson/random` 호출 → 201 응답, 신규 프로필 생성 및 강사 지정 확인

### Implementation for User Story 2

- [x] T009 [US2] `CreateRandomLessonService`에 강사 자동 생성 로직 추가 — `Latinhouse.Be/src/main/java/com/latinhouse/api/lesson/application/service/CreateRandomLessonService.java`
  - 남성 강사 없을 경우: `createProfileUseCase.createProfile(CreateProfileAppRequest(nickname="강사_M_<4자리 랜덤숫자>", sex=M))` 호출 → 반환된 id로 `setInstructorUseCase.setInstructor(SetInstructorAppRequest(profileId))` 호출
  - 여성 강사 없을 경우: 동일하게 `강사_F_<4자리 랜덤숫자>` nickname으로 생성 및 강사 지정
  - 프로필 생성 또는 강사 지정 실패 시 `RuntimeException` throw
- [x] T010 [US2] `CreateRandomLessonServiceTest` 생성 — `Latinhouse.Be/src/test/java/com/latinhouse/api/lesson/application/service/CreateRandomLessonServiceTest.java`
  - 테스트 케이스 1: 남성·여성 강사 모두 존재 → `createProfileUseCase` 호출 없음
  - 테스트 케이스 2: 강사 전혀 없음 → `createProfileUseCase` 2회, `setInstructorUseCase` 2회 호출
  - 테스트 케이스 3: 남성 강사만 존재 → 여성 강사만 자동 생성
  - 테스트 케이스 4: `createProfileUseCase` 예외 발생 시 `RuntimeException` throw 검증

**Checkpoint**: User Story 2 완료 — 초기 상태에서도 랜덤 수업 생성 가능

---

## Phase 5: User Story 3 — 생성된 수업의 필드 유효성 확인 (Priority: P3)

**Goal**: 랜덤 생성된 파라미터가 `POST /api/lesson` 비즈니스 규칙을 100% 준수하는지 확인

**Independent Test**: `CreateRandomLessonAppMapper`가 생성하는 `CreateLessonAppRequest`의 모든 필드가 유효한 값인지 단위 테스트로 검증

### Implementation for User Story 3

- [x] T011 [US3] `CreateRandomLessonAppMapperTest` 생성 — `Latinhouse.Be/src/test/java/com/latinhouse/api/lesson/application/port/in/CreateRandomLessonAppMapperTest.java`
  - `toCreateLessonAppRequest` 반복 호출(100회) 후 아래 항목 검증:
    - `genre` ∈ `{S, B}`
    - `options` 개수 1~3
    - 각 option: `endDateTime > startDateTime`
    - `amount` ∈ `{30000, 50000, 80000, 100000}`
    - `discounts` 개수 0~2
    - type=E discount의 condition이 `yyyy-MM-dd` 형식이고 가장 이른 startDate보다 7일 이전
    - type=S discount의 condition ∈ `{M, F}`
    - `isActive = true`

**Checkpoint**: User Story 3 완료 — 랜덤 생성 파라미터의 유효성 보장

---

## Phase 6: Polish & Cross-Cutting Concerns

- [x] T012 [P] Swagger 문서화 확인 — `LessonController.java`의 `@Tag`, `@Operation` 정상 렌더링 여부 서버 기동 후 `/swagger-ui.html` 접속 확인
- [x] T013 [P] 에러 응답 형식 확인 — `GlobalExceptionHandler`에 `RuntimeException` 핸들러가 `{ "status": 500, "errors": [...] }` 형식으로 반환하는지 확인. 필요 시 `Latinhouse.Be/src/main/java/com/latinhouse/api/common/exception/GlobalExceptionHandler.java` 수정

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 (Setup)**: 즉시 시작 가능
- **Phase 2 (Foundational)**: Phase 1 완료 후 시작 — Phase 3, 4, 5 전체 블로킹
- **Phase 3 (US1)**: Phase 2 완료 후 시작
- **Phase 4 (US2)**: Phase 3 완료 후 시작 (Service 파일 수정)
- **Phase 5 (US3)**: Phase 2 완료 후 시작 가능 (독립 테스트)
- **Phase 6 (Polish)**: Phase 3, 4 완료 후

### User Story Dependencies

- **US1 (P1)**: Phase 2 완료 후 즉시 시작 가능
- **US2 (P2)**: US1의 Service 파일을 수정하므로 US1 완료 후 시작
- **US3 (P3)**: AppMapper만 테스트하므로 Phase 2 완료 후 US1과 병렬 진행 가능

### Parallel Opportunities

- T001, T002: 서로 다른 파일 → 병렬 실행 가능
- T004, T005: 서로 다른 파일 → 병렬 실행 가능
- T011(US3), T006(US1): AppMapper와 Service는 다른 파일 → 병렬 실행 가능

---

## Parallel Example: Phase 2

```
Task: "CreateRandomLessonAppMapper 생성"
Task: "CreateRandomLessonWebMapper 생성"
```

## Parallel Example: Phase 3 + Phase 5

```
Task: "CreateRandomLessonService 구현 (US1)"
Task: "CreateRandomLessonAppMapperTest 생성 (US3)"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Phase 1 완료: DTO 생성
2. Phase 2 완료: Mapper 생성 (CRITICAL)
3. Phase 3 완료: Service + Controller
4. **STOP and VALIDATE**: `POST /api/lesson/random` 호출 → 201 확인
5. 강사 있는 환경에서 단독 데모 가능

### Incremental Delivery

1. Phase 1 + 2 → 기반 완료
2. Phase 3 (US1) → 강사 있을 때 수업 생성 동작 확인 → MVP
3. Phase 4 (US2) → 강사 없는 초기 상태에서도 동작 확인
4. Phase 5 (US3) → 랜덤 파라미터 유효성 보장
5. Phase 6 → Swagger + 에러 응답 완성

---

## Notes

- [P] tasks = 서로 다른 파일, 의존성 없음
- US2는 US1의 Service 파일을 수정하므로 반드시 US1 완료 후 진행
- `ThreadLocalRandom.current()` 사용 (멀티스레드 환경 안전)
- 신규 persistence adapter 불필요 — 저장은 기존 `CreateLessonUseCase` 위임
