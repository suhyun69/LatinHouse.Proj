# Tasks: Coupon 도메인 생성

**Input**: Design documents from `/specs/013-coupon-domain/`

**Prerequisites**: plan.md ✅ | spec.md ✅ | research.md ✅ | data-model.md ✅ | contracts/ ✅

**Organization**: 두 개의 User Story (쿠폰 템플릿 생성 / 쿠폰 일괄 발행)로 구성. 각 스토리는 독립적으로 구현·테스트 가능.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: 병렬 실행 가능 (다른 파일, 의존성 없음)
- **[Story]**: 해당 User Story ([US1], [US2])
- 모든 경로는 `Latinhouse.Be/src/main/java/com/latinhouse/api/` 기준

---

## Phase 1: Setup (공통 인프라)

**Purpose**: 도메인 패키지 디렉토리 구조 생성 및 에러 처리 인프라 준비

- [X] T001 Create coupon domain package directories: `coupon/domain/`, `coupon/application/port/in/`, `coupon/application/port/out/`, `coupon/application/service/`, `coupon/adapter/in/web/`, `coupon/adapter/out/persistence/`
- [X] T002 [P] Create `CouponTemplateNotFoundException` in `common/exception/CouponTemplateNotFoundException.java`
- [X] T003 Add `handleCouponTemplateNotFoundException()` handler to `common/exception/GlobalExceptionHandler.java` — 404 + field=templateId

**Checkpoint**: 패키지 구조와 예외 처리 준비 완료

---

## Phase 2: Foundational (공통 도메인·Enum)

**Purpose**: 두 User Story 모두에서 공유되는 도메인 객체와 Enum 생성

**⚠️ CRITICAL**: US1, US2 모두 이 단계 완료 후 시작 가능

- [X] T004 [P] Create `CouponTemplateType` enum in `coupon/domain/CouponTemplateType.java` — value: `LESSON`
- [X] T005 [P] Create `CouponStatus` enum in `coupon/domain/CouponStatus.java` — values: `AVAILABLE`, `USED`
- [X] T006 [P] Create `CouponTemplate` domain object in `coupon/domain/CouponTemplate.java` — `@Getter @Builder`, fields: `Long id`, `String title`, `CouponTemplateType type`, `Long target`, `BigDecimal amount`
- [X] T007 [P] Create `Coupon` domain object in `coupon/domain/Coupon.java` — `@Getter @Builder`, fields: `Long id`, `Long templateId`, `String owner` (nullable), `CouponStatus status`

**Checkpoint**: 공유 도메인 준비 완료 → US1, US2 병렬 시작 가능

---

## Phase 3: User Story 1 — 쿠폰 템플릿 생성 (Priority: P1) 🎯 MVP

**Goal**: `POST /api/coupon/template` 엔드포인트를 통해 쿠폰 템플릿을 생성하고 ID를 반환한다.

**Independent Test**: `POST /api/coupon/template`에 유효한 요청 전송 → 201 Created + `{"couponTemplateId": "1"}` 반환 확인

### Implementation for User Story 1

- [X] T008 [P] [US1] Create `CreateCouponTemplateAppRequest` in `coupon/application/port/in/CreateCouponTemplateAppRequest.java` — fields: `String title`, `CouponTemplateType type`, `Long target`, `BigDecimal amount` (`@Getter @Builder`)
- [X] T009 [P] [US1] Create `CreateCouponTemplateAppResponse` in `coupon/application/port/in/CreateCouponTemplateAppResponse.java` — field: `Long couponTemplateId`
- [X] T010 [P] [US1] Create `CreateCouponTemplateUseCase` interface in `coupon/application/port/in/CreateCouponTemplateUseCase.java` — method: `CreateCouponTemplateAppResponse createCouponTemplate(CreateCouponTemplateAppRequest)`
- [X] T011 [P] [US1] Create `SaveCouponTemplatePort` interface in `coupon/application/port/out/SaveCouponTemplatePort.java` — method: `CouponTemplate save(CouponTemplate)`
- [X] T012 [P] [US1] Create `CouponTemplateEntity` JPA entity in `coupon/adapter/out/persistence/CouponTemplateEntity.java` — table: `coupon_templates`, `@GeneratedValue(strategy = IDENTITY)`, fields: `Long id`, `String title`, `String type`, `Long target`, `BigDecimal amount(precision=15,scale=2)`
- [X] T013 [P] [US1] Create `CouponTemplateJpaRepository` in `coupon/adapter/out/persistence/CouponTemplateJpaRepository.java` — `extends JpaRepository<CouponTemplateEntity, Long>`
- [X] T014 [US1] Create `CreateCouponTemplateService` in `coupon/application/service/CreateCouponTemplateService.java` — `implements CreateCouponTemplateUseCase`, `@Service @RequiredArgsConstructor @Transactional`: `CouponTemplate` 빌드 → `SaveCouponTemplatePort.save()` → `CreateCouponTemplateAppResponse(saved.getId())` 반환 (depends on T008, T009, T010, T011)
- [X] T015 [P] [US1] Create `CreateCouponTemplateWebRequest` in `coupon/adapter/in/web/CreateCouponTemplateWebRequest.java` — `@Getter @Builder`, `@NotBlank String title`, `@NotNull String type`, `@NotNull Long target`, `@NotNull BigDecimal amount`
- [X] T016 [P] [US1] Create `CreateCouponTemplateWebResponse` in `coupon/adapter/in/web/CreateCouponTemplateWebResponse.java` — field: `String couponTemplateId`
- [X] T017 [US1] Create `CouponWebMapper` in `coupon/adapter/in/web/CouponWebMapper.java` — private 생성자, static 메서드: `toAppRequest(CreateCouponTemplateWebRequest)→CreateCouponTemplateAppRequest` (CouponTemplateType 변환 포함), `toWebResponse(CreateCouponTemplateAppResponse)→CreateCouponTemplateWebResponse` (depends on T008, T015, T016)
- [X] T018 [US1] Create `CouponPersistenceMapper` in `coupon/adapter/out/persistence/CouponPersistenceMapper.java` — private 생성자, static: `toEntity(CouponTemplate)→CouponTemplateEntity`, `toDomain(CouponTemplateEntity)→CouponTemplate` (depends on T006, T012)
- [X] T019 [US1] Create `CouponPersistenceAdapter` in `coupon/adapter/out/persistence/CouponPersistenceAdapter.java` — `@Repository @RequiredArgsConstructor`, `implements SaveCouponTemplatePort`: `save()` 구현 (depends on T011, T012, T013, T018)
- [X] T020 [US1] Create `CouponController` (template endpoint only) in `coupon/adapter/in/web/CouponController.java` — `@Tag(name="Coupon")`, `@PostMapping("/coupon/template")`, `@Valid @RequestBody`, `ResponseEntity.status(201).body(...)` (depends on T010, T014, T015, T016, T017)

**Checkpoint**: `POST /api/coupon/template` 독립 동작 검증 가능

---

## Phase 4: User Story 2 — 쿠폰 일괄 발행 (Priority: P2)

**Goal**: `POST /api/coupon` 엔드포인트를 통해 템플릿 기반으로 쿠폰을 count 개수만큼 발행한다.

**Independent Test**: `POST /api/coupon` with valid templateId + count=3 → 201 No Body + DB에 3개 쿠폰 생성 확인

### Implementation for User Story 2

- [X] T021 [P] [US2] Create `CreateCouponAppRequest` in `coupon/application/port/in/CreateCouponAppRequest.java` — `@Getter @Builder`, fields: `Long templateId`, `Integer count`
- [X] T022 [P] [US2] Create `CreateCouponUseCase` interface in `coupon/application/port/in/CreateCouponUseCase.java` — method: `void createCoupon(CreateCouponAppRequest)`
- [X] T023 [P] [US2] Create `LoadCouponTemplatePort` interface in `coupon/application/port/out/LoadCouponTemplatePort.java` — method: `Optional<CouponTemplate> findById(Long)`
- [X] T024 [P] [US2] Create `SaveCouponPort` interface in `coupon/application/port/out/SaveCouponPort.java` — method: `void saveAll(List<Coupon>)`
- [X] T025 [P] [US2] Create `CouponEntity` JPA entity in `coupon/adapter/out/persistence/CouponEntity.java` — table: `coupons`, `@GeneratedValue(strategy = IDENTITY)`, fields: `Long id`, `Long templateId`, `String owner` (nullable), `String status NOT NULL`
- [X] T026 [P] [US2] Create `CouponJpaRepository` in `coupon/adapter/out/persistence/CouponJpaRepository.java` — `extends JpaRepository<CouponEntity, Long>`
- [X] T027 [US2] Add `LoadCouponTemplatePort`, `SaveCouponPort` implementations to `CouponPersistenceAdapter.java` — `findById()`: `CouponTemplateJpaRepository.findById().map(toDomain)`, `saveAll()`: `CouponJpaRepository.saveAll(entities)` (depends on T019, T023, T024, T025, T026)
- [X] T028 [US2] Add `toCouponEntity(Coupon)→CouponEntity`, `toCouponDomain(CouponEntity)→Coupon` static methods to `CouponPersistenceMapper.java` (depends on T007, T025)
- [X] T029 [US2] Create `CreateCouponService` in `coupon/application/service/CreateCouponService.java` — `implements CreateCouponUseCase`, `@Service @RequiredArgsConstructor @Transactional`: `LoadCouponTemplatePort.findById()` → 없으면 `CouponTemplateNotFoundException`, count 루프로 Coupon 빌드(owner=null, status=AVAILABLE), `SaveCouponPort.saveAll()` (depends on T002, T021, T022, T023, T024)
- [X] T030 [P] [US2] Create `CreateCouponWebRequest` in `coupon/adapter/in/web/CreateCouponWebRequest.java` — `@Getter @Builder`, `@NotNull Long templateId`, `@NotNull @Min(1) Integer count`
- [X] T031 [US2] Add `toAppRequest(CreateCouponWebRequest)→CreateCouponAppRequest` to `CouponWebMapper.java` (depends on T021, T030)
- [X] T032 [US2] Add `POST /coupon` endpoint to `CouponController.java` — `@Operation(summary="쿠폰 일괄 발행")`, `@PostMapping("/coupon")`, `@Valid @RequestBody CreateCouponWebRequest`, `ResponseEntity<Void>` 반환 201 (depends on T022, T029, T030, T031)

**Checkpoint**: `POST /api/coupon` 독립 동작 및 오류 케이스 검증 가능

---

## Phase 5: Polish & Cross-Cutting Concerns

**Purpose**: Swagger 문서 보강 및 최종 빌드 검증

- [X] T033 Verify `@Tag(name="Coupon", description="쿠폰 관리 API")` on `CouponController`, `@Operation` on both endpoints
- [X] T034 Run `./gradlew build` from `Latinhouse.Be/` — 컴파일 오류 없음 확인
- [X] T035 Start app and verify both endpoints appear in Swagger UI (`/swagger-ui.html`)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 (Setup)**: 즉시 시작
- **Phase 2 (Foundational)**: Phase 1 완료 후 → US1, US2 모두 블로킹
- **Phase 3 (US1)**: Phase 2 완료 후 시작 가능
- **Phase 4 (US2)**: Phase 2 완료 후 시작 가능 (US1과 병렬 가능)
- **Phase 5 (Polish)**: Phase 3, 4 완료 후

### User Story Dependencies

- **US1**: Foundational 완료 후 독립 시작 가능
- **US2**: Foundational 완료 후 독립 시작 가능 (US1 불필요)

### Within Each User Story

- App 계층 클래스(AppRequest, UseCase, Port) → Service → Adapter 순서
- WebRequest/WebResponse → WebMapper → Controller 순서
- Entity → JpaRepository → PersistenceMapper → PersistenceAdapter 순서

---

## Parallel Opportunities

```bash
# Phase 2 — 모두 병렬 가능:
T004 CouponTemplateType enum
T005 CouponStatus enum
T006 CouponTemplate domain
T007 Coupon domain

# Phase 3 — 병렬 가능 그룹:
Group A: T008, T009, T010, T011  (Application Port In 클래스들)
Group B: T012, T013              (Entity + Repository)
Group C: T015, T016              (WebRequest + WebResponse)
→ T014 (Service): Group A 완료 후
→ T017 (WebMapper): Group C + T008 완료 후
→ T018 (PersistenceMapper): T006 + T012 완료 후
→ T019 (PersistenceAdapter): T018 완료 후
→ T020 (Controller): T014 + T017 + T019 완료 후

# Phase 4 — 병렬 가능 그룹:
Group A: T021, T022, T023, T024  (Application Port 클래스들)
Group B: T025, T026              (Entity + Repository)
Group C: T030                    (WebRequest)
→ T027 (PersistenceAdapter 확장): T019 + T025 완료 후
→ T028 (PersistenceMapper 확장): T025 완료 후
→ T029 (Service): T021–T024 완료 후
→ T031 (WebMapper 확장): T021 + T030 완료 후
→ T032 (Controller 확장): T029 + T031 완료 후
```

---

## Implementation Strategy

### MVP (US1만)

1. Phase 1: Setup
2. Phase 2: Foundational
3. Phase 3: US1 (쿠폰 템플릿 생성)
4. **검증**: `POST /api/coupon/template` 동작 확인
5. 필요시 배포/데모

### Incremental Delivery

1. Phase 1 + 2 → 기반 완료
2. Phase 3 (US1) → 템플릿 생성 MVP
3. Phase 4 (US2) → 쿠폰 발행 추가
4. Phase 5 → 최종 정리

---

## Notes

- `[P]` 태스크는 다른 파일을 다루므로 병렬 실행 가능
- `CouponPersistenceAdapter`는 US2에서 기존 파일에 인터페이스 추가 방식으로 확장
- `CouponPersistenceMapper`도 US2에서 메서드 추가 방식으로 확장
- `CouponController`도 US2에서 엔드포인트 추가 방식으로 확장
