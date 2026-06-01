# Research: GET /api/profiles

## Decision 1: 쿼리 파라미터 필터링 — Optional<Boolean> vs 없을 때 전체 반환

**Decision**: `isInstructor` 쿼리 파라미터를 `@RequestParam(required = false) Boolean isInstructor`로 선언. `null`이면 전체 조회, `true`/`false`이면 필터링 조회.

**Rationale**:
- Spring이 `true`/`false` 문자열을 `Boolean`으로 자동 변환하며, 변환 실패 시 400 응답을 자동 처리함.
- 단일 엔드포인트에서 전체/필터 조회를 모두 처리할 수 있어 API 표면을 최소화.
- Constitution II(Contract-First): `api-spec.md`에 파라미터 없을 때 전체 반환으로 명세 추가.

**Alternatives considered**:
- 별도 엔드포인트(`/api/profiles/instructors`): API가 늘어나고 기능이 중복됨. 기각.
- `@RequestParam(defaultValue = "false")`: 파라미터 생략 시 false로 처리되어 전체 조회 불가. 기각.

---

## Decision 2: Port 설계 — FindAllProfilesPort 신규 vs FindProfilePort 확장

**Decision**: 기존 `FindProfilePort`에 `findAll(Boolean isInstructor): List<Profile>` 메서드 추가.

**Rationale**:
- `FindProfilePort`는 이미 "프로필 조회" 책임을 갖는다. 목록 조회도 같은 책임 범주.
- 신규 Port를 만들면 `ProfilePersistenceAdapter`가 구현해야 할 인터페이스가 늘어나 복잡도 증가.
- `findAll(null)` → 전체, `findAll(true)` → 강사만, `findAll(false)` → 비강사만 의미가 명확함.

**Alternatives considered**:
- `FindAllProfilesPort` 신규 생성: 분리가 명확하지만 Port가 너무 잘게 나뉘어 관리 부담. 기각.
- `findAll()` + `findAllByIsInstructor(boolean)` 두 메서드 분리: 호출부가 분기해야 함. 기각.

---

## Decision 3: JPA 조회 전략

**Decision**: `ProfileJpaRepository`에 `findAllByIsInstructor(boolean isInstructor)` 메서드 추가. 전체 조회는 기존 `findAll()` 활용.

**Rationale**:
- Spring Data JPA 메서드명 쿼리로 별도 `@Query` 없이 간단하게 구현 가능.
- `isInstructor` 파라미터가 `null`이면 `findAll()`, 아니면 `findAllByIsInstructor()` 분기 처리.

---

## Decision 4: AppRequest DTO 생략

**Decision**: `GetProfilesAppRequest` 클래스 미생성. Service 메서드 시그니처에 `Boolean isInstructor` 직접 사용.

**Rationale**:
- 입력 파라미터가 Boolean 하나뿐이므로 DTO 래핑은 과도한 추상화.
- Constitution Two-DTO 패턴은 복잡한 입력 객체에 적용하며, 단순 파라미터는 직접 전달 가능.
- 기존 `SetInstructorAppRequest`(profileId: String)와 달리 UseCase 메서드 파라미터로 충분.

---

## Decision 5: AppResponse / WebResponse 설계

**Decision**: `GetProfilesAppResponse` (id, nickname, sex, isInstructor 포함) + `GetProfilesWebResponse` (동일 구조) 분리 생성. 각각 `List<GetProfilesAppResponse>` / `List<GetProfilesWebResponse>` 반환.

**Rationale**:
- Constitution Two-DTO 패턴 준수: App 레이어는 도메인 타입(`Sex` enum)을 그대로 사용하고, Web 레이어에서 문자열로 직렬화.
- 향후 각 레이어에서 독립적으로 필드 추가/삭제 가능.

---

## Decision 6: api-spec.md 명세 추가

**Decision**: `docs/api-spec.md`의 Profile 섹션에 `GET /api/profiles` 명세를 추가한다.

**Rationale**:
- Constitution II(Contract-First): 구현 전 api-spec.md에 명세가 있어야 함.
- 현재 api-spec.md에 해당 엔드포인트가 없으므로 이 피처 작업 시 함께 추가.

---

## 신규/수정 파일 목록

| 분류 | 파일 | 역할 |
|------|------|------|
| 신규 | `GetProfilesUseCase.java` | application/port/in 인터페이스 |
| 신규 | `GetProfilesAppResponse.java` | application/port/in DTO |
| 신규 | `GetProfilesAppMapper.java` | application/port/in Mapper |
| 신규 | `GetProfilesService.java` | application/service 구현체 |
| 신규 | `GetProfilesWebResponse.java` | adapter/in/web DTO |
| 신규 | `GetProfilesWebMapper.java` | adapter/in/web Mapper |
| 수정 | `FindProfilePort.java` | findAll(Boolean) 메서드 추가 |
| 수정 | `ProfilePersistenceAdapter.java` | findAll(Boolean) 구현 추가 |
| 수정 | `ProfileJpaRepository.java` | findAllByIsInstructor() 메서드 추가 |
| 수정 | `ProfileController.java` | GET /api/profiles 엔드포인트 추가 |
| 수정 | `docs/api-spec.md` | GET /api/profiles 명세 추가 |
