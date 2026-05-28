# Tasks: Create Profile (POST /api/profile)

**Input**: Design documents from `specs/001-create-profile/`

**Prerequisites**: plan.md ✅, spec.md ✅, research.md ✅, data-model.md ✅, contracts/ ✅

**Organization**: Tasks are grouped by layer/phase to match Hexagonal Architecture dependency order.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[US1]**: Belongs to User Story 1 — Profile Creation (only user story)
- Paths use `Latinhouse.Be/src/main/java/com/latinhouse/api/` prefix

---

## Phase 1: Setup (Build Configuration)

**Purpose**: Add missing dependency required by Constitution Quality Gate (Swagger)

- [x] T001 Add `springdoc-openapi-starter-webmvc-ui:3.0.0` to `Latinhouse.Be/build.gradle`

**Checkpoint**: Project compiles with Swagger dependency

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Shared infrastructure required by ALL layers before profile feature can be built

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [x] T002 [P] Create `ProfileIdGenerator` (SecureRandom + 55-char custom charset, excluding `i I 1 l 0 o O`) in `Latinhouse.Be/src/main/java/com/latinhouse/api/common/util/ProfileIdGenerator.java`
- [x] T003 [P] Create `ErrorResponse` record/class (`int status`, `List<FieldError> errors`) in `Latinhouse.Be/src/main/java/com/latinhouse/api/common/exception/ErrorResponse.java`
- [x] T004 Create `GlobalExceptionHandler` (`@RestControllerAdvice`) handling `MethodArgumentNotValidException` → 400 + `ErrorResponse` in `Latinhouse.Be/src/main/java/com/latinhouse/api/common/exception/GlobalExceptionHandler.java`
- [x] T005 Create `SecurityConfig` permitting all requests to `POST /api/profile` (disable CSRF for REST) in `Latinhouse.Be/src/main/java/com/latinhouse/api/common/config/SecurityConfig.java`

**Checkpoint**: Foundation ready — error handling, security, and ID generation available

---

## Phase 3: User Story 1 — Profile Creation (Priority: P1) 🎯 MVP

**Goal**: User submits nickname + sex → system creates profile → returns 8-char unique ID with 201 Created. Invalid input returns 400 with Korean field-level error messages.

**Independent Test**: `POST /api/profile` with `{"nickname":"TestUser","sex":"M"}` returns `201 {"id":"<8chars>"}`. Invalid requests return `400 {"status":400,"errors":[...]}`.

### Domain

- [x] T006 [P] [US1] Create `Sex` enum (`M`, `F`) in `Latinhouse.Be/src/main/java/com/latinhouse/api/profile/domain/Sex.java`
- [x] T007 [P] [US1] Create `Profile` domain object (`id`, `nickname`, `Sex sex`, `boolean isInstructor`) using Lombok `@Builder` in `Latinhouse.Be/src/main/java/com/latinhouse/api/profile/domain/Profile.java`

### Application — Ports

- [x] T008 [P] [US1] Create `CreateProfileAppRequest` (`String nickname`, `Sex sex`) in `Latinhouse.Be/src/main/java/com/latinhouse/api/profile/application/port/in/CreateProfileAppRequest.java`
- [x] T009 [P] [US1] Create `CreateProfileAppResponse` (`String id`) in `Latinhouse.Be/src/main/java/com/latinhouse/api/profile/application/port/in/CreateProfileAppResponse.java`
- [x] T010 [P] [US1] Create `CreateProfileUseCase` interface (`CreateProfileAppResponse createProfile(CreateProfileAppRequest)`) in `Latinhouse.Be/src/main/java/com/latinhouse/api/profile/application/port/in/CreateProfileUseCase.java`
- [x] T011 [P] [US1] Create `SaveProfilePort` interface (`Profile save(Profile)`) in `Latinhouse.Be/src/main/java/com/latinhouse/api/profile/application/port/out/SaveProfilePort.java`

### Application — Mapper & Service

- [x] T012 [US1] Create `CreateProfileAppMapper` (static methods: `toDomain(AppRequest)` sets `isInstructor=false`; `toAppResponse(Profile)`) in `Latinhouse.Be/src/main/java/com/latinhouse/api/profile/application/port/in/CreateProfileAppMapper.java`
- [x] T013 [US1] Create `CreateProfileService` (implements `CreateProfileUseCase`; calls `ProfileIdGenerator.generate()`, sets id on domain, calls `saveProfilePort.save()`) in `Latinhouse.Be/src/main/java/com/latinhouse/api/profile/application/service/CreateProfileService.java`

### Persistence Adapter

- [x] T014 [P] [US1] Create `ProfileEntity` (`@Entity`, fields: `id`, `nickname`, `sex` as String, `isInstructor`) in `Latinhouse.Be/src/main/java/com/latinhouse/api/profile/adapter/out/persistence/ProfileEntity.java`
- [x] T015 [P] [US1] Create `ProfileJpaRepository` (extends `JpaRepository<ProfileEntity, String>`) in `Latinhouse.Be/src/main/java/com/latinhouse/api/profile/adapter/out/persistence/ProfileJpaRepository.java`
- [x] T016 [US1] Create `ProfilePersistenceMapper` (static methods: `toEntity(Profile)`, `toDomain(ProfileEntity)`) in `Latinhouse.Be/src/main/java/com/latinhouse/api/profile/adapter/out/persistence/ProfilePersistenceMapper.java`
- [x] T017 [US1] Create `ProfilePersistenceAdapter` (implements `SaveProfilePort`; uses `ProfileJpaRepository` + `ProfilePersistenceMapper`) in `Latinhouse.Be/src/main/java/com/latinhouse/api/profile/adapter/out/persistence/ProfilePersistenceAdapter.java`

### Web Adapter

- [x] T018 [P] [US1] Create `CreateProfileWebRequest` (`@NotBlank String nickname`, `@NotBlank @Pattern(regexp="^[MF]$") String sex` with Korean messages) in `Latinhouse.Be/src/main/java/com/latinhouse/api/profile/adapter/in/web/CreateProfileWebRequest.java`
- [x] T019 [P] [US1] Create `CreateProfileWebResponse` (`String id`) in `Latinhouse.Be/src/main/java/com/latinhouse/api/profile/adapter/in/web/CreateProfileWebResponse.java`
- [x] T020 [US1] Create `CreateProfileWebMapper` (static methods: `toAppRequest(WebRequest)` converts `Sex.valueOf(sex)`; `toWebResponse(AppResponse)`) in `Latinhouse.Be/src/main/java/com/latinhouse/api/profile/adapter/in/web/CreateProfileWebMapper.java`
- [x] T021 [US1] Create `ProfileController` (`@RestController`, `POST /api/profile` with `@Valid`, `@Tag`, `@Operation`, returns `ResponseEntity<CreateProfileWebResponse>` 201) in `Latinhouse.Be/src/main/java/com/latinhouse/api/profile/adapter/in/web/ProfileController.java`

### Tests

- [x] T022 [P] [US1] Create `ProfileControllerTest` (`@WebMvcTest`: valid request → 201+id; missing nickname → 400+message; missing sex → 400+message; invalid sex → 400+message) in `Latinhouse.Be/src/test/java/com/latinhouse/api/profile/adapter/in/web/ProfileControllerTest.java`
- [x] T023 [P] [US1] Create `CreateProfileServiceTest` (unit test with mock `SaveProfilePort`: verifies generated id length=8, `isInstructor=false`, port called once) in `Latinhouse.Be/src/test/java/com/latinhouse/api/profile/application/service/CreateProfileServiceTest.java`

**Checkpoint**: `POST /api/profile` fully functional — valid input creates profile, invalid input returns Korean field-level errors

---

## Phase 4: Polish & Cross-Cutting Concerns

**Purpose**: Quality gates and documentation verification

- [x] T024 Verify `@Tag` and `@Operation` annotations present on `ProfileController` and confirm Swagger UI accessible at `/swagger-ui/index.html`
- [x] T025 Run `./gradlew test` from `Latinhouse.Be/` and confirm all tests pass with zero failures
- [x] T026 Validate responses match `docs/api-spec.md` exactly: 201 body `{"id":"..."}`, 400 body `{"status":400,"errors":[...]}`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies — start immediately
- **Foundational (Phase 2)**: Depends on Phase 1 — BLOCKS all user story work
- **User Story 1 (Phase 3)**: Depends on Phase 2 — can begin once T002–T005 complete
- **Polish (Phase 4)**: Depends on Phase 3 completion

### Within Phase 3 (User Story 1)

```
T006, T007 (Domain) — parallel
    ↓
T008, T009, T010, T011 (Ports) — parallel, depend on T006 (Sex enum)
    ↓
T012 (AppMapper) — depends on T006, T007, T008, T009
T014, T015 (Persistence types) — parallel, can start after T007
T018, T019 (Web DTOs) — parallel, no domain dependency
    ↓
T013 (Service) — depends on T002, T010, T011, T012
T016 (PersistenceMapper) — depends on T007, T014
T020 (WebMapper) — depends on T008, T009, T018, T019
    ↓
T017 (PersistenceAdapter) — depends on T011, T015, T016
T021 (Controller) — depends on T010, T019, T020
    ↓
T022, T023 (Tests) — parallel, depend on full stack
```

### Parallel Opportunities

- T006, T007 (Domain types) — parallel
- T008, T009, T010, T011 (Port interfaces/DTOs) — parallel
- T014, T015, T018, T019 (Persistence + Web types) — all parallel
- T022, T023 (Tests) — parallel

---

## Parallel Example: Phase 3 Setup Batch

```bash
# Batch 1 — Domain layer (start immediately after Phase 2)
Task: "Create Sex enum in ...profile/domain/Sex.java"
Task: "Create Profile domain in ...profile/domain/Profile.java"

# Batch 2 — Port interfaces (after Batch 1)
Task: "Create CreateProfileAppRequest"
Task: "Create CreateProfileAppResponse"
Task: "Create CreateProfileUseCase"
Task: "Create SaveProfilePort"
Task: "Create ProfileEntity"
Task: "Create ProfileJpaRepository"
Task: "Create CreateProfileWebRequest"
Task: "Create CreateProfileWebResponse"
```

---

## Implementation Strategy

### MVP (Single User Story)

1. Complete **Phase 1**: Add Swagger dependency
2. Complete **Phase 2**: Common infrastructure (T002–T005)
3. Complete **Phase 3**: Profile Creation end-to-end (T006–T023)
4. **VALIDATE**: Run `./gradlew test`, test with curl from `quickstart.md`
5. Complete **Phase 4**: Polish

### Solo Developer Order (Recommended)

Follow task IDs T001 → T025 sequentially, using [P] markers to identify safe parallelization opportunities when context allows.

---

## Notes

- [P] tasks operate on different files — safe to parallelize
- [US1] maps all tasks to User Story 1 (only story in this feature)
- Commit after each phase checkpoint
- Constitution mandates: no Spring/JPA imports in `domain/` package
- `@NotBlank` + `@Pattern` on sex: blank triggers `@NotBlank` message, non-blank invalid triggers `@Pattern` message
- `isInstructor=false` is set in `CreateProfileAppMapper.toDomain()`, not in the controller or service
