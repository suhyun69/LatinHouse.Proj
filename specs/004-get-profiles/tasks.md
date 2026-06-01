# Tasks: 프로필 목록 조회 (GET /api/profiles)

**Input**: Design documents from `specs/004-get-profiles/`

**Prerequisites**: plan.md ✅, spec.md ✅, research.md ✅, data-model.md ✅, contracts/get-profiles.md ✅

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (US1, US2)

## Path Conventions

- Source: `Latinhouse.Be/src/main/java/com/latinhouse/api/`
- Tests: `Latinhouse.Be/src/test/java/com/latinhouse/api/`

---

## Phase 1: Setup (API 명세 추가)

**Purpose**: Constitution II(Contract-First) — 구현 전 api-spec.md에 명세를 먼저 추가한다.

- [X] T001 `docs/api-spec.md`의 Profile 섹션에 GET /api/profiles 명세 추가 (query param, 200 응답, 400 에러 포함)

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: 조회 레이어의 핵심 Port·Repository·Persistence를 확장한다. 모든 User Story 구현의 전제조건.

**⚠️ CRITICAL**: 이 phase가 완료되기 전에 User Story 구현을 시작할 수 없다.

- [X] T002 `profile/application/port/out/FindProfilePort.java`에 `findAll(Boolean isInstructor): List<Profile>` 메서드 추가
- [X] T003 [P] `profile/adapter/out/persistence/ProfileJpaRepository.java`에 `findAllByIsInstructor(boolean isInstructor): List<ProfileEntity>` 메서드 추가
- [X] T004 `profile/adapter/out/persistence/ProfilePersistenceAdapter.java`에 `findAll(Boolean isInstructor)` 구현 추가 — `isInstructor==null`이면 `findAll()`, 아니면 `findAllByIsInstructor()` 분기 처리 (T002, T003 완료 후)

**Checkpoint**: Foundation ready — User Story 구현 시작 가능

---

## Phase 3: User Story 1 - 전체 프로필 목록 조회 (Priority: P1) 🎯 MVP

**Goal**: `GET /api/profiles` (파라미터 없음) 호출 시 전체 프로필 목록을 200 OK로 반환한다.

**Independent Test**: `GET /api/profiles` 호출 → 전체 프로필 배열 반환, 프로필 없을 때 빈 배열 반환 확인.

### Implementation for User Story 1

- [X] T005 [P] [US1] `profile/application/port/in/GetProfilesAppResponse.java` 생성 — `id: String`, `nickname: String`, `sex: Sex`, `isInstructor: boolean` 필드 포함
- [X] T006 [P] [US1] `profile/application/port/in/GetProfilesUseCase.java` 생성 — `getProfiles(Boolean isInstructor): List<GetProfilesAppResponse>` 인터페이스
- [X] T007 [P] [US1] `profile/adapter/in/web/GetProfilesWebResponse.java` 생성 — `id: String`, `nickname: String`, `sex: String`, `isInstructor: boolean` 필드 포함
- [X] T008 [US1] `profile/application/port/in/GetProfilesAppMapper.java` 생성 — `Profile → GetProfilesAppResponse` 변환 정적 메서드 (T005 완료 후)
- [X] T009 [US1] `profile/adapter/in/web/GetProfilesWebMapper.java` 생성 — `GetProfilesAppResponse → GetProfilesWebResponse` 변환 정적 메서드 (T007 완료 후)
- [X] T010 [US1] `profile/application/service/GetProfilesService.java` 생성 — `GetProfilesUseCase` 구현체. `findProfilePort.findAll(null)`로 전체 조회 후 `GetProfilesAppMapper`로 변환하여 반환 (T004, T006, T008 완료 후)
- [X] T011 [US1] `profile/adapter/in/web/ProfileController.java`에 `GET /api/profiles` 엔드포인트 추가 — `@RequestParam(required=false) Boolean isInstructor` 없이 우선 기본 전체 조회, `@Operation` Swagger 문서화 포함 (T009, T010 완료 후)
- [X] T012 [US1] `profile/application/service/GetProfilesServiceTest.java` 생성 — `isInstructor=null`일 때 전체 프로필 반환 및 빈 목록 반환 케이스 테스트 (T010 완료 후)
- [X] T013 [US1] `profile/adapter/in/web/ProfileControllerTest.java`에 `GET /api/profiles` 기본 조회 테스트 추가 — 프로필 있을 때 200+배열, 없을 때 200+빈 배열 (T011 완료 후)

**Checkpoint**: `GET /api/profiles` 호출로 전체 목록 반환 동작 확인 가능

---

## Phase 4: User Story 2 - 강사 프로필 필터링 조회 (Priority: P2)

**Goal**: `GET /api/profiles?isInstructor=true` 또는 `?isInstructor=false` 쿼리 파라미터로 필터링된 목록을 반환한다.

**Independent Test**: `?isInstructor=true` → 강사만 반환, `?isInstructor=false` → 비강사만 반환, 각각 결과 없을 때 빈 배열 반환 확인.

### Implementation for User Story 2

- [X] T014 [US2] `profile/adapter/in/web/ProfileController.java`의 `GET /api/profiles` 엔드포인트에 `@RequestParam(required=false) Boolean isInstructor` 파라미터 추가 후 `getProfilesUseCase.getProfiles(isInstructor)` 호출로 수정 (T011 완료 후)
- [X] T015 [US2] `profile/application/service/GetProfilesService.java`를 수정하여 `isInstructor` 파라미터를 `findProfilePort.findAll(isInstructor)`에 그대로 전달 (T010, T014 완료 후)
- [X] T016 [US2] `profile/application/service/GetProfilesServiceTest.java`에 `isInstructor=true` (강사만 반환) 및 `isInstructor=false` (비강사만 반환) 케이스 추가 (T015 완료 후)
- [X] T017 [US2] `profile/adapter/in/web/ProfileControllerTest.java`에 `?isInstructor=true` 필터 테스트 추가 — 강사 프로필만 반환되는지 확인 (T014 완료 후)

**Checkpoint**: `GET /api/profiles?isInstructor=true/false` 필터링 동작 확인 가능

---

## Phase 5: Polish & Cross-Cutting Concerns

**Purpose**: 에러 처리, 문서화 완성

- [X] T018 `common/exception/GlobalExceptionHandler.java`에 `InvalidParamException` 핸들러 추가 — `isInstructor`에 `true`/`false` 외 값 입력 시 `{"status":400,"errors":[{"field":"isInstructor","message":"isInstructor는 true 또는 false만 입력 가능합니다."}]}` 반환 (InvalidParamException 신규 생성)
- [X] T019 [P] `profile/adapter/in/web/ProfileControllerTest.java`에 잘못된 파라미터(`?isInstructor=yes`) 입력 시 400 응답 테스트 추가 (T018 완료 후)
- [X] T020 [P] `docs/api-spec.md`의 GET /api/profiles 섹션에 400 에러 케이스 명세가 올바른지 검토 및 보완 (T018 완료 후)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 (Setup)**: 즉시 시작 가능
- **Phase 2 (Foundational)**: Phase 1 완료 후 시작 — 모든 User Story를 블록
- **Phase 3 (US1)**: Phase 2 완료 후 시작
- **Phase 4 (US2)**: Phase 3 완료 후 시작 (같은 파일 수정)
- **Phase 5 (Polish)**: Phase 4 완료 후 시작

### Within Phase 3

- T005, T006, T007은 동시 병렬 실행 가능 (서로 다른 파일)
- T008은 T005 완료 후, T009는 T007 완료 후
- T010은 T004, T006, T008 완료 후
- T011은 T009, T010 완료 후
- T012, T013은 T010, T011 완료 후

---

## Parallel Example: Phase 3 시작 시

```bash
# 동시 실행 가능한 태스크 (모두 다른 파일):
Task T005: GetProfilesAppResponse.java 생성
Task T006: GetProfilesUseCase.java 생성
Task T007: GetProfilesWebResponse.java 생성
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Phase 1: api-spec.md 명세 추가
2. Phase 2: Port/JPA/Persistence 확장
3. Phase 3: US1 — 전체 조회 엔드포인트
4. **STOP and VALIDATE**: `GET /api/profiles` 호출 테스트
5. 준비되면 배포/데모

### Incremental Delivery

1. Phase 1+2 완료 → 기반 레이어 준비
2. Phase 3 → US1 완료 → 기본 목록 조회 동작 (MVP!)
3. Phase 4 → US2 완료 → 필터링 조회 동작
4. Phase 5 → Polish 완료 → 에러 처리 포함 완전한 기능

---

## Notes

- [P] tasks = 다른 파일, 의존 없음 → 병렬 실행 가능
- US2는 US1과 동일 파일을 수정하므로 순차 진행 필요
- `isInstructor` Boolean 파라미터는 Spring이 자동 변환하므로 별도 Validator 불필요
- `GetProfilesWebRequest` / `GetProfilesAppRequest` 미생성 (단순 파라미터, DTO 불필요)
