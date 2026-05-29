# Research: PATCH /api/profile/{profileId}/instructor

## Decision 1: Persistence Port 설계

**Decision**: `FindProfilePort` + `UpdateProfilePort` 두 개로 분리

**Rationale**:
- 기존 `SaveProfilePort`는 INSERT 전용이므로 UPDATE에 재사용 불가
- 단일 책임 원칙에 따라 조회와 수정 Port를 분리하면 독립 테스트 가능
- `FindProfilePort`는 향후 다른 조회 UseCase에서도 재사용 가능

**Alternatives considered**:
- `SetInstructorPort` (find + update 한 포트): 코드가 단순해지나 Port가 특정 UseCase에 종속됨. 기각.
- `SaveProfilePort` 재사용: 메서드 시그니처 의미가 다름 (save vs update). 기각.

---

## Decision 2: 도메인 메서드 — Profile.asInstructor()

**Decision**: `Profile` 도메인 객체에 `asInstructor()` 인스턴스 메서드 추가

**Rationale**:
- `Profile`은 불변 객체(모든 필드 final). 상태 변경은 새 인스턴스를 반환해야 함.
- 강사 지정 로직은 순수 도메인 규칙이므로 도메인 레이어에 위치가 적합.
- Constitution I 원칙: 도메인은 외부 의존 금지 → 외부 의존 없이 구현 가능.

```java
// Profile.java 추가
public Profile asInstructor() {
    return Profile.builder()
            .id(this.id)
            .nickname(this.nickname)
            .sex(this.sex)
            .isInstructor(true)
            .build();
}
```

**Alternatives considered**:
- Service 레이어에서 직접 Profile 재생성: 도메인 규칙이 서비스에 흘러들어 Constitution 위반. 기각.

---

## Decision 3: 404 예외 처리

**Decision**: `ProfileNotFoundException` 커스텀 예외 + `GlobalExceptionHandler` 핸들러 추가

**Rationale**:
- 기존 `GlobalExceptionHandler`가 `MethodArgumentNotValidException`만 처리 → 도메인 예외 핸들러 확장 필요
- 에러 응답은 Constitution III 통합 에러 형식 `{ "status": 404, "errors": [...] }` 사용
- `field`는 `"profileId"`, `message`는 `"프로필을 찾을 수 없습니다."`

**Alternatives considered**:
- `ResponseStatusException` 사용: 에러 응답 형식이 Spring 기본 형식이 되어 Constitution III 위반. 기각.

---

## Decision 4: 트랜잭션

**Decision**: `SetInstructorService`에 `@Transactional` 적용

**Rationale**:
- find + update 두 단계 DB 작업이므로 원자성 보장 필요
- `UpdateProfilePort` 구현 시 JPA `save()`(merge)를 사용하므로 트랜잭션 컨텍스트 필수

---

## Decision 5: Web Adapter — Request DTO 생략

**Decision**: `SetInstructorWebRequest` 클래스 미생성

**Rationale**:
- Path variable만 입력으로 사용하므로 별도 DTO 불필요
- `SetInstructorWebMapper.toAppRequest(String profileId)` 메서드가 직접 변환
- Constitution Two-DTO 패턴: WebRequest는 HTTP 바디 입력이 있을 때 생성. 이 케이스는 해당 없음.

**Alternatives considered**:
- 빈 `SetInstructorWebRequest` 생성: 불필요한 클래스 추가. 기각.

---

## Decision 6: JPA Entity 업데이트 전략

**Decision**: `profileJpaRepository.save(entity)` (JPA merge) 사용

**Rationale**:
- `ProfileEntity`는 기존 ID가 있으면 JPA가 `MERGE` → SQL `UPDATE` 실행
- `ProfilePersistenceMapper.toEntity(profile)` 호출로 업데이트된 Entity 생성 가능
- 별도 `@Modifying @Query` 불필요, 기존 패턴 재사용

---

## 신규 파일 목록 (Resolution)

| 분류 | 파일 | 역할 |
|------|------|------|
| 신규 | `SetInstructorUseCase.java` | application/port/in 인터페이스 |
| 신규 | `SetInstructorAppRequest.java` | application/port/in DTO |
| 신규 | `SetInstructorAppResponse.java` | application/port/in DTO |
| 신규 | `SetInstructorAppMapper.java` | application/port/in Mapper |
| 신규 | `SetInstructorService.java` | application/service 구현체 |
| 신규 | `FindProfilePort.java` | application/port/out 인터페이스 |
| 신규 | `UpdateProfilePort.java` | application/port/out 인터페이스 |
| 신규 | `SetInstructorWebResponse.java` | adapter/in/web DTO |
| 신규 | `SetInstructorWebMapper.java` | adapter/in/web Mapper |
| 신규 | `ProfileNotFoundException.java` | common/exception |
| 수정 | `Profile.java` | asInstructor() 도메인 메서드 추가 |
| 수정 | `ProfileController.java` | PATCH 엔드포인트 추가 |
| 수정 | `ProfilePersistenceAdapter.java` | FindProfilePort + UpdateProfilePort 구현 |
| 수정 | `GlobalExceptionHandler.java` | ProfileNotFoundException 핸들러 추가 |
