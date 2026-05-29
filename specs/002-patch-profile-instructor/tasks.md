# Tasks: 강사 지정 (PATCH /api/profile/{profileId}/instructor)

**Input**: Design documents from `specs/002-patch-profile-instructor/`

**Prerequisites**: plan.md ✅ spec.md ✅ research.md ✅ data-model.md ✅ contracts/ ✅

**Organization**: User Story 3개(US1 성공·US3 404·US2 멱등)를 독립적으로 구현·테스트 가능하도록 단계 구성.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: 병렬 실행 가능 (다른 파일, 미완료 태스크에 의존 없음)
- **[Story]**: 해당 User Story 레이블 (US1 / US2 / US3)
- 모든 경로는 `Latinhouse.Be/src/` 기준

---

## Phase 1: Setup

**Purpose**: 기존 Spring Boot 프로젝트를 그대로 활용. 별도 초기화 불필요.

> 프로젝트 구조·의존성·H2·springdoc-openapi 모두 `001-create-profile`에서 구성 완료.  
> 이 Phase는 스킵하고 Phase 2(Foundational)부터 시작한다.

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: 세 User Story 모두가 의존하는 핵심 타입. 이 Phase 완료 전까지 어떤 User Story도 시작 불가.

**⚠️ CRITICAL**: Phase 2 완료 후 Phase 3~5를 시작할 수 있다.

- [x] T001 Create `ProfileNotFoundException` in `main/java/com/latinhouse/api/common/exception/ProfileNotFoundException.java` — `RuntimeException` 상속, 생성자 파라미터 `String profileId`
- [x] T002 [P] Create `FindProfilePort` interface in `main/java/com/latinhouse/api/profile/application/port/out/FindProfilePort.java` — `Optional<Profile> findById(String profileId)`
- [x] T003 [P] Create `UpdateProfilePort` interface in `main/java/com/latinhouse/api/profile/application/port/out/UpdateProfilePort.java` — `Profile update(Profile profile)`
- [x] T004 Add `asInstructor()` domain method to `main/java/com/latinhouse/api/profile/domain/Profile.java` — 불변 패턴: 현재 Profile 필드 유지하고 `isInstructor=true`로 새 인스턴스 반환

**Checkpoint**: Foundational 완료 → Phase 3·4·5 시작 가능

---

## Phase 3: User Story 1 - 강사 지정 성공 (Priority: P1) 🎯 MVP

**Goal**: 유효한 profileId로 `PATCH /api/profile/{profileId}/instructor`를 호출하면 200 OK와 `{ "id": "..." }`를 반환한다.

**Independent Test**: 먼저 `POST /api/profile`로 프로필 생성 → PATCH 호출 → 200 OK + profileId 반환 확인

### Implementation for User Story 1

- [x] T005 [P] [US1] Create `SetInstructorAppRequest` in `main/java/com/latinhouse/api/profile/application/port/in/SetInstructorAppRequest.java` — `@Getter @Builder`, 필드: `String profileId`
- [x] T006 [P] [US1] Create `SetInstructorAppResponse` in `main/java/com/latinhouse/api/profile/application/port/in/SetInstructorAppResponse.java` — `@Getter @Builder`, 필드: `String id`
- [x] T007 [P] [US1] Create `SetInstructorUseCase` interface in `main/java/com/latinhouse/api/profile/application/port/in/SetInstructorUseCase.java` — `SetInstructorAppResponse setInstructor(SetInstructorAppRequest request)`
- [x] T008 [P] [US1] Create `SetInstructorAppMapper` in `main/java/com/latinhouse/api/profile/application/port/in/SetInstructorAppMapper.java` — static 메서드: `toAppResponse(Profile profile) → SetInstructorAppResponse`; private 생성자
- [x] T009 [US1] Create `SetInstructorService` in `main/java/com/latinhouse/api/profile/application/service/SetInstructorService.java` — `@Service @RequiredArgsConstructor`, `FindProfilePort` + `UpdateProfilePort` 주입; `findById` → empty 시 `ProfileNotFoundException` throw → `profile.asInstructor()` → `update()` → `SetInstructorAppMapper.toAppResponse()` 반환
- [x] T010 [US1] Implement `FindProfilePort` and `UpdateProfilePort` in `main/java/com/latinhouse/api/profile/adapter/out/persistence/ProfilePersistenceAdapter.java` — `findById`: `profileJpaRepository.findById()` → `Optional<Profile>`; `update`: `ProfilePersistenceMapper.toEntity(profile)` → `profileJpaRepository.save()` → `toDomain()` 반환
- [x] T011 [P] [US1] Create `SetInstructorWebResponse` in `main/java/com/latinhouse/api/profile/adapter/in/web/SetInstructorWebResponse.java` — `@Getter @Builder`, 필드: `String id`
- [x] T012 [P] [US1] Create `SetInstructorWebMapper` in `main/java/com/latinhouse/api/profile/adapter/in/web/SetInstructorWebMapper.java` — static 메서드: `toAppRequest(String profileId) → SetInstructorAppRequest`; `toWebResponse(SetInstructorAppResponse) → SetInstructorWebResponse`; private 생성자
- [x] T013 [US1] Add PATCH endpoint to `main/java/com/latinhouse/api/profile/adapter/in/web/ProfileController.java` — `SetInstructorUseCase` 주입 추가; `@Operation(summary = "강사 지정") @PatchMapping("/{profileId}/instructor")`; `@PathVariable String profileId`; `SetInstructorWebMapper.toAppRequest()` → useCase → `toWebResponse()` → `ResponseEntity.ok()` 반환

### Tests for User Story 1

- [x] T014 [P] [US1] Create `SetInstructorServiceTest` in `test/java/com/latinhouse/api/profile/application/service/SetInstructorServiceTest.java` — 성공 케이스: `findById` Mock → `profile.asInstructor()` Mock → `update` Mock 검증; 반환 id 검증
- [x] T015 [P] [US1] Add PATCH success test to `test/java/com/latinhouse/api/profile/adapter/in/web/ProfileControllerTest.java` — `@MockitoBean SetInstructorUseCase`; `PATCH /api/profile/Ab2Cd3Ef/instructor` → 200 OK → `$.id = "Ab2Cd3Ef"` 검증

**Checkpoint**: `PATCH /api/profile/{validId}/instructor` → 200 OK + `{ "id": "..." }` 동작 확인

---

## Phase 4: User Story 3 - 존재하지 않는 프로필 처리 (Priority: P1)

**Goal**: 존재하지 않는 profileId로 요청 시 404 Not Found와 `PROFILE_NOT_FOUND` 에러 응답을 반환한다.

**Independent Test**: 임의 문자열(예: `NOTEXIST`) profileId로 PATCH → 404 + `errors[0].field="profileId"` 확인

### Implementation for User Story 3

- [x] T016 [US3] Add `ProfileNotFoundException` handler to `main/java/com/latinhouse/api/common/exception/GlobalExceptionHandler.java` — `@ExceptionHandler(ProfileNotFoundException.class) @ResponseStatus(HttpStatus.NOT_FOUND)`; `ErrorResponse` 빌드: `status=404`, `field="profileId"`, `message="프로필을 찾을 수 없습니다."`

### Tests for User Story 3

- [x] T017 [P] [US3] Add profile-not-found test to `test/java/com/latinhouse/api/profile/application/service/SetInstructorServiceTest.java` — `findById` empty 반환 Mock → `ProfileNotFoundException` throw 검증
- [x] T018 [P] [US3] Add 404 test to `test/java/com/latinhouse/api/profile/adapter/in/web/ProfileControllerTest.java` — `setInstructor` Mock throws `ProfileNotFoundException` → `PATCH` → 404 → `$.status=404`, `$.errors[0].field="profileId"` 검증

**Checkpoint**: 존재하지 않는 profileId → 404 + `{ "status": 404, "errors": [{"field": "profileId", ...}] }` 동작 확인

---

## Phase 5: User Story 2 - 멱등성 (Priority: P2)

**Goal**: 이미 강사인 프로필에 동일 요청을 재호출해도 200 OK로 응답한다.

**Independent Test**: US1 성공 후 동일 profileId로 PATCH 재호출 → 200 OK + 동일 id 반환 확인

> **구현 참고**: Phase 3에서 구현한 `SetInstructorService`는 이미 멱등성을 자연스럽게 충족한다.  
> (`profile.asInstructor()`는 `isInstructor`가 이미 true여도 동일하게 동작하고 `update()`가 재저장)  
> 이 Phase는 테스트로만 검증한다.

### Tests for User Story 2

- [x] T019 [P] [US2] Add idempotency test to `test/java/com/latinhouse/api/profile/application/service/SetInstructorServiceTest.java` — `findById` Mock → `isInstructor=true`인 Profile 반환 → `update` 호출 검증 → 200 반환 검증
- [x] T020 [P] [US2] Add idempotency test to `test/java/com/latinhouse/api/profile/adapter/in/web/ProfileControllerTest.java` — `setInstructor` Mock → 동일 id 반환 → 200 OK + `$.id` 검증

**Checkpoint**: 세 User Story 모두 독립적으로 기능 동작 확인 완료

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: 전체 품질 검증 및 Constitution Quality Gate 통과 확인

- [ ] T021 Verify Swagger documentation — 서버 기동 후 `http://localhost:8080/swagger-ui/index.html` → Profile 섹션 → `PATCH /api/profile/{profileId}/instructor` 엔드포인트와 `@Operation` 설명 확인
- [x] T022 Run full test suite — `cd Latinhouse.Be && ./gradlew test` 전체 통과 확인; 컴파일 오류 0, 테스트 실패 0
- [ ] T023 Run quickstart.md manual validation — `quickstart.md` 시나리오 1~4 순서대로 수동 실행하여 응답 검증

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 (Setup)**: 스킵 — 이미 완료
- **Phase 2 (Foundational)**: 즉시 시작 가능 — **Phase 3·4·5를 블록**
- **Phase 3 (US1 P1)**: Phase 2 완료 후
- **Phase 4 (US3 P1)**: Phase 2 완료 후 (Phase 3와 동시 진행 가능, 단 T016은 T009 이후 의미 있음)
- **Phase 5 (US2 P2)**: Phase 3 완료 후 (구현은 이미 됐으므로 테스트만)
- **Phase 6 (Polish)**: Phase 3·4·5 모두 완료 후

### User Story Dependencies

- **US1 (P1)**: Phase 2 완료 후 시작. 다른 User Story에 의존 없음.
- **US3 (P1)**: Phase 2 + T009(SetInstructorService) 완료 후 T016 의미 있음. 테스트는 US1과 병렬 가능.
- **US2 (P2)**: Phase 3 완료 후 테스트 시작.

### Within Each Phase

- Application DTO(T005~T008) → Service(T009) → Persistence(T010) → Web(T011~T013)
- Phase 3 내 [P] 태스크(T005·T006·T007·T008·T011·T012)는 동시 작업 가능
- 테스트(T014·T015·T019·T020)는 구현 완료 후 병렬 작업 가능

---

## Parallel Example: Phase 2

```bash
# Phase 2 병렬 실행 가능 태스크:
Task T002: FindProfilePort.java
Task T003: UpdateProfilePort.java
# T001(ProfileNotFoundException)과 T004(Profile.asInstructor)는 순차 (기존 파일 수정 포함)
```

## Parallel Example: Phase 3 (US1)

```bash
# Application DTO 동시 생성:
Task T005: SetInstructorAppRequest.java
Task T006: SetInstructorAppResponse.java
Task T007: SetInstructorUseCase.java
Task T008: SetInstructorAppMapper.java

# Web DTO 동시 생성 (T005~T008와도 병렬):
Task T011: SetInstructorWebResponse.java
Task T012: SetInstructorWebMapper.java
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Phase 2: Foundational (T001~T004)
2. Phase 3: US1 구현 (T005~T013) + 테스트 (T014~T015)
3. **STOP and VALIDATE**: `PATCH /api/profile/{validId}/instructor` → 200 OK 확인
4. 필요 시 배포/데모

### Incremental Delivery

1. Phase 2 완료 → 기반 타입 준비
2. Phase 3 완료 → 성공 케이스 동작 (MVP!)
3. Phase 4 완료 → 에러 처리 추가
4. Phase 5 완료 → 멱등성 검증
5. Phase 6 완료 → Quality Gate 통과

---

## Notes

- [P] 태스크 = 다른 파일, 의존 없음 → 병렬 실행 가능
- [Story] 레이블로 각 태스크의 User Story 추적 가능
- `ProfilePersistenceAdapter`(T010) 수정 시 기존 `SaveProfilePort` 구현 절대 건드리지 않도록 주의
- `ProfileController`(T013) 수정 시 기존 `POST /api/profile` 핸들러 영향 없도록 주의
- `GlobalExceptionHandler`(T016) 수정 시 기존 `MethodArgumentNotValidException` 핸들러 유지
- Constitution Quality Gate: 컴파일 오류 없음 + 전체 테스트 통과 + Swagger 문서화 필수
