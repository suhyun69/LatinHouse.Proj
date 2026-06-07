# Tasks: 쿠폰 소유자 배정

**Input**: Design documents from `/specs/014-coupon-assign-owner/`

**Prerequisites**: plan.md ✅ | spec.md ✅ | research.md ✅ | data-model.md ✅ | contracts/ ✅

**Organization**: 단일 User Story (쿠폰 소유자 배정). 기존 coupon 도메인 파일 확장 방식.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: 병렬 실행 가능 (다른 파일, 의존성 없음)
- **[Story]**: 해당 User Story ([US1])
- 모든 경로는 `Latinhouse.Be/src/main/java/com/latinhouse/api/` 기준

---

## Phase 1: Setup (에러 처리 인프라)

**Purpose**: 새 예외 클래스 추가 및 핸들러 등록

- [X] T001 [P] Create `CouponNotFoundException` in `common/exception/CouponNotFoundException.java` — `public CouponNotFoundException(Long couponId) { super("Coupon not found: " + couponId); }`
- [X] T002 Add `handleCouponNotFoundException()` to `common/exception/GlobalExceptionHandler.java` — `@ExceptionHandler(CouponNotFoundException.class)`, `@ResponseStatus(NOT_FOUND)`, field=`"couponId"`, message=`"쿠폰을 찾을 수 없습니다."`

**Checkpoint**: 예외 처리 인프라 준비 완료

---

## Phase 2: Foundational (포트 인터페이스)

**Purpose**: US1 구현에 필요한 Port Out 인터페이스 신규 생성

**⚠️ CRITICAL**: 서비스 구현 전 포트 인터페이스가 먼저 완료되어야 함

- [X] T003 [P] Create `LoadCouponPort` interface in `coupon/application/port/out/LoadCouponPort.java` — method: `Optional<Coupon> findById(Long couponId)`
- [X] T004 [P] Create `UpdateCouponPort` interface in `coupon/application/port/out/UpdateCouponPort.java` — method: `Coupon update(Coupon coupon)`

**Checkpoint**: 포트 인터페이스 준비 완료 → US1 구현 시작 가능

---

## Phase 3: User Story 1 — 쿠폰 소유자 배정 (Priority: P1) 🎯

**Goal**: `PATCH /api/coupon/{profileId}` 호출 시 Profile 존재 확인 후 Coupon.owner를 profileId로 업데이트한다.

**Independent Test**: `PATCH /api/coupon/{존재하는 profileId}`에 존재하는 couponId 전송 → 200 OK + `{"couponId": N}` 반환, Coupon.owner 변경 확인

### Implementation for User Story 1

- [X] T005 [P] [US1] Create `AssignCouponAppRequest` in `coupon/application/port/in/AssignCouponAppRequest.java` — `@Getter @Builder`, fields: `String profileId`, `Long couponId`
- [X] T006 [P] [US1] Create `AssignCouponAppResponse` in `coupon/application/port/in/AssignCouponAppResponse.java` — `@Getter @RequiredArgsConstructor`, field: `Long couponId`
- [X] T007 [P] [US1] Create `AssignCouponUseCase` interface in `coupon/application/port/in/AssignCouponUseCase.java` — method: `AssignCouponAppResponse assignCoupon(AssignCouponAppRequest)`
- [X] T008 [US1] Add `updateOwner(String owner)` method to `coupon/adapter/out/persistence/CouponEntity.java` — `public void updateOwner(String owner) { this.owner = owner; }` (depends on T003, T004)
- [X] T009 [US1] Add `LoadCouponPort`, `UpdateCouponPort` implementations to `coupon/adapter/out/persistence/CouponPersistenceAdapter.java` — `findById()`: `couponJpaRepository.findById(id).map(toCouponDomain)`, `update()`: `findById → entity.updateOwner() → save → toCouponDomain` (depends on T003, T004, T008)
- [X] T010 [P] [US1] Create `AssignCouponWebRequest` in `coupon/adapter/in/web/AssignCouponWebRequest.java` — `@Getter @Builder`, field: `@NotNull Long couponId`
- [X] T011 [P] [US1] Create `AssignCouponWebResponse` in `coupon/adapter/in/web/AssignCouponWebResponse.java` — `@Getter @RequiredArgsConstructor`, field: `Long couponId`
- [X] T012 [US1] Add `toAppRequest(String profileId, AssignCouponWebRequest)→AssignCouponAppRequest` and `toWebResponse(AssignCouponAppResponse)→AssignCouponWebResponse` static methods to `coupon/adapter/in/web/CouponWebMapper.java` (depends on T005, T006, T010, T011)
- [X] T013 [US1] Create `AssignCouponService` in `coupon/application/service/AssignCouponService.java` — `@Service @RequiredArgsConstructor @Transactional`, `implements AssignCouponUseCase`: ① `findProfilePort.findById(profileId)` → 없으면 `ProfileNotFoundException`, ② `loadCouponPort.findById(couponId)` → 없으면 `CouponNotFoundException`, ③ `Coupon` 재빌드(owner=profileId), ④ `updateCouponPort.update(updated)`, ⑤ `return new AssignCouponAppResponse(updated.getId())` (depends on T001, T003, T004, T005, T006, T007)
- [X] T014 [US1] Add `PATCH /coupon/{profileId}` endpoint to `coupon/adapter/in/web/CouponController.java` — `@Operation(summary="쿠폰 소유자 배정")`, `@PatchMapping("/coupon/{profileId}")`, `@PathVariable String profileId`, `@Valid @RequestBody AssignCouponWebRequest`, `ResponseEntity.ok(...)` (depends on T007, T012, T013)

**Checkpoint**: `PATCH /api/coupon/{profileId}` 독립 동작 검증 가능

---

## Phase 4: Polish & Cross-Cutting Concerns

- [X] T015 Verify `@Operation(summary="쿠폰 소유자 배정")` on `PATCH /coupon/{profileId}` endpoint in `CouponController.java`
- [X] T016 Run `./gradlew build` from `Latinhouse.Be/` — 컴파일 오류 없음 확인

---

## Dependencies & Execution Order

- **Phase 1 (Setup)**: 즉시 시작. T001, T002 병렬 가능
- **Phase 2 (Foundational)**: Phase 1 완료 후. T003, T004 병렬 가능
- **Phase 3 (US1)**: Phase 2 완료 후
  - T005, T006, T007, T010, T011 → 병렬 가능
  - T008 → T003, T004 완료 후
  - T009 → T008 완료 후
  - T012 → T005, T006, T010, T011 완료 후
  - T013 → T001, T003~T007 완료 후
  - T014 → T007, T012, T013 완료 후
- **Phase 4 (Polish)**: T014 완료 후

---

## Parallel Opportunities

```bash
# Phase 1:
T001 CouponNotFoundException
T002 GlobalExceptionHandler 핸들러 추가  ← T001 완료 후

# Phase 2:
T003 LoadCouponPort
T004 UpdateCouponPort

# Phase 3 — 병렬 그룹:
Group A (포트/DTO): T005, T006, T007, T010, T011
→ T008 (CouponEntity): T003, T004 완료 후
→ T009 (PersistenceAdapter): T008 완료 후
→ T012 (WebMapper): Group A 완료 후
→ T013 (Service): T001 + T003~T007 완료 후
→ T014 (Controller): T012 + T013 완료 후
```

---

## Implementation Strategy

1. Phase 1 + 2: 인프라 준비 (T001~T004)
2. Phase 3 Group A 병렬: 포트/DTO 클래스 생성 (T005~T007, T010~T011)
3. Phase 3 순차: Entity → Adapter → Mapper → Service → Controller (T008~T014)
4. Phase 4: 빌드 검증

---

## Notes

- `AssignCouponService`는 `FindProfilePort`(profile 도메인)와 `LoadCouponPort`, `UpdateCouponPort`(coupon 도메인) 세 포트를 주입받음
- `CouponPersistenceAdapter`에 `LoadCouponPort`, `UpdateCouponPort` 인터페이스를 추가 구현 (기존 파일 수정)
- `CouponController`와 `CouponWebMapper`는 기존 파일에 메서드/엔드포인트 추가 (기존 파일 수정)
